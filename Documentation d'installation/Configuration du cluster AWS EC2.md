------------------------------------------
# - Configuration du cluster AWS EC2 -
------------------------------------------

La configuration de notre architecture nécessite de configurer un cluster EC2 sur AWS. Pour notre cas, ce cluster comportera en tout 8 serveurs :
* **2 Masters** avec Spark 2.3.2 ;
* **5 Slaves (ou Workers)** avec Spark 2.3.2 et Cassandra 3.11.3 qui serviront à stocker nos données. **2 d'entres eux** auront également Zookeeper installé pour assurer la résilience de nos Workers ;
* **1 Zookeeper** pour assurer la résilience de nos Masters ;


La résilience des Workers est automatiquement gérées par le master Spark. La résilience des Masters est assurées par Zookeeper.


La documentation qui suit permet de configurer ces instances, étapes par étapes. Dans les grandes lignes il faudra :
1. Lancer un cluster EC2 sur AWS et y accéder ;
2. Configurer préalablement les 8 serveurs avec Java et les Workers avec Python  ;
3. Installer Apache Cassandra sur les 5 Workers du cluster ;
4. Installer Zookeeper sur 2 Workers et 1 serveur dédié ;
5. Installer Spark sur les Masters 1 et 2 et sur les 5 Workers ;
6. Installer Zeppelin sur les Masters ;

--------------------------------------------
## 1. Lancer un cluster EC2 sur AWS et y accéder
A FAIRE

---------------------------------------------
## 2. Configurer préalablement les 8 serveurs avec Java et les Workers avec Python


### 2.1 Connexion SSH sur tous les nœuds

La console AWS EC2 fournit directement pour chaque serveur un lien permettant de se connecter en SSH. Pour cela, il faut commencer par se placer dans le répertoire où se trouve notre clé ***"gdeltKeyPair.pem"*** puis lancer la commande indiquée. Cela donne donc :

```bash
cd PATH_TO_YOUR_KEYPAIR
ssh -i "gdeltKeyPair.pem" ubuntu@<copy the public DNS> 
```
Cette action est donc à faire pour chaque serveurs.

### 2.2 Installation de Java SDK 8

Une fois les connexions SSH établies, nous pouvons installer Java SDK. Sur chaque serveur, on lance donc :

```bash
sudo apt-get install openjdk-8-jre
```

### 2.3 Installation de Python sur les Workers
Pour faire fonctionner `cqlsh` d'Apache-Cassandra sur les Workers, un interpréteur python est requis. Il faut donc éxécuter sur les 5 Workers :

```bash
sudo apt-get install python
```
Une fois cela fait, nous pouvons passer à l'installation d'Apache Cassandra sur les 5 Workers.

--------------------------------------------
## 3. Installer Apache Cassandra sur les 5 Workers du cluster

*Les actions qui suivent sont à effectuer sur chaque Workers du cluster.* 

### 3.1 Connexion SSH sur tous les nœuds

La console AWS EC2 fournit directement pour chaque serveur un lien permettant de se connecter en SSH. Pour cela, il faut commencer par se placer dans le répertoire où se trouve notre clé ***"gdeltKeyPair.pem"*** puis lancer la commande indiquée. Cela donne donc :

```bash
cd PATH_TO_YOUR_KEYPAIR
ssh -i "gdeltKeyPair.pem" ubuntu@<copy the public DNS> 
```
Cette action est donc à faire pour chaque serveurs.

