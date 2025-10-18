# 🛡️ Projet 1 - Sécurité Informatique

## Scénarios de tentatives d’attaques

Dans le cadre de la mise en place d’un **système de détection d’anomalies et de gestion des logs**, nous avons choisi cinq scénarios d’attaques afin de démontrer la capacité du projet à détecter, corréler et visualiser des événements de sécurité.

---

## 1️⃣ Attaque par Brute Force (authentification)

### 🎯 Contexte et objectif
L’attaque par **brute force** vise à obtenir des identifiants valides sur un service exposé (SSH ou application web) en testant de nombreuses combinaisons d’utilisateurs et de mots de passe.  
Ce type d’attaque est extrêmement fréquent et souvent automatisé.

### 🧩 Description
L’attaquant tente de se connecter de manière répétée à un service (par exemple SSH) depuis une ou plusieurs adresses IP, en provoquant une série d’échecs d’authentification, parfois suivis d’un succès.  
Ces événements apparaissent dans les logs d’authentification système (`/var/log/auth.log`) et dans les logs applicatifs (requêtes `POST /login` renvoyant des codes HTTP 401).

### 📊 Indicateurs et données collectées
- Logs système SSH (`Failed password` / `Accepted password`)
- Logs applicatifs des tentatives de connexion (codes 401 puis 200)
- Données de flux réseau indiquant un nombre anormal de connexions courtes

### 🔍 Méthode de détection
- Plus de **N échecs d’authentification** pour un même couple *(IP, utilisateur)* en moins de **T minutes**
- Corrélation : détection d’un **pic d’échecs suivi d’un succès** depuis la même IP

### 📈 Visualisation et alertes
- Tableaux de bord Kibana : échecs par IP et par utilisateur + timeline des tentatives
- Alerte : **"Brute force detected"** lorsqu’un seuil est dépassé

### ✅ Justification du scénario
Ce scénario illustre une attaque réelle et fréquente.  
Il démontre la capacité du système à **corréler des logs de sources multiples (SSH, application web, IDS)** et à réagir en temps réel.

---

## 2️⃣ Balayage de Ports (Port Scanning)

### 🎯 Contexte et objectif
Avant toute exploitation, un attaquant procède souvent à une phase de reconnaissance pour identifier les services accessibles.  
Le **balayage de ports (port scan)** permet de cartographier les hôtes et ports ouverts sur un réseau.

### 🧩 Description
Le scénario consiste à simuler une série de connexions vers de multiples ports d’une machine cible.  
Ce comportement est identifiable dans les logs pare-feu ou IDS par une succession rapide de tentatives sur des ports distincts.

### 📊 Indicateurs et données collectées
- Logs pare-feu (`iptables`, `pf`) montrant des connexions rejetées sur plusieurs ports  
- Alertes IDS signalant des séquences de SYN vers différents services  
- Données Netflow ou de routeur révélant une activité anormale

### 🔍 Méthode de détection
- Détection d’une IP contactant plus de **M ports distincts** en moins de **N secondes**  
- Détection de **scans lents** par corrélation sur plusieurs périodes

### 📈 Visualisation et alertes
- Tableau Kibana : IP source ↔ nombre de ports sondés + timeline du scan  
- Alerte : **"Port scan detected"** lorsque l’activité dépasse le seuil défini

### ✅ Justification du scénario
Le **port scanning** est une étape préliminaire essentielle à toute attaque.  
Sa détection rapide permet de **prévenir les phases ultérieures d’exploitation** et met en évidence l’importance de la corrélation temporelle.

---

## 3️⃣ Déni de Service Distribué (DDoS)

### 🎯 Contexte et objectif
Le **déni de service** vise à rendre un service indisponible en saturant sa bande passante ou ses ressources.  
Simuler un pic de requêtes permet d’observer la réaction du système de détection.

### 🧩 Description
Un afflux massif de requêtes HTTP (ou connexions TCP) est envoyé vers le serveur web cible, provoquant une surcharge temporaire.  
Les logs montrent un grand nombre de requêtes en un court laps de temps, souvent accompagnées d’erreurs HTTP `503`.

### 📊 Indicateurs et données collectées
- Taux de requêtes HTTP/seconde dans les logs serveur web  
- Nombre d’erreurs `5xx` et de timeouts  
- Statistiques réseau (trafic entrant anormalement élevé)

