# Signals — PromptGuard   (MAJ 2026-09-26)

## Actions ouvertes
- [P1|ouvert] Lancer l'exécution de `roadmap_mvp.md` (phase 0 — Socle projet) en session autonome (modèle Opus, mode auto activé).
  fait quand: `journal_roadmap_mvp.md` créé et la phase 0 passe au gate vert (`ruff`, `pytest`, `playwright`)
  réf: roadmap_mvp.md, _docs/cadrage_mvp.md, _docs/architecture_mvp.md

## Contexte chaud
- Environnement constaté le 2026-09-26 : Claude Code CLI 2.1.259 (flags d'isolation confirmés présents) ; Ollama avec `gemma4:12b`, `gemma4:12b-32k`, `gemma4:e4b` installés ; Python 3.13.14 ; uv 0.9.11 ; Node 22.16 ; gh 2.74 ; RTX 4060 8 Go ; remote `origin` = `github.com/ServOMorph/PromptGuard`.
- Exception validée à la règle CLAUDE.md du checkpoint `/compact` : `roadmap_mvp.md` s'exécute en autonomie complète, sans pause ni question à l'utilisateur. Limitée à ce fichier, ne vaut pas pour les autres roadmaps du projet.
- Budget indicatif d'appels réels au vrai Claude CLI pendant la roadmap : ≤ 40 sur l'ensemble des phases (compteur à tenir dans `journal_roadmap_mvp.md`, pas encore créé).
- Règle d'arrêt de la roadmap : seul cas d'interruption autorisé = fuite avérée d'une donnée sensible réelle de l'utilisateur (voir `roadmap_mvp.md`, règle d'exécution 4).

## Dernière session (2026-09-26)
<!-- Écrasé intégralement par /close. Synthèse < 25 lignes. -->
# Session du 2026-09-26

## Décisions prises
- Claude CLI par abonnement (pas de clé API) ; défense en profondeur wrapper isolé + proxy local `ANTHROPIC_BASE_URL`.
- Détection/pseudonymisation : `piighost` (MIT) + validateurs IA_V7 (IBAN, NIR, Luhn, SIRET) + Ollama en niveau 3.
- Validation systématique de la charge sortante avant tout envoi.
- Licence Apache 2.0 ; français seul ; PII + secrets techniques dans le périmètre.
- Roadmap MVP en autonomie complète : exception ponctuelle au checkpoint `/compact`, commit local par phase, jamais de push automatique, appels réels bornés.

## Livrables produits ou modifiés
- `_docs/cadrage_mvp.md` : créé (décisions de cadrage, critères d'acceptation, limites assumées).
- `_docs/etat_de_l_art.md` : créé (briques réutilisées, projets de référence, écarts et raisons).
- `_docs/architecture_mvp.md` : créé (flux, découpage du code, contrats API, invariants de sécurité).
- `roadmap_mvp.md` : créé (8 phases, gates, replis, règles d'exécution autonome).

## Hypothèses validées / invalidées
- EN ATTENTE : compatibilité du proxy local avec l'abonnement Claude CLI (canari prévu en phase 1).
- EN ATTENTE : `piighost` installable et fonctionnel en Python 3.13 (vérifié sur PyPI uniquement, pas encore installé).

## Prochaine étape exacte
Lancer une session autonome (Opus, mode auto) avec la consigne « Exécute roadmap_mvp.md » pour démarrer la phase 0.

## Question bloquante pour la session suivante
Aucune
