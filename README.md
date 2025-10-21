# 🧠 Projet – Système de détection d’anomalies & gestion de logs

Bienvenue dans le dépôt du projet **Système de détection d’anomalies et de gestion de logs**, réalisé dans le cadre du module de cybersécurité.  
Ce projet a pour objectif de **concevoir une architecture complète de détection**, de **collecte** et de **visualisation** d’incidents de sécurité à partir des journaux système, réseau et applicatifs.

---

## 🎯 Objectif du projet

Mettre en place une infrastructure capable de :
- **Collecter** des logs provenant de différentes sources (serveurs, réseau, applications).  
- **Détecter** des comportements suspects ou malveillants via des règles d’analyse (IDS/IPS).  
- **Corréler** les événements dans une base centralisée (Elasticsearch).  
- **Visualiser et alerter** via des tableaux de bord interactifs (Kibana).  

Le but final est de démontrer la **détection de plusieurs scénarios d’attaque simulés**, d’analyser les logs correspondants et de proposer des contre-mesures adaptées.

---

## 🧩 Architecture globale

L’architecture repose sur une chaîne d’outils interconnectés :  

[ Sources de logs ]  
↓  
Syslog-ng  
↓  
Snort / Wazuh  
↓  
Elasticsearch  
↓  
Kibana  


Chaque composant joue un rôle précis dans la détection et la gestion des logs 👇

---

## 🛠️ Outils utilisés et leur rôle

### 🔹 **Syslog-ng**
> *Collecte et centralise les logs depuis toutes les sources.*

- Reçoit les journaux système, réseau et applicatifs.  
- Formate, normalise et redirige les logs vers Elasticsearch.  
- Sert de **point d’entrée unique** de la chaîne de détection.

---

### 🔹 **Snort**
> *IDS/IPS — analyse du trafic réseau en temps réel.*

- Inspecte les paquets réseau pour repérer des signatures d’attaque.  
- Détecte des comportements anormaux (scans, tentatives d’exploitation, DoS…).  
- Génère des alertes envoyées vers le système de logs central.

---

### 🔹 **Elasticsearch**
> *Base de données et moteur d’analyse.*

- Stocke tous les logs collectés sous forme structurée.  
- Permet la **recherche rapide**, la **corrélation d’événements** et les **agrégations temporelles**.  
- Sert de cœur analytique de la plateforme.

---

### 🔹 **Kibana**
> *Interface de visualisation et d’alerting.*

- Permet de créer des **dashboards interactifs** à partir des données d’Elasticsearch.  
- Configure des **alertes automatiques** (e-mail, Slack, webhook) selon des seuils ou des anomalies.  
- Fournit la **vue globale** sur la sécurité du système.

---

## 🧠 Fonctionnement général

1. **Les machines et applications** génèrent des logs (connexion, erreur, upload, etc.).  
2. **Syslog-ng** collecte et normalise ces logs.  
3. **Snort** surveille le trafic réseau et crée des alertes lors d’activités suspectes.  
4. Tous les événements sont **indexés dans Elasticsearch** pour stockage et corrélation.  
5. **Kibana** permet de visualiser, filtrer, agréger et déclencher des alertes automatiques.  

L’ensemble forme un **SIEM** (Security Information and Event Management) simplifié et reproductible.

---

## 🔄 Analyse et Conclusion

Dans le cadre de ce projet, nous avons mis en place une infrastructure complète de collecte, corrélation et détection d’événements de sécurité, fondée sur l’écosystème Syslog – Elasticsearch – Kibana – Snort. L’objectif principal était de démontrer la capacité du système à identifier et interpréter différents types d’attaques, à partir de logs hétérogènes provenant des serveurs, pare-feux et applications.

### Fonctionnalités opérationnelles :

- L’ensemble des composants nécessaires à la collecte et à la centralisation des logs a été correctement déployé :

- Syslog reçoit et stocke les journaux des systèmes et applications.

- Les scénarios d’attaques (brute force, scan de ports, DDoS, injection SQL et upload malveillant) ont tous été simulés avec succès, générant des traces réelles exploitables.

- Les règles de détection et de corrélation ont été validées à partir de ces logs : par exemple, la détection d’un nombre anormal d’échecs d’authentification sur SSH, ou d’un grand nombre de ports sondés sur une courte période.

- Le moteur IDS a produit des alertes pertinentes, confirmant la bonne intégration du volet détection.

### Limite rencontrée : la liaison Syslog ↔ Elasticsearch

Le seul élément non fonctionnel à ce stade concerne la transmission des logs de Syslog vers Elasticsearch, nécessaire pour l’indexation et la visualisation dans Kibana. Cette anomalie empêche la création automatique des tableaux de bord et la génération en temps réel des visualisations prévues.

Le dysfonctionnement provenait probablement d'un manque de compréhension des transferts de données entre les deux outils.
Cependant, cette panne n’affecte ni la collecte locale des logs, ni la capacité du système à détecter les attaques. Les preuves de détection ont été établies manuellement par analyse des journaux, ce qui démontre la robustesse de la chaîne de détection même sans la brique d’indexation.

### Mesures compensatoires

Pour pallier ce problème, nous avons exporté les logs au format brut et analysé les indicateurs manuellement.
Ces mesures ont permis de valider la logique de détection et d’assurer la continuité du projet malgré la panne technique.

### Conclusion générale

Ce projet a permis de mettre en pratique l’ensemble des compétences liées à la supervision de la sécurité :

- collecte et centralisation des logs,

- détection et corrélation d’incidents,

- analyse et interprétation des événements,

- et documentation méthodique d’une infrastructure SIEM.

Malgré le dysfonctionnement du lien Syslog–Elasticsearch, les objectifs pédagogiques essentiels ont été atteints :

- La chaîne de détection fonctionne et produit des résultats concrets.

- Les scénarios d’attaque ont été exécutés et correctement identifiés via les traces générées.

- La méthodologie de traitement des incidents et la démarche d’analyse de logs ont été validées.

- La documentation du projet témoigne d’une bonne compréhension du fonctionnement global d’un système SIEM.

En conclusion, le projet atteint son objectif de démonstration :
il illustre clairement la détection proactive d’incidents de sécurité à partir de sources multiples, et met en évidence la capacité à analyser et exploiter les logs même sans l’aide d’un outil de visualisation automatisé.

<img width="941" height="311" alt="diagramme_final" src="https://github.com/user-attachments/assets/76bdd1d8-807e-4026-8fa6-3854554e25e0" />



---

## 📎 Liens vers les autres documents du projet

- 🔗 [**Installation des outils pour le projet**](./Documentation/Installation.md)   

- 🔗 [**Explication des scénarios**](./Documentation/Scenarios.md) 

- 🔗 [**Guide d'utilisation**](./Documentation/Utilisation.md) 
