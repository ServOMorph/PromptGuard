# État de l'art — outils réutilisables (2026-09-26)

Recherche effectuée le 2026-09-26 (web, GitHub, Hugging Face, PyPI). Chiffres de maturité (étoiles, commits) relevés ce jour, volatils.

## Verdict

Pas de projet existant couvrant le besoin complet (UI locale + coffre chiffré persistant + validation humaine + Claude CLI par abonnement + français). Le MVP assemble des briques existantes ; le code propre se limite à l'orchestration, au proxy de contrôle, à l'UI et aux validateurs FR.

## Briques retenues

| Brique | Rôle dans PromptGuard | Licence | Remarque |
|---|---|---|---|
| [piighost](https://github.com/Athroniaeth/piighost) 1.8.0 (PyPI) | Détection pluggable (regex FR/EU, GLiNER2, LLM), jetons stables `<<PERSON:1>>` par conversation, restauration, AES-GCM, mémoire SQL | MIT | 13 étoiles, ~900 commits, CI + codecov. Risque de maturité → version épinglée + interface interne `Detector`/`Pseudonymizer` pour pouvoir la remplacer. Extras : `gliner2`, `crypto`, `sqlalchemy`. API possiblement asynchrone (à confirmer en phase 2). |
| [gliner2](https://pypi.org/project/gliner2/) 2.0.0 + [fastino/gliner2-privacy-filter-PII-multi](https://huggingface.co/fastino/gliner2-privacy-filter-PII-multi) | NER PII multilingue (FR inclus), 42 types dont secrets, CPU | Apache 2.0 | Meilleur F1 moyen sur SPY (0,471) parmi les modèles testés par Fastino. Précision faible sur les noms (seuil `person` à relever). **Le paquet embarque un client d'API distante : l'inférence locale (extra `local`) doit être imposée et prouvée réseau coupé.** |
| Validateurs IA_V7 (`D:\ServOMorph\IA_V7\src\ia_v7\services\commands.py`) | IBAN mod 97, NIR, Luhn, téléphone, IP, adresses, civilités | MIT (même auteur) | Base mesurée : 21/26 cas conformes, 21 cas verrouillés. |
| Benchmark IA_V7 (`scripts/benchmark_rgpd.py`, `benchmarks/rgpd_corpus.json`) | Modèle du benchmark verrouillé | MIT | À étendre (secrets, noms sans civilité, adversarial). |
| Socle IA_V7 (`app.py`, `config.py`, `infrastructure/database.py`, `infrastructure/ollama.py`, `static/`, `templates/`) | App factory Flask, config, SQLite, client Ollama streaming, UI JS vanilla, vendors `marked` + `DOMPurify` | MIT | Reprise sélective, pas de copie intégrale. |
| [cryptography](https://pypi.org/project/cryptography/) 50.x | AES-GCM du coffre | Apache 2.0 / BSD | — |
| [keyring](https://pypi.org/project/keyring/) 25.x | Clé maître dans le Gestionnaire d'identifiants Windows | MIT | Non interactif sous Windows. |
| httpx | Proxy : relais amont streaming SSE | BSD | — |

## Projets de référence (architecture, non importés)

| Projet | Apport pour PromptGuard |
|---|---|
| [PrivAiTe](https://github.com/crp4222/PrivAiTe) (BSD-3, 43 étoiles, v0.5.0) | Preuve de concept la plus proche : proxy `ANTHROPIC_BASE_URL` pour Claude Code qui **relaie l'authentification propre du CLI** (donc a priori l'abonnement), scrubbing y compris des arguments d'appels d'outils, restauration en streaming, FR supporté (spaCy). Mapping en mémoire seulement, pas de coffre ni d'UI. Modèle pour le proxy de phase 1/3. |
| [DontFeedTheAI](https://github.com/zeroc00I/LLM-anonymization) (MIT, 655 étoiles) | Ollama local + regex, coffre de correspondances par « engagement », boucle d'amélioration par détection de fuites, 53 fixtures d'intégration. Modèle pour le corpus adversarial et la boucle fuite → règle. |
| [token-proxy](https://github.com/zolderio/token-proxy) (Apache 2.0) | Restauration de pseudonymes en streaming SSE par tampon de fin de chunk (jetons coupés entre deux chunks). Clé API seulement. |
| [kiji-proxy](https://github.com/dataiku/kiji-proxy) (Apache 2.0, 436 étoiles) | Proxy + app Electron de supervision des requêtes. Référence UX pour la vue « charge sortante ». |
| [LLM Guard](https://protectai.github.io/llm-guard/) (ProtectAI) | Pattern Anonymize/Vault/Deanonymize. Écarté : dépend de Presidio/spaCy, plus lourd que piighost pour le même service. |

## Écartés

| Outil | Raison |
|---|---|
| Microsoft Presidio + spaCy `fr_core_news_*` | Aucun reconnaisseur FR natif (NIR, SIRET à écrire), NER spaCy inférieur à GLiNER2 sur les PII, dépendances lourdes. Reste accessible via l'extra `presidio` de piighost si besoin ultérieur. |
| OpenAI privacy-filter, piiranha, betterdataai | Redondants avec GLiNER2-PII pour le MVP ; candidats de comparaison post-MVP. |
| DataFog | Orientation performance/regex, pas d'apport sur le FR. |

## Données d'évaluation

- Corpus principal : synthétique FR, construit dans le projet (aucune donnée réelle).
- Références externes pour post-MVP : [Text Anonymization Benchmark](https://huggingface.co/datasets/mattmdjaga/text-anonymization-benchmark-train), [beki/privy](https://huggingface.co/datasets/beki/privy), liste [awesome-anonymization-for-llms](https://github.com/malteos/awesome-anonymization-for-llms).

## Environnement constaté (2026-09-26)

Claude Code 2.1.259 (flags `--tools`, `--strict-mcp-config`, `--setting-sources`, `--system-prompt`, `--disable-slash-commands`, `--no-session-persistence`, `--output-format stream-json` présents) ; Ollama avec `gemma4:12b` ; Python 3.13.14 ; uv 0.9.11 ; Node 22.16 ; gh 2.74 ; RTX 4060 8 Go ; remote `origin` = `github.com/ServOMorph/PromptGuard`.
