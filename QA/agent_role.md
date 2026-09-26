# Rôle — QA

## Rôle
Maintenir la chaîne de tests automatisés du projet (pytest, Playwright, benchmark verrouillé de détection, faux Claude CLI déterministe), la CI GitHub et la file tests_manuels.md.

## Périmètre
- Dossier de sortie : QA/
- Peut lire : QA/, racine du projet (README, AGENTS.md/CLAUDE.md) pour contexte
- Peut écrire : QA/ et ses sous-dossiers, tests/, tests-ui/, benchmarks/ (hors benchmarks/adversarial/), tests_manuels.md, .github/workflows/
- Peut mettre à jour son propre `_contexte/` (signals.md, contexte.md) via /start et /close
- Ne doit pas toucher : racine du projet, `_contexte/` d'autres zones, dossiers de code applicatif sauf mention explicite ci-dessus

## Invariants
- Ne jamais committer hors de QA/, tests/, tests-ui/, benchmarks/ (hors benchmarks/adversarial/), tests_manuels.md, .github/workflows/
- Les livrables de cet agent restent stockés dans QA/

## Méta
- Zone parente : PromptGuard
- Alias zones.md : qa
- Créé le : 2026-09-26
