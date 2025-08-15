# Excalidraw Embed Object from Template Script

## Description
Script Excalidraw pour Obsidian permettant de créer et d'intégrer automatiquement des objets de workflow depuis des templates personnalisables directement dans vos dessins Excalidraw.

## Version actuelle : v1.3.0 (Stable)
Date de mise à jour : 15/08/2025

## 🎯 Fonctionnalités principales

### ✨ Création automatique d'embeds
- Création d'objets de workflow directement depuis Excalidraw
- Intégration automatique comme embed dans le dessin actif
- Positionnement intelligent au centre de la vue avec décalage en cascade
- Support complet des fichiers Excalidraw embeddés

### 🎨 Système de templates dynamique (v1.2.0+)
- **Sélection interactive** parmi les templates disponibles dans le dossier `Templates/`
- **Support des catégories** de templates (sous-dossiers)
- **Mémorisation automatique** des 5 derniers templates utilisés
- **Templates favoris** avec accès rapide (⭐)
- **Template par défaut intégré** en fallback

### 🔄 Gestion intelligente des doublons (v1.2.1+)
- Détection automatique des noms de fichiers existants
- **Propositions intelligentes** : nom(1), nom(2), nom_v2, nom_copy
- Interface intuitive avec `utils.suggester()`
- Option pour saisir un nom personnalisé

### 🖼️ Affichage immédiat des embeds (v1.3.0)
- **Toggle programmatique automatique** pour forcer l'affichage
- **Pas de manipulation manuelle** requise
- **Multiples méthodes de rafraîchissement** pour compatibilité maximale
- Résolution définitive du bug d'affichage initial

## 📋 Prérequis

- **Obsidian** v1.5.0 ou supérieur
- **Plugin Excalidraw** v2.14.0 ou supérieur
- Un dossier `Templates/` dans votre vault (optionnel)

## 🚀 Installation

### Installation via le menu Scripts d'Excalidraw

1. Ouvrez un fichier Excalidraw dans Obsidian
2. Cliquez sur l'icône **outils** dans la barre d'outils Excalidraw
3. Accédez à **Scripts** → **Downloaded** → **Install new**
4. Copiez le contenu du fichier `embed-object-from-template.md`
5. Le script apparaîtra dans la section "Downloaded"

### Installation manuelle

1. Copiez le fichier `embed-object-from-template.md` dans :
   ```
   VotreVault/Excalidraw/Scripts/Downloaded/
   ```
2. Redémarrez Obsidian ou rafraîchissez les scripts

## 🎮 Utilisation

### Utilisation basique

1. **Ouvrez** un fichier Excalidraw
2. **Lancez** le script depuis :
   - Menu Scripts → Downloaded → embed-object-from-template
   - Ou via raccourci clavier configuré
3. **Sélectionnez** un template dans la liste ou utilisez le template par défaut
4. **Entrez** le nom de votre objet de workflow
5. **L'objet est créé** et automatiquement intégré dans votre dessin !

### Configuration d'un raccourci clavier

1. Paramètres → Raccourcis clavier
2. Recherchez "Excalidraw: embed-object-from-template"
3. Assignez un raccourci (ex : `Cmd+Shift+O` ou `Ctrl+Shift+O`)

### Organisation des templates

Placez vos templates Excalidraw dans le dossier `Templates/` de votre vault :

```
Templates/
├── MonTemplateAction.md
├── MonTemplateObjet.md
├── Workflows/
│   ├── WorkflowSimple.md
│   └── WorkflowComplexe.md
├── Phases/
│   ├── Phase1.md
│   └── Phase2.md
└── CreerObjetWorkflow.md
```

Les templates sont automatiquement :
- Détectés et listés par catégorie
- Triés alphabétiquement
- Mémorisés après utilisation

## ⚙️ Configuration avancée

### Structure d'un template

