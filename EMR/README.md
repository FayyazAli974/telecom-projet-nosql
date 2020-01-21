# Configuration EMR

**Pré-requis**
* Avoir créé au préalable un bucket S3 nommé "inf728-projet", et dans ce bucket un dossier "data" ;
* Avoir créé au préalable une paire de clés EC2.

**Créer un EMR avec cette configuration:**

* **Version**: la plus récente
* **Application**: Spark
* **Type d'instance**: m4.large
* **Nombre d'instances**: 3
* **Paire de clés EC2**: charger un fichier de clés, créé précédemment


**Une fois le cluster lancé:**

* Modifier le "groupe de sécurité pour le principal" : Choisir le master, les connections entrantes, puis ajouter une règle ("Tout le trafic", "Mon IP").
* Dans l'onglet Configurations, filtrer sur une des deux instances, cliquer sur Reconfigurer > modifier en JSON, puis coller le contenu du fichier config-spark.json. Cocher la case "Appliquer cette configuration à tous les groupes d'instances actifs" puis valider. 

Zeppelin peut alors être lancé et le notebook chargé.
Ne pas oublier d'exporter le notebook Zeppelin à la fin, pour conserver ses modifications.



**Cassandra**
sudo chown -R zeppelin:zeppelin /usr/lib/zeppelin/local-repo/
sudo /usr/lib/zeppelin/bin/install-interpreter.sh --name cassandra  #org.apache.zeppelin:zeppelin-cassandra_2.10:0.6.2
