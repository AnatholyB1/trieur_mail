# trieur_mail

Routine matinale de tri des emails Gmail.

Le tri est défini comme une slash command Claude Code (`/routine-matin`) située dans `.claude/commands/routine-matin.md`. Pour qu'il s'exécute **chaque matin automatiquement**, on la branche sur une **routine planifiée Claude Code Web**.

## Mise en place de la routine (Claude Code Web)

1. Ouvre Claude Code Web : <https://claude.ai/code>
2. Sélectionne ce repo (`trieur_mail`) et la branche `main` (ou la branche où la commande est mergée).
3. Vérifie que le **MCP Gmail** est connecté à ton compte (depuis Settings → Connectors → Gmail).
4. Crée une nouvelle routine planifiée :
   - **Trigger** : Schedule (cron)
   - **Fréquence** : tous les jours à l'heure voulue (ex. `0 8 * * *` pour 8h00)
   - **Repo** : `trieur_mail`, branche `main`
   - **Prompt initial** : `/routine-matin`
5. Sauvegarde. La routine se déclenchera désormais chaque matin et exécutera la commande.

## Test manuel

Avant d'activer la planification, lance la commande à la main dans une session Claude Code :

```
/routine-matin
```

Vérifie :
- Les labels `Trieur/Important`, `Trieur/Clients`, `Trieur/Notifications`, `Trieur/Newsletters`, `Trieur/Promos` sont bien créés dans Gmail.
- Les mails d'hier ont été déplacés dans les bons labels.
- Les promos sont passées à la corbeille.
- Le récap affiché correspond à ce que tu attends.

## Ce que fait `/routine-matin`

1. Crée les 5 labels Gmail s'ils n'existent pas.
2. Cherche les threads reçus la veille en `in:inbox` non déjà classés.
3. Classe chaque thread :
   - `Trieur/Important` — mails humains nécessitant une action (restent en inbox).
   - `Trieur/Clients` — mails de clients : commandes, devis, factures (restent en inbox).
   - `Trieur/Notifications` — `noreply@`, alertes services (archivés hors inbox).
   - `Trieur/Newsletters` — digests, articles récurrents (archivés hors inbox).
   - `Trieur/Promos` — promos, soldes → **corbeille**.
   - Spam / inutiles → **corbeille**.
4. Produit un récap Markdown des mails Important + Clients.

## Personnalisation

Modifie `.claude/commands/routine-matin.md` :
- Pour ajuster les règles de classement (ex. liste blanche d'expéditeurs clients).
- Pour changer la fenêtre temporelle (autre que « la veille »).
- Pour adapter le format du récap.

Tout changement poussé sur la branche utilisée par la routine sera pris en compte au prochain déclenchement.
