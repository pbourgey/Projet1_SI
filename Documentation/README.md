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

### 🔹 **Snort** *(ou Suricata)*
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

## 📎 Liens vers les autres documents du projet

- 🔗 [**Installation des outils pour le projet**](./Installation.md)   

- 🔗 [**Explication des scénarios**](./Scenarios.md) 
