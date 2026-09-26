# Contexte — securite

## Objectif (immuable sauf décision explicite)
Garantir qu'aucune donnée sensible ne quitte la machine : modèle de menace, tests canari d'isolation de Claude CLI, corpus adversarial de fuites, revue des sorties du LLM cloud et investigation des incidents de fuite.

## Stack / contraintes techniques (stable, rarement modifié)
Hérite de la stack du projet (voir `_contexte/contexte.md` racine). Points propres au rôle : Claude Code CLI 2.1.259 en mode headless (`claude -p`, flags d'isolation `--tools`, `--strict-mcp-config`, `--setting-sources`, `--system-prompt`, `--disable-slash-commands`, `--no-session-persistence` — efficacité non prouvée, à établir par test canari) ; `--bare` exclu (exige une clé API) ; coffre SQLite + cryptography AES-GCM + keyring (Windows DPAPI) ; détection regex/validateurs + GLiNER2-PII + Ollama gemma4:12b. Cadrage : `_docs/brief_initial.md`.

## État actuel (réécrit intégralement à chaque /close)
Projet initialisé. Aucun livrable produit.

## Décisions structurantes (append only — 10 entrées max, 5 lignes max/entrée, archiver au-delà)
- 2026-09-26 : Initialisation du protocole vibecoding.
