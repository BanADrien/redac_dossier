# §5 — Périmètre et limites

## 5.1 Périmètre fonctionnel

| Dans le périmètre | Hors périmètre |
|-------------------|----------------|
| Création et gestion d'événements par les organisateurs | Application mobile native (iOS/Android) |
| Inscription avec paiement en ligne (Stripe) | Système de billetterie physique (impression) |
| Authentification via SSO de l'école (OIDC) | Gestion d'identités locale (création de compte SupEvents) |
| Génération et validation de tickets QR code | Intégration avec des plateformes tierces (Eventbrite, etc.) |
| Tableau de bord organisateur (stats, export CSV) | Streaming live des événements |
| Notifications email transactionnelles (SendGrid) | Notifications push mobiles natives |
| Espace d'administration (validation d'organisateurs) | Revente de billets entre étudiants |
| Liste d'attente pour les événements complets | Gestion de la billetterie multi-sessions complexes |
| Système de tarification earlybird | Programmes de fidélité ou points de récompense |

## 5.2 Limites techniques

| Limite technique | Justification |
|------------------|---------------|
| **Aucune donnée de paiement stockée localement** | SupEvents délègue la conformité PCI-DSS à Stripe. Aucun numéro de carte, CVV ou token de carte ne transite ou n'est stocké par nos serveurs. |
| **Pas de support d'Internet Explorer 11** | Navigateur en fin de vie depuis juin 2022. La cible utilisateur (étudiants) utilise des navigateurs modernes (Chrome, Firefox, Edge ≥ 90). |
| **Migration depuis un système existant : hors périmètre** | SupEvents est construit from scratch. Aucune base de données ou système événementiel préexistant n'a été identifié dans l'établissement. |
| **Pas de haute disponibilité multi-région** | Le budget contraint impose un déploiement single-region sur un VPS. La cible de 99,5 % de disponibilité est atteignable sans redondance géographique. |
| **Paiements uniquement en euros (EUR)** | La plateforme cible exclusivement des établissements français. Le support multi-devises est explicitement exclu pour simplifier la conformité fiscale. |
| **Capacité maximale d'un événement : 10 000 participants** | Au-delà, les contraintes de concurrence sur la gestion des places nécessiteraient une architecture distribuée hors périmètre du projet. |

---

*Dernière mise à jour : 2026-05-13*
