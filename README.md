# FinOPS-APP

## OffreAPI - Dashboard de Recrutement avec Analyse FinOps

### ✨ Présentation du projet

OffreAPI est une application web développée en Flask permettant aux recruteurs de publier des offres d'emploi et de visualiser, pour chaque offre, un **dashboard de scoring sémantique** entre les CV soumis et la description du poste.

Un modèle LLM était initialement prévu pour effectuer le scoring, mais en raison de contraintes de mémoire (**OOM sur K8s**), une simulation avec `numpy.random.uniform` est utilisée pour refléter la logique de classement des candidats.

Le projet intègre un monitoring complet **Prometheus + Grafana**, ainsi qu'une **analyse de coûts avec Kubecost**, dans une démarche FinOps.

---

## Stack technique

* **Backend** : Flask, SQLAlchemy, PDF extraction (PyMuPDF)
* **Base de données** : PostgreSQL
* **Monitoring** : Prometheus + Grafana + Flask Exporter
* **Analyse FinOps** : Kubecost
* **Infrastructure** : Docker, Kubernetes (via `k8s/*.yaml`)
* **CI/CD** : Script bash (Jenkins en développement)

---

## Dashboards de monitoring Grafana

Voici à quoi ressemble un dashboard de monitoring Grafana sur notre cluster :

### Avant afflux de trafic :

![Dashboard avant afflux](captures/Avant_requests.png)
Cette capture montre l'évolution par graphe du dashboard **Offre API - CV**, avec plusieurs graphiques :

* Latence HTTP moyenne de notre API Flask
* Utilisation CPU/RAM de notre API
* CPU/RAM de PostgreSQL
* CPU/RAM de pgAdmin

### Après afflux de trafic :

![Dashboard après afflux](captures/Apres_requests.png)
Cette capture montre le même dashboard **après afflux sur le site**, illustrant l'impact sur les ressources du cluster.

---

## Analyse FinOps via Kubecost

![Dashboard Kubecost](captures/Kubecost.png)

Voici un dashboard de notre visualisation des coûts simulés avec le modèle personnalisé (CPU, RAM, stockage). Nous avons relevé les coûts par pod et identifié les surprovisions.

---

## Fonctionnalités principales

* Authentification (recruteur / candidat)
* Publication d'offres d'emploi par les recruteurs
* Soumission de CV par les candidats (PDF)
* Extraction automatique du texte
* Scoring simulé via similarité texte
* Dashboard par offre avec graphique des scores
* Monitoring Prometheus
* Visualisation FinOps Kubecost
