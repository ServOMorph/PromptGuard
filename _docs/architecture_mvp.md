# Architecture MVP — PromptGuard

Référence technique de `roadmap_mvp.md`. Décisions : `_docs/cadrage_mvp.md`. Briques : `_docs/etat_de_l_art.md`.

## Flux principal

```
Navigateur (UI JS vanilla)
   │ 1. prompt + fichiers joints + @alias
   ▼
Flask :4024 ── analyse ──► Détection (regex+validateurs FR → GLiNER2-PII → Ollama)
   │                              │
   │ 2. détections + charge       ▼
   │    sortante pseudonymisée   Pseudonymiseur (piighost, jetons <<TYPE:n>> stables)
   │◄─────────────────────────────┘        │ mapping chiffré (SQLite)
   │ 3. validation explicite               │ Coffre (AES-GCM, clé keyring)
   ▼                                        │
Wrapper Claude CLI (claude -p, flags d'isolation, cwd = bac à sable vide)
   │  ANTHROPIC_BASE_URL=http://127.0.0.1:4025
   ▼
Proxy de sortie :4025 ── scan corps requête vs valeurs connues ──► 403 + incident
   │ sinon relais httpx (streaming SSE, auth du CLI relayée telle quelle)
   ▼
api.anthropic.com
   │ réponse (jetons)
   ▼
Contrôle de sortie (détection + correspondance exacte) ──► incident si fuite
   ▼
Réinjection locale (jetons → vraies valeurs) ──► affichage / export dossier
```

## Découpage du code

```
src/promptguard/
  __main__.py            lance Flask :4024 et le proxy :4025 (threads), options CLI
  app.py                 app factory Flask (repris d'IA_V7)
  config.py              chemins, ports, seuils, modèle Ollama, mode proxy on/off
  detection/
    base.py              interface Detector -> list[Span(start, end, type, source, score)]
    validators_fr.py     IBAN mod 97, NIR clé 97 (2A/2B), Luhn, SIRET/SIREN, tél FR, IP (repris d'IA_V7)
    secrets.py           clés API (préfixes connus + entropie), tokens, mots de passe, chaînes de connexion
    piighost_adapter.py  seul module qui importe piighost (regex catalogue FR/EU + GLiNER2)
    ollama_detector.py   niveau 3 : prompt JSON strict, timeout, dégradation silencieuse si indisponible
    pipeline.py          fusion/déduplication des spans, priorité validateurs > GLiNER2 > Ollama
  pseudonymize.py        jetons <<TYPE:n>> stables par conversation, restauration (via adaptateur piighost)
  vault.py               coffre : entrées (alias, type, valeur chiffrée), clé maître keyring
  egress/
    proxy.py             proxy HTTP local, relais streaming, blocage, journal d'incidents
    matcher.py           correspondance exacte multi-variantes (compact, espacé, casse) des valeurs connues
  cloud/
    claude_cli.py        wrapper subprocess claude -p, stream-json, isolation, bac à sable
  output_guard.py        contrôle des réponses cloud (fuite = incident)
  files.py               listing/lecture bornés du dossier de travail, export (garde anti-traversal)
  storage/
    database.py          SQLite (schéma versionné), champs sensibles chiffrés
    repository.py        dossiers, conversations, messages, mappings, incidents
  web/routes.py          API JSON + SSE
templates/index.html
static/js/app.js, static/css/app.css, static/vendor/ (marked, DOMPurify repris d'IA_V7)
tests/                   pytest (unitaires, API, intégration stubs)
tests-ui/                Playwright (Edge, stubs)
tests/fakes/             faux Claude CLI (script Python déterministe), faux Ollama
benchmarks/              corpus_fr.json, benchmark.py, rapport_benchmark.md
benchmarks/adversarial/  corpus adversarial (zone securite)
SECURITE/                modele_menace.md, rapport_canaris.md, incidents/
presentation/            index.html (page de présentation autonome)
docs/assets/             captures Playwright pour le README
```

## Contrats clés

### API HTTP (Flask :4024)

| Route | Rôle |
|---|---|
| `POST /api/workspaces` | Déclare un dossier de travail (chemin absolu existant). |
| `GET /api/workspaces/<id>/files` | Liste les fichiers texte du dossier (extensions autorisées, ≤ 200 Ko, profondeur ≤ 3). |
| `POST /api/conversations` / `GET /api/conversations/<id>` | Création / lecture (valeurs réinjectées côté serveur). |
| `POST /api/analyze` | Entrée : `{conversation_id, text, files[], manual_spans[], ignored_spans[]}`. Sortie : `{spans[], outgoing_payload, vault_suggestions[], ollama_status}`. Aucun appel cloud. |
| `POST /api/send` | Entrée : `{conversation_id, analysis_id}` (charge figée à l'analyse, non modifiable ensuite). Sortie : SSE (chunks pseudonymisés → réinjectés, puis verdict du contrôle de sortie). |
| `GET/POST/DELETE /api/vault` | Liste (valeurs masquées), ajout, suppression. |
| `POST /api/export` | Écrit la réponse réinjectée dans le dossier de travail. |
| `GET /api/incidents` | Journal des blocages proxy et fuites en sortie. |

### Invocation Claude CLI

```
claude -p --output-format stream-json --verbose
       --tools "" --strict-mcp-config --mcp-config <fichier {} vide>
       --setting-sources "" --system-prompt <prompt PromptGuard>
       --disable-slash-commands --no-session-persistence
cwd = <data>/sandbox/<uuid> (vide, supprimé après appel)
env = ANTHROPIC_BASE_URL=http://127.0.0.1:4025 (si mode proxy actif)
stdin = historique pseudonymisé + nouveau message
```

Valeurs exactes des flags (`--setting-sources ""` accepté ou non, forme de `--tools`) établies en phase 1 par test, pas supposées.

### Proxy de sortie

- Écoute `127.0.0.1:4025` uniquement. Relaie toute méthode/chemin vers `https://api.anthropic.com`, en-têtes d'authentification transmis sans lecture ni journalisation.
- Avant relais : décode le corps (JSON), le compare aux valeurs connues (coffre + mappings des conversations actives) via `matcher.py`. Correspondance → 403, incident journalisé (type, jeton, jamais la valeur).
- Mode `record` (tests canari uniquement) : enregistre les corps de requête dans `SECURITE/canaris/` après remplacement des en-têtes d'auth.

### Données locales

`data/` (hors git) : `promptguard.db`, `sandbox/`. Coffre et textes originaux chiffrés AES-GCM ; clé maître 256 bits dans keyring (service `PromptGuard`). Tests : backend keyring en mémoire, jamais le Gestionnaire d'identifiants réel.

## Invariants de sécurité (testés)

1. Le dossier de travail n'est jamais le `cwd` de Claude CLI ni passé en argument.
2. Aucune valeur du coffre ni d'un mapping ne figure dans une requête sortante (proxy) ni dans stdin du CLI (test unitaire sur le wrapper).
3. `/api/send` n'envoie que la charge figée à l'analyse et validée.
4. GLiNER2 en inférence locale : test réseau coupé (socket bloqué) pendant la détection.
5. Aucune valeur sensible dans les logs applicatifs ni dans les incidents.
6. Flask et le proxy n'écoutent que sur `127.0.0.1`.
