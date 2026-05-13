# SupEvents — Documentation de Conception Technique

DCT complète du projet **SupEvents**, plateforme de gestion d'événements étudiants.

## Sommaire

| N° | Section | Fichier |
|----|---------|---------|
| §1 | Page de garde | [01-page-de-garde.md](docs/01-page-de-garde.md) |
| §2 | Historique des révisions | [02-historique-revisions.md](docs/02-historique-revisions.md) |
| §3 | Glossaire | [03-glossaire.md](docs/03-glossaire.md) |
| §4 | Contexte et objectifs | [04-contexte-objectifs.md](docs/04-contexte-objectifs.md) |
| §5 | Périmètre et limites | [05-perimetre-limites.md](docs/05-perimetre-limites.md) |
| §6.1 | Vue logique | [06-architecture-generale/06.1-vue-logique.md](docs/06-architecture-generale/06.1-vue-logique.md) |
| §6.2 | Vue des processus | [06-architecture-generale/06.2-vue-processus.md](docs/06-architecture-generale/06.2-vue-processus.md) |
| §6.3 | Vue de déploiement | [06-architecture-generale/06.3-vue-deploiement.md](docs/06-architecture-generale/06.3-vue-deploiement.md) |
| §6.4 | Vue des données | [06-architecture-generale/06.4-vue-donnees.md](docs/06-architecture-generale/06.4-vue-donnees.md) |
| §7 | Conception détaillée des modules | [07-conception-detaillee/07-modules.md](docs/07-conception-detaillee/07-modules.md) |
| §8 | Interfaces & contrats d'API | [08-interfaces-api.md](docs/08-interfaces-api.md) |
| §9 | Exigences transverses | [09-exigences-transverses.md](docs/09-exigences-transverses.md) |
| §10 | Matrice de traçabilité | [10-matrice-tracabilite.md](docs/10-matrice-tracabilite.md) |
| §11 | Décisions d'architecture (ADR) | [11-decisions-architecture/README.md](docs/11-decisions-architecture/README.md) |
| §12 | Annexes & Audit qualité | [12-annexes.md](docs/12-annexes.md) |

## Conventions Git

**Branches :** `docs/section-XX-description`  
**Commits :** `docs(§XX): description courte au présent`

## Répartition des TPs

| Collaborateur | TP | Branche |
|--------------|-----|---------|
| BanADrien | TP 1.2 (§1-5) + TP 1.5 (§7+§11) | `docs/section-01-scaffolding` · `docs/section-07-11-modules-adr` |
| YSO-DEV | TP 1.3 (§6.1-6.3) + TP 1.6 (§9-10) | `docs/section-06-architecture` · `docs/section-09-10-exigences` |
| Tankiste07 | TP 1.4 (§6.4+§8) + TP 1.7 (§12) | `docs/section-06-4-donnees-api` · `docs/section-12-audit-ci` |
