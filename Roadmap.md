Projet SOC

1. **Mise en place de l'infra**

	- Choix des solutions 
	- Déploiement
	- Paramétrage 
	- Création de règles SIGMA et YARA suivant le hunting sur l'APT
	- (Optionnel suivant le choix de la solution) Création d'un parser SIGMA to Query language

2. **Hunting APT**

	- Chercher sur MITRE le groupe Aquatic Panda, repérer tous les indicateurs possibles en suivant la pyramid of Pain vue en cours
	- Aller chercher les IOCs, liés au groupe (Hash, Ip, domain, etc...) -> Pyramid of Pain
	- Aller chercher des rapports de Hunting, CTI, Malware analysis, Intrusion pour reconstituer les Kills Chain -> Tout savoir de leur façon de faire de l'infection à la détection. Parmi les sources de ces rapports, on trouve tous les éditeurs de logiciel de sécu (Kaspersky, Crowdstrike, etc...) les sites de ressources offensives (Vx-Underground, etc...), ainsi que les blogposts en tout genre (Medium, Linkedin, Twitter)
	- Récupérer les éventuels binaires qui peuvent trainer sur Internet (Vx-underground et VirusTotal)
	- Une fois les IOCs récupérés, faire du Hunting et de la qualification dessus. Pour les Hashs -> VirusTotal (Oubliez pas de récupérer les règles Sigma et Yara déjà existantes), pour les IPs Shodan, Pour les binaires n'importe quelle sandbox  (VirusTotal, Malcore, Any.run), Pour les domaines DnsDumpster. Le but de cette étape va être de savoir si les domaines et Ips récupérées sont toujours liées à l'APT ou non

3. **Red Team**

	- Afin de simuler une attaque on doit suivre le People Process Tool, les tools vont être les Iocs récupérés (Binaires, Hashs, Ips, Domaines, TTP)
	- Créer des customs attaques avec Atomic Red Team 
	- Créer un environnement automatique 

4. **Mise en place de contremesures**

	- Création de règles Sigma sur mesure suivant la réussite des attaques 
	- Mise en place de moyens DFIR sur l'environnement


N'hésitez pas à rajouter des éléments si besoin