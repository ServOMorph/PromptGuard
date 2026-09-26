# Contexte — qa

## Objectif (immuable sauf décision explicite)
Maintenir la chaîne de tests automatisés du projet (pytest, Playwright, benchmark verrouillé de détection, faux Claude CLI déterministe), la CI GitHub et la file tests_manuels.md.

## Stack / contraintes techniques (stable, rarement modifié)
Hérite de la stack du projet (voir `_contexte/contexte.md` racine). Points propres au rôle : pytest (+ pytest-cov), Playwright (Node 22, Edge), uv, ruff ; benchmark à N cas verrouillés (modèle : `IA_V7/scripts/benchmark_rgpd.py`, 21 cas obligatoires) ; CI sans Ollama ni Claude (stubs déterministes). Cadrage : `_docs/brief_initial.md`.

## État actuel (réécrit intégralement à chaque /close)
Projet initialisé. Aucun livrable produit.

## Décisions structurantes (append only — 10 entrées max, 5 lignes max/entrée, archiver au-delà)
- 2026-09-26 : Initialisation du protocole vibecoding.
