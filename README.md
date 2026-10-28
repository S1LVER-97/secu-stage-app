# DevSecOps Pipeline — Trivy · Harbor · Kubernetes (k3d)

Pipeline DevSecOps complet automatisant la construction, le scan de sécurité,
le déploiement et la supervision continue d'images Docker au sein d'un
cluster Kubernetes.

![CI](https://github.com/S1LVER-97/secu-stage-app/actions/workflows/build-push.yml/badge.svg)

## Ce que fait ce projet

À chaque push, la chaîne suivante s'exécute automatiquement, sans
intervention manuelle :

```
Push GitHub → Build & Push (GitHub Actions) → Scan Harbor/Trivy
→ Déploiement Kubernetes → Scan continu (Trivy Operator)
→ Visualisation (Prometheus + Grafana)
```

## Stack technique

| Catégorie | Technologies |
|---|---|
| Conteneurisation | Docker |
| Orchestration | Kubernetes (k3d/k3s), Helm |
| Registre & scan de sécurité | Harbor, Trivy, Trivy Operator |
| Intégration continue | GitHub Actions (runner self-hosted) |
| Supervision | Prometheus, Grafana (kube-prometheus-stack) |
| Base de données | PostgreSQL |
| Infrastructure | Microsoft Azure (VM Ubuntu 24.04) |

## Points forts du projet

- **Registre sécurisé** avec scan automatique de vulnérabilités à chaque push (Harbor + Trivy)
- **Pipeline CI/CD** entièrement automatisé avec authentification par compte de service à privilèges limités
- **Scan continu** de l'ensemble du cluster (pas seulement au moment du build) via Trivy Operator
- **Observabilité** avec tableau de bord Grafana dédié aux vulnérabilités
- **Diagnostic rigoureux** de deux blocages d'infrastructure majeurs (voir plus bas) — la partie la plus formatrice du projet

## Les deux défis techniques résolus

### 1. Limitation réseau de virtualisation
Le scan de sécurité échouait systématiquement au téléchargement de sa base
de données. Après avoir écarté Harbor, Kubernetes et Trivy un par un, la
cause s'est révélée être le **mode réseau de la VM de développement**
(Bridge vs NAT) — un écart de débit mesuré de 15 Ko/s à 640 Ko/s après
correction (facteur ×40).

### 2. Saturation de ressources lors du scan continu
L'activation du scan continu sur tout le cluster a saturé le CPU de
l'environnement de test (load average à 15 sur 4 vCPU), provoquant une
instabilité générale. Décision prise : migration vers une infrastructure
cloud (Azure) avec des ressources dédiées.

*Le détail complet de la démarche de diagnostic est disponible dans
[`docs/rapport-stage.pdf`](docs/rapport-stage.pdf).*

## Architecture

![Architecture](docs/architecture.png)

## Structure du dépôt

```
├── app/                    # Application de démonstration + Dockerfile
├── .github/workflows/      # Pipeline CI/CD GitHub Actions
├── k8s/                    # Manifestes de déploiement Kubernetes
├── helm-values/            # Configuration Helm (Harbor, registres)
└── docs/                   # Rapport complet, architecture, captures d'écran
```

## Résultats

- 774 vulnérabilités détectées sur l'image de démonstration, dont 433 corrigeables
- Pipeline complet validé de bout en bout en démonstration live
- Scan continu opérationnel sur l'ensemble du cluster

## Documentation complète

Le rapport de stage complet (contexte, méthodologie, conception,
démarche de diagnostic détaillée) est disponible dans
[`docs/rapport-stage.pdf`](docs/rapport-stage.pdf).

## Auteur

**Abdessamad EL KAHIRI** — Ingénierie Informatique et Réseaux, EMSI Tanger
