# Architecture

## Vue d'ensemble

```
Client (widget @n8n/chat)
        │
        ▼
Orchestrateur (gemini-2.5-flash, mémoire de session)
        ├── agent_compatibilite ── chercher_compatibilite, lire_designation
        ├── agent_stock_prix ───── lire_piece, lire_stock
        └── agent_commandes ────── lire_commande, preparer_commande*
                                             * soumis à validation Telegram du gérant
```

L'orchestrateur n'accède à aucune donnée : il route chaque demande vers l'agent compétent
et rédige la réponse finale à partir de leurs retours. Seul `agent_commandes` dispose d'un
outil d'écriture, exécuté uniquement après approbation humaine.

## Données (n8n Data tables)

| Table | Colonnes | Lignes |
|---|---|---|
| `pieces` | ref, designation, categorie, marque_piece, prix_fcfa | 12 |
| `compatibilites` | ref_piece, marque, modele, annee_debut, annee_fin | 15 |
| `stock` | ref_piece, quantite | 12 |
| `commandes` | numero, client, telephone, ref_piece, quantite, montant_fcfa, ville, statut, date_commande | 3 |

Le filtrage par année (`annee_debut <= annee <= annee_fin`) est fait par la requête de la
table et non par le modèle.

## Agents

| Agent | Modèle | Max iterations | Outils |
|---|---|---|---|
| Orchestrateur | gemini-2.5-flash | 8 | 3 agents |
| agent_compatibilite | gemini-2.5-flash-lite | 10 | chercher_compatibilite, lire_designation |
| agent_stock_prix | gemini-2.5-flash-lite | 10 | lire_piece, lire_stock |
| agent_commandes | gemini-2.5-flash-lite | 5 | lire_commande, preparer_commande |

Les paramètres des outils sont renseignés par le modèle via `$fromAI()`. Les champs
`numero`, `statut` et `date_commande` de `preparer_commande` sont fixés par n8n.

## Garde-fous

- Consignes : pas de remise, pas de confirmation de commande par l'IA, hors-sujet refusé,
  tentatives de modification des règles ignorées.
- Outils : lecture seule partout sauf `preparer_commande`, au statut fixe.
- Humain : chaque commande est approuvée ou refusée par le gérant sur Telegram.

## Choix techniques

- Quota gratuit Gemini limité par modèle (5 req/min sur gemini-2.5-flash) : l'orchestrateur
  et les agents spécialisés utilisent deux modèles distincts, avec 3 tentatives espacées de 5 s.
- `agent_compatibilite` et `agent_stock_prix` montent à 10 itérations : un véhicule peut
  avoir plusieurs pièces compatibles, chacune nécessitant un appel à `lire_designation`.
