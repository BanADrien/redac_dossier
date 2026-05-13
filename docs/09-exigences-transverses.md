# §9 — Exigences transverses

## 9.1 Sécurité — Analyse STRIDE

### 9.1.1 Flux 1 : Inscription à un événement payant

Ce flux couvre le chemin de l'étudiant depuis le clic "S'inscrire" jusqu'à la confirmation du ticket, impliquant `AuthModule`, `TicketModule`, `PaymentModule`, PostgreSQL, Stripe et RabbitMQ.

#### Diagramme de flux (rappel)

```
Étudiant → Frontend → API Gateway → AuthModule → EventModule → TicketModule → PaymentModule → Stripe
                                                                                            ↓
                                              NotificationWorker ← RabbitMQ ←──────────────┘
```

#### Fiche STRIDE

| Catégorie | Menace | Risque concret | Mesure de défense |
|-----------|--------|----------------|-------------------|
| **S — Spoofing** | Un attaquant usurpe l'identité d'un étudiant pour s'inscrire à sa place | Inscription frauduleuse, paiement imputé à la victime | JWT signé HS256 avec secret serveur, validé à chaque requête par `AuthModule`. Token OIDC vérifié via la clé publique du SSO. |
| **T — Tampering** | Modification du montant du ticket dans la requête `POST /tickets` | Inscription à un prix inférieur | Le montant n'est jamais fourni par le client : il est calculé côté serveur à partir du prix de l'événement en base. La requête client contient uniquement `eventId`. |
| **R — Repudiation** | Un organisateur prétend n'avoir jamais annulé un événement | Litige sans preuve | Toutes les mutations critiques sont journalisées (action, userId, timestamp, état avant/après) via un middleware d'audit. Les logs sont en append-only. |
| **I — Information Disclosure** | Fuite des emails des participants dans les réponses API | Violation RGPD, spam ciblé | Les endpoints organisateur retournent uniquement les champs nécessaires (pas l'email complet en liste). L'export CSV est protégé par `JWT + role:organizer` et limité aux événements de l'organisateur connecté. |
| **D — Denial of Service** | Flood de `POST /tickets` pour épuiser les places d'un événement sans payer | Événement complet sans vrai inscrit | Rate limiting sur `POST /tickets` : 5 req/min par IP + par userId. Les tickets `pending` expirent après 15 min si non confirmés (TTL Redis + job CRON). |
| **E — Elevation of Privilege** | Un étudiant tente d'accéder à `GET /organizer/events` sans être organisateur | Accès non autorisé aux données des organisateurs | RBAC vérifié par `AuthModule` sur chaque route : le JWT contient le rôle, vérifié côté serveur. Le rôle `organizer` n'est accordé qu'après validation admin (`PATCH /admin/organizers/:id`). |

---

### 9.1.2 Flux 2 : Réception du webhook Stripe

Ce flux couvre la réception du `POST /webhooks/stripe` depuis Stripe jusqu'à la mise à jour du statut du ticket et la publication de `ticket.confirmed`. Implique `PaymentModule`, Redis, PostgreSQL et RabbitMQ.

#### Fiche STRIDE

| Catégorie | Menace | Risque concret | Mesure de défense |
|-----------|--------|----------------|-------------------|
| **S — Spoofing** | Un attaquant envoie un faux webhook Stripe pour confirmer un ticket sans payer | Ticket confirmé sans paiement réel | Signature HMAC vérifiée via le secret webhook Stripe (`STRIPE_WEBHOOK_SECRET`) et l'en-tête `Stripe-Signature`. Sans signature valide : rejet 401. |
| **T — Tampering** | Modification du corps du webhook en transit (montant, statut) | Ticket confirmé pour un paiement échoué | Le HMAC SHA-256 couvre le corps entier du webhook. Toute modification invalide la signature → rejet. Le corps brut (Buffer) est utilisé pour la vérification, jamais le JSON parsé. |
| **R — Repudiation** | SupEvents prétend ne pas avoir reçu un webhook Stripe (litige de paiement) | Paiement capturé non créédité en ticket | Chaque webhook est tracé en base avec `stripe_event_id`, timestamp de réception, et statut de traitement. Le dashboard Stripe conserve l'historique côté Stripe. |
| **I — Information Disclosure** | Exposition du secret webhook dans les logs | Compromission du mécanisme d'authentification des webhooks | `STRIPE_WEBHOOK_SECRET` injecté via variable d'environnement (jamais en dur dans le code). Les logs ne journalisent jamais les headers d'authentification bruts. |
| **D — Denial of Service** | Flood de faux webhooks pour saturer l'endpoint | Indisponibilité du service de traitement des paiements | L'endpoint `/webhooks/stripe` rejette immédiatement (401) tout webhook sans signature valide. Rate limiting sur l'IP de Stripe (range connue). |
| **E — Elevation of Privilege** | Rejouer un webhook `payment_intent.succeeded` déjà traité | Double confirmation de ticket, double notification | Idempotence via `stripe_event_id` stocké en Redis (TTL 24h). Un `stripe_event_id` déjà présent en cache → 200 OK sans re-traitement. |

---

## 9.2 Performance — SLO

### Objectifs de niveau de service

| Identifiant | Exigence | Métrique | Seuil | Justification |
|-------------|---------|---------|-------|---------------|
| **ENF-01** | Latence endpoints critiques | P95 temps de réponse | < 500 ms | Standard UX acceptable pour une interaction utilisateur directe |
| **ENF-02** | Disponibilité mensuelle | Uptime | ≥ 99,5 % | ≤ 3h39 d'indisponibilité/mois — acceptable pour un service étudiant non critique 24/7 |
| **ENF-03** | Charge maximale sans dégradation | Requêtes simultanées | 500 req/s | Pic estimé lors de la rentrée universitaire |
| **ENF-04** | Latence Redis (cache/sessions) | P99 temps de réponse | < 10 ms | Redis en mémoire locale — seuil très conservatif |
| **ENF-05** | Throughput RabbitMQ | Messages/seconde | ≥ 200 msg/s | Pic de notifications lors d'un événement populaire complet (annulation) |
| **ENF-06** | Temps de génération du QR code | P95 | < 2 s | Acceptable pour une opération post-paiement non-bloquante |

### Stratégie de performance

- **Connexion pool PostgreSQL** : pool de 20 connexions maximum pour le backend, configurable via `DATABASE_POOL_SIZE`.
- **Cache Redis** : sessions JWT (TTL 1h), résultats de liste d'événements (TTL 5min, invalidé à chaque modification).
- **Pagination obligatoire** : `GET /events` retourne au maximum 50 événements par page, paramètre `limit` plafonné à 100.
- **Compression gzip** : activée sur nginx pour toutes les réponses JSON > 1 Ko.

---

## 9.3 RGPD

### a) Registre des traitements

| Traitement | Données traitées | Base légale | Durée de rétention | Responsable |
|------------|-----------------|-------------|-------------------|-------------|
| Authentification SSO | Email, prénom, nom (fournis par le SSO) | Intérêt légitime (accès service institutionnel) | Durée de la session + 30 jours | SSO École (SupEvents ne stocke qu'une copie) |
| Inscription événement | UserId, EventId, montant payé | Exécution du contrat (achat de billet) | 5 ans (obligation comptable) | SupEvents |
| Notification email | Email du destinataire | Intérêt légitime (confirmation de transaction) | 1 an | SupEvents |
| Tableau de bord organisateur | Liste des participants (prénom, nom, email) | Exécution du contrat (gestion de l'événement) | Durée de l'événement + 3 mois | Organisateur (SupEvents sous-traitant) |
| Logs d'audit | UserId, action, timestamp | Obligation légale / sécurité | 1 an | SupEvents |

### b) Mécanismes de protection

- **Chiffrement en transit** : TLS 1.3 obligatoire sur tous les endpoints (nginx) ; HTTPS forcé via redirection 301.
- **Chiffrement au repos** : les volumes Docker contenant PostgreSQL et MinIO sont sur un système de fichiers chiffré (LUKS sur le VPS).
- **Pseudonymisation** : l'exercice du droit à l'oubli anonymise `email`, `first_name`, `last_name` dans la table `user` sans suppression physique (intégrité référentielle).
- **Minimisation** : l'API ne retourne jamais plus de données que nécessaire (pas d'email complet dans les listes publiques d'inscrits).
- **Séparation des responsabilités** : SupEvents n'a pas accès aux mots de passe (délégation SSO), ni aux données de carte (délégation Stripe).

### c) Procédures liées aux droits des personnes

**Droit d'accès (Art. 15 RGPD)** : endpoint `GET /users/me` retourne toutes les données de l'utilisateur connecté (profil, tickets, notifications). Pour les données hébergées ailleurs (SSO, Stripe), l'utilisateur est renvoyé vers les DPO respectifs.

**Droit à l'effacement (Art. 17 RGPD)** : endpoint `DELETE /users/me`. La procédure pseudonymise les champs personnels de `user` (email → `deleted_[uuid]@anon.local`, prénom/nom → `Anonyme Utilisateur`), conserve les tickets pour obligations comptables (5 ans), et supprime les préférences de notification. Un email de confirmation est envoyé avant suppression via l'adresse à anonymiser.

**Droit de portabilité (Art. 20 RGPD)** : endpoint `GET /users/me/export` retourne un JSON structuré contenant toutes les données personnelles de l'utilisateur (profil, historique des inscriptions, notifications). Délai de traitement : immédiat (pas de traitement batch nécessaire à ce stade).

---

*Dernière mise à jour : 2026-05-13*
