# trieur_mail

Routine matinale de tri des emails Gmail, intégrée à l'**🌱 OS de Vie** Notion.

Le tri est défini comme une slash command Claude Code (`/routine-matin`) située dans `.claude/commands/routine-matin.md`. Pour qu'il s'exécute **chaque matin automatiquement**, on la branche sur une **routine planifiée Claude Code Web**.

## Ce que fait la routine

1. Crée les labels `Trieur/Important`, `Trieur/Clients`, `Trieur/Comptabilité`, `Trieur/Notifications`, `Trieur/Newsletters`, `Trieur/Promos` (idempotent).
2. Récupère les threads reçus la veille (en `in:inbox`, non déjà classés).
3. Classe chaque thread :
   - `Trieur/Important` & `Trieur/Clients` → **restent en inbox**.
   - `Trieur/Comptabilité` (factures fournisseurs : Apple, Stripe, OVH, etc.) → **archivés** (retire `INBOX`, conservés pour la compta).
   - `Trieur/Notifications` & `Trieur/Newsletters` → **archivés** (retire `INBOX`).
   - `Trieur/Promos` → **corbeille**.
   - Spam évident → **corbeille** sans label.
4. **Notion** :
   - Pour chaque mail Important / Clients → 1 entrée dans la **📥 Inbox** GTD (`Type pressenti = Engagement | Tâche`, `Traité = false`).
   - 1 entrée dans la DB **📨 Tri Mails** sous OS de Vie : compteurs par catégorie + récap markdown du jour.
5. Imprime un récap final (mails à traiter + clients + lien vers le log Notion).

## Mise en place de la routine planifiée (Claude Code Web)

### Pré-requis (une seule fois)

1. Sur Claude Code Web → **Connectors**, vérifie que les 2 MCP sont actifs :
   - **Gmail** (le compte à trier)
   - **Notion** (workspace contenant 🌱 OS de Vie)

### Créer la routine

1. Ouvre <https://claude.ai/code>.
2. Sélectionne ce repo (`trieur_mail`), branche `main`.
3. Va dans la section **Schedules / Routines** (icône ⏰ ou « Schedule » dans le menu).
4. **+ New schedule** avec ces paramètres :

   | Champ | Valeur |
   |---|---|
   | Repo | `anatholyb1/trieur_mail` |
   | Branche | `main` |
   | Cron | `0 6 * * *` (= 8h Paris en été, 7h en hiver) |
   | Prompt | `/routine-matin` |

5. **Save**. La routine se déclenchera chaque matin à 8h.

### Note sur le timezone du cron

Claude Code Web exécute les crons en **UTC**. Avec `0 6 * * *` :
- Été (Paris UTC+2) → 8h00 locale ✅
- Hiver (Paris UTC+1) → 7h00 locale

Si tu veux **8h toute l'année**, configure 2 schedules (ou utilise `0 6 * * *` mars→oct + `0 7 * * *` oct→mars), ou laisse l'heure flottante.

### Permissions auto-accordées

`.claude/settings.json` (committé) auto-allow les tools nécessaires pour que la routine tourne sans intervention :
- `Bash(date:*)` et `Bash(TZ=Europe/Paris date:*)` pour le calcul de date
- 5 tools Gmail MCP (list_labels, create_label, search_threads, get_thread, label_thread)
- 1 tool Notion MCP (notion-create-pages)

Si la routine se bloque sur un prompt de permission, c'est qu'un tool manque dans cette liste — l'ajouter au fichier et re-pousser.

## Test manuel

Avant d'activer la planification, fais un dry-run pour vérifier la classification sans rien modifier :

```
/routine-matin --dry-run
```

Puis un vrai passage :

```
/routine-matin
```

Vérifie :
- Les labels `Trieur/*` ont été créés dans Gmail.
- Les mails d'hier ont été correctement classés.
- Les promos sont passées à la corbeille.
- La 📥 Inbox Notion contient les nouvelles captures (Important + Clients).
- La DB **📨 Tri Mails** a une nouvelle ligne pour la date d'hier avec le récap.

## Notion — les assets utilisés

- **📥 Inbox** (existante) — `data_source_id = 86b1a677-2ef1-4e5a-825e-c9b663bd596a`
- **📨 Tri Mails** (créée pour cette routine) — `data_source_id = 436f094e-143f-46dc-af92-aff16521db5f` · [ouvrir](https://www.notion.so/7f01cb73495942beae950b5e845c852f)

Schéma de **📨 Tri Mails** :

| Champ | Type |
|---|---|
| Date | Title (format `YYYY-MM-DD`) |
| Total | Number |
| Important | Number |
| Clients | Number |
| Comptabilité | Number |
| Notifications | Number |
| Newsletters | Number |
| Promos→corbeille | Number |
| Spam→corbeille | Number |
| Récap | Rich text (markdown du récap) |
| Reviewé | Checkbox |

## Personnalisation

Modifie `.claude/commands/routine-matin.md` :
- Pour ajuster les règles de classement (mots-clés, liste blanche d'expéditeurs).
- Pour changer la fenêtre temporelle (autre que « la veille »).
- Pour adapter le format du récap.

Tout changement poussé sur la branche utilisée par la routine sera pris en compte au prochain déclenchement.
