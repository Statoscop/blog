Title: DevOps : comprendre l'essentiel  
Author: Louis
Date: '2025-06-24'
Category: Dev
Tags: DevOps, DevWeb, automatisation, versionning, codes, scripts
Cover: images/cover_25.png
twitter_image: images/cover_25.png
Summary: Les concepts et outils cachés derrière le DevOps

[TOC]  


Plus qu'un simple buzzword technique, le **DevOps** est une manière de travailler, un ensemble d'outils et de pratiques pour les entreprises qui veulent livrer/déployer plus rapidement, plus souvent et avec plus de fiabilité. Si vous vous interrogez sur ce qui se cache derrière ce mot-valise, cet article est une introduction aux grands principes et outils du **DevOps**.


# DevOps : de quoi parle-t-on ?

Avant tout, le **DevOps** est une **démarche** qui vise à rapprocher les équipes de **développement (Dev)** et d'**exploitation (Ops)**. L'objectif ? Briser les silos organisationnels, fluidifier la collaboration, et automatiser tout ce qui peut l'être avec pour bénéfices directs une **accélération des déploiements** et une **réduction du _time-to-market_** (le temps entre la conception d'une fonctionnalité et sa mise à disposition aux utilisateurs)

La mise en place de méthodes **DevOps** doit permettre de :

- Améliorer la **confiance entre les équipes**  
- Accélérer les **livraisons**  
- Résoudre plus vite les **incidents**  
- Mieux gérer les **urgences** et les imprévus  

L'adoption du **DevOps** transforme non seulement les processus, mais aussi la **culture d'entreprise**.


# Les origines du DevOps

Le terme **DevOps** est apparu autour de **2008**, dans un contexte où les équipes de développement et d'exploitation poursuivaient des objectifs souvent contradictoires avec d'un côté, les **devs** qui veulent livrer rapidement des (nouvelles) fonctionnalités et de l'autre côté, les **ops** cherchent à garantir la stabilité et la disponibilité.

Ce conflit, souvent appelé **"le mur de la confusion"**, rendait les mises en production longues, risquées et stressantes.

Le **DevOps** est né comme une réponse à cette problématique : un **modèle collaboratif**, basé sur l'**automatisation des processus**, l'**intégration continue**, et le **déploiement continu**.


> 👋 Nous c'est Antoine et Louis de Statoscop, une coopérative de statisticiens / data scientists.
> Vous cherchez un prestataire pour vous accompagner dans votre migration vers R?
<div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>   

# Le DevOps en entreprise

Entre une méthodologie théorique et son application au sein d'une organisation, petite ou grande, la marche peut sembler haute et il peut donc être rassurant de voir comment cela a été mis en place ailleurs.

## Les approches DevOps

Il n'y a pas une seule bonne manière d'implémenter **DevOps**. L'organisation, la culture et les contraintes techniques dictent souvent l'approche. Certaines configurations sont efficaces, d'autres sont à éviter. Allez donc visiter le site [DevOps Topologies](https://web.devopstopologies.com/) qui propose des **patterns** et **anti-patterns** d'implémentation **DevOps**.

## Le modèle CALMS

Le modèle **CALMS** est souvent utilisé pour évaluer la maturité **DevOps** d'une organisation. Il repose sur 5 piliers :

- **Culture** : Dev et Ops collaborent autour d'objectifs communs.
- **Automation** : tout ce qui peut être automatisé doit l'être.
- **Lean** : éliminer les gaspillages, optimiser les flux de travail.
- **Mesure** : mesurer pour comprendre et améliorer.
- **Sharing** : partager les responsabilités, les réussites… et les échecs.

## Des exemples d'entreprises 

Un petit tableau récapitulatif des géants du numérique qui sont, bien évidemment, passé à l'approche **DevOps**.