Les templates doivent être des fichiers Excalidraw valides avec :
- Header YAML avec `excalidraw-plugin: parsed`
- Sections standard Excalidraw
- Variable `${objectName}` pour le contenu dynamique

### Exemple de template minimal

```markdown
---
excalidraw-plugin: parsed
tags: [excalidraw]
---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠==

# Text Elements
${objectName}

# Embedded files

# Drawing
```json
{
    "type": "excalidraw",
    "version": 2,
    "source": "https://github.com/zsviczian/obsidian-excalidraw-plugin",
    "elements": [],
    "appState": {
        "theme": "light",
        "viewBackgroundColor": "#ffffff"
    }
}
```
%%
```

### Personnalisation des dimensions

Modifiez les lignes de création de l'élément image dans le script :
```javascript
width: 400,   // Largeur de l'embed
height: 300,  // Hauteur de l'embed
```

## 🔧 Résolution des problèmes

| Problème | Solution |
|----------|----------|
| L'embed ne s'affiche pas | ✅ Résolu dans v1.3.0 avec toggle automatique |
| "Aucun fichier actif" | Ouvrez d'abord un dessin Excalidraw |
| Templates non trouvés | Vérifiez le dossier `Templates/` et le format des fichiers |
| Nom de fichier déjà existant | Le script propose automatiquement des alternatives |
| Erreur de création | Vérifiez les permissions sur le dossier |

## 📊 Architecture technique

### Workflow du script

1. **Sélection du template** : Menu interactif avec mémoire
2. **Saisie du nom** : Prompt utilisateur
3. **Gestion des doublons** : Vérification et propositions
4. **Création du fichier** : Génération depuis template
5. **Intégration dans Excalidraw** :
   - Génération d'un fileId unique
   - Ajout dans `## Embedded Files`
   - Insertion de l'élément image dans le JSON
   - Status "saved" pour affichage immédiat
6. **Toggle programmatique** : Rafraîchissement forcé
7. **Notification** : Confirmation de succès

### Mécanisme de rafraîchissement (v1.3.0)

Le script utilise plusieurs méthodes pour garantir l'affichage :
1. Toggle source/preview pour les vues Markdown
2. setState() pour forcer le rechargement
3. Fermeture/réouverture via fichier temporaire
4. Déclenchement d'événements Obsidian

## 📈 Historique des versions

### v1.3.0 (15/08/2025) - **Version stable actuelle**
- ✅ Toggle programmatique pour affichage immédiat
- ✅ Résolution définitive du bug d'affichage
- ✅ Optimisation des délais et méthodes de rafraîchissement
- ✅ Support complet de tous les types de templates

### v1.2.0 - v1.2.9 (14-15/08/2025)
- Système de sélection dynamique de templates
- Gestion avancée des doublons
- Multiples corrections d'interface
- Synchronisation nom fichier/embed

### v1.0.0 (13/08/2025)
- Version initiale stable
- Création basique avec template fixe

## 🤝 Contribution

Les contributions sont les bienvenues ! 

- **Signaler des bugs** : Via les [issues GitHub](https://github.com/RollandMELET/EXCALIDRAW_Script_EmbededDrawingFromTemplate/issues)
- **Proposer des améliorations** : Pull requests acceptées
- **Partager des templates** : Créez des exemples dans le dossier Templates

## 📝 Licence

MIT License - Voir le fichier LICENSE pour plus de détails

## 👥 Auteurs

- **Rolland MELET** - CEO 360SmartConnect
- **Claude Code** - Assistant IA Anthropic

## 💬 Support

Pour toute question ou problème :
- Ouvrez une issue sur le [dépôt GitHub](https://github.com/RollandMELET/EXCALIDRAW_Script_EmbededDrawingFromTemplate)
- Contact : rm@360sc.io

---

*Script développé pour optimiser les workflows de création de diagrammes dans Obsidian avec Excalidraw.*

**Version Stable 1.3.0** - 15/08/2025