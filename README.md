# PromptGuard

## Objectif

Détecte et anonymise les données sensibles d'un prompt avant envoi à un LLM cloud (Claude, via abonnement), avec stockage chiffré local pour réutilisation.

## Stack

Python 3.13, Flask, SQLite, JS vanilla, Ollama (`gemma4:12b`), GLiNER2-PII, cryptography AES-GCM + keyring, Claude Code CLI headless, pytest, Playwright, uv, ruff.

## Structure

- `_docs/` : cadrage et documentation de conception (brief initial, cadrage MVP, état de l'art, architecture).
- `SECURITE/`, `QA/`, `DOCUMENTATION/` : zones-agents du protocole vibecoding.
- `roadmap_mvp.md` : feuille de route du MVP (8 phases, exécution autonome).

## État actuel

Cadrage du MVP terminé (`_docs/cadrage_mvp.md`, `_docs/etat_de_l_art.md`, `_docs/architecture_mvp.md`). Roadmap `roadmap_mvp.md` prête (8 phases). Aucun code applicatif encore écrit.

## Licence

Apache 2.0 (fichier `LICENSE` prévu en phase 0 de la roadmap).
