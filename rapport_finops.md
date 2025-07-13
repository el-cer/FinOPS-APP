# Rapport FinOps – OffreAPI

## 1. Objectif

Ce rapport présente l'analyse des coûts et l'efficacité des ressources du projet **OffreAPI**, dans le cadre d’un déploiement Kubernetes avec surveillance Prometheus/Grafana et estimation FinOps via Kubecost.

---

## 2. Méthodologie

- **Métriques collectées** : CPU / RAM usage, requests, limits (Prometheus)
- **Visualisation** : Dashboards Grafana personnalisés (latence, consommation)
- **Estimation de coût** : via Kubecost + modèle de tarification personnalisé
- **Simulation de trafic** : afflux d’utilisateurs simulé manuellement
- **Période d’observation** : 30 minutes avant/après pic d’activité

---

## 3. Modèle de coût personnalisé

| Ressource | Coût simulé |
|-----------|-------------|
| CPU       | 0.02 €/vCPU/heure |
| RAM       | 0.01 €/Go/heure   |
| Stockage  | 0.03 €/Go/mois    |

Appliqué dans Kubecost (`Settings > Pricing configuration`).

---

## 4. Résultats observés (Kubecost)

| Composant   | Coût simulé mensuel | Efficiency CPU | Efficiency RAM |
|-------------|----------------------|----------------|----------------|
| `offre-api` | ~11 €                | 14 %           | 22 %           |
| `postgres`  | ~6 €                 | 40 %           | 60 %           |
| `pgadmin`   | ~4 €                 | 70 %           | 80 %           |
| `__idle__`  | ~2 €                 | —              | —              |

> **Surprovision détectée** sur `offre-api` (requests CPU trop élevés, mémoire inutilisée).

---

## 5. Recommandations d’optimisation

| Composant   | Action recommandée              | Avant          | Après         | Gain        |
|-------------|----------------------------------|----------------|---------------|-------------|
| `offre-api` | CPU 1 → 0.5 vCPU, RAM 512Mi → 256Mi | 11 €/mois      | 5.5 €/mois    | ~50 %       |
| `postgres`  | RAM 1Go → 512Mi                  | 6 €/mois       | 3.2 €/mois    | ~47 %       |
| `pgadmin`   | Changer image → Alpine Slim      | 4 €/mois       | 2 €/mois      | ~50 %       |
| Service API | NodePort → ClusterIP             | 2 €/mois       | 0.5 €/mois    | ~1.5 €      |

---

## 6. Suivi graphique (Grafana)

### Avant pic de trafic :

- Latence stable ~120ms
- CPU/RAM utilisés < 20 %
- Coût CPU/min ~0.003 €

📸 `captures/Avant_requests.png`

---

### Après afflux d’utilisateurs :

- Latence montée à ~450ms
- CPU usage stabilisé à 60 %
- Coût CPU/min ~0.01 €

📸 `captures/Apres_requests.png`

---

## 7. Conclusion

Grâce à l’intégration de Kubecost, nous avons pu :

- Identifier une surprovision importante sur certains services
- Proposer des actions concrètes (requests, images, services)
- Visualiser l’impact d’un pic de charge sur la consommation
- Simuler un coût total optimisé de -45 % en moyenne

Ce travail sera intégré à la CI/CD pour ajustement automatique des ressources à terme.
