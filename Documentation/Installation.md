# Projet1_SI
projet 1 de sécurité informatique à l'UQAC

Projet SI
## Machine 
Hyperviseur : VirtualBox
OS : Ubuntu 24
RAM : 8192 Mo
Processeurs : 4

## Mise en place environnement :

$ sudo apt update && sudo apt upgrade 

========================================

## Mise en place Snort :

$ sudo apt install snort
-> Address range for the local network : 192.168.0.0/16

Vérif installation : 
$ snort -V
-> 29

Test mode IDS :
Console 1 : 
$ mkdir ~/snort-test
$ cd ~/snort-test
$ echo 'alert icmp any any -> any any (msg:"Ping detected"; sid:1000001; rev:1;)' > local.rules
$ sudo snort -A console -q -c ~/snort-test/local.rules -i lo

Console 2 :
$ ping localhost
[**] [1:1000001:1] Ping detected [**] ...

========================================

## Mise en place syslog-ng 

$ sudo apt install syslog-ng

Verif installation :
$ syslog-ng --version
$ sudo systemctl status syslog-ng

Test log : 
$ logger "Test NOW"
$ sudo tail -n 20 /var/log/syslog

========================================

## Intégration Snort -> syslog-ng

Dans fichier /etc/snort/snort.conf : 
Décommenter ligne output alert_syslog: LOG_AUTH LOG_ALERT
ajouter la ligne sous la ligne précédente 
	include rules/local.rules

$ sudo cp ~/test-snort/local.rules /etc/snort/rules/
$ sudo systemctl restart snort

Ajouts logs Snort dans fichier :

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

Les logs d'alertes doivent apparaître dans le fichier /var/log/snort.log

========================================

## Mise en place ElasticSearch

$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
$ sudo apt-get install apt-transport-https
$ echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-9.x.list
$ sudo apt-get update && sudo apt-get install elasticsearch

========================================

## Mise en place Kibana

sudo apt install kibana
sudo systemctl enable kibana
sudo systemctl start kibana

Verif : 
Firefox -> http://localhost:5601

========================================

## Configuration Elactisearch + Kibana
Sur http://localhost:5601 :
Mettre le token donné par sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrllment-token --scope kibana

Si code demandé : 
Mettre le code donné par sudo /usr/share/kibana/bin/kibana-verification-code

user par défaut : elastic
mdp par défaut : 
Sur un terminal faire /usr/share/elasticsearch/bin/elasticsearch-reset-password
Récupérer mot de passe 
