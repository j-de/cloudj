---
title: Transformer votre site en Progressive Web App
date: 2019-06-04 10:43:35
tags:
  - PWA
  - GoogleI/O
  - Javascript
  - Mobile
category:
  - Web Development
---
![Progressive Web App][pwa-logo]

On parle beaucoup des Progressive Web App cette année et si vous avez suivi la GoogleI/O le mois dernier, vous avez certainement vu pas mal de choses intéressantes à ce sujet. 
Maintenant qu'on a un [site rapide][serverless-blog], nous allons voir comment ajouter les fonctionnalités PWA à votre site web. 

<!-- more -->

## PWA, c'est quoi ? 

L'idée principale est de dédier sa propre fenêtre à votre site web pour donner à vos utilisateurs une expérience full screen. 
On propose à l'utilisateur d'ajouter votre site à son écran d'accueil et voilà, vous avez une application installée en tout point similaire à ce qu'on obtenir sur un appstore en ligne. 
![Ce que ca donne sur ce blog][pwa-cloudj]
Au delà de la compatibilité sur toutes les plateformes et de l'énergie que met Google à mettre cette technologie en avant, je pense qu'on n'a pas fini d'en parler, tellement ca améliore l'expérience de l'utilisateur final.

## Les avantages 
* Plus besoin de maintenir des applications natives pour être présent directement sur l'écran de vos utilisateur. 
De nombreux sites aujourd'hui développent des applications natives pour Android & iOS qui permettent de susciter plus d'engament. Cela signifie donc : maintenir des flux pour nourrir ces apps en contenus et faire 2 autres développements supplémentaires pour les devices mobiles. Peut importe votre organisation et les moyens que vous y mettrez; ces applications mobiles seront toujours en retard par rapport à votre site web car par essence, elles n'en sont qu'un reflet qui en dépend par des flux. 
* Compatible avec tous les OS / écrans 
* Couplé aux autres technologies in-browser (ex [WebAssembly][wasm]), on peut virtuellement faire tout ce qu'on veut comme développement.  
* La gestion du cache au niveau client amenée par le _service worker_ (qu'on détaillera par la suite) permet une expérience offline pour vos visiteurs. 
* Cette meme fonctionnalité permet d'améliorer **radicalement** l'expérience de navigation sur votre site web. 

## Les pré-requis

![Les 12 pratiques à implémenter pour etre PWA compliant][pwa-req]

Avant d'entrer dans le vif du sujet, il est important que votre site respecte plusieurs standards, les points 1, 4 & 7 sont critiques et cela ne marchera pas correctement si votre site n'est pas compliant 
* Rapide sur réseau mobile : dans tous les cas, votre site doit interractif en dessous des 10 secondes. Cela peut poser problème pour certains sites monolithiques et il faudra parfois travailler vos backend & front end pour arriver à ce niveau de performance.
* Utiliser le HTTPS : Votre site doit fonctionner en HTTPS 
* Rediriger le HTTP vers le HTTPS : plus rien ne doit passer en HTTP sur votre domaine. 

### 1ere étape; le manifest.json 

Ci-dessous le manifest.json de ce blog : 
{% codeblock lang:javascript %}
{
name: "CloudJ",
short_name: "CloudJ",
icons: [
	{
		src: "/images/icon-72x72.png",
		sizes: "72x72",
		type: "image/png"
	},
	{
		src: "/images/icon-96x96.png",
		sizes: "96x96",
		type: "image/png"
	},
	{
		src: "/images/icon-128x128.png",
		sizes: "128x128",
		type: "image/png"
	},
	{
		src: "/images/icon-144x144.png",
		sizes: "144x144",
		type: "image/png"
	},
	{
		src: "/images/icon-384x384.png",
		sizes: "384x384",
		type: "image/png"
	},
	{
		src: "/images/icon-512x512.png",
		sizes: "512x512",
		type: "image/png"
	}
],
start_url: "/",
theme_color: "#000000",
background_color: "#000000",
display: "standalone"
}
{% endcodeblock %}

Ce fichier vous permet de déclarer vos icônes, le nom à donner à votre app, des couleurs de theming et la "start_url" qui sera en fait l'écran d'accueil quand l'utilisateur ouvrira votre app. 
Ce [générateur de manifest][manif-gen] vous permet de générer le fichier ainsi que les icônes nécessaires dans tous les formats. 
Il faut ensuite déclarer votre manifest dans la partie _head_ de toutes les pages de votre site 

{% codeblock lang:html %}
<link rel="manifest" href="/manifest.json">
{% endcodeblock %}

### Le Service Worker 

Le service worker est un javascript qui sera chargé - au minimum - de gérer le mode offline et d'implémenter une logique de caching pour délivrer la meilleure expérience possible.

**Attention, ce javascript doit être à la root de votre site pour lui donner un scope global sur le site. S'il est dans un sous dossier, il n'aura accès qu'aux éléments plus bas que lui.**

Le code de www.delsoir.com/sw.js :  
{% codeblock lang:javascript %}
self.addEventListener('install', function () {
  return self.skipWaiting();
});
self.addEventListener('active', function () {
  return self.clients.claim();
});

var precacheUrls = [];

  precacheUrls.push('/');

  precacheUrls.push('/css/style.css');

  precacheUrls.push('/2019/06/03/Mettre-en-place-un-blog-Serverless-sur-AWS-avec-integration-continue/');

  precacheUrls.push('/2019/06/02/A-bit-of-music/');

  precacheUrls.push('/2019/06/01/First-Post/');

toolbox.precache(precacheUrls);
toolbox.options = {"networkTimeoutSeconds":5};


toolbox.router.any(/.*\.(js|css|jpg|jpeg|png|gif)$/, toolbox.cacheFirst);

toolbox.router.any(/\//, toolbox.networkFirst);

{% endcodeblock %} 

On peut voir que la page d'accueil, la css et les derniers posts sont automatiquement mis en cache lorsqu'on arrive sur le site. 

On définit ensuite grace à toolbox les strategies de caching pour chaque type de route : 
* cacheFirst
* networkFirst 
* networkOnly

**NB:sw-toolbox qui est utilisé ici par le plugin [hexo-pwa][hexo-pwa-plugin] est deprecated en faveur de [Workbox][workbox]** 

Reste à enregistrer votre service worker dans votre html :
{% codeblock lang:html %}
<script>if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js?t=1559657386669')
    .then(function () {console.log('ServiceWorker Register Successfully.')})
    .catch(function (e) {console.error(e)});
}
</script>
{% endcodeblock %}

Et voilà, l'app devrait désormais fonctionner en mode offline et la pop-up qui propose l'installation devrait apparaitre. 
Vous pouvez vous convaincre du bon fonctionnement en fermant toutes vos app, en passant en mode avion puis en ouvrant votre PWA. 
Pour valider votre mise en place, utilisez Lighthouse dans Chrome (F12 puis onglet _Audit_) 

Naviguer sur les pages en cache sera donc **complètement instantané** après la première page, difficile de faire mieux comme expérience. 
Autre avantage, si votre site devait rencontrer des problèmes de disponibilité, vos visiteurs auront quand meme une page d'accueil le temps que les problèmes soient résolus. 

## Conclusion : PWA, il faut le faire :-)  


## Quelques ressources 
* [Un plugin Hexo][hexo-pwa-plugin] utilisé ici 
* [Un plugin Wordpress][wp-pwa] 
* [Un module Drupal][drupal-pwa]
* [La doc de Google sur les Services Workers][google-pwa-doc] 
* La présentation faite à la Google I/O, qui explique tous les concepts qu'on a abordés ici et les illustre de pas mal d'exemples concrets d'intégration réussies (HULU, Spotify) :
{% oembed https://www.youtube.com/watch?v=2KhRmFHLuhE 600 %}
* Vidéo également intéressante, explique la création de [Proxx][proxx]; un démineur codé comme PWA qui fonctionne sur tous les devices, notament les feature phone à touches : 
{% oembed https://www.youtube.com/watch?v=w8P5HLxcIO4 600 %}

[pwa-logo]: https://user-images.githubusercontent.com/3104648/28351989-7f68389e-6c4b-11e7-9bf2-e9fcd4977e7a.png
[googleio-pwa]: https://www.youtube.com/watch?time_continue=15&v=2KhRmFHLuhE
[googleio-proxx]: https://www.youtube.com/watch?v=w8P5HLxcIO4
[proxx]: https://proxx.app/
[wasm]: https://webassembly.org/
[pwa-req]: https://i.imgur.com/i5FnmN3.png
[manif-gen]: https://app-manifest.firebaseapp.com/
[pwa-cloudj]: https://i.imgur.com/IrjqVb3.jpg
[workbox]: https://developers.google.com/web/tools/workbox/
[hexo-pwa-plugin]: https://github.com/lavas-project/hexo-pwa
[drupal-pwa]: https://www.drupal.org/project/pwa
[wp-pwa]: https://wordpress.org/plugins/super-progressive-web-apps/
[google-pwa-doc]: https://developers.google.com/web/fundamentals/primers/service-workers/
[serverless-blog]: /2019/06/03/Mettre-en-place-un-blog-Serverless-sur-AWS-avec-integration-continue/
