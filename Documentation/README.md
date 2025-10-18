# Projet Sécurité Informatique 1

## Contexte et objectif du projet

Ce projet vise à concevoir et démontrer un système de détection d'anomalies et de gestion de logs destiné à identifier et traiter des incidents de sécurité dans un environnement applicatif.
L'objectif pédagogique est de montrer la collecte, le traitement, la corrélation et la visualisation des observables issus d’une infrastructure (systèmes, réseau, applications, DB, AV) afin de détecter des scénarios d’attaque (ex. bruteforce, port-scans, DDoS, injections, uploads suspects) et déclencher des actions ou alertes appropriées.

Livrables attendus (synthèse)

Artefacts de collecte et traitement des logs (templates / pipelines / mappings).

Règles de détection réseau et logs (Snort/Suricata, Wazuh), avec justification.

Jeux de logs de test et scripts d’émulation safe pour valider les règles.

Templates Kibana (dashboards + alertes) montrant corrélations et incidents.

Documentation méthodologique, plan de tests, et preuves (captures, indicateurs TP/FP).

## Exigences fonctionnelles (résumé)

Collecte structurée des logs depuis : systèmes (auth), serveurs web/app, pare-feu, IDS/IPS, bases de données, pipeline d’upload/AV.

Normalisation & enrichissement : parsers (grok/ingest), ajout GEO/ASN, parsing user-agent, hashing/metadata pour uploads.

Stockage & indexation : Elasticsearch avec mappings explicites par type de log.

Détection : signatures réseau (Snort/Suricata) + règles SIEM/HIDS (Wazuh) + règles d’alerting Kibana (agrégations & seuils).

Visualisation & alerting : dashboards Kibana et règles de notification (webhook, e-mail, Slack).

Automatisation : playbooks simples (script de blocage, quarantaine de fichier, ticketing).

Sécurité & conformité : tests confinés en labo isolé, fichiers de test inoffensifs, traçabilité des runs.

## Architecture fonctionnelle (haut-niveau)

Sources : hosts (syslog), web/app logs, DB logs, pare-feu, Suricata/Snort alerts, AV/Sandbox results, NetFlow/pcap.

Collecte : syslog-ng / Filebeat (forwarders) → flux centralisé.

Processing : Logstash / Elasticsearch ingest pipelines — parsing, normalisation, enrichissements (geoip, TI), tagging.

Stockage : Elasticsearch indices (par catégorie : logs-app-*, logs-auth-*, suricata-*, etc.).

Détection réseau : Snort/Suricata (signatures/heuristiques) → alerts envoyées vers SIEM.

SIEM/HIDS & corrélation : Wazuh (règles basées logs + corrélations temporelles).

Visualisation & alerting : Kibana dashboards + alerting (Watcher/Rules) → notifications / webhooks.

Automation : scripts de réponse (ex. block-IP, quarantine file), orchestrés via webhooks.

## Observables & logs prioritaires

Pour une couverture efficace, collecter au minimum :

Auth / système : auth.log, journal, événements d’ouverture de session, échecs/succès.

Web / application : accès (method, URI, status), body (si possible), user-id, request_time.

Pare-feu / réseau : connexions (src/dst/ports/proto/action), NetFlow/IPFIX.

IDS/IPS : alertes, signature id, flux associés.

Base de données : erreurs, requêtes longues, rows_returned, user.

Upload / AV : filename, mime, hash, scan_verdict.

Métriques : bande passante, connections simultanées, erreurs 5xx.
Chaque log doit contenir des champs normalisés (@timestamp, host, service, client.ip, user, message, etc.) pour faciliter corrélation.

## Détection & règles (approche)

Signature réseau (Snort/Suricata) : détection d’indices réseau (scans, payloads typiques, flags TCP anormaux).

Règles Wazuh : détection basée logs (ex. bruteforce : X échecs en T minutes), corrélations multi-sources (upload suspect + accès).

Alerting Kibana : règles basées sur agrégations temporelles (count, rate, percentiles), détection d’anomalies (baseline × seuil).