<table>
<thead>
<tr>
<th><strong>Entreprise</strong></th>
<th><strong>Création</strong></th>
<th><strong>Début</strong></th>
<th class="break-word"><strong>Avant</strong></th>
<th class="break-word"><strong>Après</strong></th>
<th><strong>Résultat</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Amazon</strong></td>
<td>1994</td>
<td>~2011</td>
<td class="break-word">Déploiements manuels, très longs et risqués. Problèmes fréquents de coordination entre développeurs et équipes opérationnelles. Faible fréquence de mise en production</td>
<td class="break-word">Automatisation complète des déploiements (CI/CD). Des milliers de déploiements par jour. Forte amélioration de la résilience, de la vitesse de livraison et du retour client</td>
<td class="break-word">1 déploiement toutes les <strong>11,6 secondes</strong></td>
</tr>
<tr>
<td><strong>Netflix</strong></td>
<td>1997</td>
<td>~2009</td>
<td class="break-word">Monolithes complexes. Dépendance à une infrastructure physique. Temps d'indisponibilité importants lors de bugs ou d'erreurs de déploiement</td>
<td class="break-word">Passage au cloud et microservices. Automatisation poussée avec Chaos Monkey. Culture DevOps forte : "You build it, you run it"</td>
<td class="break-word">Disponibilité quasi <strong>totale</strong>, déploiements rapides</td>
</tr>
<tr>
<td><strong>Etsy</strong></td>
<td>2005</td>
<td>~2009</td>
<td class="break-word">Déploiements peu fréquents, manuels et risqués. Environnements de test et de production très différents. Développeurs peu impliqués dans la gestion de production</td>
<td class="break-word">Mise en place de pipelines CI/CD automatisés. Culture de l'amélioration continue et de la transparence. Déploiements plusieurs fois par jour sans interruption de service</td>
<td class="break-word">50+ déploiements/jour, <strong>MTTR réduit de plusieurs heures à &lt; 4 minutes</strong></td>
</tr>
<tr>
<td><strong>Facebook</strong></td>
<td>2004</td>
<td>~2010</td>
<td class="break-word">Croissance rapide qui rendait difficile la gestion manuelle des infrastructures. Intégration et déploiement complexes à l'échelle</td>
<td class="break-word">Automatisation avancée. Chaque commit peut aller en production rapidement. Culture "move fast and break things"</td>
<td class="break-word"><strong>Centaines de déploiements/jour</strong></td>
</tr>
</tbody>
</table>

Le **MTTR** est le **Mean Time to Recovery** : il correspond au temps moyen nécessaire à la restauration d'un service.

# Les outils DevOps

Au delà d'une simple approche méthodologique, le DevOps ne fonctionne pas sans outillage adapté. Voici une liste, loin d'être exhaustive, des outils les plus utilisés :

<table>
<thead>
<tr>
<th><strong>Domaine</strong></th>
<th class="break-word"><strong>Définition</strong></th>
<th><strong>Exemples d'outils</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>CI/CD</strong></td>
<td class="break-word">Intégration et déploiement continus : automatisation du cycle de vie de développement, de l'intégration du code jusqu'à la mise en production</td>
<td>Jenkins, GitLab CI, GitHub Actions</td>
</tr>
<tr>
<td><strong>Conteneurisation</strong></td>
<td class="break-word">Technique de *packaging* d'une application avec ses dépendances pour garantir son exécution sur tout environnement</td>
<td>Docker</td>
</tr>
<tr>
<td><strong>Orchestration</strong></td>
<td class="break-word">Gestion automatisée du déploiement, de la mise à l'échelle et du fonctionnement de conteneurs dans un cluster</td>
<td>Kubernetes</td>
</tr>
<tr>
<td><strong>Versioning</strong></td>
<td class="break-word">Suivi et gestion des modifications du code source dans le temps, avec historique et collaboration</td>
<td>Git</td>
</tr>
<tr>
<td><strong>Automatisation / Configuration</strong></td>
<td class="break-word">Outils pour déployer et configurer automatiquement des infrastructures et des environnements</td>
<td>Ansible, Terraform</td>
</tr>
<tr>
<td><strong>Monitoring &amp; Observabilité</strong></td>
<td class="break-word">Surveillance des performances, collecte de métriques, visualisation des logs et analyse du comportement en temps réel</td>
<td>Prometheus, Grafana, ELK Stack</td>
</tr>
<tr>
<td><strong>Sécurité / Secrets</strong></td>
<td class="break-word">Gestion sécurisée des identifiants, mots de passe, clés API et autres données sensibles utilisées</td>
<td>HashiCorp Vault</td>
</tr>
</tbody>
</table>

Chaque entreprise fait ses choix en fonction de son environnement technique et de ses priorités.


# Le MLOps

Si DevOps optimise le **cycle de vie des applications**, **MLOps** (pour Machine Learning Operations) s'intéresse à la **mise en production des modèles de machine learning**. Il en reprend les grands principes (CI/CD, monitoring, collaboration,...) tout en y ajoutant les spécificités de la data science :

- Suivi des données d'entraînement
- Versioning des modèles
- Tests sur la dérive des performances
- Automatisation du retraining
- Déploiement multi-environnements (batch, temps réel)

**MLOps** est aujourd'hui un complément naturel au **DevOps** pour les organisations orientées **data-science**.


# Conclusion  

Adopter DevOps, ce n'est pas installer Jenkins ou Docker du jour au lendemain. C'est amorcer une **transformation culturelle**, **technique** et **humaine**. C'est une manière de penser, de collaborer, et de livrer en continu un contenu de qualité.   
Quoiqu'il en soit, si cet article vous a donné envie de réagir ou d'en savoir plus, vous pouvez [nous contacter sur le site de Statoscop](https://www.statoscop.fr/contact) pour discuter des prochaines étapes!   

 <div class = "d-flex justify-content-center mt-4">
   <a href="https://statoscop.fr" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Visiter notre site</a>
   <a href="https://statoscop.fr/contact" target=_blank class="btn btn-primary btn-custom text-uppercase" type="button">Nous contacter</a>
</div>
<br>

