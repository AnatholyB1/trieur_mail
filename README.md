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

1. Ouvre Claude Code Web : <https://claude.ai/code>.
2. Sélectionne ce repo (`trieur_mail`) sur la branche `main` (mergée depuis `claude/email-sorting-automation-QZAFQ`).
3. Vérifie que les **2 connectors MCP** sont actifs sur ton compte :
   - **Gmail** (le compte à trier)
   - **Notion** (workspace contenant 🌱 OS de Vie)
4. Crée une routine planifiée :
   - Trigger : Schedule (cron)
   - Fréquence : tous les jours à l'heure voulue, **expression cron en UTC** par défaut sur Claude Code Web.
     - Exemple Europe/Paris : `0 7 * * *` (UTC) = 8h du matin en hiver, 9h en été. Si tu veux 8h pile toute l'année, programme `0 6 * * *` en hiver et `0 6 * * *` toujours… ou utilise `0 7 * * *` si une heure flottante te suffit.
   - Repo : `trieur_mail`, branche `main`
   - Prompt initial : `/routine-matin`
5. Sauvegarde.

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
