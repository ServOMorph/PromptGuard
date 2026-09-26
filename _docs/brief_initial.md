# Brief initial — PromptGuard (2026-09-26)

Cadrage issu de la session de création du projet (`/create_projet`, depuis le kit VibeObs).

## Brief utilisateur

- Retirer toutes les données sensibles des échanges avec des IA cloud.
- Base de code : `D:\ServOMorph\IA_V7` (importer ce qui est utile).
- Workflow : l'UI s'ouvre, l'utilisateur choisit un dossier physique de travail sur le PC, puis converse.
- À chaque prompt, un LLM local (Ollama) analyse le prompt, détecte les données sensibles (ex. RIB), les affiche et propose de les stocker chiffrées localement (clé de chiffrement) pour ne plus avoir à les redonner.
- Le LLM local repère les actions à faire : si la tâche exige les données sensibles (ex. écrire un mail avec le RIB), il la traite lui-même ; sinon, ou s'il peut substituer des valeurs, il transmet à Claude CLI.
- Chaque retour de Claude CLI est contrôlé : présence de données sensibles = bug à investiguer manuellement.
- But ultime : environnement de travail IA garantissant le contrôle total des données, branchable sur une IA comme Claude.
- MVP rapide de démonstration, fonctionnel sur la machine de l'utilisateur ; limité à Claude CLI (déjà installé).
- Tout doit être automatisé (tests, tests manuels) car l'IA teste seule.
- Centre d'expérimentation : faire tourner une IA en autonomie de A à Z, avec un workflow complet d'écriture de documentation ; protocole amélioré en continu (comme VibeObs).
- Visée commerciale, open source sur GitHub. Prendre le temps de bien configurer.

## Décisions validées

- Stack : voir `_contexte/contexte.md`.
- Détection en 3 niveaux : regex + validateurs (repris d'IA_V7) → GLiNER2-PII → Ollama (contexte + routage local/cloud).
- Stockage sensible : coffre chiffré (AES-GCM, clé dans le Gestionnaire d'identifiants Windows), accessible au seul LLM local, jamais transmis ni exposé au LLM cloud.
- Agents retenus : `securite`, `qa`, `documentation`. Agent `methode` envisagé plus tard.
- Cohérence `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` : skill local à créer, déclenché à chaque modification ; tronc commun canonique conservé séparément ; une section propre par fichier pour les divergences ; liste extensible des fichiers `.md` requis par LLM. Expérimentation locale, pas de généralisation au kit pour l'instant.

## Points critiques identifiés (à traiter dans la roadmap)

1. La garantie de 100 % ne peut pas reposer sur la détection seule (benchmark IA_V7 : 80,8 % de cas conformes, 40 % en adversarial). Proposition : l'UI affiche la charge exacte sortante et rien ne part sans validation explicite.
2. Isolation de Claude CLI : `--bare` exige une clé API (pas l'abonnement). Sans lui, `claude -p` charge `~/.claude` (CLAUDE.md, mémoire, MCP). Parade à prouver par test canari : `--tools ""`, `--strict-mcp-config`, `--setting-sources`, `--system-prompt`, `--disable-slash-commands`, `--no-session-persistence`, cwd = dossier bac à sable vide. Flags présents dans Claude Code 2.1.259 ; efficacité non prouvée.
3. Le dossier de travail de l'utilisateur n'est jamais passé à Claude CLI ; seul PromptGuard le lit et transmet du contenu nettoyé.
4. Matériel : RTX 4060 8 Go (gemma4:12b = 7,6 Go) → GLiNER sur CPU (48 Go RAM).
5. Usage commercial via l'abonnement Claude d'un utilisateur : conditions Anthropic à vérifier ; prévoir un mode clé API. Licence (MIT ou Apache 2.0) à trancher.
