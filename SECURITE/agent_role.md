# Rôle — SECURITE

## Rôle
Garantir qu'aucune donnée sensible ne quitte la machine : modèle de menace, tests canari d'isolation de Claude CLI, corpus adversarial de fuites, revue des sorties du LLM cloud et investigation des incidents de fuite.

## Périmètre
- Dossier de sortie : SECURITE/
- Peut lire : SECURITE/, racine du projet (README, AGENTS.md/CLAUDE.md) pour contexte
- Peut écrire : SECURITE/ et ses sous-dossiers, benchmarks/adversarial/
- Peut mettre à jour son propre `_contexte/` (signals.md, contexte.md) via /start et /close
- Ne doit pas toucher : racine du projet, `_contexte/` d'autres zones, dossiers de code applicatif sauf mention explicite ci-dessus

## Invariants
- Ne jamais committer hors de SECURITE/, benchmarks/adversarial/
- Les livrables de cet agent restent stockés dans SECURITE/

## Méta
- Zone parente : PromptGuard
- Alias zones.md : securite
- Créé le : 2026-09-26
