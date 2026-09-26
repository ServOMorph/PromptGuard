# Contexte — PromptGuard

## Objectif (immuable sauf décision explicite)
Détecte et anonymise les données sensibles d'un prompt avant envoi à un LLM cloud, avec stockage chiffré local pour réutilisation.

## Stack / contraintes techniques (stable, rarement modifié)
Python 3.13, Flask, SQLite, JS vanilla, Ollama (gemma4:12b), GLiNER2-PII, cryptography AES-GCM + keyring, Claude Code CLI headless, pytest, Playwright, uv, ruff

## État actuel (réécrit intégralement à chaque /close)
Projet initialisé. Aucun livrable produit.

## Décisions structurantes (append only — 10 entrées max, 5 lignes max/entrée, archiver au-delà)
- 2026-09-26 : Initialisation du protocole vibecoding.
