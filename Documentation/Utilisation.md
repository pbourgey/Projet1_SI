# IMPLEMENTATION DU PROJET 

Nous allons voir dans cette section les différents résultats en fonction de l'attaque. L'analyse est en trois partie, la première est la règle établie dans le fichier /etc/snort/rules/local.rules. La seconde est le commande pour executer l'attaque. Et enfin la troisième partie représente les résultats. 

Pour effectuer ces attaques, nous avons créé une deuxième VM qui fera office de machine attaquante. Nous avons établi un sous-réseau, avec 192.168.10.1 pour l'adresse IP de la VM avec Snort et 192.168.10.2 celle de la VM d'attaques. 

Sur la VM Snort, nous avons 3 console avec une commande lancée :  

Console 1 : sudo snort -q -c /etc/snort/snort.conf -i enp0s3  
-> Lancement du détecteur Snort.  

Console 2 : sudo tail -f /var/log/snort.log  
-> Lecture des logs 

Console 3 : sudo python3 -m http.server 80
-> Lancement d'un serveur http  

---

## 1️⃣ Attaque par Brute Force (authentification)

### Configuration de snort : 

alert tcp any any -> $HOME_NET 22 (msg: "DETECT - Possible SSH brute force (many connections)"; flow: to_server, established; detection_filter:track by_dst, count 10, seconds 60; sid:1000004; rev:1;)

## Commande de l'attaque 

for i in $(seq 1 12); do  
curl -s -X POST "http://192.168.10.1/login" -d "username=user${i}&password=badpass" -o /dev/null  
sleep 1  
done  

## Résultat 
### Nous pouvons voir ici que l'attaque s'est bien effectué
<img width="1503" height="23" alt="AttaqueBF" src="https://github.com/user-attachments/assets/b717a8b8-41bd-4c11-a508-568914a68888" />

### Le serveur web a bien reçu les nombreuses requêtes de l'attaquant
<img width="868" height="597" alt="HTTPServerBF" src="https://github.com/user-attachments/assets/48b692ea-4de0-4019-8d1e-3ede7c0e40bd" />

### Malheureusement Snort n'a pas envoyé d'alerte...
<img width="632" height="82" alt="RésultatBF" src="https://github.com/user-attachments/assets/c14f2bcd-fe5d-4354-a62c-d0c45edd9d7c" />

---

## 2️⃣ Balayage de Ports (Port Scanning)

### Configuration de snort : 

alert tcp any any -> any 1:65535 (msg:"DETECT - Possible TCP SYN port scan"; flags:S; detection_filter:track by_src, count 20, seconds 10; sid:1000002; rev:1;)

## Commande de l'attaque 
Aucune attaque

## Résultat 
Aucun résultat

---

## 3️⃣ Déni de Service Distribué (DDoS)

### Configuration de snort : 

alert tcp any any -> any any (msg: "DETECT - Potential SYN flood (DOOS)"; flags:S; detection_filter:track by_dst, count 200, seconds 1; sid:1000003; rev:1;)

## Commande de l'attaque 
Aucune attaque

## Résultat 
Aucun résultat

---

## 4️⃣ Injection SQL

### Configuration de snort : 

alert tcp $EXTERNAL_NET any -> $HOME_NET $HTTP_PORTS (msg: "SQL Injection test"; content: "select"; nocase; http uri; sid:1000005; rev:1;)

## Commande de l'attaque 

curl -G "http://192.168.10.1/search" --data-urlencode "q=select"

## Résultat 

### Nous pouvons voir ici que l'attaque s'est bien effectué et l'utilisateur a obtenu la page demandée
<img width="797" height="305" alt="AttaqueSQL" src="https://github.com/user-attachments/assets/1be25fc2-d2f6-40e4-a9d0-726ad4d3b3da" />

### Le serveur web a bien reçu la requête de l'attaquant et lui a répondu
<img width="781" height="152" alt="HTTPServerSQL" src="https://github.com/user-attachments/assets/6058394a-3771-44e5-9c76-12416de7a870" />

### On retrouve le log d'alerte envoyé par Snort dans snort.log
<img width="1202" height="116" alt="RésultatSQL" src="https://github.com/user-attachments/assets/2da765ab-7469-447a-baa3-a91ad2c46f83" />

---

## 5️⃣ Injection de Fichier via Formulaire d’Upload (fichier malveillant déguisé)


### Configuration de snort : 

alert tcp $EXTERNAL_NET any -> $HOME_NET $HTTP_PORTS (msg: "DETECT - Suspicious file upload with executable extension" flow: to_server, established; content:"filename="; nocase; http_client_body; pcre:"/filename=\s *= \s*\"?[^\"]+?\. ( ?: php|jsp|asp|aspx|exe|sh)\b\"?/Ui"; sid:1000006; rev:1;)

## Commande de l'attaque 

curl -s -X POST "http://192.168.10.1/upload" -F "file=@/dev/null:filename=profile.php" -o /dev/null

## Résultat 

### L'attaque est bien lancé
<img width="1153" height="53" alt="AttaqueUP" src="https://github.com/user-attachments/assets/9dc75a85-122a-4090-9e0b-66217ff801e2" />

### Le serveur HTTP ne reçoit pas la requête
<img width="645" height="80" alt="HTTPServerUP" src="https://github.com/user-attachments/assets/afe4e49e-56eb-45aa-854e-3149c127ca8e" />

### Il est donc logique que Snort ne puisse pas envoyer d'alerte
<img width="633" height="188" alt="RésultatUP" src="https://github.com/user-attachments/assets/a61631a1-c0f1-4848-a4ff-680c5e917bd0" />



---

📘 **Résumé de l'analyse :**

Nous n'avons seulement réussi qu'à réaliser l'analyse sur l'injection SQL, pour le reste nous n'avons pas pu obtenir les logs d'alertes. Nous pensons que c'est probablement dû à l'établissement des règles qui sont probablement fausse, où pas assez restreintes. 