### 3.2 Téléchargement du package
* Depuis le [site Apache Cassandra](http://www.apache.org/dyn/closer.lua/cassandra/3.11.3/apache-cassandra-3.11.3-bin.tar.gz) copier le lien qui ressemble à celui-ci en haut de la page :
[http://archive.apache.org/dist/cassandra/3.11.3/apache-cassandra-3.11.3-bin.tar.gz](http://archive.apache.org/dist/cassandra/3.11.3/apache-cassandra-3.11.3-bin.tar.gz)

* Dans le répertoire `/home` de chaque instance, lancer les commandes suivantes : 
```bash
cd /home/ubuntu/
wget http://archive.apache.org/dist/cassandra/3.11.3/apache-cassandra-3.11.3-bin.tar.gz
```

* Décompressez ensuite le fichier :
```bash
tar -xzvf apache-cassandra-3.11.3-bin.tar.gz
```

* Et retirez le **.tar.gz** précédemment téléchargé :
```bash
rm apache-cassandra-3.11.3-bin.tar.gz
```

### 3.3 Modification du fichier `cassandra.yaml` 

* Le fichier se trouve dans le répertoire `conf` :
```bash
cd apache-cassandra-3.11.3/conf/
```

* On commence par modifier le fichier `cassandra.yaml` (voir fichier de conf en exemple)
```bash
vi cassandra.yaml
```
> Les commandes utiles avec vim sont : `i` pour commencer une insertion, `echap` pour sortir du mode choisi, `:w` pour écrire les modifications, `:x` pour quitter et enregistrer.

Dans le fichier, il faut modifier les lignes suivantes :
* `cluster_name` : nom de notre cluster
* `seed_provider` : adresses IP privées des nœuds Cassandra 
* `listen_address` : adresse IP privée du nœud sur lequel on se trouve
* `rpc_address` : adresse IP privée du nœud sur lequel on se trouve
* `endpoint_snitch` : mettre `Ec2Snitch`


Par exemple, cela donne : 
```yaml
cluster_name: 'Test Cluster'

seed_provider:
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
          # seeds is actually a comma-delimited list of addresses.
          # Ex: "<ip1>,<ip2>,<ip3>"
          - seeds: "172.31.92.67,172.32.91.100,172.31.80.158,172.31.87.77,172.31.94.200"

listen_address: 172.31.80.158

rpc_address: 172.31.80.158

endpoint_snitch: Ec2Snitch
```

Une astuce intéressante pour gagner du temps dans la modification des fichiers de configuration est de préparer les fichiers sur notre machine locale dans notre éditeur de texte préféré, pour ensuite copier ces fichier avec `scp` sur les différentes machines. La commande générique permettant de faire cela est la suivante (en se plaçant au préalable dans le dossier où se trouve notre clé `.pem`) :
```bash
scp -i "Key_file_Name.pem" Chemin/du/fichier/a/copier/fichier.ext ubuntu@ecx-xx-xxx-xxx-xxx.compute-1.amazonaws.com:Chemin/du/repertoire/de/destination/
```

Il est de même possible de récupérer un fichier de configuration sur notre machine locale avec le même type de commande :
```bash
scp -i "Key_File_Name.pem" ubuntu@ecx-xx-xxx-xxx-xxx.compute-1.amazonaws.com:Chemin/du/fichier/a/recuperer/fichier.ext Chemin/local/de/destination/
```

### 3.4 Modification du fichier `cassandra-rackdc.properties`

Le fichier se trouve également dans le répertoire `conf`. 
```bash
vi cassandra-rackdc.properties
```

On considère ici la structure la plus simple où ne spécifie aucun nom de rack ou de datacenter. Il suffit donc de **commenter les deux lignes correspondantes**, non commentées initialement.


### 3.5 Test de la connexion entre les différents nœuds

* On se rend dans le répertoire `bin` :
```bash
cd ./bin
```

* On exécute sur chaque nœud la commande suivante :
```bash
./cassandra
```

Certaines lignes contiennent le mot ***"Handshaking"*** qui signifie que les nœuds communiquent. On peut également utiliser les commandes suivantes pour avoir le détail de notre cluster :
```bash
./nodetool status

./nodetool describecluster
```

Nous pouvons maintenant passer à l'installation et la configuration de Zookeeper pour la résilience de Spark.



--------------------------------------------
## 4. Installer Zookeeper sur 2 Workers et 1 serveur dédié 

Zookeeper (ZK) peut être installé indépendamment sur un nœud ou directement avec Spark / Cassandra sur des Workers. Chaque nœud ZK doit communiquer avec les autres instances ZK pour former un quorum de 3. Nous avons choisi d'installer Zookeeper avant Spark car la configuration est plus simple ainsi.

### 4.1 Connexion SSH sur tous les nœuds

La console AWS EC2 fournit directement pour chaque serveur un lien permettant de se connecter en SSH. Pour cela, il faut commencer par se placer dans le répertoire où se trouve notre clé ***"gdeltKeyPair.pem"*** puis lancer la commande indiquée. Cela donne donc :

```bash
cd PATH_TO_YOUR_KEYPAIR
ssh -i "gdeltKeyPair.pem" ubuntu@<copy the public DNS> 
```
Cette action est donc à faire pour chaque serveurs.


### 4.2 Installer Apache-Zookeeper sur nos instances

* On commence par se placer dans le répertoire `home` :
```bash
cd /home/ubuntu/

```

* On télécharge le .tar.gz, on l'extrait et on le supprime :
```bash
wget https://www-eu.apache.org/dist/zookeeper/zookeeper-3.5.6/apache-zookeeper-3.5.6-bin.tar.gz && tar -xzvf apache-zookeeper-3.5.6-bin.tar.gz && rm apache-zookeeper-3.5.6-bin.tar.gz
```

* On renomme le fichier décompressé avec une indentation *X* différente pour chaque nœud (1 pour Worker1, 2 pour Worker2 et 3 pour le serveur ZK dédié) : 
```bash
mv apache-zookeeper-3.5.6-bin/ zookeeper_X   
```

On peut alors passer à la configuration des nœuds.


### 4.3 Configurer les nœuds

* On créé les dossiers `data` et `logs` :
```bash
cd zookeeper_X/

mkdir data
mkdir logs
```

* On créé un nouveau fichier `myid` qui contiendra uniquement un digit entre 1 et 3 :
```bash
cd data && touch myid
vi myid
```
En fonction du nœud sur lequel on se trouve, ajouter le digit dans le fichier .txt : 
* 1: si vous travaillez sur le Worker 1
* 2: si vous travaillez sur le Worker 2
* 3: si vous travaillez sur le nœud "Zookeeper"


* On créé et modifie le fichier `zoo.cfg` à l'aide du fichier `zoo_sample.cfg` :
```bash
cd ..
cd conf
cp zoo_sample.cfg zoo.cfg
vi zoo.cfg
```
Voir le fichier `zoo.cfg` pour un exemple de configuration. En résumé cela demande :
* d'ajouter la ligne : `datadir=<Path to the data dir>` par exemple `/home/ubuntu/zookeeper_1/data`
* d'ajouter la ligne : `clientPort=218X` en remplaçant X par le digit défini plus haut
* d'ajouter les lignes : `server.1=<PRIVATE.DNS.1>:2891:3881` ; `server.2=<PRIVATE.DNS.2>:2892:3882` ; `server.3=<PRIVATE.DNS.3>:2893:3883`

* On compile le fichier de configuration. Dans le répertoire `zookeeper_X`, exécuter cette ligne : 
```bash
java -cp lib/zookeeper-3.5.6.jar:lib/log4j-1.2.17.jar:lib/slf4j-log4j12-1.7.25.jar:lib/slf4j-api-1.7.25.jar:conf org.apache.zookeeper.server.quorum.QuorumPeerMain ./conf/zoo.cfg
```
Et également :
```bash
java -cp lib/zookeeper-3.5.6.jar:lib/log4j-1.2.17.jar:lib/slf4j-log4j12-1.7.25.jar:lib/slf4j-api-1.7.25.jar:conf org.apache.zookeeper.server.quorum.QuorumPeerMain ./conf/zoo.cfg >> logs/zookeeper.log &
```

### 4.4 Lancer le service sur chaque nœuds

On peut alors lancer le service sur chaque nœud avec :
```bash
cd /home/ubuntu/zookeeper_X/bin
./zkServer.sh start
```
Et pour arrêter :
```bash
./zkServer.sh stop
```


--------------------------------------------
## 5. Installer Spark sur les Masters 1 et 2 et sur les 5 Workers

Cette partie vise à détailler l'installation d'Apache-Spark sur notre cluster EC2 AWS et de le faire fonctionner avec nos instances Apache-Cassandra en totale résilience. Nous prendrons une configuration standard qui permet d'élire un Master qui répartira ensuite ses jobs sur les Workers. L'élection du master primaire est gérée par Zookeeper.

Cette partie se divise en 5 sections :
* Installer Apache-SparK sur les Masters et Workers
* Configurer le fichier spark-env.sh
* Configurer le fichier spark-default.conf
* Ajouter les dépendances pour connecter Spark et Cassandra
* Lancer les Masters et les Workers


### 5.1 Installer Apache-SparK sur les Masters et Workers

Il suffit de passer la commande suivante pour récupérer, extraire et supprimer l'archive sur chaque nœuds :
```bash
wget https://archive.apache.org/dist/spark/spark-2.3.2/spark-2.3.2-bin-hadoop2.7.tgz && tar -xzvf spark-2.3.2-bin-hadoop2.7.tgz && rm spark-2.3.2-bin-hadoop2.7.tgz
```
Puis de configurer les variables d'environnement pour tous les nœuds :
```bash
vi ~/.bashrc
```
Dans le fichier, on rajoute à la fin les lignes suivantes :
```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=$JAVA_HOME/jre
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
export SPARK_HOME=/home/ubuntu/spark-2.3.2-bin-hadoop2.7
```

On termine en sourçant le fichier pour que le système prenne en compte les modifications apportées :
```bash
source ~/.bashrc
```

On peut vérifier la bonne prise en compte des modifications avec par exemple :
```bash
echo $SPARK_HOME
```

### 5.2 Configurer le fichier spark-env.sh

#### Sur les Masters

On commence par se rendre dans le dossier `conf` de spark et par copier les fichiers `spark-env.sh.template` et `spark-defaults.conf.template` (en prévision de l'étape suivante) dans de nouveaux fichiers :
```bash
cd /home/ubunut/spark-2.3.2-bin-hadoop2.7/conf && cp spark-env.sh.template spark-env.sh && cp spark-defaults.conf.template spark-defaults.conf
```
On ouvre ensuite le fichier `spark-env.sh` que l'on vient de créer pour y renseigner les informations suivantes :

```bash
export SPARK_LOCAL_IP=ip-172-31-89-119.ec2.internal
export SPARK_MASTER_HOST=ip-172-31-89-119.ec2.internal
export SPARK_MASTER_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=ip-172-31-92-67.ec2.internal:2181,ip-172-31-91-100.ec2.internal:2182,ip-172-31-82-144.ec2.internal:2183"
```

#### Sur les Workers

``` bash



vi spark-defaults.conf (voir fichier pour modifs)
vi spark-env.sh (voir fichier pour modifs)

cd spark-2.3.2-bin-hadoop2.7/jars
wget https://repo1.maven.org/maven2/com/twitter/jsr166e/1.1.0/jsr166e-1.1.0.jar
wget https://repo1.maven.org/maven2/com/datastax/spark/spark-cassandra-connector_2.11/2.4.0/spark-cassandra-connector_2.11-2.4.0.jar
```

Pour vérifier : IPpublic:8080 (ex : http://3.93.186.250:8080/)



### 5.3 Configurer le fichier spark-default.conf

Le fichier est cette fois quasiment identique pour les Masters et les Workers.


### 5.4 Ajouter les dépendances pour connecter Spark et Cassandra

### 5.5 Lancer les Masters et les Workers

---------------------------------------------
## 6. Installer Zeppelin sur les Masters 
Zeppelin servira à vernir requêter en Spark SQL notre base de données Cassandra. C'est pour cette raison que nous l'installons sur nos deux Masters, il conviendra donc de répeter les opérations suivantes pour les deux serveurs.

### 6.1 Installation de Zeppelin
On télécharge l'archive, la décompresse et la supprime en étant dans le `/home/ubuntu` :
```bash
wget https://www-eu.apache.org/dist/zeppelin/zeppelin-0.8.2/zeppelin-0.8.2-bin-all.tgz && tar -xzvf zeppelin-0.8.2-bin-all.tgz && rm zeppelin-0.8.2-bin-all.tgz 
```

L'installation est ainsi réalisée, reste à la paramétrer (si besoin).

### 6.2 Configuration

Dans notre cas Zeppelin fonctionne sur l'un ou l'autre des masters (selon disponibilité), sur lesquels tournent déjà Spark. Comme l'UI Spark se récupère via le port 8080 du localhost et qu'il en est de même pour Zeppelin (même s'il se lance sur une adresse de serveur différente), nous avons préféré choisir le port 8090 pour accéder à l'interface de Zeppelin. Aussi, il convient de créer et modifier les fichiers de configuration correspondants.

* Création des fichiers :
```bash
cd zeppelin-0.8.2-bin-all/conf
cp zeppelin-env.sh.template zeppelin-env.sh && cp zeppelin-site.xml.template zeppelin-site.xml
```

* Modification des fichiers :
```bash
vi zeppelin-env.sh
```
Il faut alors décommenter et renseigner la ligne : 
```bash
export ZEPPELIN_PORT=8090
```
On enregistre et on quitte.
```bash
vi zeppelin-site.xml
```
Il faut alors modifier le passage suivant :
```bash
<property>
    <name>zeppelin.server.port</name>
    <value>8090</value>
    <description>Server port.</description>
</property>
```
D'autres changement dans les configurations peuvent être nécessaires.

La totalité de notre instance EC2 étant maintenant paramétrée, nous pouvons lancer les différents services au démarrage du cluster en se connectant sur chaques nœuds et en respectant un certain ordre, détaillé dans la partie qui suit.

--------------------------------------------
## 7. Commandes et instructions pour le lancement des services de notre cluster EC2

Après avoir démarré les serveurs via la console de l'EC2 sur AWS, l'ordre de démarrage des services est le suivant :

### 7.1 Zookeeper

Lancer sur chaque nœud :
```bash
cd /home/ubuntu/zookeeper_X/bin && ./zkServer.sh start
```

### 7.2 Spark sur Masters

Lancer sur chaque Master :
```bash
cd /home/ubuntu/spark-2.3.2-bin-hadoop2.7/sbin && ./start-master.sh
```

On vérifie le bon fonctionnement via : `IPpublic:8080` (ex : http://3.93.186.250:8080/)


### 7.3 Spark sur Workers

Lancer sur chaque Worker :

```bash
cd /home/ubuntu/spark-2.3.2-bin-hadoop2.7/sbin && ./start-slave.sh spark://ip-172-31-89-119.ec2.internal:7077,ip-172-31-82-26.ec2.internal:7077
```
La commande générique est donc :
```bash
cd /home/ubuntu/spark-2.3.2-bin-hadoop2.7/sbin && ./start-slave.sh spark://Private_DNS_Master1:7077,Private_DNS_Master2:7077
```

On vérifie le bon fonctionnement via : `IPpublic:8081` (ex : http://3.93.186.250:8081/)


Test résilience : éteindre master1, regarder spark UI master 2 => les workers doivent y apparaître
on peut vérifier l'élection via :
cat /home/ubuntu/spark-2.3.2-bin-hadoop2.7/logs/spark-ubuntu-org.apache.spark.deploy.master.Master-1-ip-172-31-82-26.out

2020-01-19 11:51:08 INFO  ZooKeeperLeaderElectionAgent:54 - We have gained leadership
2020-01-19 11:51:08 INFO  Master:54 - I have been elected leader! New state: RECOVERING



### 7.4 Cassandra sur Workers 

A effectuer en commençant par le Worker 5 jusqu'au 1 :
```bash
cd /home/ubuntu/apache-cassandra-3.11.3/bin && ./cassandra
```

On peut ensuite vérifier le bon fonctionnement en faisant un `Ctrl+C` et `./nodetool status` ou encore `./nodetool describecluster`

```bash
./cqlsh $(hostname -i)
```

### 7.5 Zeppelin sur un Master
```bash
cd /home/ubuntu/zeppelin-0.8.2-bin-all/bin && ./zeppelin-daemon.sh start
cd /home/ubuntu/zeppelin-0.8.2-bin-all/bin && ./zeppelin-daemon.sh stop
```


Pour accéder à zeppelin depuis sa machine locale, il faut commencer par copier le fichier de permission sur le serveur où tourne Zeppelin, puis de redigirer, toujours depuis sa machine locale, le port du service vers notre port local :

```bash
scp -i gdeltKeyPair.pem gdeltKeyPair.pem ubuntu@ec2-xx-xxx-xxx-xxx.compute-1.amazonaws.com:
ssh -L 8090:127.0.0.1:8090 -i gdeltKeyPair.pem ubuntu@ec2-xx-xxx-xxx-xxx.compute-1.amazonaws.com
```
La commande `-L` redirige le port `127.0.0.1:8890` de l'instance Zeppelin de l'EC2 sur notre port local `8090` (port qui est spécifié en premier). On utilise de plus le DNS public du serveur pour s'y connecter en ssh.
Il suffit alors d'ouvrir un navigateur et de se connecter à [http://localhost:8090/#/](http://localhost:8090/#/). 

### 7.6 Observer la charge des serveurs en opération

Pour observer la charge des serveurs, on peut se connecter à ceux-ci et lancer la commande :
```bash
htop
```

## Annexes
* Détail sur les types d'instances utilisables :
https://aws.amazon.com/fr/ec2/instance-types/
* Détail sur la facturation des instances utilisées :
https://aws.amazon.com/fr/ec2/pricing/on-demand/
