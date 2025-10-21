# GUIDE D'INSTALLATION DES DIFFERENTS OUTILS

---

## Caractéristique VM

Hyperviseur : VirtualBox  
OS : Ubuntu 24  
RAM : 8192 Mo  
Processeurs : 4  

---

## Mise en place environnement :
$ sudo apt update && sudo apt upgrade 

---

## Déploiement de Snort :

1) Installation du paquet snort :
$ sudo apt install snort  
  
3) Une fenêtre apparaît, à la question : Address range for the local network, mettre la plage d'adresse suivante : 192.168.0.0/16  
  
4) Vérification de l'installation :   
$ snort -V  
-> On doit trouver la version : 2.9.x  
  
5) Test du mode IDS :  
Console 1 :   
$ mkdir ~/snort-test  
$ cd ~/snort-test  
$ echo 'alert icmp any any -> any any (msg:"Ping detected"; sid:1000001; rev:1;)' > local.rules  
$ sudo snort -A console -q -c ~/snort-test/local.rules -i lo  
  
Console 2 :  
$ ping localhost  
  
Nous devons voir des logs sous la forme suivante dans Console 1 :   
MM/DD-HH:MM:SS [\*\*] [1:1000001:1] Ping detected [\*\*] ...  

---

## Déploiement de syslog-ng 

1) Installation du paquet syslog-ng  
$ sudo apt install syslog-ng  
  
2) Vérification de l'installation :  
$ syslog-ng --version  
-> Version 4.3.1  

$ sudo systemctl enable syslog-ng  
$ sudo systemctl start syslog-ng  
$ sudo systemctl status syslog-ng  
-> Active : active (running)  

3) Test d'un log :   
$ logger "Test NOW"  
$ sudo tail -n 20 /var/log/syslog
-> Log "test" doit apparaître
    
---

## Intégration Snort -> syslog-ng

1) Dans le fichier /etc/snort/snort.conf :   
	Il faut décomment la ligne :
		output alert_syslog: LOG_AUTH LOG_ALERT  

	Puis ajouter la ligne en dessous de la ligne précédente     
		include rules/local.rules

2) Déplacement du fichier local.rules (voir partie Test du mode IDS dans déploiement de Snort)  
$ sudo cp ~/test-snort/local.rules /etc/snort/rules/  
$ sudo systemctl restart snort  

3) Configuration pour l'ajout des logs Snort dans le fichier /var/log/snort.log :  
Dans /etc/syslog-ng/syslog-ng.conf ajouter :  
	filter f_snort { program("snort"); };  
	destination d_snort { file("/var/log/snort.log"); };  
	log { source(s_src); filter(f_snort); destination(d_snort); };  

$ sudo systemctl restart syslog-ng  

Test Verif :  
Console 1 :  
$ sudo snort -q -c /etc/snort/snort.conf -i lo  

Console 2 :   
$ sudo tail -f /var/log/snort.log  

Console 3 :   
$ ping localhost  
  
Les logs d'alertes doivent apparaître dans le fichier /var/log/snort.log.

---

## Déploiement d'ElasticSearch

1) Installation d'ElasticSearch  
$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg  
$ sudo apt-get install apt-transport-https  
$ echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-9.x.list  
$ sudo apt-get update && sudo apt-get install elasticsearch  

---

## Déploiement de Kibana

1) Installation de Kibana
$ sudo apt install kibana    
$ sudo systemctl enable kibana  
$ sudo systemctl start kibana  

2) Verification de l'installation :
   Ouvrir Firefox  
   Dans la barre de recherche : http://localhost:5601 
   La page d'accueil de Kibana (ElasticSearch) apparaît

---

## Configuration Elactisearch + Kibana

$ sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrllment-token --scope kibana  
-> Un token apparaît
   Mettre ce token dans la page d'accueil de Kibana (http://localhost:5601)

Si un code est demandé :  
$ sudo /usr/share/kibana/bin/kibana-verification-code  
Mettre le code qui est donné par cette commande  
  
user par défaut : elastic  
mdp par défaut : Le mot de passe donné par la commande ci-dessous :   
-> $ sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password

---

## Intégration Syslog-ng -> ElasticSearch (non fonctionnel) 

Dans le fichier /etc/syslog-ng/syslog-ng.conf :  
Ajouter les lignes suivantes :   

destination d_elasticsearch_http {  
    elasticsearch-http(  
        index("syslog-ng")  
        type("")  
        url("http://localhost:9200/_bulk")  
        template("$(format-json --scope rfc5424 --scope dot-nv-pairs  
        \-\-rekey .* --shift 1 --scope nv-pairs  
        \-\-exclude DATE --key ISODATE @timestamp=${ISODATE})")  
    );  
};  
  
Modifier log { source(s_src); filter(f_snort); destination(d_snort); }; -> log { source(s_src); filter(f_snort); destination(d_snort); destination(d_elasticsearch_http) };    
