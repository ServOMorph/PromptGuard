# Roadmap — MVP de présentation PromptGuard

Documents de référence (à lire avant la phase 0) : `_docs/brief_initial.md`, `_docs/cadrage_mvp.md`, `_docs/architecture_mvp.md`, `_docs/etat_de_l_art.md`.

## Mode d'exécution : autonomie complète

Exception validée par l'utilisateur le 2026-09-26 à la règle CLAUDE.md (section Roadmap) : **aucun checkpoint `/compact`, aucune question à l'utilisateur**. Les phases s'enchaînent sans interruption jusqu'à la fin de la phase 7.

### Règles d'exécution

1. **Journal** : l'avancement est tenu dans `journal_roadmap_mvp.md` (racine), pas dans ce fichier (statuts de phase mis à jour par `/close` uniquement, règle CLAUDE.md inchangée). À chaque tâche terminée : une ligne `AAAA-MM-JJ HH:MM | Px.y | fait | preuve (commande + résultat)`. À chaque décision de repli : une ligne `DÉCISION`. Après une compaction automatique du contexte, reprendre en relisant ce fichier puis le journal.
2. **Gate de phase** : une phase n'est terminée que si toutes les commandes de son gate passent. Puis commit local unique `feat(mvp): phase N — <titre>` (message généré via `python ollama_call.py`, cf. CLAUDE.md). **Jamais de push.**
3. **Blocage** : un problème non résolu après 3 tentatives de correction distinctes → appliquer la règle de repli de la phase si elle existe ; sinon consigner `BLOCAGE` dans le journal, marquer la tâche contournée (`xfail` explicite ou fonctionnalité désactivée derrière un flag de `config.py`), et continuer. Ne jamais désactiver un test de sécurité pour faire passer un gate.
4. **Arrêt obligatoire** (seul cas d'interruption) : si un test canari ou un test du proxy montre qu'une valeur sensible *réelle de l'utilisateur* (contenu de `~/.claude`, fichier personnel) a été transmise à Anthropic de façon non prévue → arrêter, consigner dans `SECURITE/incidents/` et `_contexte/signals.md` (action P1), ne pas continuer.
5. **Données de test** : exclusivement synthétiques (jamais de vraies données de l'utilisateur). Valeurs fictives valides (IBAN à clé correcte, NIR à clé correcte, cartes Luhn de test).
6. **Appels réels** : vrai Claude CLI limité aux tests marqués `@pytest.mark.real_claude` (canaris phase 1, E2E phases 4 et 6) ; budget indicatif ≤ 40 appels sur toute la roadmap, compteur tenu dans le journal. Vrai Ollama autorisé (`@pytest.mark.real_ollama`). Les suites par défaut et la CI n'exécutent ni l'un ni l'autre.
7. **Ne jamais modifier** `~/.claude/`, la configuration globale de Claude Code, ni `D:\ServOMorph\IA_V7` (lecture seule).
8. **Périmètres agents** : les livrables respectent les dossiers des zones (`SECURITE/`, `benchmarks/adversarial/` pour la sécurité ; `tests/`, `tests-ui/`, `benchmarks/`, `.github/workflows/` pour la QA ; `DOCUMENTATION/`, `README.md` pour la doc). L'exécution se fait depuis la zone racine.
9. **Modèle** : lancer la roadmap avec Opus (plan + debug + sécurité).

### Prérequis de lancement (vérifiés en P0.1, à défaut : arrêt avant tout code)

- Session Claude Code en mode auto (ou permissions équivalentes) : sans cela les demandes d'autorisation interrompent l'autonomie.
- Accès réseau (PyPI, Hugging Face ~1 Go pour GLiNER2-PII, npm).
- Ollama démarré, `gemma4:12b` présent ; `claude` connecté à l'abonnement.

---

## Phase 0 — Socle projet [TODO]

- P0.1 Vérifier les prérequis (`claude --version`, `ollama list`, `uv --version`, `node --version`, accès PyPI/HF). Créer `journal_roadmap_mvp.md`.
- P0.2 `pyproject.toml` (uv, Python ≥ 3.13, paquet `src/promptguard`), dépendances : `flask`, `httpx`, `cryptography`, `keyring`, `python-dotenv`, `piighost[gliner2,crypto,sqlalchemy]==1.8.0`, `gliner2[local]`. Dev : `pytest`, `pytest-cov`, `ruff`. Verrou `uv.lock`.
- P0.3 `.gitignore` (`data/`, `.venv/`, caches, rapports Playwright, modèles), `LICENSE` Apache 2.0 (copyright 2026 ServOMorph), `package.json` + Playwright (Edge).
- P0.4 Reprise sélective d'IA_V7 (adapter, ne pas copier en bloc) : app factory, config, `database.py`, client Ollama, `templates/index.html`, `static/` (charte, vendors `marked`, `DOMPurify`). Renommage `ia_v7` → `promptguard`, suppression de ce qui est hors périmètre (commandes `/write`, mode Code, capture, arrêt serveur Ollama).
- P0.5 `__main__.py` : Flask `127.0.0.1:4024`, page d'accueil servie.
- P0.6 Fakes : `tests/fakes/fake_claude.py` (lit stdin, renvoie un stream-json déterministe qui recopie les jetons reçus) et faux Ollama.
- P0.7 CI `.github/workflows/ci.yml` : ruff + pytest (sans marqueurs réels) + Playwright stubs, runner Windows.

**Repli** : `piighost` incompatible Python 3.13 ou installation impossible → épingler la dernière version compatible ; à défaut, retirer la dépendance et implémenter `piighost_adapter.py` directement sur `gliner2` (jetons et mapping maison), `DÉCISION` au journal.

**Gate** : `uv sync` ; `uv run ruff check .` ; `uv run pytest -q` (smoke : app démarre, `/` répond 200) ; `npx playwright test` (page chargée).

## Phase 1 — Preuves d'isolation Claude CLI (canaris) [TODO]

Risque principal du projet, traité avant toute fonctionnalité.

- P1.1 `SECURITE/modele_menace.md` : actifs (coffre, dossier, `~/.claude`), canaux de fuite (stdin CLI, contexte chargé par le CLI, outils, MCP, proxy contourné, réponse, logs), mesures, risques résiduels.
- P1.2 `egress/proxy.py` en mode passthrough + mode `record` (en-têtes d'auth masqués avant écriture).
- P1.3 `cloud/claude_cli.py` : construction de la commande, bac à sable, parsing stream-json, timeout, erreurs lisibles. Tests unitaires avec le faux CLI.
- P1.4 Canari A (réel) : `claude -p "Réponds uniquement OK"` via proxy, abonnement. Attendu : requête vue par le proxy + réponse OK. **Décide le mode proxy.**
- P1.5 Canari B (réel, via proxy `record`) : même appel **sans** flags d'isolation puis **avec**. Comparer les corps enregistrés : liste des outils, prompt système, présence de marqueurs extraits (lecture seule) de `~/.claude/CLAUDE.md` et du `CLAUDE.md` projet, définitions MCP. Déterminer par essais la forme valide de chaque flag.
- P1.6 `SECURITE/rapport_canaris.md` : tableau avant/après, flags retenus, preuves (extraits anonymisés des corps), verdict.
- P1.7 Tests de non-régression : les flags retenus sont figés dans un test unitaire du wrapper ; un test `real_claude` rejoue le canari B.

**Repli** : canari A échoue (CLI ignore `ANTHROPIC_BASE_URL` avec l'abonnement, erreur d'auth) → mode proxy désactivé dans `config.py`, contrôle déterministe reporté sur le stdin du wrapper (même `matcher.py`), limite documentée dans le rapport et le README ; `DÉCISION` au journal. Canari B montre qu'un contenu de `~/.claude` part malgré tous les flags → consigner comme risque résiduel majeur dans le rapport, garder le cwd bac à sable, continuer (ce n'est pas une donnée PromptGuard) ; appliquer la règle d'arrêt 4 uniquement si le contenu transmis est une donnée sensible réelle.

**Gate** : `uv run pytest -q` ; `uv run pytest -m real_claude tests/security -q` vert ; rapport canari rédigé avec verdict explicite.

## Phase 2 — Détection et benchmark [TODO]

- P2.1 `detection/base.py` (Span, interface Detector) et `pipeline.py` (fusion, chevauchements, priorité validateurs > GLiNER2 > Ollama).
- P2.2 `validators_fr.py` : reprise des validateurs IA_V7 + SIRET/SIREN (Luhn), NIR 2A/2B, date de naissance contextuelle. Tests unitaires par validateur (valeurs valides/invalides).
- P2.3 `secrets.py` : préfixes connus (`sk-`, `sk-ant-`, `ghp_`, `AKIA`, `xox`…), JWT, chaînes de connexion (`postgres://user:pass@`), `mot de passe : …`, entropie de Shannon bornée.
- P2.4 `piighost_adapter.py` : catalogue regex FR/EU + détecteur GLiNER2 (`fastino/gliner2-privacy-filter-PII-multi`, CPU, chargement unique paresseux). Seuil `person` relevé (point de départ 0,6, calibré sur le corpus). Gestion de l'API asynchrone éventuelle.
- P2.5 Test réseau coupé : détection GLiNER2 exécutée avec sockets bloqués (monkeypatch) → aucun appel distant.
- P2.6 `ollama_detector.py` : `gemma4:12b`, sortie JSON contrainte (`format: json`), liste de spans avec extrait exact vérifié dans le texte (rejet des hallucinations), timeout 45 s, statut `indisponible` sans erreur bloquante.
- P2.7 Corpus `benchmarks/corpus_fr.json` : reprise des 26 cas IA_V7 + ≥ 40 nouveaux cas (noms sans civilité, secrets, fichiers type CSV/code, faux positifs à ne pas détecter), ≥ 30 cas verrouillés. Script `benchmarks/benchmark.py` (dérivé de `benchmark_rgpd.py`) : conformité exacte, rappel, précision, par catégorie et par niveau (regex seul / + GLiNER2 / + Ollama). Rapport `benchmarks/rapport_benchmark.md`.
- P2.8 Corpus adversarial `benchmarks/adversarial/corpus.json` (≥ 20 cas : fragmentation, espaces insérés, homoglyphes, encodage base64, dictée « F R 7 6 », mélange langues). Mesuré, non verrouillé.

**Repli** : GLiNER2 > 10 s par prompt de 2 000 caractères sur CPU → découpage en fenêtres + cache par hash ; si toujours > 10 s, GLiNER2 passe en option désactivable (défaut activé), mesure au journal. Ollama trop lent ou incohérent → niveau 3 désactivé par défaut, mesures publiées.

**Gate** : `uv run pytest -q` ; `uv run python benchmarks/benchmark.py --check` : 100 % des cas verrouillés conformes, aucun cas IA_V7 verrouillé régressé ; `uv run pytest -m real_ollama -q` vert.

## Phase 3 — Coffre, pseudonymisation, proxy bloquant [TODO]

- P3.1 `vault.py` : clé maître 256 bits générée au premier lancement et stockée via keyring ; AES-GCM (nonce 96 bits aléatoire par valeur) ; entrées `alias` (unique, `[a-z0-9_]+`), type, valeur chiffrée, date. Tests avec backend keyring en mémoire.
- P3.2 `pseudonymize.py` : jetons `<<TYPE:n>>` stables par conversation, restauration, restauration en flux (tampon de fin de chunk pour jetons coupés). Résolution `@alias` → jeton lié à la valeur du coffre.
- P3.3 `storage/` : schéma versionné (workspaces, conversations, messages [original chiffré, pseudonymisé], mappings chiffrés, analyses figées, incidents).
- P3.4 `egress/matcher.py` : variantes normalisées (sans espaces/tirets/points, casse), longueur minimale 4, pas de valeur en clair en mémoire plus longtemps que nécessaire.
- P3.5 Proxy bloquant : 403 + incident si correspondance ; test d'intégration (requête forgée contenant un IBAN du coffre → bloquée ; même requête pseudonymisée → relayée vers un faux amont).
- P3.6 Garde du wrapper : refus d'écrire sur stdin du CLI toute charge contenant une valeur connue (même `matcher.py`), indépendant du proxy.

**Gate** : `uv run pytest -q` (dont tests invariants 2, 5, 6 de l'architecture) ; ruff.

## Phase 4 — Pipeline d'envoi et contrôle de sortie [TODO]

- P4.1 `/api/analyze` : détection + fichiers joints + spans manuels/ignorés → charge sortante figée (`analysis_id`).
- P4.2 `/api/send` : exécution du wrapper sur la charge figée uniquement, SSE, historique pseudonymisé multi-tours (le CLI est sans session : l'historique est reconstruit à chaque tour).
- P4.3 `output_guard.py` : chaque réponse passe détection + matcher ; fuite → incident, message marqué dans l'UI.
- P4.4 `files.py` + routes : listing borné, lecture UTF-8, export `.md` horodaté, garde anti-traversal.
- P4.5 Tests API complets avec faux CLI (dont : `/api/send` sans analyse → 400 ; analyse modifiée après coup → refus).
- P4.6 E2E réel unique (`real_claude`) : prompt synthétique avec IBAN + nom + clé API → réponse de Claude contient les jetons, réinjection correcte, 0 incident.

**Gate** : `uv run pytest -q` ; `uv run pytest -m real_claude tests/e2e -q` vert.

## Phase 5 — Interface utilisateur [TODO]

- P5.1 Écran dossier de travail (saisie chemin absolu, validation, liste des dossiers récents).
- P5.2 Chat : saisie, `@alias` avec autocomplétion, joindre un fichier du dossier.
- P5.3 Panneau d'analyse : texte surligné par type, retrait d'une détection (faux positif), ajout manuel par sélection, suggestions de mise au coffre, statut des 3 niveaux.
- P5.4 Vue « charge sortante » exacte + bouton « Envoyer à Claude » (seul déclencheur d'envoi).
- P5.5 Réponse en streaming réinjectée, bascule « voir ce que Claude a reçu/renvoyé » (jetons), badge d'incident, bouton d'export.
- P5.6 Panneau coffre (liste masquée, ajout, suppression) et panneau incidents.
- P5.7 Playwright (Edge, stubs, desktop + mobile) couvrant le parcours d'acceptation n° 2 et 3 du cadrage.

**Gate** : `uv run pytest -q` ; `npx playwright test` vert.

## Phase 6 — Durcissement et revue de sécurité [TODO]

- P6.1 Refacto si dette visible des phases 2-5 (duplication, contournements) ; sinon consigner « rien à refactorer ».
- P6.2 Revue de sécurité du diff complet (skill `security-review`) ; corrections.
- P6.3 Boucle adversariale : pour chaque fuite du corpus adversarial, ajouter une règle si possible sans faux positif massif ; mesurer avant/après.
- P6.4 Tests des 6 invariants de sécurité de l'architecture regroupés dans `tests/security/test_invariants.py`.
- P6.5 E2E réel complet (`real_claude` + `real_ollama`) sur 3 scénarios de démo ; résultats au journal.
- P6.6 Couverture : `pytest --cov` ≥ 80 % sur `src/promptguard` (hors `__main__`).

**Gate** : ruff ; `uv run pytest -q --cov=promptguard --cov-fail-under=80` ; benchmark `--check` ; Playwright ; E2E réels verts.

## Phase 7 — Présentation [TODO]

- P7.1 Script `scripts/demo_seed.py` : dossier de démo et coffre préremplis de données fictives.
- P7.2 Captures automatiques (`tests-ui/screenshots.spec.js`) → `docs/assets/*.png` : analyse, charge sortante, réponse réinjectée, blocage proxy, coffre.
- P7.3 `README.md` vitrine : problème, démonstration (captures), architecture (schéma), preuves (chiffres du benchmark et du rapport canari tels que mesurés), installation, lancement, tests, limites honnêtes (cadrage § Limites), licence.
- P7.4 `presentation/index.html` : page autonome (sans dépendance externe hors polices), pitch problème → solution → preuves → limites → feuille de route post-MVP, thème clair/sombre, lisible mobile. Chiffres repris des rapports, jamais inventés.
- P7.5 `DOCUMENTATION/INDEX.md` + fiches minimales (`10_concepts/pseudonymisation.md`, `20_guides/installation.md`, `30_decisions/` reprenant les décisions du cadrage, `40_specs/api.md`).
- P7.6 `CHANGELOG.md` (v0.1.0) ; `tests_manuels.md` : recette utilisateur finale (parcours de démo sur la vraie machine, lecture du rapport canari, relecture avant push).
- P7.7 Mise à jour `_contexte/signals.md` : action P1 « Relire le MVP et décider du push », renvoi au journal.

**Gate** : suite complète de la phase 6 verte ; liens du README valides (test) ; `presentation/index.html` chargée sans erreur console (Playwright).

---

## Post-MVP (hors roadmap, pour mémoire)

Routage local Ollama ; mode clé API ; conditions Anthropic pour un usage commercial via abonnement ; comparaison OpenAI privacy-filter / piiranha ; anglais ; skill de cohérence `CLAUDE.md`/`AGENTS.md`/`GEMINI.md` (brief).
