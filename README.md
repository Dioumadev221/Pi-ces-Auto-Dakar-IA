# Pièces Auto Dakar IA — assistant de chat multi-agents (n8n)

TP « Orchestration d'agents IA en low-code pour une PME de pièces auto ».

Assistant de chat pour le site d'une PME dakaroise (fictive) de pièces détachées :
un agent orchestrateur délègue à trois agents spécialisés (compatibilité, stock et prix,
commandes) construits dans n8n avec Google Gemini. Chaque nouvelle commande est validée
par le gérant sur Telegram avant d'être enregistrée.

## Contenu

| Dossier / fichier | Rôle |
|---|---|
| `donnees/` | Les 4 tables de la boutique en CSV : `pieces`, `compatibilites`, `stock`, `commandes` |
| `site/index.html` | Page web de la boutique avec la bulle de chat `@n8n/chat` |
| `docs/architecture.md` | Agents, outils, garde-fous et choix techniques |
| `workflows/` | Workflows n8n exportés (agent simple, équipe d'agents) |
| `compte-rendu/` | Tableau de tests (étape 8) et compte rendu |
| `captures/` | Captures d'écran demandées |

## Lancer n8n (Docker)

```bash
docker run -d --name n8n -p 5678:5678 -e GENERIC_TIMEZONE="Africa/Dakar" -e TZ="Africa/Dakar" -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true -v n8n_data:/home/node/.n8n n8nio/n8n
```

Puis ouvrir http://localhost:5678.

Aucune clé (Gemini, Telegram) n'est stockée dans ce dépôt : elles restent dans les identifiants chiffrés de n8n.
