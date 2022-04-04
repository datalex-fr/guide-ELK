![](https://files.gitbook.com/v0/b/gitbook-legacy-files/o/assets%2F-MhdSzY_7K9n5aQAlJq8%2F-MhjO8LUqTyOFXapnFW8%2F-MhjbPCHLlWnm6PkgDr0%2Felk-architecture.jpg?alt=media&token=3829f366-7fca-4d39-9f57-b62c0f40770a)

## 

I Introduction.

Cette stack permet l’indexation et l’analyse de presque tout type de source de données structurée et non structurée, comme des logs et les visualiser ensuite sous forme de tableaux et de graphiques. Chaque couche peut être installé en instance client-serveur. Optimisé pour la sécurité informatique des infrastructures, il peut être aussi utilisé en data science avec le client Python "Eland" sur elasticsearch et ainsi pouvoir lancer des opérations de type Numpy, Pandas, Scikit-learn etc. Mais il est aussi possible de faire du machine learning avec le X-pack intégré à la suite Elastic. De plus, Metricbeat et Packetbeat seront utiles pour la surveillance réseaux et systèmes.

![](https://files.gitbook.com/v0/b/gitbook-legacy-files/o/assets%2F-MhdSzY_7K9n5aQAlJq8%2F-MhjO8LUqTyOFXapnFW8%2F-Mhjqh7iMQFwch5zaTe1%2Fblog-machine-learning-5-4-release.jpg?alt=media&token=97e20b58-debb-4570-a301-1dcea19ec8c9)

## 

II Mise en place et étude de solutions Big Data pour traiter les logs d’activités.

-   **Logstash** : C’est un ETL (Extract Transform Load) destiné principalement au traitement des logs et compte plus de 160 plugins. il sert à rassembler et normaliser tous les types de données provenant de différentes sources et de les rendre disponibles pour une utilisation ultérieure. Cette couche Java est très gourmande en ressource, Beat et Filebeat peut être une alternative.
    
-   **Elasticsearch** : C’est moteur de recherche libre open source en charge de l'indexation et du stockage de vos informations sur une base de données NoSQL et il est construit pour fournir des APIS rest. Il propose également des requêtes avancées pour effectuer une analyse détaillée et stocke toutes les données de manière centralisée.
    
-   **Kibana** : C’est une interface web offrant aux utilisateurs la possibilité d'analyser et de visualiser les données récupérées Elasticsearch. C'est un outil puissant offrant pour vos tableaux de bord divers diagrammes interactifs, données géographiques et graphiques pour visualiser les données les plus complexes
    

## 

III Développement de traitements pour exploiter les logs d’activités.

L'analyse des logs est un processus complexe qui doit suivre les bonnes pratiques suivantes :

-   **La détection et la reconnaissance des patterns** : cela fait référence au filtrage des messages entrants sur la base d'un pattern souvent combiné avec l'utilisation des expressions régulières. Elle fait partie intégrante de l'analyse des logs car elle permet de détecter plus rapidement les anomalies.
    
-   **La normalisation des logs** : ce sont le processus de conversion des éléments de logs tels que les adresses IP ou les horodatages, dans un format commun pour toutes vos équipes.
    
-   **La classification et le balisage** : ce sont le processus de balisage (tag) des messages avec des mots-clés et de leur catégorisation en classes. Cela vous permet de filtrer et de personnaliser la façon dont vous visualisez les données.
    
-   **L'analyse de corrélation** : fait référence à la collecte de données provenant de différentes sources et à la recherche de messages appartenant à un événement spécifique. Par exemple, en cas d'activité malveillante, il vous permet de filtrer et de corréler les logs provenant de vos périphériques réseaux, pare-feu, serveurs et autres sources afin de très rapidement détecter la source du problème.
    
-   **Alerte** : L'analyse de corrélation est généralement associée aux systèmes d'alerte, en fonction du pattern que vous avez identifié, vous pouvez créer des alertes lorsque votre analyseur de logs détecte une activité anormale.
    

![](https://files.gitbook.com/v0/b/gitbook-legacy-files/o/assets%2F-MhdSzY_7K9n5aQAlJq8%2F-MhoeGmccg6iOY6xLDiY%2F-MhoiKwFeMn7cgewMoTm%2FDfqTyH7WkAUDW5F.gif?alt=media&token=a6669dbb-50d9-4211-b86e-45c698e75fd3)

ETL, les 3 pipelines de Logstash :

Les points de départ de toute configuration sont vos entrées. Les entrées sont des

plugins

Logstash responsables de la récupération des données de différentes sources de données.

Les filtres sont des modules qui vont traiter les données récupérées depuis l'input en s'aidant lui aussi de

différents plugins de filtrage

. Pour notre exemple, nous utiliserons le plugin

grok

qui permet d'analyser facilement des données de logs non structurées en quelque chose de plus exploitable en utilisant des expressions régulières. Cet outil est parfait pour tout format de logs généralement écrit pour les humains comme les logs applicatifs (ex: mysql, apache, nginx, etc ...).

Nous allons découvrir et utiliser des patterns APACHE grok pré-configurés. Vous pouvez utiliser soit le pattern COMMONAPACHELOG qui récupère des informations de base (IP source, code HTTP, etc ...) ou le pattern COMBINEDAPACHELOG qui récupère des informations supplémentaires comme le user-agent. J'utilise également le filtre mutate avec l'action convert afin de convertir la réponse HTTP et la taille de la requête qui sont par défaut en type string vers le type entier, car en effet, Kibana gère ses visualisations différemment selon le type de données que vous lui envoyez.

Les points d'arrivée de toute configuration Logstash sont vos sorties, lui-même utilise

différents plugins de sortie

qui envoient les données traitées par la phase de filtrage à une destination particulière (ex: elasticsearch). Par ailleurs, les sorties sont la dernière étape du pipeline Logstash.

IV Procédure d'installation Elasticsearch.

wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

sudo apt-get install apt-transport-https

echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

sudo apt-get update -y && sudo apt-get install elasticsearch

Par défaut à son lancement Elasticsearch consomme 1go de mémoire de la JVM (machine virtuelle java), si votre machine n'est pas assez puissante vous pouvez modifier les valeurs Xms et Xmx situé dans le fichier /etc/elasticsearch/jvm.options pour une consommation réduite.

\# Avant (1go)

\-Xms1g

\-Xmx1g

\# Après (512 mo)

\-Xms512mo

\-Xmx512mo

Les configurations Elasticsearch sont effectuées à l'aide du fichier de configuration /etc/elasticsearch/elasticsearch.yml qui vous permet de configurer les paramètres généraux comme par exemple le nom du nœud, ainsi que les paramètres réseau comme par exemple l'hôte et le port, l'emplacement des données stockées, la mémoire, les fichiers de logs, etc. Nous laisserons par défaut.

Pour exécuter Elasticsearch :

sudo systemctl start elasticsearch

Pour lire les logs d'installation :

sudo journalctl -f -u elasticsearch

{

"name" : "debian-inux",

"cluster\_name" : "elasticsearch",

"cluster\_uuid" : "U1zH\_yFqTUmdMRckR1gHLQ",

"version" : {

"number" : "7.8.0",

"build\_flavor" : "default",

"build\_type" : "deb",

"build\_hash" : "757314695644ea9a1dc2fecd26d1a43856725e65",

"build\_date" : "2020-06-14T19:35:50.234439Z",

"build\_snapshot" : false,

"lucene\_version" : "8.5.1",

"minimum\_wire\_compatibility\_version" : "6.8.0",

"minimum\_index\_compatibility\_version" : "6.0.0-beta1"

},

"tagline" : "You Know, for Search"

}

Pour initialiser le service à chaque démarrage de la machine, lancez la commande suivante :

sudo systemctl enable elasticsearch

V Procédure d'installation Kibana.

sudo apt-get install kibana

Le fichier de configuration de kibana se retrouve dans /etc/kibana/kibana.yml

elasticsearch.hosts: \["http://localhost:9200"\]

sudo systemctl start kibana

sudo journalctl -f -u kibana

Pour tester Kibana

http://localhost:5601

Pour initialiser le service Kibana à chaque démarrage de la machine, lancez la commande suivante :

sudo systemctl enable kibana

VI Procédure d'installation Logstash.

Logstash nécessite au minimum la version 8 de java pour fonctionner

sudo apt-get install default-jre

java -version

openjdk version "11.0.7" 2020-04-14

...

sudo apt-get install logstash

Le fichier de configuration de Logstash est le suivant : /etc/logstash/logstash.yml et permet de configurer des paramètres généraux comme par exemple le nom du nœud, le port, le niveau des logs etc. Nous laisserons la configuration par défaut.

sudo systemctl start logstash

sudo journalctl -f -u logstash

sudo systemctl enable logstash

Procédure d'installation d'Apache.

Afin d'analyser en temps réel les logs d’accès d'un site web sur notre machine ELK

sudo apt-get install -y apache2

sudo systemctl start apache2

sudo usermod -aG adm logstash (ajout de l'utilisateur logstash au groupe adm)

#Le groupe d'utilisateurs adm est utilisé

#pour permettre à ses membres de lire le contenu des fichiers de logs

Binding logs Apache => Logstash

Traitement des logs sous forme d'un ou plusieurs pipelines :

Il est capable d'extraire des données de presque n'importe quelle source de données à l'aide des plugins d'entrée, et d'appliquer une grande variété de transformations et d'améliorations de données à l'aide de plugins de filtre et enfin d'expédier ces données vers un grand nombre de destinations à l'aide de plugins de sortie. Logstash joue un rôle très important dans la pile en récupérant, filtrant et expédiant vos données afin de les utiliser plus facilement sur le reste des composants.

Pour utiliser Logstash, il faut créer des fichiers de configuration qui contiendront trois sections principales que nous allons par passer en revue ci-dessous. À savoir que chacune de ces sections est responsable de différentes fonctions et utilisant différents plugins Logstash.

Création du fichier apache.conf dans le dossier de configuration Logstash situé dans /etc/logstash/conf.d/

input {

file { path => "/var/log/apache2/access.log" }

}

filter {

grok {

match => { "message" => "%{COMBINEDAPACHELOG}" }

}

date {

match => \[ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" \]

}

mutate {

convert => {

"response" => "integer"

"bytes" => "integer"

}

}

}

output {

elasticsearch {

hosts => "localhost:9200"

index => "apache-%{+YYYY.MM.dd}"

}

}

sudo systemctl start logstash

curl "localhost:9200/\_cat/indices?v"

yellow open apache-2020.06.30
