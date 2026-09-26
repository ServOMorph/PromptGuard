# Cadrage MVP — PromptGuard (2026-09-26)

Source des décisions : session de cadrage du 2026-09-26 (questions/réponses utilisateur). Complète `_docs/brief_initial.md`, ne le remplace pas.

## Objectif du MVP

MVP de présentation, fonctionnel sur la machine de l'utilisateur, démontrant qu'on peut converser avec Claude (abonnement, via Claude Code CLI) sans qu'aucune donnée sensible connue ne quitte la machine, avec preuves chiffrées publiées (benchmark de détection, tests canari d'isolation).

## Décisions validées

| Sujet | Décision |
|---|---|
| Accès Claude | Claude Code CLI avec l'abonnement de l'utilisateur. Pas de clé API. `--bare` exclu. |
| Contrôle de sortie | Défense en profondeur : wrapper `claude -p` isolé (flags + cwd bac à sable vide) ET proxy local via `ANTHROPIC_BASE_URL` qui bloque toute requête contenant une valeur connue. Compatibilité proxy/abonnement à prouver par test canari ; repli automatique sur wrapper seul si échec (cf. roadmap, règles de repli). |
| Moteur de détection | `piighost` (MIT) en dépendance épinglée, derrière une interface interne remplaçable. Compléments : validateurs d'IA_V7 (IBAN mod 97, NIR clé, Luhn, SIRET) et niveau 3 Ollama (détection contextuelle). |
| Validation avant envoi | Systématique : l'UI affiche la charge exacte sortante ; rien ne part sans clic. |
| Périmètre fonctionnel | Coffre chiffré ; réinjection des vraies valeurs dans la réponse (affichage local uniquement) ; joindre un fichier du dossier de travail (lu, nettoyé) ; export de la réponse dans le dossier. |
| Hors périmètre | Routage local Ollama (traitement de la tâche par le LLM local) ; mode clé API ; multi-utilisateur ; autres OS que Windows ; autres LLM cloud. |
| Langue | Français seul (UI, corpus, benchmark). Anglais non garanti. |
| Catégories | PII (nom, email, téléphone, adresse, IBAN, NIR, carte bancaire, SIRET/SIREN, IP, date de naissance) + secrets techniques (clés API, tokens, mots de passe, chaînes de connexion). |
| Licence | Apache 2.0. |
| Livrables de présentation | README vitrine (captures générées par Playwright) + page HTML de présentation autonome. |
| Roadmap | Exécution autonome complète, sans checkpoint `/compact` (exception à la règle CLAUDE.md, limitée à `roadmap_mvp.md`). |
| Git | Un commit local par phase terminée (tests verts). Aucun push : décidé par l'utilisateur après relecture. |
| Appels réels | Autorisés, bornés : stubs pour les tests courants ; vrai Claude CLI uniquement pour canaris d'isolation et un E2E par phase concernée ; vrai Ollama autorisé. |

## Critères d'acceptation du MVP

1. `uv run python -m promptguard` lance l'UI sur `http://127.0.0.1:4024`.
2. Parcours démontrable : choisir un dossier → saisir un prompt contenant un IBAN, un nom, une clé API → l'UI affiche les détections, la charge sortante pseudonymisée → validation → réponse de Claude affichée avec les vraies valeurs réinjectées → export dans le dossier.
3. Une valeur stockée au coffre est réutilisable par `@alias` sans la retaper.
4. Le proxy bloque (HTTP 403 + incident journalisé) toute requête contenant une valeur du coffre ou de la conversation — prouvé par test.
5. Toute réponse de Claude contenant une valeur sensible connue est signalée comme incident.
6. Benchmark FR verrouillé : 100 % des cas verrouillés conformes ; rappel global et adversarial mesurés et publiés tels quels.
7. Rapport canari publié : ce que Claude CLI envoie réellement (outils, prompt système, contenu `~/.claude`) avec et sans flags d'isolation.
8. Suite automatisée verte : ruff, pytest, Playwright (stubs), CI GitHub (stubs).
9. README vitrine et page de présentation, avec limites énoncées honnêtement (la détection n'est jamais à 100 %).

## Limites assumées (à afficher dans le README)

- La garantie déterministe ne porte que sur les valeurs connues (coffre + détections validées de la conversation). Une donnée jamais détectée ni marquée manuellement peut partir si l'utilisateur valide l'envoi.
- Compatibilité proxy/abonnement dépendante du comportement de Claude Code (version testée : 2.1.259).
- Usage commercial via abonnement Claude d'un tiers : conditions Anthropic non vérifiées (point ouvert du brief).
