/*
Script Excalidraw pour créer et intégrer des objets depuis un template
Version 1.2.7 - Interface de doublons finalisée
Author: Rolland MELET & Claude Code
Date: 2025-08-15
Changelog:
- v1.2.7: Approche simplifiée en deux étapes pour les doublons
- v1.2.6: Fix titre "false" et permettre saisie directe de nouveaux noms
- v1.2.5: Utilisation de utils.suggester() pour un menu de sélection élégant
- v1.2.4: Fix - Entrée accepte maintenant correctement la suggestion par défaut
- v1.2.3: Correction du message (Tab ne fonctionne pas avec inputPrompt)
- v1.2.2: Tentative d'ajout de Tab (non fonctionnel - limitation API)
- v1.2.1: Ajout de la gestion des doublons avec proposition automatique de nom
- v1.2.0: Ajout de la sélection dynamique de templates
- v1.0.0: Version initiale stable
*/

// Configuration
const TEMPLATES_FOLDER = "Templates";
const DEFAULT_TEMPLATE_NAME = "Embedded-Object-Template";
const STORAGE_KEY = "excalidraw-embed-preferences";

// Template par défaut intégré (fallback ultime)
const TEMPLATE_DEFAULT = `---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠==


# Text Elements

# Embedded files

# Drawing
\`\`\`json
{
	"type": "excalidraw",
	"version": 2,
	"source": "https://github.com/zsviczian/obsidian-excalidraw-plugin",
	"elements": [],
	"appState": {
		"theme": "light",
		"viewBackgroundColor": "#ffffff",
		"currentItemStrokeColor": "#000000",
		"currentItemBackgroundColor": "transparent",
		"currentItemFillStyle": "hachure",
		"currentItemStrokeWidth": 1,
		"currentItemStrokeStyle": "solid",
		"currentItemRoughness": 1,
		"currentItemOpacity": 100,
		"currentItemFontFamily": 1,
		"currentItemFontSize": 20,
		"currentItemTextAlign": "left",
		"currentItemStartArrowhead": null,
		"currentItemEndArrowhead": "arrow",
		"scrollX": 0,
		"scrollY": 0,
		"zoom": {
			"value": 1
		},
		"gridSize": null,
		"gridColor": {
			"Bold": "#C9C9C9FF",
			"Regular": "#EDEDEDFF"
		}
	},
	"files": {}
}
\`\`\`
%%`;

// Générer un ID unique pour le fichier embed
function generateFileId() {
    const chars = 'abcdef0123456789';
    let result = '';
    for (let i = 0; i < 40; i++) {
        result += chars[Math.floor(Math.random() * chars.length)];
    }
    return result;
}

// Charger les préférences sauvegardées
function loadPreferences() {
    try {
        const stored = localStorage.getItem(STORAGE_KEY);
        if (stored) {
            return JSON.parse(stored);
        }
    } catch (e) {
        console.error("Erreur lors du chargement des préférences:", e);
    }
    return {
        lastUsedTemplate: null,
        recentTemplates: []
    };
}

// Sauvegarder les préférences
function savePreferences(prefs) {
    try {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(prefs));
    } catch (e) {
        console.error("Erreur lors de la sauvegarde des préférences:", e);
    }
}

// Mettre à jour les templates récents
function updateRecentTemplates(templatePath, prefs) {
    // Retirer le template s'il existe déjà dans la liste
    prefs.recentTemplates = prefs.recentTemplates.filter(t => t !== templatePath);
    
    // Ajouter en début de liste
    prefs.recentTemplates.unshift(templatePath);
    
    // Garder seulement les 5 derniers
    prefs.recentTemplates = prefs.recentTemplates.slice(0, 5);
    
    // Mettre à jour le dernier utilisé
    prefs.lastUsedTemplate = templatePath;
    
    savePreferences(prefs);
}

