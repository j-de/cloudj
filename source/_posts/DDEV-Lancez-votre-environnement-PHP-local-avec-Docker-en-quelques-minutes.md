---
title: 'DDEV : Lancez un environnement PHP local avec Docker en quelques minutes'
date: 2020-06-20 13:41:43
tags: 
  - PHP
  - Drupal
  - DDEV 
  - Docker
category:
  - Architecture
---
![ddev-logo]

Si vous avez bossé dans le web pendant quelques années; vous avez certainement de nombreux projets PHP installés sur votre machine et vous avez certainement rencontré les problèmes suivants : 

* Vous devez jongler avec plusieurs versions de vos librairies (Par exemple PHP 5.6 et PHP 7.X mais aussi les différents mods apache)
* Du coup, vous avez certainement une machine où TOUT est activé 
* Globalement, le serveur de prod ne réagit pas comme votre environnement local et quand on déploye; on obtient plein de petites erreurs inattendues
  -- Mais, ça marche sur ma machine, chef!
  -- Ok backuppes tes mails; on met ton portable en prod  

<!-- more -->

Dans la plupart des cas, vous faites fonctionner l'ensemble avec XAMP sur Windows, MAMP sur OSX ou vous avez tout installé à la main sous Linux si comme moi, vous avez le goût d'apprendre sous la torture. 
Dans les 3 cas, vous avez un serveur web unique installé sur votre machine et vous le configurez / déconfigurez / reconfigurez au gré des projets. 

Vous activez l'option X pour vous débarasser de l'erreur 1 du projet A. Et sans vous en rendre compte, vous venez d'introduire le problème 2 dans le projet B qui vous attendra bien au chaud quelques mois le temps que client B ait besoin d'une modification *"simple en toute urgence"*. 
-ddev-
Plus vous avez de projets, plus vous maintenez un même environnement unique, plus les problèmes se multiplieront: c'est inévitable. 


## Docker à la rescousse 

![docker-logo]
[Docker][docker-site] est un "systeme de gestion de conteneurs" qui permet; sur un même OS; de cloisonner des processus et leur ressources qui fonctionnent cependant avec le coeur de l'OS hôte. 

Bien que ce soit litt-é-ra-le-ment révolutionnaire, je ne compte pas m'étendre sur la technologie et ses implications ici. Contentons nous de voir les avantages principaux dans le cas de notre environnement de développement local : 
* La possibilité d'encapsuler l'ensemble de nos sources, configs, libraires et dépendances dans une "*boite*" (le container) que l'on pourra ouvrir; fermer; modifier à volonté. 
* Cette même boite pourra être ré-utilisée sur des machines différentes avec un OS différent mais en gardant l'assurance qu'elle fonctionnera exactement de la meme manière. 

#### Mais Docker, c'est pas si simple
C'est ce qui m'a fait me tourner vers Docker mais malheureusement, ce n'était pas si simple. J'ai vite compris qu'il faudrait :
* gérer plusieurs containers différents pour un meme environnement ( php sans mysql, ce n'est pas si fun )
* leur donner la possibilité de communiquer entre eux 
* s'assurer de leur persistance une fois qu'ils sont arrêtés. 
  
Bref en termes techniques, en plus d'automatiser ma config; mes librairies en l'installation de mes sources; ca revient à m'occuper de:
* mapping de ports
* réseau virtuel
* gestion de points de montage sur le filesystem hôte. 

En somme, il reste beaucoup d' "*heavy lifting*" et impossible d'installer cela chez tout le monde facilement comme prévu. 

[Docker Compose][dockerc-link] fait ce boulot d'*orchestration* de manière relativement simple en configurant tout avec un fichier YML et j'ai travaillé avec ce concept pendant plusieurs années. Mais j'ai rencontré pas mal de soucis; notament pour faire fonctionner ceci sous Windows (probleme de filesystem et autres liens symboliques essentiellement). 

Pour faciliter cette portabilité; j'en étais arrivé à écrire des scripts bash qui permettaient de générer les *DockerFile* et *dockercompose.yml* en fonction du projet qu'on voulait créer. 

Et puis, ce mois ci, je suis tombé sur DDEV qui fait ce que j'imaginais avec mes scripts bash. Et bien plus. En beaucoup mieux.  

## DDEV 

[DDEV][ddev-site] est un outil simple en ligne de commande qui va vous permettre d'instancier automatiquement les conteneurs nécessaires pour disposer d'une stack locale pré-configurée en fonction de votre projet. 
Sous le capot; il utilise Docker & Docker Compose; ce qui en fait un outil relativement standard et extensible. 

J'ai découvert DDEV en voulant tester Drupal 9 qui est sorti début juin; une documentation de comment installer D9 avec DDEV se trouve sur le [site officiel de Drupal][d9-ddev]. Comme d'habitude; je ne vais pas réecrire ce qui existe déja, si vous voulez faire ce test; suivez [le lien][d9-ddev]

**NEXT** migrer un projet existant sur ddev






* [Installez DDEV][install-ddev]
* [Tester Drupal 9 avec DDEV][d9-ddev]
## COMMANDS 
{% codeblock lang:bash %}
ddev config
{% endcodeblock %}

{% codeblock lang:bash %}
ddev start
{% endcodeblock %}


## Plus

### ... sur Docker 
Video de NetworkChuck sur les bases de Docker et ses avantages (Anglais)
{% oembed https://www.youtube.com/watch?v=eGz9DS-aIeY 600 %}

### ... sur DDEV
Présentation de DDEV au Drupal Camp Colorado en 2019 (Anglais)
{% oembed https://www.youtube.com/watch?v=A5qH8RzgiF0 600 %}

[docker-site]: http://docker.com
[ddev-site]: http://ddev.com
[ddev-logo]: https://i.imgur.com/kxbImam.png
[install-ddev]: https://ddev.readthedocs.io/en/stable/
[d9-ddev]: https://www.drupal.org/docs/official_docs/en/_local_development_guide.html
[docker-logo]: https://w7.pngwing.com/pngs/256/416/png-transparent-docker-github-node-js-mongodb-computer-software-github-blue-marine-mammal-logo.png
[dockerc-link]: https://docs.docker.com/compose/
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1OTc3NDI5NDksLTcyNzY0NzMxMF19
-->