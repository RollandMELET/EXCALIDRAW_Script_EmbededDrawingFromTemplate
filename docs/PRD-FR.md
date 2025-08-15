
## **PRD : Création et Intégration Dynamique d'Objets de Workflow dans Excalidraw pour Obsidian**

**Version :** 1.3
**Date :** 2025-08-15
**Auteur :** SuperAssistant (sur la base des besoins et de l'analyse technique de l'utilisateur)
**Mise à jour :** Version stable avec affichage immédiat des embeds
**Sources d'Information :**
*   **Dépôt GitHub du Plugin Excalidraw :** [https://github.com/zsviczian/obsidian-excalidraw-plugin](https://github.com/zsviczian/obsidian-excalidraw-plugin)
*   **Documentation de l'API Excalidraw Automate :** [https://zsviczian.github.io/obsidian-excalidraw-plugin/](https://zsviczian.github.io/obsidian-excalidraw-plugin/)
* **Dépôt GitHub de Templater Obsidian Plugin**  https://github.com/SilentVoid13/Templater

### 1. Descriptif Précis du Rendu de l'Opération (Le "Quoi")

L'utilisateur se trouve dans une vue Excalidraw active dans Obsidian.

#### Version 1.0 (Actuelle)
1.  En utilisant un raccourci clavier unique et dédié (ex: `Cmd+Option+O`), une boîte de dialogue apparaît, lui demandant de nommer un nouvel "objet de workflow".
2.  L'utilisateur saisit un nom (ex: "Étape 1 - Authentification") et valide.
3.  **Immédiatement et sans aucune autre action manuelle**, les événements suivants se produisent de manière atomique :
    *   Un nouvel élément visuel apparaît sur le canevas Excalidraw. Cet élément est l'intégration ("embed") d'un nouveau fichier. **Il doit être positionné au centre de la vue visible de l'utilisateur à l'instant de la création.**
    *   Dans l'explorateur de fichiers d'Obsidian, un nouveau fichier Markdown (ex: `Étape 1 - Authentification.md`) est créé.
    *   Ce nouveau fichier est impérativement localisé dans le **même dossier** que le dessin Excalidraw actif.
    *   Le contenu de ce nouveau fichier est pré-rempli à partir d'un template spécifié (ex: `Templates/MonTemplateObjet.md`).

#### Version 1.2 (Nouvelle - Sélection de Template)
1.  En utilisant un raccourci clavier unique et dédié (ex: `Cmd+Option+O`), **un menu de sélection de templates apparaît**.
2.  **L'utilisateur choisit parmi les templates disponibles** :
    *   Liste des templates du dossier `Templates/` (fichiers `.md` et `.excalidraw`)
    *   Templates récemment utilisés (3-5 derniers)
    *   Option "Template par défaut" (Embedded-Object-Template)
    *   Support des catégories si sous-dossiers présents
3.  Une fois le template sélectionné, l'utilisateur saisit le nom de l'objet.
4.  Le reste du processus reste identique à la version 1.0.

Le processus doit être fluide, rapide (moins de 2 secondes) et ne doit laisser aucun artefact. L'utilisateur doit pouvoir enchaîner cette opération plusieurs fois pour construire rapidement son diagramme.

### 2. Contexte et Objectif Stratégique (Le "Pourquoi")

Cette fonctionnalité est la pierre angulaire d'un système de **"Visual Product Management"** dont l'objectif final est de transformer des workflows visuels en spécifications techniques (PRD) exploitables par une IA (projet 360SmartConnect).

Chaque "objet" sur le canevas doit être une entité "intelligente" possédant deux facettes :
*   **Une facette visuelle :** Un pictogramme ou une forme simple sur le canevas Excalidraw.
*   **Une facette "données" :** Un fichier Markdown structuré contenant tous les paramètres de l'objet, suivant le principe de la "fiche" (back of the card).

L'objectif est de réduire la friction entre l'idéation (le dessin) et la spécification (le texte), pour ensuite automatiser la génération de documentation via des outils comme le script "Excalidraw Writing Machine".

### 3. User Stories

#### User Story 1 (Version de base)
**En tant que** Product/Process Designer,
**Je veux** utiliser un raccourci clavier pour créer instantanément un nouvel objet de workflow à partir d'un template et l'intégrer au centre de ma vue sur le canevas Excalidraw actif,
**Afin de** construire visuellement mes processus de manière fluide, tout en générant automatiquement la structure de données sous-jacente nécessaire pour une documentation et une automatisation futures.

#### User Story 2 (Sélection de template)
**En tant que** Product/Process Designer,
**Je veux** pouvoir choisir parmi différents templates disponibles lors de la création d'un objet,
**Afin de** créer rapidement des objets de différents types (Process, Data, Interface, etc.) avec la structure appropriée.

#### User Story 3 (Mémorisation)
**En tant que** utilisateur régulier,
**Je veux** que le système mémorise mes templates récemment utilisés,
**Afin de** gagner du temps en accédant rapidement à mes templates favoris sans parcourir toute la liste.

### 4. Analyse Technique de l'Intégration Manuelle

L'observation d'une intégration manuelle réussie révèle la mécanique interne précise d'Excalidraw :

1.  Une nouvelle section **`## Embedded Files`** est créée dans le fichier Markdown du dessin, positionnée entre `## Text Elements` et `## Drawing`.
2.  Cette section contient un manifeste liant un **token de signature (hash)** au chemin du fichier intégré.
    *   Exemple : `de59b03237d1e12e71b4fdb15566266f919fe31c: [[Chemin/Vers/Objet.md]]`
3.  Dans la section `## Drawing`, le JSON de la scène est modifié pour inclure un nouvel élément de type **`image`**.
4.  Cet élément `image` contient une clé **`"fileId"`** dont la valeur est le **token de signature** correspondant, assurant ainsi la liaison entre l'objet visuel et sa source.

Toute solution automatisée devra impérativement reproduire cette structure en trois parties (ajout à la section `Embedded Files`, ajout de l'élément `image` au JSON, et synchronisation du `fileId`).

### 5. Brief sur les Tentatives et Échecs

Les multiples tentatives ont révélé des instabilités et des conflits de contexte entre Templater et l'API Excalidraw Automate (`ea`).

*   **Tentative 1 : API Excalidraw directe (`addEmbeddable`)**
    *   **Échec :** Les fonctions se sont révélées inexistantes ou ont échoué en raison d'un contexte non défini (`targetView not set`). L'API, appelée depuis Templater, est instable et n'arrive pas à effectuer la triple modification (section `Embedded Files`, JSON, `fileId`) requise.

*   **Tentative 2 : Contournement par `app.vault.append()`**
    *   **Succès partiel :** Le fichier a été créé au bon endroit et le dessin modifié pour y ajouter `![[...]]`.
    *   **Échec :** Le rafraîchissement de la vue pour qu'Excalidraw interprète le lien `![[...]]` et génère la structure complète (section `Embedded Files`, `fileId`) a échoué. Les fonctions de l'API pour forcer ce rafraîchissement se sont révélées tout aussi instables.

### 6. Apprentissages Clés

1.  **L'API Excalidraw Automate (`ea`) est une boîte noire instable** lorsqu'elle est pilotée par un script Templater externe. Ses fonctions ne sont pas fiables pour des opérations complexes comme l'intégration.
2.  **La manipulation directe du fichier est plus robuste**, mais incomplète si elle ne reproduit pas la structure précise d'Excalidraw (JSON + section `Embedded Files`).
3.  **Les fonctions de base d'Obsidian (`app.workspace`, `app.vault`) sont la seule source de vérité fiable** pour manipuler les fichiers et connaître le contexte.

### 7. Nouvelles Fonctionnalités (Version 1.2)

#### 7.1 Sélection Dynamique de Templates

**Exigences fonctionnelles :**
1. **Découverte automatique** : Scanner le dossier `Templates/` pour lister tous les fichiers compatibles
2. **Interface de sélection** : Menu déroulant ou suggérer pour choisir le template
3. **Templates récents** : Afficher les 3-5 derniers templates utilisés en haut de la liste
4. **Catégorisation** : Support des sous-dossiers (ex: `Templates/Process/`, `Templates/Data/`)
5. **Template par défaut** : Fallback sur "Embedded-Object-Template" si aucun template sélectionné

**Exigences techniques :**
- Utiliser `app.vault.getFiles()` ou `app.vault.getMarkdownFiles()` pour lister les templates
- Stocker les préférences dans `localStorage` ou un fichier de configuration
- Validation du format Excalidraw avant utilisation
- Cache de la liste des templates pour optimiser les performances

#### 7.2 Mémorisation et Personnalisation

**Fonctionnalités :**
1. **Dernier template utilisé** : Option pour réutiliser automatiquement le dernier template
2. **Templates favoris** : Marquer certains templates comme favoris
3. **Raccourcis personnalisés** : Assigner des raccourcis à des templates spécifiques

**Stockage des préférences :**
```javascript
{
  "lastUsedTemplate": "Templates/Process/Step.md",
  "recentTemplates": ["Template1.md", "Template2.md"],
  "favoriteTemplates": ["MainTemplate.md"],
  "templateShortcuts": {
    "Cmd+Shift+1": "Templates/Process.md",
    "Cmd+Shift+2": "Templates/Data.md"
  }
}
```

### 8. Orientations Techniques Possibles (Historique)

1.  **Option A (La plus Robuste) : Script Excalidraw Pur**
    *   **Concept :** Abandonner Templater comme "pilote". Le script doit être un **script du moteur Excalidraw natif**, placé dans le dossier des scripts d'Excalidraw (voir `ea-scripts/` sur le GitHub).
    *   **Avantages :** Le script s'exécutera dans un contexte 100% Excalidraw. L'objet `ea` sera stable et correctement initialisé. Il n'y aura plus de conflit. L'API `ea` devrait alors être capable d'effectuer correctement la triple modification requise.
    *   **Défis :** Il faudra ré-implémenter la création de fichier depuis un template en utilisant l'API de base d'Obsidian, mais c'est la garantie d'une solution stable et propre.

2.  **Option B (L'Ingénierie Inverse) : Script Templater de Haute Précision**
    *   **Concept :** Créer un script Templater qui **simule manuellement et parfaitement** ce que fait Excalidraw en interne.
    *   **Séquence :**
        1.  Créer le nouveau fichier `Objet.md` (partie maîtrisée).
        2.  Lire le contenu du fichier `Brouillon.md` actif.
        3.  Générer un token de signature (hash) unique.
        4.  Créer un nouvel objet `image` en JSON avec ce `fileId`.
        5.  Insérer cet objet `image` dans le JSON de la section `## Drawing`.
        6.  Insérer la ligne `hash: [[Chemin/Objet.md]]` dans la section `## Embedded Files` (en la créant si elle n'existe pas).
        7.  Ré-écrire la totalité du fichier `Brouillon.md` avec ces modifications.
        8.  Forcer le rafraîchissement de la vue avec `leaf.openFile()`.
    *   **Avantages :** Reste dans l'écosystème Templater.
    *   **Défis :** Extrêmement complexe, fragile aux mises à jour du plugin Excalidraw, et risque de corruption de fichiers. **Non recommandé.**

### 9. Roadmap de Développement

#### Phase 1 - Version 1.0.0 ✅ (Complétée)
- Création d'objets avec template fixe
- Positionnement intelligent au centre
- Format `|100%` pour affichage correct
- Sauvegarde forcée et rafraîchissement

#### Phase 2 - Version 1.2.0 ✅ (Complétée)
- Sélection dynamique de templates
- Interface de sélection intuitive
- Templates récents et mémorisation
- Support des catégories
- Gestion intelligente des doublons

#### Phase 3 - Version 1.3.0 ✅ (Complétée - Version Stable)
- Toggle programmatique pour affichage immédiat
- Résolution définitive du bug d'affichage
- Multiples méthodes de rafraîchissement
- Optimisation des performances
- Version de production stable

#### Phase 4 - Version 2.0.0 (Vision long terme)
- Création de templates depuis objets existants
- Synchronisation des templates entre vaults
- Templates partagés (GitHub, communauté)
- IA pour suggestion de templates contextuels