// Scanner le dossier Templates pour trouver les fichiers disponibles
async function getAvailableTemplates() {
    const templates = [];
    
    try {
        // Vérifier si le dossier Templates existe
        const templatesFolder = app.vault.getAbstractFileByPath(TEMPLATES_FOLDER);
        
        if (!templatesFolder) {
            console.log("Dossier Templates non trouvé");
            return templates;
        }
        
        // Récupérer tous les fichiers du vault
        const allFiles = app.vault.getFiles();
        
        // Filtrer pour garder seulement ceux dans le dossier Templates
        for (const file of allFiles) {
            if (file.path.startsWith(TEMPLATES_FOLDER + "/")) {
                // Vérifier si c'est un fichier Markdown ou Excalidraw
                if (file.extension === "md" || file.extension === "excalidraw") {
                    // Vérifier si c'est un fichier Excalidraw valide
                    const content = await app.vault.read(file);
                    if (content.includes("excalidraw-plugin: parsed") || file.extension === "excalidraw") {
                        const relativePath = file.path.substring(TEMPLATES_FOLDER.length + 1);
                        const category = relativePath.includes("/") ? 
                            relativePath.substring(0, relativePath.lastIndexOf("/")) : "";
                        
                        templates.push({
                            name: file.basename,
                            path: file.path,
                            category: category,
                            displayName: category ? `${category}/${file.basename}` : file.basename
                        });
                    }
                }
            }
        }
        
        // Trier par catégorie puis par nom
        templates.sort((a, b) => {
            if (a.category !== b.category) {
                return a.category.localeCompare(b.category);
            }
            return a.name.localeCompare(b.name);
        });
        
    } catch (e) {
        console.error("Erreur lors du scan des templates:", e);
    }
    
    return templates;
}

// Afficher le menu de sélection de template
async function selectTemplate() {
    const prefs = loadPreferences();
    const templates = await getAvailableTemplates();
    
    // Construire la liste des options
    const options = [];
    const templateMap = {};
    
    // Ajouter les templates récents en premier (s'ils existent encore)
    if (prefs.recentTemplates.length > 0) {
        const recentTemplates = [];
        for (const recentPath of prefs.recentTemplates) {
            const template = templates.find(t => t.path === recentPath);
            if (template) {
                const recentOption = `⭐ ${template.displayName}`;
                recentTemplates.push(recentOption);
                templateMap[recentOption] = template;
            }
        }
        
        if (recentTemplates.length > 0) {
            options.push(...recentTemplates);
            options.push("───────────────"); // Séparateur
        }
    }
    
    // Ajouter tous les templates
    for (const template of templates) {
        const option = template.displayName;
        options.push(option);
        templateMap[option] = template;
    }
    
    // Ajouter l'option template par défaut
    if (options.length > 0) {
        options.push("───────────────"); // Séparateur
    }
    options.push("📦 Template par défaut (intégré)");
    
    // Si aucun template disponible, utiliser directement le template par défaut
    if (templates.length === 0 && prefs.recentTemplates.length === 0) {
        new Notice("Aucun template trouvé dans le dossier Templates/. Utilisation du template par défaut.");
        return null;
    }
    
    // Afficher le menu de sélection
    let selectedOption;
    try {
        // Utiliser utils.suggester si disponible (plus joli)
        if (typeof utils !== 'undefined' && utils.suggester) {
            selectedOption = await utils.suggester(
                options,
                options,
                false,
                "Sélectionnez un template"
            );
        } else {
            // Fallback sur inputPrompt avec la liste numérotée
            const numberedOptions = options.map((opt, idx) => 
                opt.startsWith("───") ? opt : `${idx + 1}. ${opt}`
            ).join("\n");
            
            const choice = await utils.inputPrompt(
                `Sélectionnez un template (entrez le numéro):\n\n${numberedOptions}\n\nVotre choix:`
            );
            
            if (choice && !isNaN(choice)) {
                const index = parseInt(choice) - 1;
                if (index >= 0 && index < options.length && !options[index].startsWith("───")) {
                    selectedOption = options[index];
                }
            }
        }
    } catch (e) {
        console.error("Erreur lors de la sélection:", e);
        return null;
    }
    
    // Traiter la sélection
    if (!selectedOption) {
        return null; // Annulation
    }
    
    if (selectedOption === "📦 Template par défaut (intégré)") {
        return { isDefault: true };
    }
    
    if (selectedOption.startsWith("───")) {
        return null; // Séparateur sélectionné par erreur
    }
    
    // Retirer le préfixe ⭐ si présent
    const cleanOption = selectedOption.startsWith("⭐ ") ? 
        selectedOption.substring(2) : selectedOption;
    
    const selectedTemplate = templateMap[selectedOption] || templateMap[cleanOption];
    
    if (selectedTemplate) {
        // Mettre à jour les templates récents
        updateRecentTemplates(selectedTemplate.path, prefs);
        return selectedTemplate;
    }
    
    return null;
}

