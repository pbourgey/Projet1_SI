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
Aucun résultat

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


---

## 5️⃣ Injection de Fichier via Formulaire d’Upload (fichier malveillant déguisé)


### Configuration de snort : 

alert tcp $EXTERNAL_NET any -> $HOME_NET $HTTP_PORTS (msg: "DETECT - Suspicious file upload with executable extension" flow: to_server, established; content:"filename="; nocase; http_client_body; pcre:"/filename=\s *= \s*\"?[^\"]+?\. ( ?: php|jsp|asp|aspx|exe|sh)\b\"?/Ui"; sid:1000006; rev:1;)

## Commande de l'attaque 

curl -s -X POST "http://192.168.10.1/upload" -F "file=@/dev/null:filename=profile.php" -o /dev/null

## Résultat 
Aucun résultat

---

📘 **Résumé de l'analyse :**

Nous n'avons seulement réussi qu'à réaliser l'analyse sur l'injection SQL, pour le reste nous n'avons pas pu obtenir les logs d'alertes. Nous pensons que c'est probablement dû à l'établissement des règles qui sont probablement fausse, où pas assez restreintes. 
