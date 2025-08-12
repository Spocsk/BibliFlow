# Bibliflow

Bibliflow est une plateforme complète de gestion de bibliothèque, conçue pour faciliter la gestion, le prêt et la consultation de ressources documentaires. Le projet est structuré en deux parties principales : un backend basé sur NestJS et un frontend développé avec Angular.

## Fonctionnalités principales

- **Gestion des livres** : Ajout, modification, suppression et recherche de livres dans la bibliothèque.
- **Gestion des utilisateurs** : Création de comptes, authentification, gestion des rôles (administrateur, utilisateur, etc.).
- **Gestion des prêts** : Emprunt et retour de livres, suivi des prêts en cours et historiques.
- **Tableau de bord** : Statistiques sur l’utilisation de la bibliothèque, nombre de prêts, livres populaires, etc.
- **Notifications** : Alertes pour les retards de retour, rappels de réservation, etc.
- **Interface moderne** : Application web responsive et intuitive pour les utilisateurs et les administrateurs.

## Architecture du projet

- **bibliflow-backend/** : API RESTful développée avec [NestJS](https://nestjs.com/), connectée à une base de données MongoDB. Elle gère la logique métier, la sécurité, l’authentification et l’accès aux données.
- **bibliflow-frontend/** : Application web développée avec [Angular](https://angular.io/), offrant une interface utilisateur riche et interactive.
- **database/** : Scripts d’initialisation et configuration de la base de données MongoDB.
- **jenkins/** : Fichiers de configuration pour l’intégration et le déploiement continus (CI/CD) avec Jenkins.
- **compose.yml** : Orchestration des services via Docker Compose pour faciliter le développement et le déploiement.

## Prérequis

- [Node.js](https://nodejs.org/) >= 18.x
- [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/)
- [MongoDB](https://www.mongodb.com/)

## Installation rapide

1. **Cloner le dépôt**
   ```bash
   git clone <url-du-repo>
   cd bibliflow
   ```
2. **Lancer les services avec Docker Compose**
   ```bash
   docker compose up --build
   ```
3. **Accéder à l’application**
   - Frontend : http://localhost:4200
   - Backend : http://localhost:3000

## Structure des dossiers

```
bibliflow/
├── bibliflow-backend/    # Backend NestJS
├── bibliflow-frontend/   # Frontend Angular
├── database/             # Scripts et init DB
├── jenkins/              # CI/CD Jenkins
├── compose.yml           # Docker Compose
└── README.md             # Ce fichier
```

## Contribution

Les contributions sont les bienvenues ! Merci de soumettre vos issues ou pull requests pour toute amélioration ou correction.
