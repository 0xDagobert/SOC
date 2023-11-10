# Architecture du projet

### Le but de ce document est d'expliquer ce qu'il se trouve dans chaque dossier:

## OSQUERY FOLDER

### **OSQUERY.CONF**

Ce fichier est le ficher de configuration principal sur lequel va se baser le daemon osqueryd.

#### Les bonnes pratiques pour modifier ce fichier sont les suivantes:

    - Toute modification doit être faite après avoir arrété le daemon car il a besoin de redémarrer afin d'appliquer les modifications
    - Avant d'ajouter / modifier  un pack ou l'attribut decorator veuillez tester les queries dans la partie schedule
    - La partie option est relativement sensible veuillez vous réferer à la documentation officielle avant toute modification

#### Pour stopper le daemon:

Linux: systemctl stop osqueryd
Windows: net stop osqueryd

#### Patern d'OSQUERY.CONF

{
	"options": {
        ...
	},
	"schedule": {
        ...
	},
	"decorators": {
        ...
	},
	"packs": {
        ...
	},
}

Ci-dessous sont détaillées chacune des sections 
Ces sections sont obligatoires, d'autres existent et sont détaillées dans la documentation officielle


### Options

#### Description

Le segment Option vise à renseigner les options de configuration du Daemon osqueryd.
L'ensemble des options est donnée dans le manuel osqueryd accessible via la commande osqueryd --help sous le titre "osquery configuration options"

Ces options sont renseignables de 2 manières: Via osquery.conf ou directement via CLI lors du lancement du daemon.


### Schedule

#### Description

Le segment Schedule doit être considérée comme la seconde instance de test des queries. 

Elle ne doit accueillir que des queries préalablement testées via l'utilitaire osqueryi.

Les queries renseignées dans ce segment doivent être temporaires et répondant uniquement à 2 besoins:

    - Accueillir des requêtes encore en phase de test et visant à être transférées dans les Packs (Queries visant à détecter un comportement malveillant lié à une nouvelle faille par exemple)
    - Accueillir des requêtes de "Débug" système ou réseau temporaires liées à la santé de l'infrastructure

Si votre query ne respecte pas ces besoins, vous devez exclusivement utiliser l'utilitaire osqueryi pour exécuter vos requêtes, en le couplant éventuellement à un cron pour créer une périodicité.

#### Patern

Le segment schedule s'organise de la manière suivant. Les champs requis sont marqués d'une *

{
    "schedule": {
        "nom de la recherche": {
            "query": string ,
            "interval": int ,
            "removed": bool ,
            "snapshot": bool ,
            "platform": string ,
            "version": string ,
            "shard": int(1 - 100) ,
            "denylist": bool 
       }
    }
}

Tel que:
    - query *: La recherche SQL à exécuter
    - interval *: La durée en secondes entre 2 lancement de cette query (à noter que ce temps se calcule via l'uptime du daemon)
    - removed: Détermine si les actions supprimées doivent être logguées, par défaut True
    - snapshot: Détermine l'activation du mode snapshot, par défaut False
    - platform: Détermine la / les plateformes d'éxécution de la query ('all', 'windows', 'linux', 'posix', 'darwin')
    - version: La version minimale d'osquery sue laquelle les query vont s'éxécuter
    - shard: Restreint l'éxécution de la query sur un pourcentage de machien hôtes (1-100)
    - denylist: Détermine si cette query peut être stoppée par le watcher osquery si les ressources matérielles utilisées sont trop importantes. Lié au flag de control du daemon --disable_watchdog=false


### Decorators

#### Description

Les decorators existent pour ajouter des informations additionnelles indépendantes des logs générés.
Ils vont apparaître en premier dans la ligne de log.

3 types de decorators existent:
    - Load: La query décrite par ce décorateur est exécutée lors du load de la configuration
    - always: La query est exécutée avant chaque scheduled query
    - interval: La query est exécutée selon l'interval spécifié

Chaque décorator doit retourner au maximum 1 colonne, sinon un log de comportement inconnu est levé

#### Patern et exemple

{
  "decorators": {
    "load": [
      "SELECT version FROM osquery_info;",
      "SELECT uuid AS host_uuid FROM system_info;"
    ],
    "always": [
      "SELECT user AS username FROM logged_in_users WHERE user <> '' ORDER BY time LIMIT 1;"
    ],
    "interval": {
      "3600": [
        "SELECT total_seconds AS uptime FROM uptime;"
      ]
    }
  }
}

L'exemple précédent va retourner:

{"decorations": {"user": "you", "uptime": "10000", "version": "1.7.3"}}

On remarque que load contient 2 colonnes, la seconde est donc ignorée pour display uniquement la version d'osquery


### Packs

#### Description

Dans le folder Osquery Packs vous allez trouver différents fichiers de confs qui vont consister à donner des recherches à faire à Osquery sous forme de batch de scheduled requests

Le but de ces packs va être de créer du Log lorsqu'un comportement non voulu se présente.

Il est recommandé de coupler ces queries avec des règles Sigma afin de couvrir l'ensemble de l'endpoint

Dans les packs, les queries peuvent avoir un interval différent cf: WindowsHardening.

Il est possible de spécifier dans l'objet packs plusieurs queries sous la même forme que les scheduled queries, cependant il est recommandé de spécifier le path d'un fichier contenant des queries par thème cf OsqueryPacks

