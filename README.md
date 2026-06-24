# SentimentAI — DevOps CI/CD Project

Projet réalisé dans le cadre du module DevOps M4.

## Objectif

Mettre en place une chaîne DevOps complète autour d'une API FastAPI de prédiction de sentiment :

- Git, Docker et Docker Compose
- Pipeline Jenkins CI/CD
- Analyse qualité avec SonarQube
- Scan sécurité avec Trivy
- Infrastructure as Code avec Terraform
- Monitoring avec Prometheus et Grafana

## Stack technique

- Python / FastAPI
- Docker / Docker Compose
- Jenkins
- GitHub Container Registry
- SonarQube
- Trivy
- Terraform Docker Provider
- Prometheus
- Grafana

## Pipeline CI/CD

Le pipeline Jenkins exécute les étapes suivantes :

1. Checkout
2. Lint
3. IaC Validate
4. Build & Test
5. SonarQube Analysis
6. Quality Gate
7. Security Scan
8. Push image Docker vers GHCR
9. IaC Apply
10. Deploy Staging
11. Smoke Test

## Registry Docker

Image publiée sur GitHub Container Registry :

```bash
docker pull ghcr.io/satlix/sentiment-ai:latest
