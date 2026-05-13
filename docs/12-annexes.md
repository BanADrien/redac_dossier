# §12 — Annexes

## 12.1 Audit qualité — Grille d'évaluation (TP 1.7)

Audit auto-évalué en posture d'auditeur externe. Barème : 0 = absent, 1 = partiel, 2 = conforme.

### Catégorie 1 — Structure (max. 12 points)

| N° | Critère | Score | Commentaire |
|----|---------|-------|-------------|
| S1 | Toutes les sections du squelette de référence sont présentes | 2 | §1 à §12 présentes, §11/ADR présent |
| S2 | Page de garde complète (titre, auteurs, date, version, cours) | 2 | Tous les champs renseignés dans 01-page-de-garde.md |
| S3 | Sommaire présent, navigable et cohérent avec la structure | 2 | README.md avec liens Markdown vers chaque fichier |
| S4 | Historique des révisions à jour (version, date, auteur, description) | 2 | Toutes les versions TP 1.2 à 1.7 tracées |
| S5 | Glossaire avec ≥ 10 termes contextualisés | 2 | 21 termes définis et contextualisés à SupEvents |
| S6 | Numérotation cohérente sans saut | 1 | Numérotation §6.1-6.4 correcte ; §11 absent (ADR dans /adr/) — à clarifier |
| **Total Structure** | | **11 / 12** | |

### Catégorie 2 — Complétude (max. 14 points)

| N° | Critère | Score | Commentaire | Action recommandée |
|----|---------|-------|-------------|-------------------|
| C1 | Chaque module documenté (rôle, responsabilités, interfaces) | 2 | TicketModule, PaymentModule, NotificationModule documentés | — |
| C2 | Flux critiques avec diagramme de séquence | 2 | Inscription nominale + échec paiement en §6.2 | — |
| C3 | Modèle de données complet (entités, attributs, relations) | 2 | 8 entités ER + dictionnaire complet en §6.4 | — |
| C4 | Contrats d'API documentés (endpoint, méthode, params, codes) | 2 | 21 endpoints + 2 événements async en §8 | — |
| C5 | Exigences transverses traitées (sécurité, performance, RGPD) | 2 | STRIDE, SLO, RGPD en §9 | — |
| C6 | Matrice de traçabilité (EF + ENF → composants) | 2 | 13 EF + 8 ENF en §10 | — |
| C7 | ≥ 3 ADR couvrant des décisions significatives | 2 | ADR-001 (RabbitMQ), ADR-002 (verrous), ADR-003 (SSO) | — |
| **Total Complétude** | | **14 / 14** | |

### Catégorie 3 — Cohérence (max. 12 points)

| N° | Critère | Score | Commentaire |
|----|---------|-------|-------------|
| CO1 | Terminologie identique entre toutes les sections | 2 | `TicketModule`, `PaymentModule` cohérents partout |
| CO2 | Entités du modèle de données présentes dans les contrats API | 2 | `User`, `Event`, `Ticket`, `Payment` cohérents §6.4 ↔ §8 |
| CO3 | Composants des diagrammes = modules documentés en §7 | 2 | AuthModule, EventModule, TicketModule, PaymentModule, NotificationModule |
| CO4 | Choix technologiques DCT cohérents avec ADR | 2 | RabbitMQ (ADR-001), verrous PostgreSQL (ADR-002), SSO (ADR-003) |
| CO5 | Aucune contradiction détectable | 2 | Aucune incohérence détectée entre sections |
| CO6 | Versions des diagrammes = version courante du document | 1 | Les fichiers indiquent la date mais pas de numéro de version par diagramme |
| **Total Cohérence** | | **11 / 12** | |

### Catégorie 4 — Lisibilité (max. 12 points)

| N° | Critère | Score | Commentaire | Action recommandée |
|----|---------|-------|-------------|-------------------|
| L1 | Diagrammes lisibles sans zoom | 2 | Diagrammes Mermaid synthétiques | — |
| L2 | Texte en voix active, phrases courtes (≤ 25 mots) | 2 | Rédigé en voix active tout au long | — |
| L3 | Tableaux correctement formatés | 2 | En-têtes présents, colonnes alignées | — |
| L4 | Code Mermaid se rend sans erreur | 1 | Non vérifié dans cet environnement — à valider sur mermaid.live | Valider chaque diagramme sur mermaid.live |
| L5 | Pas de redondance texte/diagramme | 2 | Les textes d'accompagnement apportent de l'analyse, pas de paraphrase | — |
| L6 | Structure en pyramide inversée (conclusion en premier) | 1 | Certaines sections commencent par le contexte avant la conclusion | Restructurer les introductions de §7 pour mettre la décision en premier |
| **Total Lisibilité** | | **10 / 12** | |

