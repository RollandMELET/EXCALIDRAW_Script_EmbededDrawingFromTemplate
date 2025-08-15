# Script Excalidraw : Embed Object from Template

Script permettant de créer et intégrer automatiquement des objets Excalidraw depuis des templates dans Obsidian.

## 🎯 Fonctionnalités

- ✅ **Création instantanée** d'objets de workflow via raccourci clavier
- ✅ **Templates Excalidraw** : Les nouveaux objets sont des dessins Excalidraw complets
- ✅ **Intégration automatique** dans le canevas actif avec le format `|100%` pour affichage en mode dessin
- ✅ **Positionnement intelligent** au centre de la vue avec décalage en cascade
- ✅ **Sauvegarde forcée** pour préserver les modifications
- ✅ **Rafraîchissement automatique** de la vue après insertion

## 📋 Prérequis

- Obsidian avec le plugin Excalidraw installé
- Un dossier `Templates/` dans votre vault
- Un template Excalidraw (fourni par défaut si absent)

## 🚀 Installation

### 1. Installer le script

Copiez le fichier `embed-object-from-template.md` dans :
```
VotreVault/Excalidraw/Scripts/
```

### 2. Créer un template (optionnel)

Le script utilise un template Excalidraw par défaut si aucun n'est trouvé.

Pour créer votre propre template :
1. Créez un nouveau dessin Excalidraw
2. Sauvegardez-le dans `Templates/MonTemplateObjet.md`
3. Le script utilisera automatiquement ce template

**Note** : Utilisez `${objectName}` dans votre template pour qu'il soit remplacé par le nom saisi.

## 🎮 Utilisation

### Méthode 1 : Menu Excalidraw

1. Ouvrez un dessin Excalidraw
2. Menu Scripts (engrenage → Scripts)
3. Sélectionnez "embed-object-from-template"
4. Saisissez le nom de l'objet
5. L'objet apparaît immédiatement !

### Méthode 2 : Raccourci clavier (recommandé)

1. Paramètres → Raccourcis clavier
2. Recherchez "Excalidraw: embed-object-from-template"
3. Assignez un raccourci (ex : `Cmd+Shift+O`)
4. Utilisez-le dans n'importe quel dessin

## ⚙️ Configuration

### Chemin du template

Modifiez la ligne 9 du script :
```javascript
const TEMPLATE_PATH = "Templates/MonTemplateObjet.md";
```

### Taille des objets

Modifiez les lignes 186-187 :
```javascript
width: 400,   // Largeur
height: 300,  // Hauteur
```

## 🔧 Dépannage

| Problème | Solution |
|----------|----------|
| "Aucun fichier actif" | Ouvrez d'abord un dessin Excalidraw |
| L'objet n'apparaît pas | Utilisez `Cmd/Ctrl+Shift+E` (2x) pour rafraîchir |
| Affichage en mode texte | Le script v1.0 corrige ce problème avec `|100%` |
| Position perdue après toggle | Sauvegarde forcée intégrée dans v1.0 |

## 📊 Fonctionnement Technique

1. **Prompt utilisateur** : Demande le nom de l'objet
2. **Création du fichier** : Génère un fichier Excalidraw depuis le template
3. **Calcul de position** : Centre de vue + décalage cascade
4. **Modification du dessin** :
   - Ajout dans `## Embedded Files` avec format `[[nom|100%]]`
   - Insertion de l'élément image dans le JSON
   - Synchronisation du `fileId`
5. **Sauvegarde et rafraîchissement** automatiques

## 🆕 Nouveautés v1.0

- **Template Excalidraw par défaut** : Plus besoin de créer un template texte
- **Format `|100%`** : Force l'affichage en mode dessin
- **Positionnement intelligent** : Centré dans la vue active
- **Sauvegarde forcée** : Préserve les positions après modifications
- **Sans logs console** : Version de production optimisée

## 📝 Notes

- Les fichiers créés sont placés dans le même dossier que le dessin actif
- Compatible avec la compression des fichiers Excalidraw
- Gère automatiquement la création de sections manquantes
- Support des insertions multiples avec décalage automatique

## 🤝 Auteurs

- Rolland MELET (360SmartConnect)
- Claude Code (Assistant IA)

---

*Version Finale 1.0.0 - 2025-08-14*