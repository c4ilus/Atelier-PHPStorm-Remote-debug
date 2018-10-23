# Préparation du projet (avec des vrais morceaux de docker dedans)

## Prérequis

* Veiller à ce que les paquets __Docker__ et __Docker Compose__ soient bien installés sur le système.
* Cloner le projet, et se placer à la racine de ce dernier (au même niveau que le fichier __docker-compose.yml__)

Nous allons dans un premier temps construire les images docker (dont la configuration est détaillée dans les Dockfiles présetns dans le dossier __context__)

    docker-compose build
    
Une fois les images construites, créons les containers

    docker-compose up
  
Nous sommes prêts à travailler, passons à la suite.

## Xdebug  

Nous allons ici utiliser une extension navigateur afin que le navigateur utilisé "communique" avec Xdebug

_Note : Xdebug est installé dans le container que nous avons appellé (dans le fichier docker-compose.yml) __atelier_remote_debug_php__. Pour voir plus d'informations concernant l'installation
de l'extension, se référer au fichier __context > php7-fpm > Dockerfile__. La variable __XDBG_IDEKEY__ est définie dans le fichier __.env__ à la racine du projet. Elle ici a la valeur __PHPSTORM___

* Installer __Xdebug Helper__ sur [Firefox](https://addons.mozilla.org/en-US/firefox/addon/xdebug-helper-for-firefox/) ou [Chrome](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc)
* Configurer l'extension pour utiliser un IDE Key unique (utilisé pour identifier la session de debug) :
    * Sous __Firefox__ : _Modules complémentaires_ > _Préférences_ de Xdebug Helper > Dans la liste de selection de la partie IDE KEY, choisir __PHPSTORM__
    * Sous __Chrome / Chromium__ : Faire un clic droit sur l'icône Xdebug Helper à droite de la barre de recherche, et cliquer sur _Options_ > Dans la liste de selection de la partie IDE KEY, choisir __PHPSTORM__
* Lancer PHPStorm et ouvrir le projet.
* Cliquer sur __ADD CONFIGURATION__ pour accéder à l'écran de configuration du debugger
* Cliquer sur le bouton __+__, et dans la liste qui apparait (Add new configuration), cliquer sur __PHP Remote Debug__
* Donner un nom à la configuration (on préferera une configuration propre à chaque projet, avec un Nom unique)
* Cocher la case __Filter debug connections by IDE key__
* Dans le champ IDE key(session id) saisir PHPSTORM (clé choisie dans la configuration de l'extension, et dans le fichier .env)

Nous allons également devoir créer un server.

_Note : Avant d'effectuer cette opération, nous devons associer (dans notre fichier /etc/hosts) l'adresse IP choisie dans le fichier __.env__, avec un domaine._

    echo "10.13.0.100 atelier.local" >> /etc/hosts
    
* Toujours sur le même écran de configuration, en face du champ __server__ cliquer sur les __...__
* Cliquer sur __+__,
* Choisir un nom à notre server
* Saisir l'Host (ici, atelier.local, choisi dans notre fichier _hosts_)

Nous alons également devoir associer le dossier contenant le code sur notre machine hôte, avec le dossier contenant le code dans notre container.

_Note : Ce partage de volumes est configuré dans le fichier docker-compose.yml)_

        volumes:
        - type: bind
          source: "${HOST_WEBAPP_PATH}"
          target: "/var/www/code"
          
_Nous avons une association de volumes. Le volume __source__ (machine hôte), dont le chemin est défini dans le fichier __env__ avec la variable globale __HOST_WEBAPP_PATH__
est associé au volume __target__ présent dans le container ( __/var/www/code__ )._

* Cocher la cache __Use path mappings__
* Déplier l'arborescence de dossiers jusqu'a voir __volumes > code__
* Cliquer sur l'icone bleu à droite de _code_, et saisir le chemin __/var/www/code__
* Cliquer sur __Apply__ pour enregistrer la configuration server.
* Cliquer sur __Apply__ sur l'écran encore affiché pour valider la configuration, et sur __Ok__ pour fermer l'écran.

YEAAAAAAYYY, notre Xdebug est prêt à être utilisé.

Vous me croyez pas ? Et bah on teste alors !

Dans notre dossier __volumes__ > __code__ est présent un symfony, qui est servit par nginx (container __atelier_remote_debug_nginx__). Nous pouvons
y accéder en tapant _atelier.local_ dans l'URL de notre navigateur préféré.

* Pour effectuer le test, nous allons ouvrir le fichier __index.php__ de symfony (dans le dossier __public__), et mettre un point d'arrêt (cliquer
dans la colonne à droite du numéro d'une ligne) sur une ligne au hasard, puis, cliquer sur l'icone de débug (bouton rouge à droite de l'accès à la configuration du débug).
* Ouvrir le navigateur, cliquer sur l'îcone Xdebug helper dna sla barre navigateur, et rafraîchir la page d'accueil de notre symfony.

Si nous regardons PHPStorm, nous voyons en bas de l'IDE un groupe d'onglet s'ouvrir. 

* Cliquer sur l'onglet __Debugger__. Nous avons maintenant accès au contenu des différentes variables (sous forme de liste dépliante), utilisées dans le fichier index.php, présentes jusqu'avant le point d'arrêt.

SUCCESS !!!
