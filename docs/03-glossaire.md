# Glossaire

| Terme | Définition | Première occurrence |
|-------|-----------|---------------------|
| **SSO** | Single Sign-On — mécanisme permettant à un étudiant de s'authentifier une seule fois via le portail de l'école pour accéder à SupEvents sans recréer de compte. | §4 |
| **OIDC** | OpenID Connect — protocole d'authentification basé sur OAuth 2.0, utilisé par le SSO de l'école pour émettre des tokens d'identité vérifiables par SupEvents. | §6.1 |
| **RBAC** | Role-Based Access Control — modèle d'autorisation de SupEvents attribuant des permissions selon le rôle : `étudiant`, `organisateur`, `administrateur`. | §7 |
| **RGPD** | Règlement Général sur la Protection des Données — cadre légal européen imposant à SupEvents de protéger les données personnelles des étudiants (email, historique d'inscriptions). | §9 |
| **PCI-DSS** | Payment Card Industry Data Security Standard — norme de sécurité des paiements. SupEvents délègue la conformité PCI-DSS à Stripe et ne stocke aucune donnée de carte. | §6.4 |
| **Webhook** | Appel HTTP sortant déclenché par un événement tiers. SupEvents reçoit un webhook de Stripe lors de la confirmation ou de l'échec d'un paiement. | §8 |
| **Broker de messages** | Composant (RabbitMQ) qui découple les producteurs d'événements (TicketModule, PaymentModule) de leurs consommateurs (NotificationModule), garantissant la livraison asynchrone. | §6.1 |
| **API REST** | Interface de programmation respectant les contraintes REST (HTTP, ressources, stateless). SupEvents expose ses fonctionnalités via une API REST versionnée `/api/v1/`. | §8 |
| **Earlybird (billet)** | Tarif réduit proposé aux premiers inscrits à un événement SupEvents, géré par une règle de pricing dans EventModule avant épuisement du quota earlybird. | §7 |
| **CRUD** | Create, Read, Update, Delete — quatre opérations de base sur les ressources. Les endpoints REST de SupEvents couvrent le CRUD pour les entités Event, Ticket et User. | §8 |
| **SLA** | Service Level Agreement — engagement de disponibilité. SupEvents cible 99,5 % de disponibilité mensuelle, soit moins de 3h39 d'indisponibilité tolérable par mois. | §9 |
| **P95** | Percentile 95 — seuil en dessous duquel se situent 95 % des temps de réponse. L'exigence ENF-01 fixe p95 < 500 ms pour les endpoints critiques de SupEvents. | §9 |
| **WCAG** | Web Content Accessibility Guidelines — standard d'accessibilité web. SupEvents vise le niveau AA pour être accessible aux étudiants en situation de handicap. | §5 |
| **i18n** | Internationalisation — capacité de SupEvents à afficher son interface en français (par défaut) et en anglais, sans modification du code. | §5 |
| **CI/CD** | Continuous Integration / Continuous Deployment — pipeline automatisé qui valide chaque commit (tests, lint, build) et déploie la DCT ou l'application en production. | §11 |
| **JWT** | JSON Web Token — jeton signé émis par le SSO après authentification OIDC, transporté dans l'en-tête `Authorization: Bearer` pour prouver l'identité de l'appelant. | §6.2 |
| **QR Code** | Code bidimensionnel généré par TicketModule, encodant l'identifiant unique du ticket signé. Scanné à l'entrée de l'événement pour valider la présence. | §7 |
| **Dead Letter Queue (DLQ)** | File RabbitMQ recevant les messages qu'un consommateur n'a pas pu traiter après N tentatives. Permet d'inspecter et rejouer les notifications échouées. | §7 |
| **PaymentIntent** | Objet Stripe représentant l'intention de paiement d'un étudiant. SupEvents crée un PaymentIntent et suit son cycle de vie (created → processing → succeeded / failed). | §7 |
| **Idempotence** | Propriété d'une opération produisant le même résultat quelle que soit la répétition. SupEvents garantit l'idempotence des webhooks Stripe via déduplication par `event.id`. | §7 |
| **STRIDE** | Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege — modèle d'analyse des menaces de sécurité appliqué aux flux critiques de SupEvents. | §9 |
| **SLO** | Service Level Objective — objectif chiffré de qualité de service (p95, disponibilité) que SupEvents s'engage à respecter en production. | §9 |

---

*Dernière mise à jour : 2026-05-13*
