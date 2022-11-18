# Module 2 : Réplication de base de données

Il y a plein de façons de mettre en place de la réplication de base données de type MySQL (comme MariaDB).

MariaDB possède un mécanisme de réplication natif qui peut très bien faire l'affaire pour faire des tests comme les nôtres.

Une réplication simple est une configuration de type "master-slave". Un des deux serveurs est le *master* l'autre est un *slave*.

Le *master* est celui qui reçoit les requêtes SQL (des applications comme NextCloud) et qui les traite.

Le *slave* ne fait que répliquer les donneés que le *master* possède.

La [doc officielle de MariaDB](https://mariadb.com/kb/en/setting-up-replication/) ou encore [cet article cool](https://cloudinfrastructureservices.co.uk/setup-mariadb-replication/) expose de façon simple comment mettre en place une telle config.

Pour ce module, vous aurez besoin d'un deuxième serveur de base de données.

✨ **Bonus** : Faire en sorte que l'utilisateur créé en base de données ne soit utilisable que depuis l'autre serveur de base de données

- inspirez-vous de la création d'utilisateur avec `CREATE USER` effectuée dans le TP2

✨ **Bonus** : Mettre en place un setup *master-master* où les deux serveurs sont répliqués en temps réel, mais les deux sont capables de traiter les requêtes.

![Replication](../pics/replication.jpg)
