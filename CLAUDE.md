# Pixel Agents

> Hiérarchie : hérite de C:\dev\claude\CLAUDE.md et C:\dev\claude\oscardcstudio\CLAUDE.md

Extension VS Code avec webview React : bureau en pixel art où les agents Claude Code (terminaux) sont des personnages animés. Fork de pablodelucca/pixel-agents. Carte des fichiers : [llms.txt](llms.txt). Détail technique (protocole, suivi des agents, éditeur, assets) : [docs/reference.md](docs/reference.md).

## Build & Dev

```sh
npm install && cd webview-ui && npm install && cd .. && npm run build
```
Build : type-check, lint, esbuild (extension), vite (webview). F5 pour l'Extension Dev Host. Import de tilesets : `npm run import-tileset`.

## TypeScript

- Pas d'`enum` (`erasableSyntaxOnly`) : objets `as const`
- `import type` obligatoire pour les types seuls (`verbatimModuleSyntax`)
- `noUnusedLocals` / `noUnusedParameters`

## Constantes

Aucun nombre ou chaîne magique inline : tout vit dans `src/constants.ts` (backend), `webview-ui/src/constants.ts` (webview, y compris les couleurs rgba des overlays canvas) ou le bloc `:root` de `webview-ui/src/index.css` (variables `--pixel-*`). `webview-ui/src/office/types.ts` ré-exporte les constantes de grille.

## Pièges

- `fs.watch` est peu fiable sous Windows : toujours doubler d'un polling
- Lecture d'un fichier append-only : garder la ligne non terminée en tampon
- Retarder `agentToolDone` de 300 ms (sinon React masque les états actifs brefs)
- Idle : signal `turn_duration` + timer texte de 5 s (détail dans `docs/reference.md`)
- `content` d'un prompt user = string ou tableau (tool_results) : gérer les deux
- `/clear` crée un NOUVEAU fichier JSONL (l'ancien s'arrête simplement)
- `--output-format stream-json` exige un stdin non-TTY : inutilisable dans les terminaux VS Code
- IPC par hooks : échec (hooks capturés au démarrage, variables d'env non propagées). Le suivi JSONL fonctionne
- PNG vers SpriteData : pngjs pour le buffer RGBA, seuil alpha 128
- Sélection dans `OfficeCanvas` impérative (`editorState.selectedFurnitureUid`) : appeler `onEditorSelectionChange()` pour re-rendre la toolbar
- `crypto.randomUUID()` fonctionne dans l'extension host ; l'option `cwd` d'un terminal fixe le répertoire de travail. `/add-dir <path>` donne à la session l'accès à un répertoire supplémentaire

## Décisions

- `WebviewViewProvider` (pas `WebviewPanel`) : vit dans la zone panneau à côté du terminal
- Problem matcher esbuild inline (pas d'extension en plus)
- Webview = projet Vite séparé avec ses propres `node_modules` et `tsconfig`