// Obtenir le contenu du template sélectionné
async function getTemplateContent(templateInfo) {
    // Si c'est le template par défaut
    if (templateInfo.isDefault) {
        return TEMPLATE_DEFAULT;
    }
    
    try {
        const templateFile = app.vault.getAbstractFileByPath(templateInfo.path);
        if (templateFile) {
            const content = await app.vault.read(templateFile);
            // Vérifier si c'est un fichier Excalidraw valide
            if (!content.includes("excalidraw-plugin: parsed")) {
                new Notice("Le template sélectionné n'est pas un fichier Excalidraw valide.");
                return TEMPLATE_DEFAULT;
            }
            return content;
        }
    } catch (e) {
        console.error("Erreur lors de la lecture du template:", e);
        new Notice("Erreur lors de la lecture du template. Utilisation du template par défaut.");
    }
    
    return TEMPLATE_DEFAULT;
}

// Obtenir le centre de la vue actuelle
function getViewCenter(excalidrawContent) {
    try {
        // Extraire appState du contenu
        const appStateMatch = excalidrawContent.match(/"appState":\s*{([^}]*)}/);
        if (appStateMatch) {
            const appStateStr = `{${appStateMatch[1]}}`;
            // Extraire scrollX, scrollY et zoom
            const scrollXMatch = appStateStr.match(/"scrollX":\s*([-\d.]+)/);
            const scrollYMatch = appStateStr.match(/"scrollY":\s*([-\d.]+)/);
            const zoomMatch = appStateStr.match(/"zoom":\s*{[^}]*"value":\s*([\d.]+)/);
            
            if (scrollXMatch && scrollYMatch) {
                const scrollX = parseFloat(scrollXMatch[1]);
                const scrollY = parseFloat(scrollYMatch[1]);
                const zoom = zoomMatch ? parseFloat(zoomMatch[1]) : 1;
                
                // Calculer le centre de la vue
                const viewportWidth = 800;
                const viewportHeight = 600;
                
                const centerX = -scrollX + (viewportWidth / 2) / zoom;
                const centerY = -scrollY + (viewportHeight / 2) / zoom;
                
                return { x: centerX, y: centerY };
            }
        }
    } catch (e) {
        // Retourner les valeurs par défaut en cas d'erreur
    }
    
    return { x: 0, y: 0 };
}