### 🔍 Méthode de détection
- Seuils dynamiques basés sur la charge habituelle  
- Corrélation entre **volume de requêtes** et **taux d’erreurs HTTP**

### 📈 Visualisation et alertes
- Graphique Kibana : charge (requêtes/seconde) + IP générant le plus de requêtes  
- Alerte : **"Traffic anomaly – possible DoS"** lorsque le seuil est dépassé

### ✅ Justification du scénario
Ce scénario montre une attaque **fréquente et critique**.  
Une détection rapide permet de **prévenir les utilisateurs** et de réagir efficacement.

---

## 4️⃣ Injection SQL

### 🎯 Contexte et objectif
Les attaques par **injection SQL** exploitent une mauvaise validation des entrées utilisateur pour exécuter des requêtes arbitraires.  
Ce scénario illustre la détection d’une tentative d’exploitation au niveau applicatif.

### 🧩 Description
L’attaquant envoie une requête HTTP contenant un paramètre malveillant (`' OR '1'='1` ou `UNION SELECT`).  
Le serveur web journalise ces requêtes dans les logs d’accès, et la base de données peut générer des erreurs SQL.

### 📊 Indicateurs et données collectées
- Logs applicatifs : URI, paramètres, code de retour HTTP  
- Logs de base de données : erreurs de syntaxe, exécutions suspectes  
- Alertes IDS sur signatures d’injection SQL

### 🔍 Méthode de détection
- **Détection par signature** : recherche de motifs typiques d’injection  
- **Détection comportementale** : erreurs SQL ou volumes de données inhabituels

### 📈 Visualisation et alertes
- Tableau Kibana : paramètres suspects, endpoints ciblés, fréquence d’alertes  
- Alerte : **"SQL Injection detected"** lorsqu’un motif connu est identifié

### ✅ Justification du scénario
L’injection SQL reste une des vulnérabilités les plus critiques.  
Ce scénario démontre l’intérêt de **corréler logs applicatifs, base de données et IDS**.

---

## 5️⃣ Injection de Fichier via Formulaire d’Upload (fichier malveillant déguisé)

### 🎯 Contexte et objectif
De nombreux sites web permettent le **téléversement de fichiers** (ex : CV, image de profil).  
Sans contrôles côté serveur, un attaquant peut téléverser un **fichier malveillant** (ex : script PHP déguisé en PDF) afin d’exécuter du code ou d’obtenir un accès.

### 🧩 Description
L’attaquant exploite un formulaire HTTP `POST /upload` pour envoyer un fichier nommé `cv.pdf.php` contenant du code exécutable.  
Le serveur accepte le fichier sans validation et le stocke dans `/uploads/`.  
L’attaquant y accède ensuite via `GET /uploads/cv.pdf.php`, déclenchant l’exécution du code.

### 📊 Indicateurs et données collectées
- Logs applicatifs : nom du fichier, type MIME, chemin, IP et utilisateur  
- Logs web (`access.log`) : requêtes `POST /upload` et accès à `/uploads/`  
- Logs système : création de fichier exécutable dans un répertoire surveillé  
- Alertes IDS (Snort/Wazuh) : détection de contenu suspect ou extension `.php` dans un upload

### 🔍 Méthode de détection
1. **Analyse applicative** : comparer type MIME déclaré et extension réelle  
2. **Surveillance du système de fichiers** : détection de fichier exécutable uploadé  
3. **Corrélation d’événements** : upload suspect suivi d’un accès au même fichier  
4. **Alerte IDS** : règle “Malicious file upload” lors du transfert

### 📈 Visualisation et alertes
- Tableau Kibana : liste des fichiers uploadés (nom, extension, IP source)  
- Timeline des uploads et accès associés  
- Alerte : **"Suspicious File Upload Detected"** pour extension interdite ou exécution détectée

### ✅ Justification du scénario
Ce scénario illustre une vulnérabilité **fréquente et dangereuse** : l’upload sans contrôle suffisant.  
Il est simple à reproduire et très représentatif d’une **menace réelle** dans les applications web modernes.

---

📘 **Résumé des scénarios :**
1. Attaque par Brute Force  
2. Balayage de Ports  
3. Déni de Service Distribué (DDoS)  
4. Injection SQL  
5. Injection de Fichier via Formulaire d’Upload
