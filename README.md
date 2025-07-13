# FinOPS-APP

Lien du projet GitHub : [https://github.com/Clement-tc/APP\_AUTO/tree/main](https://github.com/Clement-tc/APP_AUTO/tree/main)

## OffreAPI - Dashboard de Recrutement avec Analyse FinOps

### Présentation du projet

OffreAPI est une application web développée en Flask permettant aux recruteurs de publier des offres d'emploi et de visualiser, pour chaque offre, un dashboard de scoring sémantique entre les CV soumis et la description du poste.

Un modèle LLM était initialement prévu pour effectuer le scoring, mais en raison de contraintes de mémoire (OOM sur Kubernetes), une simulation avec `numpy.random.uniform` est utilisée pour reproduire la logique de classement des candidats.

Le projet intègre aussi un monitoring avec Prometheus + Grafana, et une analyse FinOps avec Kubecost.

---

## Stack technique

## Explication des manifests Kubernetes

Le répertoire `k8s/` contient l’ensemble des fichiers de configuration nécessaires au déploiement sur Kubernetes :

* `deployment.yaml` : déploie l'application Flask avec ses ressources (CPU, RAM, etc.)
* `pgadmin-deployment.yaml` : déploiement de pgAdmin pour l’accès à l’administration PostgreSQL
* `postgres-deployment.yaml` : déploiement de la base PostgreSQL
* `postgres-service.yaml` : expose PostgreSQL pour que l’application Flask puisse s’y connecter
* `postgres-config.yaml` : configuration de la base (utilisateur, mot de passe, nom de la base, etc.)
* `secret.yaml` : stockage des secrets sensibles (ex: mots de passe base de données)
* `service-monitoring.yaml` : ServiceMonitor pour que Prometheus scrape les métriques `/metrics` de Flask

Tous les composants sont déployés dans le namespace `offreapi` (sauf Prometheus/Grafana dans `monitoring`).

* Backend : Flask, SQLAlchemy, extraction PDF (PyMuPDF)
* Base de données : PostgreSQL
* Monitoring : Prometheus + Grafana + Flask Exporter
* Analyse FinOps : Kubecost
* Infrastructure : Docker, Kubernetes (fichiers `k8s/*.yaml`)
* CI/CD : Script bash (intégration Jenkins en cours)

---

## Déploiement Kubernetes

Les fichiers de configuration Kubernetes se trouvent dans le dossier `k8s/` et permettent de déployer l'ensemble des composants de l'application.

Voici une vue d'ensemble de leur rôle :
Déploiment Via script Bash (Réflexion sur la mise en place de Jenkins pour faciliter le déploiement.)
Les étape de deploiement sont:
* Pour simplement l'application Flask 
minikube start 
docker build -t elcer/offre-api
docker push elcer/offre-api
kubectl create namespace offreapi
kubectl apply -f k8s/
kubectl get pods -n offreapi 

* `deployment.yaml` : déploie l'application Flask dans un pod avec les ressources (requests/limits) pour l'analyse FinOps
* `service-monitoring.yaml` : expose l'endpoint `/metrics` de l'API Flask via un ServiceMonitor pour Prometheus
* `pgadmin-deployment.yaml` : déploiement de pgAdmin pour visualiser la base PostgreSQL
* `postgres-deployment.yaml` : pod PostgreSQL avec PVC et configuration de persistance
* `postgres-service.yaml` : expose PostgreSQL pour l'application Flask
* `postgres-config.yaml` : contient les variables d'environnement PostgreSQL (nom base, user, etc.)
* `secret.yaml` : stocke les informations sensibles (utilisateur / mot de passe de la base)

Chaque composant est déployé dans un namespace dédié (`offreapi`), avec un suivi des ressources pour faciliter l'audit FinOps.

---

## Dashboards de monitoring Grafana

Voici à quoi ressemble un dashboard Grafana sur notre cluster :

### Avant afflux de trafic :

![Dashboard avant afflux](captures/Avant_requests.png)

Cette capture montre l'évolution de plusieurs indicateurs clés sur l'application :

* Latence HTTP moyenne (extraites via Flask Exporter)
* Utilisation CPU et RAM de l'API Flask
* Suivi des ressources PostgreSQL
* Charges observées sur pgAdmin

### Après afflux de trafic :

![Dashboard après afflux](captures/Apres_requests.png)

Cette vue permet de comparer les variations en charge lors d'une utilisation intensive de la plateforme.

---

## Analyse FinOps avec Kubecost

![Dashboard Kubecost](captures/Kubecost.png)

Kubecost nous permet d'estimer le coût simulé des composants, d'identifier les surprovisions et de proposer des ajustements. Le modèle de coût utilisé est personnalisé avec :

* CPU : 0.02 € / vCPU / heure
* RAM : 0.01 € / Go / heure
* Stockage : 0.03 € / Go / mois

L'analyse s'appuie sur les metrics Prometheus, les requests/limits Kubernetes et l'observation en conditions de charge.

---

## Fonctionnalités principales

* Authentification utilisateur (recruteur / candidat)
* Publication d'offres par les recruteurs
* Soumission de CV au format PDF par les candidats
* Extraction de texte automatique depuis les PDF
* Simulation de scoring sémantique via analyse de texte
* Dashboard par offre (classement des candidats + graphique)
* Monitoring des performances via Grafana
* Suivi de coûts et efficacité via Kubecost
