# trieur_mail

Routine matinale de tri des emails Gmail via Claude Code + MCP Gmail.

## Utilisation

Lancer la commande slash dans une session Claude Code :

```
/trier-mails
```

La routine :
1. Crée les labels `Trieur/Important`, `Trieur/Clients`, `Trieur/Notifications`, `Trieur/Newsletters`, `Trieur/Promos` s'ils n'existent pas.
2. Récupère tous les threads reçus la veille (en boîte de réception, non encore classés).
3. Classe chaque thread dans la bonne catégorie.
4. Met les promos et mails inutiles en corbeille (label système `TRASH`).
5. Archive hors inbox `Notifications` et `Newsletters`. Garde dans l'inbox `Important` et `Clients`.
6. Affiche un récap des mails à traiter et des mails clients.

## Planification quotidienne

Pour une exécution chaque matin, programmer une session Claude Code récurrente qui exécute `/trier-mails`.
Sur Claude Code Web : utiliser une session planifiée pointant sur ce repo.
En local : un cron qui lance `claude -p "/trier-mails"` fait l'affaire.

## Configuration

Le MCP Gmail doit être connecté au compte à trier. Aucun secret n'est stocké dans ce repo.

## Personnalisation

Modifier `.claude/commands/trier-mails.md` pour :
- Ajuster les catégories ou les règles de classement
- Changer la fenêtre temporelle (autre que « la veille »)
- Adapter le format du récap