// Script principal
(async () => {
    console.log("Script Excalidraw Embed v1.2.0 - Démarrage");
    
    // Étape 1: Sélectionner le template
    const templateInfo = await selectTemplate();
    
    if (!templateInfo) {
        new Notice("Sélection de template annulée");
        return;
    }
    
    // Afficher le template sélectionné
    if (templateInfo.isDefault) {
        console.log("Template par défaut sélectionné");
    } else {
        console.log(`Template sélectionné: ${templateInfo.displayName}`);
    }
    
    // Étape 2: Demander le nom de l'objet
    let objectName = await utils.inputPrompt("Nom de l'objet de workflow:");
    if (!objectName) {
        new Notice("Création annulée");
        return;
    }
    
    // Obtenir le fichier actif
    const activeFile = app.workspace.getActiveFile();
    if (!activeFile) {
        new Notice("Aucun fichier actif");
        return;
    }
    
    // Fonction pour générer un nom unique
    async function getUniqueFileName(baseName, folder) {
        let fileName = `${baseName}.md`;
        let filePath = folder ? `${folder.path}/${fileName}` : fileName;
        
        // Si le fichier n'existe pas, retourner directement
        if (!app.vault.getAbstractFileByPath(filePath)) {
            return { name: baseName, fileName: fileName, filePath: filePath };
        }
        
        // Le fichier existe, proposer des alternatives
        const choices = [];
        let counter = 1;
        
        // Générer plusieurs suggestions
        // Style Windows : nom(1), nom(2)
        for (let i = 1; i <= 3; i++) {
            const suggestion = `${baseName}(${i})`;
            const suggestionPath = folder ? 
                `${folder.path}/${suggestion}.md` : 
                `${suggestion}.md`;
            
            // Vérifier si ce nom est disponible
            if (!app.vault.getAbstractFileByPath(suggestionPath)) {
                choices.push(suggestion);
                if (choices.length >= 2) break; // On veut au moins 2 options
            }
        }
        
        // Style version : nom_v2
        const versionName = `${baseName}_v2`;
        const versionPath = folder ? 
            `${folder.path}/${versionName}.md` : 
            `${versionName}.md`;
        if (!app.vault.getAbstractFileByPath(versionPath)) {
            choices.push(versionName);
        }
        
        // Style copie : nom_copy
        const copyName = `${baseName}_copy`;
        const copyPath = folder ? 
            `${folder.path}/${copyName}.md` : 
            `${copyName}.md`;
        if (!app.vault.getAbstractFileByPath(copyPath)) {
            choices.push(copyName);
        }
        
        // Ajouter l'option pour choisir un nom personnalisé
        choices.push("💡 Saisir un autre nom...");
        
        // Approche simplifiée : Menu de sélection sans saisie libre
        let selected;
        try {
            if (typeof utils !== 'undefined' && utils.suggester) {
                // Créer les labels d'affichage
                const displayLabels = choices.map(choice => {
                    if (choice === "💡 Saisir un autre nom...") {
                        return choice;
                    }
                    return `✓ ${choice}`;
                });
                
                // Utiliser suggester simple sans allowCustomValue
                selected = await utils.suggester(
                    displayLabels,  // Ce qui est affiché
                    choices,        // Les valeurs retournées
                    false,          // Pas de saisie libre ici
                    `"${baseName}.md" existe déjà. Choisissez une option :`
                );
            } else {
                // Fallback si suggester n'est pas disponible
                console.log("utils.suggester non disponible, utilisation du fallback");
                // Utiliser automatiquement la première suggestion
                selected = choices[0];
                new Notice(`Fichier créé sous le nom "${selected}.md" (nom déjà existant)`);
            }
        } catch (e) {
            console.error("Erreur avec suggester:", e);
            // En cas d'erreur, utiliser la première suggestion
            selected = choices[0];
        }
        
        // Traiter la sélection
        if (!selected) {
            return null; // Annulation
        }
        
        if (selected === "💡 Saisir un autre nom...") {
            // Deuxième étape : Demander un nom personnalisé
            let customName = await utils.inputPrompt(
                `Le fichier "${baseName}.md" existe déjà.\n\n` +
                `Suggestions disponibles : ${choices.slice(0, -1).join(", ")}\n\n` +
                `Entrez un nouveau nom :`,
                baseName + "_new"  // Suggestion par défaut
            );
            
            if (!customName) {
                return null; // Annulation
            }
            
            // Vérifier récursivement le nouveau nom
            return await getUniqueFileName(customName, folder);
        }
        
        // Utiliser le nom sélectionné dans la liste
        const selectedFileName = `${selected}.md`;
        const selectedFilePath = folder ? 
            `${folder.path}/${selectedFileName}` : 
            selectedFileName;
        
        return { 
            name: selected, 
            fileName: selectedFileName, 
            filePath: selectedFilePath 
        };
    }
    
    // Créer le nouveau fichier depuis le template
    const folder = activeFile.parent;
    
    // Obtenir un nom de fichier unique
    const fileInfo = await getUniqueFileName(objectName, folder);
    
    if (!fileInfo) {
        new Notice("Création annulée");
        return;
    }
    
    // Mettre à jour le nom de l'objet si nécessaire
    objectName = fileInfo.name;
    const newFileName = fileInfo.fileName;
    const newFilePath = fileInfo.filePath;
    
    try {
        // Obtenir le contenu du template
        const templateContent = await getTemplateContent(templateInfo);
        const content = templateContent.replace(/\${objectName}/g, objectName);
        
        // Créer le nouveau fichier avec gestion d'erreur
        let newFile;
        try {
            newFile = await app.vault.create(newFilePath, content);
        } catch (createError) {
            // Si la création échoue malgré nos vérifications
            console.error("Erreur lors de la création du fichier:", createError);
            new Notice(`Erreur: Impossible de créer le fichier "${newFileName}". ${createError.message}`);
            return;
        }
        
        // Générer un ID unique pour l'embed
        const fileId = generateFileId();
        
        // Lire le contenu actuel du fichier Excalidraw
        let excalidrawContent = await app.vault.read(activeFile);
        
        // Parser le JSON du dessin
        const drawingMatch = excalidrawContent.match(/## Drawing\n```json\n([\s\S]*?)\n```/);
        if (!drawingMatch) {
            new Notice("Erreur: Structure du fichier non reconnue");
            return;
        }
        
        const drawingData = JSON.parse(drawingMatch[1]);
        
        // Calculer la position - centré dans la vue actuelle
        const viewCenter = getViewCenter(excalidrawContent);
        let x = viewCenter.x - 200; // Centrer horizontalement (largeur/2)
        let y = viewCenter.y - 150; // Centrer verticalement (hauteur/2)
        
        // Si des embeds existent déjà, décaler légèrement pour éviter la superposition
        if (drawingData.elements && drawingData.elements.length > 0) {
            const embedElements = drawingData.elements.filter(el => el.type === "image" && el.fileId);
            if (embedElements.length > 0) {
                // Décalage en cascade pour chaque nouvel embed
                const offset = (embedElements.length % 5) * 30;
                x += offset;
                y += offset;
            }
        }
        
        // Créer l'élément image pour l'embed
        const imageElement = {
            id: generateFileId().substring(0, 16),
            type: "image",
            x: x,
            y: y,
            width: 400,
            height: 300,
            angle: 0,
            strokeColor: "transparent",
            backgroundColor: "transparent",
            fillStyle: "hachure",
            strokeWidth: 1,
            strokeStyle: "solid",
            roughness: 1,
            opacity: 100,
            groupIds: [],
            frameId: null,
            roundness: null,
            seed: Math.floor(Math.random() * 2000000000),
            versionNonce: Math.floor(Math.random() * 2000000000),
            isDeleted: false,
            boundElements: null,
            updated: Date.now(),
            link: null,
            locked: false,
            status: "pending",
            fileId: fileId,
            scale: [1, 1]
        };
        
        // Ajouter l'élément au dessin
        drawingData.elements = drawingData.elements || [];
        drawingData.elements.push(imageElement);
        
        // Gérer la section Embedded Files avec le format 100% pour forcer le mode Excalidraw
        const embedEntry = `${fileId}: [[${objectName}|100%]]`;
        
        // Rechercher la section Embedded Files
        const embeddedFilesRegex = /## Embedded Files\n([\s\S]*?)(?=\n%%|$)/;
        const embeddedFilesMatch = excalidrawContent.match(embeddedFilesRegex);
        
        if (embeddedFilesMatch) {
            // La section existe déjà
            let existingContent = embeddedFilesMatch[1].trim();
            
            // Ajouter la nouvelle entrée
            const newContent = existingContent ? 
                `${existingContent}\n${embedEntry}` : 
                embedEntry;
            
            // Remplacer la section
            excalidrawContent = excalidrawContent.replace(
                embeddedFilesRegex, 
                `## Embedded Files\n${newContent}\n`
            );
        } else {
            // Créer la section Embedded Files si elle n'existe pas
            const textElementsIndex = excalidrawContent.indexOf("## Text Elements");
            const drawingIndex = excalidrawContent.indexOf("## Drawing");
            const percentIndex = excalidrawContent.indexOf("%%");
            
            let insertPosition;
            
            // Insérer avant %% si présent, sinon avant ## Drawing
            if (percentIndex > -1 && percentIndex < drawingIndex) {
                insertPosition = percentIndex;
            } else if (drawingIndex > -1) {
                insertPosition = drawingIndex;
            } else {
                return;
            }
            
            // Insérer la nouvelle section
            excalidrawContent = 
                excalidrawContent.slice(0, insertPosition) + 
                `## Embedded Files\n${embedEntry}\n\n` +
                excalidrawContent.slice(insertPosition);
        }
        
        // Remplacer le JSON du dessin
        const newDrawingJson = JSON.stringify(drawingData, null, "\t");
        excalidrawContent = excalidrawContent.replace(
            /## Drawing\n```json\n[\s\S]*?\n```/,
            `## Drawing\n\`\`\`json\n${newDrawingJson}\n\`\`\``
        );
        
        // Sauvegarder les modifications
        await app.vault.modify(activeFile, excalidrawContent);
        
        // Forcer la sauvegarde immédiate sur le disque
        await app.vault.adapter.write(activeFile.path, excalidrawContent);
        
        // Attendre que la sauvegarde soit complète
        await new Promise(resolve => setTimeout(resolve, 500));
        
        // Rafraîchir la vue
        try {
            const activeLeaf = app.workspace.activeLeaf;
            if (activeLeaf) {
                await activeLeaf.openFile(activeFile);
            }
        } catch (e) {
            // Ignorer les erreurs de rafraîchissement
        }
        
        // Message de succès avec info sur le template utilisé
        const templateName = templateInfo.isDefault ? 
            "template par défaut" : 
            templateInfo.displayName;
        new Notice(`✅ Objet "${objectName}" créé avec ${templateName}!`);
        
        console.log(`Objet créé avec succès: ${objectName} (Template: ${templateName})`);
        
    } catch (error) {
        console.error("Erreur lors de la création:", error);
        new Notice(`Erreur: ${error.message}`);
    }
})();