# §4 — Contexte et objectifs

## 4.1 Contexte technique

SupEvents est une plateforme web de gestion d'événements étudiants. Elle doit permettre à des étudiants de découvrir, s'inscrire et payer des événements organisés par d'autres étudiants ou par l'école. Le système doit s'intégrer à l'infrastructure existante de l'établissement (SSO OIDC) et déléguer les paiements à Stripe pour rester hors de portée PCI-DSS.

Du point de vue technique, la contrainte principale est la concurrence : lors des événements populaires (rentrée, gala), jusqu'à 500 étudiants peuvent tenter de s'inscrire simultanément à la même session. Le système doit garantir l'intégrité des places (pas de surbooking), la fiabilité des paiements (un paiement capturé correspond toujours à un ticket confirmé) et la livraison des notifications sans bloquer le flux critique d'inscription.

L'architecture retenue est une API REST découplée d'un frontend React, avec des communications asynchrones via RabbitMQ pour les notifications. La base de données PostgreSQL est choisie pour ses garanties ACID nécessaires à la gestion concurrentielle des places.

## 4.2 Objectifs techniques

| Objectif technique | Origine dans le CDC | Niveau de priorité |
|--------------------|--------------------|--------------------|
| Scalabilité horizontale du backend | "500 utilisateurs simultanés lors de la rentrée" | Élevé |
| Conformité PCI-DSS déléguée à Stripe | "paiement en ligne sécurisé" | Élevé |
| Conformité WCAG 2.1 AA | "accessible aux étudiants en situation de handicap" | Moyen |
| Authentification SSO OIDC sans compte local | "connexion via le portail de l'école" | Élevé |
| Gestion de la concurrence sur les places | "prévenir le surbooking des événements" | Élevé |
| Idempotence des paiements | "ne pas débiter deux fois un étudiant" | Élevé |
| Notifications multi-canaux fiables | "confirmation par email systématique" | Moyen |
| Internationalisation FR/EN | "communauté internationale d'étudiants" | Faible |
| Disponibilité 99,5 % | "service disponible en permanence" | Moyen |

---

*Dernière mise à jour : 2026-05-13*
