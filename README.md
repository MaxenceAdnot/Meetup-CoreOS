# Meetup-CoreOS

Installation et configuration de CloudMonkey
--------------------------------------------

> pip install cloudmonkey
>
> cloudmonkey
>
>   set url https://cloudstack.ikoula.com:443/client/api
>
>   set apikey "votrecléAPI"
>
>   set secretkey "votreclésecrète"
>
>   create sshkeypair name=TestCore
>
>   exit

Copier le contenu la clé générée, créer un fichier TestCore avec les permissions 600 et coller le contenu copié dans ce fichier

Modification du fichier user-data
--------------------------------------

Se rendre sur la page https://discovery.etcd.io/new et modifier dans le fichier user-data la valeur "https://discovery.etcd.io/token par l'URL retournée 
> base64 user-data

Copier le retour de la commande, on l'insérera en paramètre dans la commande de déploiement

Déploiement de trois hôtes CoreOS
---------------------------------

cloudmonkey
> deploy virtualmachine zoneid=f7d3d897-bea7-4e20-bebf-d7a19e358c1f templateid=3d884327-e0f9-47fb-a4be-b77af392de27 serviceofferingid=aa53a6f6-9812-405f-92ca-1a515a85a86b securitygroupids="tab pour autocompleter" keypair=TestCore userdata='insérer le user-data encodé en base64' name=COREOS01
> 
> deploy virtualmachine zoneid=f7d3d897-bea7-4e20-bebf-d7a19e358c1f templateid=3d884327-e0f9-47fb-a4be-b77af392de27 serviceofferingid=aa53a6f6-9812-405f-92ca-1a515a85a86b securitygroupids="tab pour autocompleter" keypair=TestCore userdata='insérer le user-data encodé en base64' name=COREOS02
> 
> deploy virtualmachine zoneid=f7d3d897-bea7-4e20-bebf-d7a19e358c1f templateid=3d884327-e0f9-47fb-a4be-b77af392de27 serviceofferingid=aa53a6f6-9812-405f-92ca-1a515a85a86b securitygroupids="tab pour autocompleter" keypair=TestCore userdata='insérer le user-data encodé en base64' name=COREOS03

Note : Chaque commande deploy vous un JSON contenant entre autre l'IP publique de la machine

Lancement des services
----------------------

Se connecter sur l'une des instances CoreOS
> ssh -i TestCore core@IP

On se place dans le répertoire /units
> cd /units

Vérifier que les 3 machines sont bien listées
> fleetctl list-machines

Démarrer le service de base de données
> fleetctl start wp-db.service

Remplacer dans le fichier la valeur IP-db par l'IP retournée par la commande précédente
> sudo vi wp-web\@.service

Démarrer les deux frontaux Wordpress
> fleetctl start wp-web@{1..2}.service

Charger le unit du LoadBalancer
> fleetctl load wp-haproxy.service

Se connecter sur le serveur dont l'IP a été retournée par la commande précédente
> ssh -i TestCore core@IP

Modifier les deux valeurs IP-web1 et IP-web2 dans le fichier /units/haproxy.cfg  par les IP des deux frontaux lancés (fleetctl list-units | grep wp-web permet de retrouver ces IPs)
> sudo vi haproxy.cfg

Démarrer le service de loadbalancing
> fleetctl start wp-haproxy.service

Après quelques instants, vérifier que tous les units sont bien démarrer (Status : Running)
> fleetctl list-units
