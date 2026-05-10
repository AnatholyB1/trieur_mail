---
description: Trie les mails Gmail de la veille (labels, corbeille, récap des importants)
allowed-tools:
  - mcp__e884104c-3419-429f-b197-89ac2131a646__list_labels
  - mcp__e884104c-3419-429f-b197-89ac2131a646__create_label
  - mcp__e884104c-3419-429f-b197-89ac2131a646__search_threads
  - mcp__e884104c-3419-429f-b197-89ac2131a646__get_thread
  - mcp__e884104c-3419-429f-b197-89ac2131a646__label_thread
  - mcp__e884104c-3419-429f-b197-89ac2131a646__label_message
  - mcp__e884104c-3419-429f-b197-89ac2131a646__unlabel_thread
  - Bash
---

# Routine matinale — tri des mails de la veille

Tu es l'assistant de tri d'emails. Effectue les étapes ci-dessous **dans l'ordre**, sans demander confirmation à l'utilisateur (c'est une routine automatique).

## Catégories cibles

| Label Gmail | Quand l'appliquer |
|---|---|
| `Trieur/Important` | Mail nécessitant une action, une réponse ou une décision personnelle. Inclut sujets professionnels urgents, factures dues, rendez-vous à confirmer. |
| `Trieur/Clients` | Mail provenant d'un client (commande, demande de support, devis, échange contractuel). |
| `Trieur/Notifications` | Notifications automatiques de services (GitHub, Slack, Stripe, alertes monitoring, mots de passe, OTP). |
| `Trieur/Newsletters` | Newsletters, articles récurrents, digests éditoriaux. |
| `Trieur/Promos` | Promotions commerciales, soldes, codes promo, pubs. **Après labellisation, mettre aussi en corbeille.** |
| (corbeille) | Spam évident, notifications totalement inutiles (newsletter dont l'utilisateur ne lit jamais le contenu, pub déguisée). Appliquer le label système `TRASH`. |

> Règle de prudence : en cas de doute entre `Important` et autre chose, choisir `Important`. Ne jamais mettre en corbeille un mail provenant d'un humain qui s'adresse personnellement à l'utilisateur.

## Étape 1 — Préparer les labels

1. Appelle `list_labels` pour récupérer la liste actuelle.
2. Pour chaque label de la liste suivante qui **n'existe pas** déjà, crée-le avec `create_label` :
   - `Trieur/Important`
   - `Trieur/Clients`
   - `Trieur/Notifications`
   - `Trieur/Newsletters`
   - `Trieur/Promos`
3. Mémorise les IDs de label retournés.

## Étape 2 — Récupérer les mails de la veille

1. Calcule la date d'hier au format `YYYY/MM/DD` avec `Bash` : `date -d 'yesterday' +%Y/%m/%d` et la date du jour `date +%Y/%m/%d`.
2. Appelle `search_threads` avec la query Gmail :
   ```
   after:<HIER> before:<AUJOURDHUI> in:inbox -label:Trieur/Important -label:Trieur/Clients -label:Trieur/Notifications -label:Trieur/Newsletters -label:Trieur/Promos
   ```
   (le `-label:` évite de retraiter un thread déjà classé si la routine est relancée)
3. Si la liste est vide → affiche « Aucun mail à trier pour <date d'hier> » et termine.

## Étape 3 — Classer chaque thread

Pour chaque thread retourné :

1. Appelle `get_thread` pour lire l'expéditeur, le sujet et le début du contenu.
2. Décide la catégorie en suivant le tableau ci-dessus. Critères concrets :
   - Adresse `noreply@`, `no-reply@`, `notifications@`, `automated@` → **Notifications** (sauf alerte critique → Important).
   - Mots-clés `unsubscribe`, `newsletter`, `weekly digest`, `roundup` dans corps ou sujet → **Newsletters**.
   - Mots-clés `% off`, `sale`, `promo`, `deal`, `coupon`, `black friday`, `solde`, `réduction` → **Promos** (puis corbeille).
   - Sujet contenant `facture`, `invoice`, `devis`, `commande`, `bon de commande`, `purchase order`, ou expéditeur identifié comme client connu → **Clients**.
   - Mail rédigé par un humain s'adressant directement à l'utilisateur (sujet personnel, demande, question) → **Important**.
3. Applique le label avec `label_thread` (pass `add_label_ids: [<id du label>]`).
4. Si la catégorie est **Promos** ou si le mail est jugé totalement inutile (spam, pub déguisée) :
   - Applique aussi le label système `TRASH` via `label_thread` avec `add_label_ids: ["TRASH"]` et retire `INBOX` avec `remove_label_ids: ["INBOX"]`.
5. Pour les autres catégories, retire `INBOX` avec `remove_label_ids: ["INBOX"]` afin que le mail soit archivé hors de la boîte de réception (sauf `Important` et `Clients` qui restent dans l'inbox).

## Étape 4 — Récapitulatif

À la fin, produis un message Markdown structuré comme suit :

```
# Récap mails du <date d'hier au format JJ/MM/AAAA>

**Traités : N threads** — X important · Y clients · Z notifications · W newsletters · P promos (→corbeille) · D supprimés

## À traiter en priorité
- **<Sujet>** — <Expéditeur> · <résumé en 1 ligne de l'action attendue>
- ...

## Clients
- **<Sujet>** — <Expéditeur> · <résumé en 1 ligne>
- ...

## Notifications notables (optionnel — uniquement si action requise)
- ...
```

Si aucune entrée pour une section, omets-la. Limite-toi à des résumés d'**une ligne** par mail. N'invente rien : si tu n'es pas sûr du contenu, dis « contenu non lu » plutôt que paraphraser.

## Étape 5 — Fin

Termine ta réponse uniquement par le récap (Markdown ci-dessus). Pas de commentaire de processus, pas de « j'ai bien fait X et Y ».