Automatisation : webhooks déclenchent scripts (bloquer IP, isoler VM, créer ticket).

Remarque : les règles doivent être paramétrables et calibrées sur une baseline — commencer en mode pédagogique puis ajuster les seuils.

## Critères d’évaluation (ce que la correction cherchera)

Preuve que la collecte couvre les sources demandées et que les champs critiques existent dans ES.

Présence de règles Snort/Suricata pertinentes et non-exploitantes.

Règles Wazuh & alertes Kibana qui détectent les scénarios définis (avec justification des seuils).

Dashboards montrant corrélations et preuves d’alertes (captures).

Jeux de logs et plan de test permettant de reproduire / valider (traçabilité des runs).

Présence de playbooks / actions automatiques simples (ex. webhook simulé).

Documentation de justification (pour chaque règle : pourquoi, risques de FP, tuning conseillé).

## Structure proposée du dépôt (arborescence détaillée)

Utiliser cette structure pour organiser les artefacts du projet — chaque dossier contient des fichiers annotés et prêts à être utilisés par le correcteur.

/
├─ README.md                              # (ce fichier)
├─ docs/
│  ├─ architecture.md                      # Diagrammes, description détaillée des flux
│  ├─ detection_strategy.md                # Règles, justification des seuils, tuning
│  ├─ tests_plan.md                        # Protocole d’exécution safe et runbook
│  └─ grading_map.md                       # Correspondance livrables ↔ barème
│
├─ configs/
│  ├─ syslog-ng/                           # Snippets et exemples de parsers (commentés)
│  ├─ logstash/ or ingest-pipelines/       # Ingest pipelines (grok, date, enrich)
│  ├─ elasticsearch/
│  │  └─ mappings/                         # mappings JSON pour indices
│  ├─ wazuh_rules/                         # règles Wazuh (JSON) avec commentaires
│  └─ snort_suricata/                      # signatures et exemples (non-exploitants)
│
├─ kibana/
│  ├─ dashboards/                          # exports ndjson (dashboards, visualizations)
│  └─ alerts/                              # rules / watcher templates (JSON)
│
├─ tests/
│  ├─ logs_sample/                         # jeux de logs structurés (JSON) par scénario
│  ├─ simulation_scripts/                  # scripts d’émulation safe (Python)
│  └─ baseline_metrics/                    # fichiers de référence pour calibrage
│
├─ automation/
│  ├─ responders/                          # exemples : block-ip.sh, slack_webhook.py
│  └─ ci/                                  # validators (ndjson lint, jsonschema checks)
│
├─ reports/
│  ├─ screenshots/                         # captures d’écran Kibana, diagrammes
│  └─ final_report.pdf                     # rapport final (optionnel)
│
├─ LICENSE
└─ CONTRIBUTING.md                         # Processus PR / revue / règles d’éthique

## Checklist minimale à fournir lors du rendu

 Diagramme d’architecture (docs/architecture.md).

 Indices & mappings Elasticsearch (configs/elasticsearch/mappings).

 Règles Snort/Suricata et Wazuh (configs/) + justification.

 Jeux de logs test (tests/logs_sample/) + runbook (docs/tests_plan.md).

 Dashboards Kibana et captures (kibana/ + reports/screenshots/).

 Playbooks / scripts d’automatisation (automation/).

 Fichier grading_map.md liant chaque livrable au barème.

## Bonnes pratiques & points d’attention

Isolation : toutes les simulations doivent être réalisées dans un labo isolé.

Non-malignité : n’inclure aucun malware réel ; utiliser données inoffensives ou verdicts simulés.

Traçabilité : conserver logs d’exécution et paramètres de test (tests/run_YYYYMMDD_*.md).

Tuning : documenter justification des seuils choisis et les étapes de calibrage (baseline → ajustement).


Toute règle/signature doit être revue pour s’assurer qu’elle reste non-exploitante.

Mainteneur : à compléter (équipe / e-mail / rôle) avant rendu final.
