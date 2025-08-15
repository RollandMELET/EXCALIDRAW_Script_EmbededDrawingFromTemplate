# Plan d'implémentation : Sélection de Template Dynamique

## 🎯 Objectif
Permettre à l'utilisateur de choisir parmi plusieurs templates disponibles lors de la création d'un objet, avec un système de mémorisation et de catégorisation.

## 📋 Étapes d'implémentation

### 1. **Détection et listage des templates**
- Scanner le dossier `Templates/` pour trouver tous les fichiers `.md` et `.excalidraw`
- Créer une fonction `getAvailableTemplates()` qui retourne la liste des templates
- Gérer le cas où le dossier Templates n'existe pas

### 2. **Interface de sélection améliorée**
- Remplacer le template fixe par un système de sélection
- Options d'implémentation :
  - **Option A**: Menu suggérer avec `app.workspace.getSuggestions()` (plus natif Obsidian)
  - **Option B**: Double prompt (1: choisir template, 2: nommer l'objet)
  - **Option C**: Syntaxe spéciale dans le nom (ex: `MonObjet@template1`)

### 3. **Système de template par défaut intelligent**
- Mémoriser le dernier template utilisé dans les settings
- Template "Embedded-Object-Template" comme fallback ultime
- Ordre de priorité :
  1. Template sélectionné par l'utilisateur
  2. Dernier template utilisé (si option activée)
  3. Template par défaut configuré
  4. Template intégré dans le script

### 4. **Catégorisation des templates (optionnel)**
- Support des sous-dossiers dans `Templates/`
- Ex: `Templates/Process/`, `Templates/Data/`, `Templates/UI/`
- Affichage hiérarchique dans le menu de sélection

### 5. **Validation et feedback**
- Vérifier que le template sélectionné est valide (format Excalidraw)
- Message d'erreur clair si template invalide
- Prévisualisation rapide du template (nom + description)

## 🔧 Modifications du code

### Fichiers à modifier :
1. **embed-object-from-template.md** (script principal)
   - Ajouter `getAvailableTemplates()`
   - Modifier le workflow de sélection
   - Implémenter la mémorisation du dernier choix

### Nouvelles fonctions :
```javascript
// Lister les templates disponibles
async function getAvailableTemplates() {
    // Scanner le dossier Templates/
    // Retourner un array d'objets {name, path, category}
}

// Afficher le menu de sélection
async function selectTemplate(templates) {
    // Utiliser utils.suggester() ou utils.inputPrompt()
    // Retourner le template sélectionné
}

// Sauvegarder le dernier template utilisé
function saveLastUsedTemplate(templatePath) {
    // Stocker dans localStorage ou settings
}

// Charger le dernier template utilisé
function getLastUsedTemplate() {
    // Récupérer depuis localStorage ou settings
}
```

## ⚡ Workflow utilisateur final

1. **Raccourci clavier** → Script activé
2. **Menu de sélection** apparaît avec :
   - Templates récents (3-5 derniers)
   - Séparateur
   - Tous les templates disponibles
   - Option "Template par défaut"
3. **Sélection du template**
4. **Saisie du nom** de l'objet
5. **Création et intégration** instantanée

## 🎨 Options d'amélioration future

- **Aperçu du template** : Miniature ou description
- **Templates favoris** : Marquer certains templates comme favoris
- **Raccourcis directs** : Assigner des raccourcis à des templates spécifiques
- **Import de templates** : Depuis GitHub ou URL externe
- **Création de template** : À partir d'un objet existant

## ✅ Tests à prévoir

1. Dossier Templates vide
2. Dossier Templates inexistant
3. Templates invalides (non-Excalidraw)
4. Sélection rapide (mémorisation)
5. Annulation de la sélection
6. Performance avec beaucoup de templates (50+)

## 📊 Priorités d'implémentation

### Phase 1 - MVP (Version 1.1.0)
- [x] Listage basique des templates
- [x] Menu de sélection simple
- [x] Template par défaut comme fallback

### Phase 2 - Améliorations (Version 1.2.0)
- [ ] Mémorisation du dernier template
- [ ] Templates récents
- [ ] Validation des templates

### Phase 3 - Avancé (Version 1.3.0)
- [ ] Catégorisation
- [ ] Templates favoris
- [ ] Aperçu des templates

## 🔐 Considérations techniques

### Stockage des préférences
- **Option 1**: LocalStorage d'Obsidian
  ```javascript
  localStorage.setItem('excalidraw-last-template', templatePath);
  ```
- **Option 2**: Plugin settings (nécessite modification du plugin)
- **Option 3**: Fichier de configuration `.excalidraw-script-config.json`

### Performance
- Cache de la liste des templates (rafraîchi toutes les 5 minutes)
- Chargement asynchrone des templates
- Limite de 100 templates affichés simultanément

### Compatibilité
- Maintenir la rétrocompatibilité avec l'ancienne méthode
- Support des anciens formats de templates
- Migration automatique des anciens settings