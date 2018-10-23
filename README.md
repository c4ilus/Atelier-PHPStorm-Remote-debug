# Préparation du projet (avec des vrais morceaux de Docker dedans)

## Prérequis

* Veiller à ce que les paquets __Docker__ et __Docker Compose__ soient bien installés sur le système.
* Cloner le projet, et se placer à la racine de ce dernier (au même niveau que le fichier __docker-compose.yml__)

Nous allons dans un premier temps construire les images Docker (dont la configuration est détaillée dans les Dockfiles présents dans le dossier __context__)

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

Nous allons maintenant configurer PHPSTorm, afin qu'il utilise PHP Code sniffer en tant que linter (inspecteur en temps réel)

## PHPCS

Les standards d'ecriture de code pour le monde PHP sont définis par la [PSR-2](https://www.php-fig.org/psr/psr-2/).

De même que Xdebug est configuré pour ce projet, nous allons faire une configuration spécifique pour phpcs.

### L'interpreteur en ligne de commandes.

Avant toute chose, nous allons devoir configurer un "CLI interpreter", afin d'acceder a la ligne de commande __"php"__ du container _atelier_remote_debug_php__.

* Cliquer sur __File__ > _Settings...__
* Dans la nouvelle fenêtre, déplier la section __Languages & Frameworks__, et cliquer sur __PHP__
* Dans la partie de droite, cliquer sur les __...__ en face de __CLI Interpreter__
* Cliquer sur __+__, puis sur __From Docker, ...___

Nous serions tentés ici d'utiliser __Docker Compose__, QUE NENNI !! Nous travaillons ici avec le version 3.2 de la
syntaxe de Docker Compose. Cette version entraine cependant un bug empêchant l'utilisation de cette option. Nous allons
nous contenter de l'utilisation plus "simple" de Docker.

* Cocher l'option __Docker__

En face de Server, nous voyons __Docker__ déja selectionné. A ce stade, l'IDE est déja connexté au daemon Docker via
un socket Unix. Nous n'avons qu'a choisir l'image à utiliser.

* En face de __Image name__, cliquer sur la flèche, et choisir dans la liste l'image qui nous interesse (crée avec le nom atelierphpstormremotedebug_php:latest
dans notre cas).
* Dans le champ __PHP interpreter path__, saisir __php__, et cliquer sur __OK__

Dans l'écran qui suit, si tout s'est déroulé comme prévu, nous devrions voir l'information __PHP version: 7.2.<qqch> (selon version de l'image php d'origine),
et __Debugger: Xdebug 2.6.1__

* Cliquer sur __APPLY__ et __OK__

### Code sniffer

Nous revenons ici à l'écran __Settings__

* Toujours dans la section __Languages & Framework__, déplier __PHP__, et cliquer sur __Code sniffer__.
* En face du champ __Configuration__, cliquer sur __...__.
* Cliquer sur __+__.
* Dans la fenêtre qui s'ouvre, choisir le nouvel interpreter que nous venons de créer, et cliquer sur __OK__.
* Dans le champ __PHP Code Sniffer path__, saisir __phpcs__, et cliquer sur __Valider__. En cas de succès,
une pop-up verte devrais s'ouvrir affichant la version de PHP Code sniffer présente dans le container.
* Cliquer sur __APPLY__ puis __OK__

### L'inspecteur

Dernière chose à faire ... ça commence à être long là non ? Mais non, mais non !

Nous revenons (pour la dernière fois) à l'écran __Settings__

* Déplier la section __Editor__ et cliquer sur __Inspection__
* Dans la partie de droite, déplier __PHP__ > __Quality tools__
* Cocher __PHP Code snifer validation__
* Cliquer sur __APPLY__, puis en face du champ __Coding standard__, cliquer sur le bouton de recherche bleu (afin de charger la liste des standards utilisables par phpcs)
* Dans la liste qui apparaît enfin, selectionner __PSR2__
* Cliquer sur __APPLY__ puis __OK__

Vous voilà prêts !!

Désormais dans PHPStorm, chaque ligne présentant une erreur de respect à un standard sera soulignée en jaune. Le message
d'avertissement associé apparaîtra dans une bulle au survol de la dite ligne, ou en bas de l'editeur, en cliquand sur cette même ligne.