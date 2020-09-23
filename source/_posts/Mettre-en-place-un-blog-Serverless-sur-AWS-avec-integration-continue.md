---
title: Mettre en place un blog Serverless sur AWS avec intégration continue
date: 2019-06-03 11:29:42
tags: 
  - AWS
  - CI/CD
  - Hexo 
  - Cloudfront
  - S3 
  - Route 53 
  - CircleCI
  - GitHub
  - Markdown
  - Docker
category:
  - Architecture  
---

![GitHub - CircleCI - Hexo - S3][cloudj-techno]

Après avoir passé un bon mois à me former aux techniques cloud d'Amazon Web Services, je voulais mettre ces nouvelles connaissances à l'épreuve en faisant un projet réel. Un blog me permettant de partager mes découvertes était certainement le meilleur point de départ.  

<!-- more -->

J'ai été réellement bluffé par les possibilités incroyables du _"Cloud"_ (mais c'est une autre histoire), j'avais donc décidé d'héberger ce blog sur AWS. 

## First Draft : High Availability Wordpress
L'avantage du Cloud, c'est qu'on peut faire à peu près tout ce qu'on veut. Pas de limitation technique donc.  
L'inconvénient, c'est que si on s'y prend mal, ca peut coûter très... très... cher. Le Cloud n'est pas plus cher qu'un hébergement classique ou _"on premises"_, au contraire, pour pas mal d'usages, on peut faire des économies monstrueuses. Mais il faut le faire bien, et surtout éviter les gaspillages (que ce soit en terme de computing, de stockage, de bande passante, j'en passe et des meilleurs). 

Je construis des sites web depuis maintenant une bonne quinzaine d'années et je suis passé par de nombreuses technologies, langages et autres CMS pour arriver à mes fins. Chacun avec ses avantages et inconvénients. 
J'étais parti sur le bon vieux réflexe _Blog = Wordpress_ et j'avais même commencé à en déployer un sur mon laptop. 
Et puis je me suis posé la question de comment héberger ça et lui assurer les meilleures conditions : 
* haute disponibilité
* sécurité
* temps de réponse
* backup en cas de problème

Au final, après pas mal de recherches sur les méthodes et bonnes pratiques, ca donnait quelque chose comme ca : 

![Wordpress Haute Disponibilité sur AWS][wordpress-ha-aws]

Un peu complexe mais ca fait le job :
* Des machines virtuelles hébergent le code php de Wordpress, elles sont réparties sur plusieurs datacenters en autoscaling groups pour faire face à la charge s'il y a lieu.  
* Un cache frontal avec le CDN CloudFront
* Un cache applicatif avec Memcached (Elasticache) 
* Une base de données haute disponibilité avec Amazon Aurora. 

Et c'était même gérable en terme de maintenance car je me suis formé au passage à [Terraform][2] pour faire de l'_Infrastructure as Code (IaC)_. Je recommande d'ailleurs fortement l'outil si vous êtes amenés à déployer des infrastructures complexes dans le cloud. 

Et puis est venu le moment de budgétiser tout cela et je suis rapidement revenu sur terre. 
Il y avait quand même un minimum de 2 serveurs actifs en permanence et même s'il y a pas mal de méthodes de [pricing sur Amazon EC2][4] (le service de machines virtuelles), il fallait faire des concessions à moins de se retrouver avec un hébergement à 1000$/an. 
Ce qui n'est finalement pas si cher pour un site professionnel qui rapporte de l'argent vu ce qu'apporte cette architecture mais pour ce blog, qui démarre, et que je ne compte pas monétiser; ce n'était pas une bonne solution. Je ne voulais pas payer pour des serveurs wordpress qui dorment en attendant qu'un visiteur se pointe pour consulter ce blog. 


![Ne jetez pas vos ordures dans la nature][poubelle]


## Let's go Serverless

Les architectures en microservices ont le vent en poupe ces derniers temps, on parle beaucoup de découplage, de séparations d'intérêts en opposition à des architectures monolithiques comme Wordpress ou Drupal. 
En somme, avec les CMS, vous avez un gros programme qui fait tout dynamiquement pour vous et il n'est malheureusement pas rare qu'un grain de sable vienne gripper tout le mécanisme (un plugin mal codé, une base de données qui cale sur une query exécutée partout ou même un service tiers qui met du temps à répondre). Vous faites alors face à la redoutée erreur 503; ceux qui ont fait tourner des CMS à grande échelle savent de quoi je parle. 
Sans parler des failles découvertes régulièrement et qui sont littéralement guettées par les hackers qui scannent votre site avec des bots à la recherche d'un exploit dans lequel ils pourraient s'engouffrer. Après avoir passé des nuits à mettre à jour une quarantaine d'instances Drupal pour un client, je peux vous assurer que c'est réellement un problème. 

#### Je veux juste un blog et qu'on me laisse tranquille

On reprend donc : 
* haute disponibilité
* sécurité
* temps de réponse
* backup en cas de problème

Et j'ajoute : 
* pas cher 
* facile à maintenir (en terme de sécurité ET de performance) et donc; exit les monolithes.  

Après quelques recherches, je suis tombé sur cet article : 
["Build a Serverless Production-Ready Blog"][5]
Et j'ai suivi cette technique.

Je ne vais pas refaire l'article en Français mais je vais vous redécrire la technique et souligner les problèmes que j'ai rencontrés. Cet article a plus d'un an aujourd'hui et certaines choses ne fonctionnaient pas directement; d'autres sont trop brièvement expliquées pour qui ne connait pas Amazon Web Services en détail. Je vous renvoie à l'article si vous voulez essayer de faire une mise en place similaire, j'ajouterai ici des liens vers les documents permettant de comprendre et refaire ce qui n'est pas détaillé.     

Amazon S3 (pour Simple Storage Service) permet de stocker dans un "bucket" vos fichiers statiques et les rendre disponible sur internet. Il est même possible d'[utiliser un bucket pour héberger un site web statique à moindre frais][1].



**NB: votre bucket doit avoir exactement le même nom que le domaine qui va pointer dessus : ici, www.delsoir.com, donc.** Pensez donc à vérifier que le bucket est toujours disponible avant de réserver votre nom de domaine. 


L'architecture que voici est donc ultra simple :  

![CloudJ Website Delivery][cloudj-delivery]

Pour la mettre en place : 
1. Créer le bucket S3 et activer le website hosting. Créer un index.html dans le bucket pour tester. 
2. Créer un certificat SSL pour le HTTPS avec ACM
3. Créer une distribution Cloudfront qui pointe sur votre bucket S3
  * Une distribution CloudFront met dans les 15-20m à etre déployée. 
  * Vérifiez bien de renseigner le nom de domaine que vous voulez utilisez dans "Alternate Domain Names", sans quoi ca ne marchera pas (403 bad request); la mise à jour prendra à nouveau 15-20m
4. Enregistrer un nom de domaine avec Route 53; le faire pointer sur la distribution CloudFront. 

Cet article explique dans le détail comment réaliser ces points : ["Setup AWS S3 static website hosting using SSL (ACM)"][7]

Arrivé ici, on dispose d'un site web de quelques lignes mais qui est prêt pour la haute disponibilité et qui est délivré en HTTPS. 

![La première version de Delsoir.com sur AWS S3][delsoircom-v1]

#### Notre TODO-LIST :
* ~~haute disponibilité~~: Cloudfront assure la mise en cache des éléments statiques pour ne pas trop solliciter le bucket.  
* ~~sécurité~~: Qui peut hacker un site statique ?
* ~~temps de réponse~~: CloudFront, avec les nombreuses edges locations d'Amazon, assure un temps de réponse minimum à l'utilisateur
* ~~pas cher~~: La tarification de la plupart des services AWS se fait en _Pay as You Go_ et le premier _Tier_ d'usage est généralement gratuit. 
  * S3 : en fonction du nombre de requêtes (limité via le cache de CloudFront) et de la quantité stockée. 
  * CloudFront : en fonction du trafic que génère mon site.  
  * le certificat SSL HTTPS public est gratuit chez Amazon.   
* backup en cas de problème : TODO
* facile à maintenir : TODO

Il y a déja pas mal de requirements qui sont rencontrés mais... 

![C'est juste un site blanc d'une page de 2 lignes!!! On voulait un vrai blog!][onlyone]

C'est là qu'on entre dans le vif du sujet : Comment faire pour mettre à jour le bucket S3 avec le contenu de mon blog ? 
Je ne veux pas : 
* Editer le HTML à la main et uploader les fichiers manuellement 
* être bloqué avec un CMS dont les articles contiendraient du code spécifique au CMS susmentionné. 

#### Here comes Hexo

![Hexo a blog framework][hexo-log]

[Hexo][6] se définit lui même comme étant un _blog framework_ et utilise le [_Markdown_][8] comme format pour créer ses pages html. 
En somme, avec Hexo, on crée des fichier _.md_ et le logiciel, en NodeJS, se charge lui même de la conversion html. Ceux qui sont habitué à utiliser GitHub connaissent bien ces fichiers, le README.md étant souvent créés par défaut à l'initialisation d'un repository. Les articles sont donc stockés dans des fichiers et non en base de données... bienvenue au versionning (~~backup~~). 
Je vous renvoie simplement à l'article ["Build a Serverless Production-Ready Blog"][5] pour démarrer rapidement un projet Hexo. 

Les étapes nécessaires sont : 
* créer un projet hexo en local
* le versionner avec github pour disposer d'un backup de l'ensemble : contenant & contenu 

#### Now how to publish  ? 

Hexo vient avec un serveur Node qui interprète les fichiers qu'on lui fournit mais on ne l'utilisera pas, si ce n'est en local pour visualiser les articles qu'on rédige. 
Il y a surtout la commande suivante qui permet de générer complètement un site statique : 

{% codeblock lang:bash %}
> hexo generate
{% endcodeblock %}

Comme l'ensemble est désormais versionné, on peut sereinement utiliser des techniques d'[intégration continue][9] pour réagir à chaque modification du contenu. 
[CircleCI][3] offre 1000 minutes de build gratuitement par mois, on peut donc l'utiliser pour générer le contenu et envoyer les fichiers sur Amazon S3. Le paramétrage est décrit dans l'article source et le principe est le suivant :
A chaque commit poussé sur GitHub, CircleCI va : 
* lancer un container docker
* installer hexo et ses dépendances dans le container
* installer aws-cli pour se synchroniser avec Amazon
* générer le site statique 
* envoyer les modifications sur Amazon S3 

L'ensemble prend environs 30s par build au lancement du blog, j'ai donc droit à ~2000 déploiements CircleCI par mois gratuitement. 

NB: 
* l'image docker utilisée dans la config de CircleCI m'a posé problème les dépendances pour installer AWS-CLI n'étaient pas rencontrées. J'ai fini par utiliser [cette image docker][10] dans ma configuration. 
* il faudra [créer un utilisateur IAM sur AWS][13] et lui [assigner la permission d'écrire dans votre bucket S3][11]. Vous recevrez les credentials nécessaires à renseigner dans les settings de votre projet CircleCI. **Ne mettez jamais vos access key AWS dans le code de votre projet, des bots scannent github à la recherche de ces identifiants.**  

Si je fais la moindre erreur dans mes configurations, le build échouera et mon site ne sera pas mis à jour. 
Il me suffit d'un accès à mon compte github pour pouvoir publier de nouveaux articles ou modifier le site; tout le reste se fait automatiquement. Un simple git push suffit à démarrer la procédure de publication. 
Et cerise sur le gateau, je dispose désormais d'une timeline qui reprend tous les update que j'ai réalisé sur mon site. 

![Le workflow de publication sur CircleCI][circleCI-workflow]

Et la boucle est bouclée : ~~facile à maintenir~~ DONE  

Le résultat, vous l'avez devant les yeux.
![CloudJ v1][cloudj1]

## Conclusion 

#### Les qualités
* Mise en place rapide, quelques heures pour boucler le tout.  
* Tous les procédés sont standards, on peut envisager sereinement de changer chaque brique du système. 
GitHub => GitLab
CircleCI => GitLabCI
AWS => Azure / GCP / autre hébergeur
* Le coût opérationnel est pratiquement nul. Seule la gestion DNS avec Route53 coûte 0,50$/mois. Pour les autres services utilisés, tout est gratuit tant qu'on reste sous certains paliers (Je limite d'ailleurs en hébergeant les images ci dessus sur imgur) _À valider dans le temps le réel coût/visiteur, nous sommes au jour 2. A l'heure d'écrire ces lignes, je n'ai rien payé en dehors de la gestion DNS_.     
* Le front end est complètement fiable en termes de performances et de sécurité car statique.  

#### Les défauts 
* N'importe qui ne peut pas administrer le blog facilement, il faut pouvoir écrire le Markdown pour rédiger des articles et connaitre git pour publier; c'est définitivement un outil pour les initiés en l'état. **MAIS** Il existe pas mal de [logiciels qui produisent du markdown][12] et on peut envisager facilement un client qui automatiserait ces processus. 
* Aucune fonction dynamique, mais c'est aussi ce qui permet d'être complètement tranquille au niveau sécurité et stabilité. Les eventuelles fonctionnalités dynamiques devront être gérées par des microservices dédiés comme le veut l'architecture. 
* A grande échelle (site avec des milliers/millions d'articles-pages-tags-categories et de nombreux éditeurs), la publication faite par Hexo risque de prendre beaucoup de temps et donc de limiter la vitesse de publication d'un article. 30 secondes, c'est de toute manière déja trop pour un éditeur professionnel. 

La prochaine fois, on parlera PWA; vous pouvez déja ajouter ce site à votre homescreen et celui ci fonctionnera offline sur votre smartphone. Je vous envoie donc ces lignes avec 

{% codeblock lang:bash %}
> git push
{% endcodeblock %}

  
[1]: https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html
[2]: https://www.terraform.io/
[3]: https://circleci.com
[4]: https://aws.amazon.com/ec2/pricing/
[5]: https://hackernoon.com/build-a-serverless-production-ready-blog-b1583c0a5ac2
[6]: https://hexo.io
[7]: https://aws.amazon.com/getting-started/tutorials/get-a-domain/
[8]: https://en.wikipedia.org/wiki/Markdown
[9]: https://fr.wikipedia.org/wiki/Int%C3%A9gration_continue
[10]: https://hub.docker.com/r/jtredoux/node-aws/
[11]: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_change-permissions.html
[12]: https://stackedit.io/app
[13]: https://docs.aws.amazon.com/fr_fr/IAM/latest/UserGuide/id_users_create.html
[cloudj-techno]: https://i.imgur.com/7KeZY4m.png
[cloudj-delivery]: https://i.imgur.com/Bj7LAng.png 
[wordpress-ha-aws]: https://i.imgur.com/GEIczFb.png
[poubelle]: https://i.imgur.com/Y8UqtQI.png
[delsoircom-v1]: https://i.imgur.com/8y0iWrc.png
[onlyone]: https://i.imgur.com/kSBNkPr.png
[circleCI-workflow]: https://i.imgur.com/1hBpup1.png
[hexo-log]: https://oded.blog/images/2017/07/hexo-logo.png
[cloudj1]: https://i.imgur.com/EeQuMM7.png
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5OTc1Njc1MjAsLTE5OTc1Njc1MjAsLT
UyNjM2MzI1OSw0Nzg0Mjc3NjNdfQ==
-->