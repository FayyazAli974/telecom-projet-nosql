# Telecom - Projet NoSQL - Données GDELT

**Groupe**: Fayyaz Ali, Philippe Bénézeth, Thomas Koch, Xavier Bracquart

## Présentation

L’objectif du projet est de mettre en place une architecture de système de stockage distribué, résilient et performant sur AWS, pour requêter des données venant du jeu de données  [The Global Database of Events, Language, and Tone (GDELT)](https://www.gdeltproject.org/).

Le diaporama présenté lors du rendu se trouve dans le dossier [Présentation](https://github.com/xavierbrt/telecom-projet-nosql/tree/master/Presentation).


## Installation de l'architecture

L'architecture utilise des instances EC2 sur lesquelles sont installées Spark, Cassandra et Zookeeper. Les données sont stockées sur S3.

Les instructions pour déployer l'infrastructure sur AWS est disponible dans le dossier [Documentation d'installation cluster EC2](https://github.com/xavierbrt/telecom-projet-nosql/tree/master/Documentation%20d'installation%20cluster%20EC2).

Les fichiers de configuration des différents composants sont dans les dossiers [Conf Spark](https://github.com/xavierbrt/telecom-projet-nosql/tree/master/Conf%20Spark), [Conf Cassandra](https://github.com/xavierbrt/telecom-projet-nosql/tree/master/Conf%20Cassandra) et [Conf Zookeeper](https://github.com/xavierbrt/telecom-projet-nosql/tree/master/Conf%20Zookeeper).


## Manipulation des données

Le code pour manipuler les données est écrit sur 3 [notebooks Zeppelin](https://github.com/xavierbrt/telecom-projet-nosql/tree/master/Notebooks): 
* **feed-s3:** Charge les fichiers provenant de la base de données GDELT dans un bucket S3, sur une période donnée ;
* **feed-cassandra:** Charge les fichiers depuis S3 dans des dataframes. Nous retraitons les dataframes en fonction des besoins des requêtes. Nous écrivons ensuite ces dataframes dans les tables Cassandra ;
* **execute-requests:** Charge les données depuis les tables Cassandra, puis exécute les requêtes sur les données en Spark SQL.

Les notebooks sont en .json. Nous les avons exportés en html également, pour une meilleure visualisation.

Les requêtes ont été préparées dans un premier temps en Pyspark, dans le dossier [Preparation des requetes](https://github.com/xavierbrt/telecom-projet-nosql/tree/master/Preparation%20des%20requetes).