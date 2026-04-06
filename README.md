# Configuration Repository — Teranga Biz Platform

Ce dépôt contient les configurations centralisées pour tous les microservices de la plateforme Teranga Biz.
Il est chargé au démarrage par chaque service via le **Config Server** (Spring Cloud Config).

## Structure des fichiers

```
configurations-teranga-biz/
├── application.yml           # Configuration globale (tous les services)
├── config-server.yml         # Config Server (pointe vers ce dépôt Git)
├── eureka-server.yml         # Service Discovery (Eureka)
├── api-gateway.yml           # API Gateway + routage + sécurité JWT
├── utilisateur-service.yml   # MS-01 — Utilisateurs & Authentification
├── produits-service.yml      # MS-02 — Produits & Catalogue
├── boutique-service.yml      # MS-03 — Boutiques personnalisées
└── README.md                 # Ce fichier
```

## Services configurés

| Service | Fichier | Port | Description |
|---------|---------|------|-------------|
| Config Server | `config-server.yml` | 8888 | Distribution centralisée des configurations |
| Eureka Server | `eureka-server.yml` | 8761 | Registre et découverte de services |
| API Gateway | `api-gateway.yml` | 8080 | Point d'entrée unique, routage, sécurité JWT |
| MS-01 Utilisateurs | `utilisateur-service.yml` | 8081 | Comptes, authentification, rôles, profils |
| MS-02 Produits | `produits-service.yml` | 8082 | Catalogue, catégories, recherche Elasticsearch |
| MS-03 Boutiques | `boutique-service.yml` | 8083 | Mini-boutiques personnalisées des vendeurs |

## Plan de déploiement (phases)

### Phase 1 — MVP (en cours)
- [x] MS-01 Utilisateurs & Authentification
- [x] MS-02 Produits & Catalogue
- [x] MS-03 Boutiques
- [ ] MS-04 Inventaire
- [ ] MS-05 Commandes
- [ ] MS-06 Paiements
- [ ] MS-12 Avis & Évaluations

### Phase 2 — Social & Académie
- [ ] MS-07 Social
- [ ] MS-08 Messagerie
- [ ] MS-09 Académie
- [ ] MS-13 Certification & Gamification
- [ ] MS-14 Intelligence Artificielle (basique)

### Phase 3 — Avancé
- [ ] MS-10 Partenaires
- [ ] MS-11 Publicité
- [ ] MS-14 Intelligence Artificielle (avancé)
- [ ] MS-15 Statistiques & Analyses

## Comment ça fonctionne

### Chargement au démarrage

Chaque microservice charge sa configuration dans cet ordre :
1. `application.yml` — paramètres communs (Eureka, Actuator, logs)
2. `{nom-du-service}.yml` — paramètres spécifiques au service

```yaml
# Dans chaque microservice (application.yml local)
spring:
  application:
    name: boutique-service   # ← détermine quel fichier est chargé ici
  config:
    import: optional:configserver:http://config-server:8888
```

### Mettre à jour une configuration

```bash
git add boutique-service.yml
git commit -m "config: mise à jour boutique-service"
git push origin main
# Les services rechargent sans rebuild
```

### Rafraîchissement sans redémarrage (si @RefreshScope activé)

```bash
curl -X POST http://localhost:8083/actuator/refresh
```

## Ajouter un nouveau microservice

1. Créer le fichier `{nom-service}.yml` dans ce dépôt
2. Ajouter la route dans `api-gateway.yml` (section `routes`)
3. Si le service a des endpoints publics, les ajouter dans `routes-publiques`
4. Pousser sur GitHub — le Config Server distribue automatiquement

## Variables d'environnement sensibles

Les secrets ne sont **jamais** stockés dans ce dépôt. Ils sont injectés via des variables d'environnement dans `docker-compose.yml` ou Kubernetes Secrets.

| Variable | Description | Défaut (dev) |
|----------|-------------|--------------|
| `JWT_SECRET` | Clé secrète JWT — doit être identique sur gateway + utilisateur-service | clé de dev |
| `BDD_UTILISATEUR` | Utilisateur PostgreSQL | `postgres` |
| `BDD_MOT_DE_PASSE` | Mot de passe PostgreSQL | `postgres` |
| `SMTP_PASSWORD` | Clé API SendGrid / SMTP | vide |
| `TWILIO_COMPTE_SID` | Compte Twilio (SMS OTP) | vide |
| `TWILIO_JETON_AUTH` | Token Twilio | vide |
| `OTP_FOURNISSEUR` | Mode OTP : `console` (dev) ou `twilio` / `email` (prod) | `console` |

## Sécurité

- Ne jamais commiter de secrets ou mots de passe dans ce dépôt
- En production : utiliser Kubernetes Secrets, AWS Secrets Manager ou HashiCorp Vault
- La clé JWT doit être identique sur `api-gateway` et `utilisateur-service`

## Support

- Repository principal : https://github.com/AIMEROU-DIA/plateforme-teranga-biz
- Configuration : https://github.com/AIMEROU-DIA/configurations-teranga-biz

---

**Dernière mise à jour** : 2026-04-06
**Version** : 1.0.0-MVP
