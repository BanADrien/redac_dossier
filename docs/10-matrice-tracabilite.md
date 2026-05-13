# §10 — Matrice de traçabilité

La matrice de traçabilité ci-dessous croise l'ensemble des exigences du cahier des charges SupEvents avec les éléments de la DCT qui les implémentent ou les documentent. Elle permet à un auditeur externe de vérifier en 30 minutes que toutes les exigences sont couvertes et traitées dans la documentation technique.

**Méthodologie** : chaque exigence est identifiée par un code (EF = Exigence Fonctionnelle, ENF = Exigence Non-Fonctionnelle). La colonne "Couverture DCT" pointe vers la section ou le module qui l'implémente. La colonne "Statut" indique si la couverture est complète (✅), partielle (⚠️) ou à traiter (❌).

## 10.1 Matrice principale

### Exigences Fonctionnelles (EF)

| ID | Exigence | Module(s) | Section DCT | ADR | Statut |
|----|---------|-----------|-------------|-----|--------|
| EF-01 | Authentification via SSO OIDC de l'école | `AuthModule` | §6.1, §6.2 | ADR-003 | ✅ |
| EF-02 | Inscription d'un étudiant à un événement | `TicketModule`, `EventModule` | §7.1, §6.2 | ADR-002 | ✅ |
| EF-03 | Paiement en ligne sécurisé (Stripe) | `PaymentModule` | §7.2, §8 | — | ✅ |
| EF-04 | Génération et validation de tickets QR code | `TicketModule` | §7.1, §8 | — | ✅ |
| EF-05 | Création et gestion d'événements (organisateur) | `EventModule` | §6.1, §8 | — | ✅ |
| EF-06 | Tableau de bord organisateur (participants, KPI) | `EventModule`, API | §8 (GET /organizer) | — | ✅ |
| EF-07 | Export CSV des participants | `EventModule`, API | §8 (GET /organizer/export) | — | ✅ |
| EF-08 | Validation/révocation des organisateurs par admin | `UserModule`, API | §8 (PATCH /admin/organizers) | — | ✅ |
| EF-09 | Notifications email transactionnelles | `NotificationModule` | §7.3, §8.2 | ADR-001 | ✅ |
| EF-10 | Liste d'attente pour événements complets | `TicketModule` | §7.1, §6.2 | — | ✅ |
| EF-11 | Tarification earlybird | `EventModule`, `TicketModule` | §6.4, §7.1 | — | ✅ |
| EF-12 | Droit à l'oubli RGPD (suppression compte) | `UserModule` | §9.3, §8 | — | ✅ |
| EF-13 | Remboursement lors d'annulation | `PaymentModule`, `TicketModule` | §7.1, §7.2 | — | ✅ |

### Exigences Non-Fonctionnelles (ENF)

| ID | Exigence | Section DCT | Mesure technique | Statut |
|----|---------|-------------|-----------------|--------|
| ENF-01 | Latence P95 < 500 ms endpoints critiques | §9.2 | Pool PostgreSQL, cache Redis, pagination | ✅ |
| ENF-02 | Disponibilité ≥ 99,5 % | §9.2 | Docker Compose avec restart: always, monitoring uptime | ✅ |
| ENF-03 | 500 utilisateurs simultanés sans dégradation | §9.2 | SELECT FOR UPDATE (ADR-002), RabbitMQ découplage (ADR-001) | ✅ |
| ENF-04 | Conformité PCI-DSS | §6.4, §9.1 | Délégation totale à Stripe, aucune donnée carte stockée | ✅ |
| ENF-05 | Conformité RGPD | §9.3 | Pseudonymisation, minimisation, registre des traitements | ✅ |
| ENF-06 | Accessibilité WCAG 2.1 AA | §5 | Décision de périmètre — frontend hors DCT technique | ⚠️ |
| ENF-07 | Internationalisation FR/EN | §5 | Hors périmètre phase 1 | ⚠️ |
| ENF-08 | Sécurité — absence d'injection SQL | §9.1 | ORM NestJS/TypeORM avec requêtes paramétrées uniquement | ✅ |

## 10.2 Synthèse

- **Couverture EF** : 13 / 13 (100 %)
- **Couverture ENF** : 6 / 8 complètes, 2 partielles

### Exigences en attente ou partielles

| ID | Description | Raison | Renvoi |
|----|-------------|--------|--------|
| ENF-06 | WCAG 2.1 AA | Le frontend React est hors périmètre de cette DCT technique. La conformité WCAG sera documentée dans une DCT frontend dédiée. | Sprint final (phase 2) |
| ENF-07 | Internationalisation FR/EN | Explicitement hors périmètre phase 1 (cf. §5 Limites). Implémentation reportée si budget le permet. | Sprint final ou phase 2 |

---

*Dernière mise à jour : 2026-05-13*