### Catégorie 5 — Maintenabilité (max. 10 points)

| N° | Critère | Score | Commentaire | Action recommandée |
|----|---------|-------|-------------|-------------------|
| M1 | CODEOWNERS présent et assigne chaque fichier à un responsable | 2 | CODEOWNERS à la racine — voir §12.2 | — |
| M2 | Convention de nommage kebab-case respectée | 2 | Tous les fichiers en kebab-case sans espaces | — |
| M3 | Liens internes fonctionnels (pas de 404) | 1 | À vérifier — certains liens README vers des fichiers non encore créés | Vérifier tous les liens README |
| M4 | Date de dernière mise à jour visible dans chaque fichier | 2 | Pied de page "Dernière mise à jour" présent dans tous les fichiers | — |
| M5 | Pas de code source copié-collé directement | 2 | Uniquement du pseudo-code et des extraits symboliques | — |
| **Total Maintenabilité** | | **9 / 10** | |

### Récapitulatif

| Catégorie | Score | Maximum |
|-----------|-------|---------|
| Structure | 11 | 12 |
| Complétude | 14 | 14 |
| Cohérence | 11 | 12 |
| Lisibilité | 10 | 12 |
| Maintenabilité | 9 | 10 |
| **TOTAL GÉNÉRAL** | **55** | **60** |

### Top 5 des défauts les plus impactants

1. **L4 — Diagrammes Mermaid non vérifiés** : risque de rendu cassé sur GitHub/GitLab. Impact : diagrammes illisibles pour le correcteur.
2. **S6 — Numérotation §11** : la section "Décisions d'architecture" est dans `/adr/` mais référencée §11 dans le README. À clarifier.
3. **L6 — Pyramide inversée non respectée** : les introductions de §7 commencent par le contexte avant la décision clé.
4. **CO6 — Pas de versioning par diagramme** : impossible de savoir si un diagramme est à jour sans relire tout le contexte.
5. **M3 — Liens README non vérifiés** : certains liens pointent vers des fichiers créés tardivement — risque de 404 si le repo était incomplet.

---

## 12.2 Workflow CI — Validation documentaire

Fichier : `.github/workflows/doc-lint.yml`

```yaml
name: Documentation Lint & Validate

on:
  push:
    branches: ['**']
  pull_request:
    branches: ['main']

jobs:
  lint-markdown:
    name: Lint Markdown
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install markdownlint-cli
        run: npm install -g markdownlint-cli

      - name: Run markdownlint
        run: markdownlint 'docs/**/*.md' 'README.md' --config .markdownlint.json

  validate-links:
    name: Validate internal links
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check dead links (internal)
        uses: gaurav-nelson/github-action-markdown-link-check@v1
        with:
          use-quiet-mode: 'yes'
          config-file: '.mlc-config.json'

  validate-mermaid:
    name: Validate Mermaid diagrams
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install mermaid-js/mermaid-cli
        run: npm install -g @mermaid-js/mermaid-cli

      - name: Extract and validate Mermaid blocks
        run: |
          find docs -name "*.md" | while read file; do
            python3 - <<'PYEOF'
          import re, sys, subprocess, tempfile, os
          with open("$file") as f:
              content = f.read()
          blocks = re.findall(r'```mermaid\n(.*?)```', content, re.DOTALL)
          for i, block in enumerate(blocks):
              with tempfile.NamedTemporaryFile(suffix='.mmd', mode='w', delete=False) as tmp:
                  tmp.write(block)
                  tmp_path = tmp.name
              result = subprocess.run(['mmdc', '-i', tmp_path, '-o', '/dev/null'], capture_output=True)
              if result.returncode != 0:
                  print(f"ERROR in {os.environ.get('file','?')} block {i}: {result.stderr.decode()}")
                  sys.exit(1)
              os.unlink(tmp_path)
          PYEOF
          done
```

---

*Dernière mise à jour : 2026-05-13*
