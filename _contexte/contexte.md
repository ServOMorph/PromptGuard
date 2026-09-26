# Contexte — PromptGuard

## Objectif (immuable sauf décision explicite)
Détecte et anonymise les données sensibles d'un prompt avant envoi à un LLM cloud, avec stockage chiffré local pour réutilisation.

## Stack / contraintes techniques (stable, rarement modifié)
Python 3.13, Flask, SQLite, JS vanilla, Ollama (gemma4:12b), GLiNER2-PII, cryptography AES-GCM + keyring, Claude Code CLI headless, pytest, Playwright, uv, ruff

## État actuel (réécrit intégralement à chaque /close)
Cadrage du MVP terminé : `_docs/cadrage_mvp.md` (décisions, critères d'acceptation, limites), `_docs/etat_de_l_art.md` (briques réutilisées), `_docs/architecture_mvp.md` (flux, code, API, invariants sécurité). `roadmap_mvp.md` créé : 8 phases en autonomie complète, gates et replis définis. Aucun code applicatif écrit ; phase 0 à lancer.

## Décisions structurantes (append only — 10 entrées max, 5 lignes max/entrée, archiver au-delà)
- 2026-09-26 : Initialisation du protocole vibecoding.
- 2026-09-26 : Cadrage MVP validé — Claude CLI par abonnement, défense en profondeur wrapper+proxy local, détection par piighost+validateurs IA_V7+Ollama, validation systématique avant envoi, licence Apache 2.0, français seul, PII+secrets techniques. Détails : `_docs/cadrage_mvp.md`.
- 2026-09-26 : Roadmap MVP en autonomie complète (`roadmap_mvp.md`) — exception ponctuelle au checkpoint `/compact` de CLAUDE.md, limitée à ce fichier ; commit local par phase, jamais de push automatique.
