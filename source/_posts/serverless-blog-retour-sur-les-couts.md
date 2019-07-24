---
title: Serverless Blog - Retour sur les coûts 
date: 2019-07-24 18:15:35
tags:
  - AWS
  - IT Costs
category:
  - Architecture
---
![Alors combien ça coute ?][money] 

Si vous avez lu l'[article sur la mise en place de ce blog serverless][blogserverless]; vous savez que l'un des objectif était d'en minimiser les coûts au maximum. 
Après un mois d'exploitation, et la première facture; voici les chiffres concrets. 
<!-- more -->


## Cloudfront; le CDN 

J'ai activé le CDN pour les régions Amérique / Canada / Europe. Je juge qu'il y a peu de chance d'avoir des visiteurs des autres parties du globe et on peut donc accepter un peu de latence pour eux. Si cela devait changer, il est toujours possible d'adapter. 

Pour les USA et le Canada, j'ai eu peu de visites et je suis resté dans le Free Tier, c'est donc gratuit à ce niveau.  

### Détail pour l'Europe : 

> Amazon CloudFront EU-Requests-Tier2-HTTPS $0.01
> $0.0120 per 10,000 HTTPS Requests => **8,837 Requests : $0.01**
> Bandwidth 
> $0.085 per GB - first 10 TB / month data transfer out => **0.119 GB  : $0.01**



## S3, le stockage

> Amazon Simple Storage Service EUW3-Monitoring-Automation-INT
> $0.0025 per 1,000 Objects per month in Intelligent-Tiering  7.467 Objects : $0.00
> Amazon Simple Storage Service EUW3-Requests-INT-Tier1 
> $0.0053 per 1,000 PUT, COPY, POST, or LIST requests to Intelligent-Tiering 8 Requests : $0.00
> Amazon Simple Storage Service EUW3-Requests-INT-Tier2 
> $0.0042 per 10,000 GET and all other requests to Intelligent-Tiering 292 Requests : $0.00
> Amazon Simple Storage Service EUW3-Requests-Tier1 
> $0.0053 per 1,000 PUT, COPY, POST, or LIST requests **2,365 Requests : $0.01**
> Amazon Simple Storage Service EUW3-Requests-Tier2 
> $0.0042 per 10,000 GET and all other requests 6,364 Requests : 0.00
> Amazon Simple Storage Service EUW3-TimedStorage-ByteHrs 
> $0.024 per GB - first 50 TB / month of storage used  0.001 GB-Mo : $0.00
> Amazon Simple Storage Service EUW3-TimedStorage-INT-FA-ByteHrs 
> $0.024 per GB - first 50 TB / month of storage used in Intelligent-Tiering, Frequent Access Tier 0.000129 GB-Mo : $0.00


On peut voir que le seul coût provient des requetes en PUT, COPY, POST, ou LIST. EN somme, cela représente les multiples déploiements que j'ai du faire pour construire le blog. Il étaient nombreux au début mais à moins de mettre en place des nouvelles fonctionnalités, ceci devrait rester gratuit. 
Le CDN, qui met en cache, limite les appels à S3, on ne devrait donc pas voir exploser le nombre de requetes en GET si le traffic devait augmenter d'un coup. 


## Route 53, le DNS 

> Amazon Route 53 DNS-Queries 
> $0.40 per 1,000,000 queries for the first 1 Billion queries 3,694 Queries : $0.00
> Amazon Route 53 HostedZone 
> $0.50 per Hosted Zone for the first 25 Hosted Zones **1 HostedZone : $0.50**
> Amazon Route 53 Intra-AWS-DNS-Queries 
> Queries to Alias records are free of charge 2,736 Queries : $0.00 

C'est le seul réel coût, incompressible, qu'on ne peut pas éviter quand on veut avoir son propre nom de domaine sur AWS. 
 

## That's it 

Pour tous les autres services que j'utilise ( voir [article sur la conception du blog][blogserverless] ), je reste dans le Free Tier. 
En somme, **0,53$/mois** pour un site rapide et qui ne devrait pas souffrir de problème de disponibilité; pas mal, non ? 

## Next : Lambda & FAAS 

J'ai développé un formulaire de contact avec [AWS Lambda][lmbda] ; il fonctionne correctement mais il n'est vraiment pas assez sexy et encore trop générique pour le mettre en ligne. 
Une fois que ce sera prêt; j'en décrirai le fonctionnement et l'intérêt d'utiliser du FAAS ou Function As A Service. 

[money]: https://i.imgur.com/T6VD4Lp.gif
[blogserverless]: https://www.delsoir.com/2019/06/03/Mettre-en-place-un-blog-Serverless-sur-AWS-avec-integration-continue/
[lmbda]: https://aws.amazon.com/lambda
[sems]: https://aws.amazon.com/ses/
