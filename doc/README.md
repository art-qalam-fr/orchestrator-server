# MCP Orchestrator Server — Documentation

## Aperçu
Le **MCP Orchestrator Server** fournit des fonctionnalités de gestion et de coordination de tâches entre instances MCP (ex. Claude Desktop, Cline). Il permet aux agents de créer, partager et exécuter des tâches avec dépendances, suivi d’état et stockage persistant.

- Version actuelle : 1.1.0
- Licence : MIT
- Objectif : orchestrer des tâches distribuées et robustes entre plusieurs clients MCP.

## Fonctionnalités (v1.1.0)
- Mise à jour des tâches pendantes
- Suppression sécurisée avec vérification des dépendances
- Détection de cycles de dépendances
- Documentation des outils (listage complet)
- Transitions d’état améliorées
- Stockage persistant des tâches (fichier JSON)

## Prérequis
- Node.js ≥ 18
- Dépendance SDK MCP TypeScript (`@modelcontextprotocol/sdk`)
- Optionnel : Zod (schémas) selon le serveur

## Installation (projet Node autonome)
```bash
npm install
npm run build
```

## Démarrage
```bash
node build/index.js
```
Le serveur fonctionne en **stdio** et expose des outils MCP.

## Outils exposés
Les outils suivants sont disponibles et documentés par le serveur :

- `create_task`
  - Entrée : `{ id: string, description: string, dependencies?: string[] }`
  - Effet : crée une tâche en état `pending` avec dépendances optionnelles

- `update_task`
  - Entrée : `{ task_id: string, description?: string, dependencies?: string[] }`
  - Effet : modifie une tâche `pending` (vérifie les dépendances et évite les cycles)

- `delete_task`
  - Entrée : `{ task_id: string }`
  - Effet : supprime une tâche si aucune autre tâche ne dépend d’elle

- `get_next_task`
  - Entrée : `{ instance_id: string }`
  - Effet : récupère la prochaine tâche disponible (sans dépendances incomplètes) et l’assigne en `in_progress`

- `complete_task`
  - Entrée : `{ task_id: string, instance_id: string, result: string }`
  - Effet : marque la tâche comme `completed`, enregistre le `result` et renvoie les tâches déverrouillées

- `get_task_status`
  - Entrée : `{}`
  - Effet : renvoie la liste de toutes les tâches avec leur état

- `get_task_details`
  - Entrée : `{ task_id: string }`
  - Effet : renvoie le détail d’une tâche

## Stockage persistant
- Fichier : `data/tasks.json`
- Le serveur lit/écrit ce fichier pour persister les tâches.

## Exemples d’utilisation
Créer une tâche :
```js
await create_task({
  id: 'setup',
  description: 'Initial setup'
});
```
Obtenir la prochaine tâche :
```js
const task = await get_next_task({ instance_id: 'worker-1' });
```
Compléter une tâche :
```js
await complete_task({
  task_id: 'setup',
  instance_id: 'worker-1',
  result: 'System initialized'
});
```

## Intégration dans la passerelle MCP
Ajouter la déclaration suivante sous `servers` dans le `config.json` de la passerelle :
```json
"orchestrator-server": {
  "command": "node",
  "args": [
    "<HEPHAISTOS_ROOT>\\mcp\\servers\\orchestrator-server\\build\\index.js"
  ],
  "enabled": true
}
```

Ensuite, la passerelle exposera un **outil proxy** nommé `orchestrator-server`. Les clients MCP (Devin/RooCode) invoquent ses sous-outils via :
```json
{
  "tool_name": "create_task",
  "arguments": { "id": "setup", "description": "Initial setup" }
}
```

## Bonnes pratiques
- Définir des identifiants de tâches uniques (`id`)
- Éviter les cycles de dépendance
- Utiliser des `instance_id` stables pour le suivi d’attribution
- Surveiller le fichier `data/tasks.json` pour les sauvegardes/restaurations

## Feuille de route
- v1.2.0 : priorités, délais, gestion d’instances
- v1.3.0 : groupes de tâches, analytics, tableau de bord

## Ressources et liens
- Smithery : https://smithery.ai/server/orchestrator-server
- Glama : https://glama.ai/mcp/servers/@mokafari/orchestrator-server
- GitHub : https://github.com/mokafari/orchestrator-server#

## Licence
- MIT
