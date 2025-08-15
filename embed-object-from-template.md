/*
excalidraw-script-icon: <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <path d="M23 18C23 18.75 22.79 19.46 22.42 20.06C22.21 20.42 21.94 20.74 21.63 21C20.93 21.63 20.01 22 19 22C17.78 22 16.69 21.45 15.97 20.59C15.95 20.56 15.92 20.54 15.9 20.51C15.78 20.37 15.67 20.22 15.58 20.06C15.21 19.46 15 18.75 15 18C15 16.74 15.58 15.61 16.5 14.88C17.19 14.33 18.06 14 19 14C20 14 20.9 14.36 21.6 14.97C21.72 15.06 21.83 15.17 21.93 15.28C22.59 16 23 16.95 23 18Z" stroke="#292D32" stroke-width="1.5" stroke-miterlimit="10" stroke-linecap="round" stroke-linejoin="round"></path> <path d="M20.4898 17.98H17.5098" stroke="#292D32" stroke-width="1.5" stroke-miterlimit="10" stroke-linecap="round" stroke-linejoin="round"></path> <path d="M19 16.52V19.51" stroke="#292D32" stroke-width="1.5" stroke-miterlimit="10" stroke-linecap="round" stroke-linejoin="round"></path> <g opacity="0.4"> <path d="M3.16992 7.43994L11.9999 12.5499L20.7699 7.46991" stroke="#292D32" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path> <path d="M12 21.61V12.54" stroke="#292D32" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path> <path d="M21.6106 9.17V14.83C21.6106 14.88 21.6106 14.92 21.6006 14.97C20.9006 14.36 20.0006 14 19.0006 14C18.0606 14 17.1906 14.33 16.5006 14.88C15.5806 15.61 15.0006 16.74 15.0006 18C15.0006 18.75 15.2106 19.46 15.5806 20.06C15.6706 20.22 15.7806 20.37 15.9006 20.51L14.0706 21.52C12.9306 22.16 11.0706 22.16 9.93062 21.52L4.59062 18.56C3.38062 17.89 2.39062 16.21 2.39062 14.83V9.17C2.39062 7.79 3.38062 6.11002 4.59062 5.44002L9.93062 2.48C11.0706 1.84 12.9306 1.84 14.0706 2.48L19.4106 5.44002C20.6206 6.11002 21.6106 7.79 21.6106 9.17Z" stroke="#292D32" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"></path> </g> </g></svg>
Script Excalidraw pour créer et intégrer des objets depuis un template
Excalidraw Script for creating and embedding objects from templates
Version 1.3.0 - Bilingual Edition
Author: Rolland MELET & Claude Code
Date: 2025-08-15
Changelog:
- v1.3.0: Bilingual support (EN/FR) + Custom icon + Programmatic toggle for instant display
- v1.2.9: Fix critical - Sync filename/embed entry  
- v1.2.8: Fix immediate display (status "saved" + operation order)
- v1.2.7: Simplified two-step approach for duplicates
- v1.2.6: Fix "false" title and allow direct name input
- v1.2.5: Use utils.suggester() for elegant selection menu
- v1.2.4: Fix - Enter now correctly accepts default suggestion
- v1.2.3: Message correction (Tab doesn't work with inputPrompt)
- v1.2.2: Tab attempt (non-functional - API limitation)
- v1.2.1: Add duplicate handling with automatic name suggestion
- v1.2.0: Add dynamic template selection
- v1.0.0: Initial stable version
*/

// === CONFIGURATION ===
const TEMPLATES_FOLDER = "Templates";
const DEFAULT_TEMPLATE_NAME = "Embedded-Object-Template";
const STORAGE_KEY = "excalidraw-embed-preferences";

// === INTERNATIONALIZATION ===
// Detect user language
const getUserLanguage = () => {
    const lang = moment?.locale() || 
                 navigator.language || 
                 app.locale || 
                 'en';
    return lang.startsWith('fr') ? 'fr' : 'en';
};

const lang = getUserLanguage();

// Translation dictionary
const i18n = {
    en: {
        selectTemplate: "Select a template",
        objectName: "Workflow object name:",
        defaultTemplate: "📦 Default template (built-in)",
        fileExists: "already exists. Choose an option:",
        enterCustomName: "💡 Enter another name...",
        suggestionsAvailable: "Available suggestions:",
        enterNewName: "Enter a new name:",
        cancelled: "Creation cancelled",
        noActiveFile: "No active file",
        noTemplateFound: "No template found in Templates/. Using default template.",
        templateSelected: "Template selected:",
        defaultSelected: "Default template selected",
        fileCreatedAs: "File created as",
        objectCreated: "✅ Object",
        createdWith: "created with",
        error: "Error",
        errorCreatingFile: "Unable to create file",
        errorReadingTemplate: "Error reading template. Using default.",
        invalidTemplate: "Selected template is not a valid Excalidraw file.",
        scriptStarting: "Excalidraw Embed Script v1.3.0 - Starting",
        scanningTemplates: "Scanning templates folder",
        templatesFolder: "Templates folder",
        notFound: "not found",
        fileEmbed: "File embed created:",
        errorFileStructure: "Error: File structure not recognized",
        attemptingToggle: "Attempting programmatic toggle...",
        viewDetected: "Excalidraw/Markdown view detected, toggling...",
        toSourceMode: "Switching to source mode",
        toPreviewMode: "Back to preview mode",
        usingSetState: "Using setState to refresh",
        toggleViaReopen: "Toggle via close/reopen file",
        triggeringEvent: "Triggering file-open event",
        toggleComplete: "Programmatic toggle complete",
        toggleError: "Error during programmatic toggle:",
        refreshError: "Error during fallback refresh:",
        suggesterUnavailable: "utils.suggester unavailable, using fallback",
        nameAlreadyExists: "(name already exists)"
    },
    fr: {
        selectTemplate: "Sélectionnez un template",
        objectName: "Nom de l'objet de workflow:",
        defaultTemplate: "📦 Template par défaut (intégré)",
        fileExists: "existe déjà. Choisissez une option :",
        enterCustomName: "💡 Saisir un autre nom...",
        suggestionsAvailable: "Suggestions disponibles :",
        enterNewName: "Entrez un nouveau nom :",
        cancelled: "Création annulée",
        noActiveFile: "Aucun fichier actif",
        noTemplateFound: "Aucun template trouvé dans le dossier Templates/. Utilisation du template par défaut.",
        templateSelected: "Template sélectionné :",
        defaultSelected: "Template par défaut sélectionné",
        fileCreatedAs: "Fichier créé sous le nom",
        objectCreated: "✅ Objet",
        createdWith: "créé avec",
        error: "Erreur",
        errorCreatingFile: "Impossible de créer le fichier",
        errorReadingTemplate: "Erreur lors de la lecture du template. Utilisation du template par défaut.",
        invalidTemplate: "Le template sélectionné n'est pas un fichier Excalidraw valide.",
        scriptStarting: "Script Excalidraw Embed v1.3.0 - Démarrage",
        scanningTemplates: "Scan du dossier Templates",
        templatesFolder: "Dossier Templates",
        notFound: "non trouvé",
        fileEmbed: "Fichier embed créé :",
        errorFileStructure: "Erreur: Structure du fichier non reconnue",
        attemptingToggle: "Tentative de toggle programmatique...",
        viewDetected: "Vue Excalidraw/Markdown détectée, toggle en cours...",
        toSourceMode: "Passage en mode source",
        toPreviewMode: "Retour en mode preview",
        usingSetState: "Utilisation de setState pour rafraîchir",
        toggleViaReopen: "Toggle via fermeture/réouverture du fichier",
        triggeringEvent: "Déclenchement d'un événement file-open",
        toggleComplete: "Toggle programmatique terminé",
        toggleError: "Erreur lors du toggle programmatique:",
        refreshError: "Erreur lors du rafraîchissement fallback:",
        suggesterUnavailable: "utils.suggester non disponible, utilisation du fallback",
        nameAlreadyExists: "(nom déjà existant)"
    }
};

// Helper function to get translation
const t = (key) => i18n[lang][key] || i18n['en'][key];

// Default template (fallback)
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

// Generate unique file ID
function generateFileId() {
    const chars = 'abcdef0123456789';
    let result = '';
    for (let i = 0; i < 40; i++) {
        result += chars[Math.floor(Math.random() * chars.length)];
    }
    return result;
}

// Load saved preferences
function loadPreferences() {
    try {
        const stored = localStorage.getItem(STORAGE_KEY);
        if (stored) {
            return JSON.parse(stored);
        }
    } catch (e) {
        console.error("Error loading preferences:", e);
    }
    return {
        lastUsedTemplate: null,
        recentTemplates: []
    };
}

// Save preferences
function savePreferences(prefs) {
    try {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(prefs));
    } catch (e) {
        console.error("Error saving preferences:", e);
    }
}

// Update recent templates
function updateRecentTemplates(templatePath, prefs) {
    prefs.recentTemplates = prefs.recentTemplates.filter(t => t !== templatePath);
    prefs.recentTemplates.unshift(templatePath);
    prefs.recentTemplates = prefs.recentTemplates.slice(0, 5);
    prefs.lastUsedTemplate = templatePath;
    savePreferences(prefs);
}

// Scan Templates folder for available files
async function getAvailableTemplates() {
    const templates = [];
    
    try {
        const templatesFolder = app.vault.getAbstractFileByPath(TEMPLATES_FOLDER);
        
        if (!templatesFolder) {
            console.log(`${t('templatesFolder')} ${t('notFound')}`);
            return templates;
        }
        
        const allFiles = app.vault.getFiles();
        
        for (const file of allFiles) {
            if (file.path.startsWith(TEMPLATES_FOLDER + "/")) {
                if (file.extension === "md" || file.extension === "excalidraw") {
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
        
        templates.sort((a, b) => {
            if (a.category !== b.category) {
                return a.category.localeCompare(b.category);
            }
            return a.name.localeCompare(b.name);
        });
        
    } catch (e) {
        console.error(`${t('error')} ${t('scanningTemplates')}:`, e);
    }
    
    return templates;
}

// Show template selection menu
async function selectTemplate() {
    const prefs = loadPreferences();
    const templates = await getAvailableTemplates();
    
    const options = [];
    const templateMap = {};
    
    // Add recent templates first
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
            options.push("───────────────");
        }
    }
    
    // Add all templates
    for (const template of templates) {
        const option = template.displayName;
        options.push(option);
        templateMap[option] = template;
    }
    
    // Add default template option
    if (options.length > 0) {
        options.push("───────────────");
    }
    options.push(t('defaultTemplate'));
    
    // If no templates available, use default directly
    if (templates.length === 0 && prefs.recentTemplates.length === 0) {
        new Notice(t('noTemplateFound'));
        return null;
    }
    
    // Show selection menu
    let selectedOption;
    try {
        if (typeof utils !== 'undefined' && utils.suggester) {
            selectedOption = await utils.suggester(
                options,
                options,
                false,
                t('selectTemplate')
            );
        } else {
            const numberedOptions = options.map((opt, idx) => 
                opt.startsWith("───") ? opt : `${idx + 1}. ${opt}`
            ).join("\n");
            
            const choice = await utils.inputPrompt(
                `${t('selectTemplate')}:\n\n${numberedOptions}\n\n:`
            );
            
            if (choice && !isNaN(choice)) {
                const index = parseInt(choice) - 1;
                if (index >= 0 && index < options.length && !options[index].startsWith("───")) {
                    selectedOption = options[index];
                }
            }
        }
    } catch (e) {
        console.error(`${t('error')}:`, e);
        return null;
    }
    
    if (!selectedOption) {
        return null;
    }
    
    if (selectedOption === t('defaultTemplate')) {
        return { isDefault: true };
    }
    
    if (selectedOption.startsWith("───")) {
        return null;
    }
    
    const cleanOption = selectedOption.startsWith("⭐ ") ? 
        selectedOption.substring(2) : selectedOption;
    
    const selectedTemplate = templateMap[selectedOption] || templateMap[cleanOption];
    
    if (selectedTemplate) {
        updateRecentTemplates(selectedTemplate.path, prefs);
        return selectedTemplate;
    }
    
    return null;
}

// Get template content
async function getTemplateContent(templateInfo) {
    if (templateInfo.isDefault) {
        return TEMPLATE_DEFAULT;
    }
    
    try {
        const templateFile = app.vault.getAbstractFileByPath(templateInfo.path);
        if (templateFile) {
            const content = await app.vault.read(templateFile);
            if (!content.includes("excalidraw-plugin: parsed")) {
                new Notice(t('invalidTemplate'));
                return TEMPLATE_DEFAULT;
            }
            return content;
        }
    } catch (e) {
        console.error(`${t('errorReadingTemplate')}:`, e);
        new Notice(t('errorReadingTemplate'));
    }
    
    return TEMPLATE_DEFAULT;
}

// Get current view center
function getViewCenter(excalidrawContent) {
    try {
        const appStateMatch = excalidrawContent.match(/"appState":\s*{([^}]*)}/);
        if (appStateMatch) {
            const appStateStr = `{${appStateMatch[1]}}`;
            const scrollXMatch = appStateStr.match(/"scrollX":\s*([-\d.]+)/);
            const scrollYMatch = appStateStr.match(/"scrollY":\s*([-\d.]+)/);
            const zoomMatch = appStateStr.match(/"zoom":\s*{[^}]*"value":\s*([\d.]+)/);
            
            if (scrollXMatch && scrollYMatch) {
                const scrollX = parseFloat(scrollXMatch[1]);
                const scrollY = parseFloat(scrollYMatch[1]);
                const zoom = zoomMatch ? parseFloat(zoomMatch[1]) : 1;
                
                const viewportWidth = 800;
                const viewportHeight = 600;
                
                const centerX = -scrollX + (viewportWidth / 2) / zoom;
                const centerY = -scrollY + (viewportHeight / 2) / zoom;
                
                return { x: centerX, y: centerY };
            }
        }
    } catch (e) {
        // Return default values on error
    }
    
    return { x: 0, y: 0 };
}

// Main script
(async () => {
    console.log(t('scriptStarting'));
    
    // Step 1: Select template
    const templateInfo = await selectTemplate();
    
    if (!templateInfo) {
        new Notice(t('cancelled'));
        return;
    }
    
    // Display selected template
    if (templateInfo.isDefault) {
        console.log(t('defaultSelected'));
    } else {
        console.log(`${t('templateSelected')} ${templateInfo.displayName}`);
    }
    
    // Step 2: Get object name
    let objectName = await utils.inputPrompt(t('objectName'));
    if (!objectName) {
        new Notice(t('cancelled'));
        return;
    }
    
    // Get active file
    const activeFile = app.workspace.getActiveFile();
    if (!activeFile) {
        new Notice(t('noActiveFile'));
        return;
    }
    
    // Function to generate unique filename
    async function getUniqueFileName(baseName, folder) {
        let fileName = `${baseName}.md`;
        let filePath = folder ? `${folder.path}/${fileName}` : fileName;
        
        if (!app.vault.getAbstractFileByPath(filePath)) {
            return { name: baseName, fileName: fileName, filePath: filePath };
        }
        
        const choices = [];
        
        // Generate suggestions - Windows style: name(1), name(2)
        for (let i = 1; i <= 3; i++) {
            const suggestion = `${baseName}(${i})`;
            const suggestionPath = folder ? 
                `${folder.path}/${suggestion}.md` : 
                `${suggestion}.md`;
            
            if (!app.vault.getAbstractFileByPath(suggestionPath)) {
                choices.push(suggestion);
                if (choices.length >= 2) break;
            }
        }
        
        // Version style: name_v2
        const versionName = `${baseName}_v2`;
        const versionPath = folder ? 
            `${folder.path}/${versionName}.md` : 
            `${versionName}.md`;
        if (!app.vault.getAbstractFileByPath(versionPath)) {
            choices.push(versionName);
        }
        
        // Copy style: name_copy
        const copyName = `${baseName}_copy`;
        const copyPath = folder ? 
            `${folder.path}/${copyName}.md` : 
            `${copyName}.md`;
        if (!app.vault.getAbstractFileByPath(copyPath)) {
            choices.push(copyName);
        }
        
        choices.push(t('enterCustomName'));
        
        // Simplified approach: Selection menu without free input
        let selected;
        try {
            if (typeof utils !== 'undefined' && utils.suggester) {
                const displayLabels = choices.map(choice => {
                    if (choice === t('enterCustomName')) {
                        return choice;
                    }
                    return `✓ ${choice}`;
                });
                
                selected = await utils.suggester(
                    displayLabels,
                    choices,
                    false,
                    `"${baseName}.md" ${t('fileExists')}`
                );
            } else {
                console.log(t('suggesterUnavailable'));
                selected = choices[0];
                new Notice(`${t('fileCreatedAs')} "${selected}.md" ${t('nameAlreadyExists')}`);
            }
        } catch (e) {
            console.error(`${t('error')} suggester:`, e);
            selected = choices[0];
        }
        
        if (!selected) {
            return null;
        }
        
        if (selected === t('enterCustomName')) {
            let customName = await utils.inputPrompt(
                `"${baseName}.md" ${t('fileExists')}\n\n` +
                `${t('suggestionsAvailable')} ${choices.slice(0, -1).join(", ")}\n\n` +
                `${t('enterNewName')}`,
                baseName + "_new"
            );
            
            if (!customName) {
                return null;
            }
            
            return await getUniqueFileName(customName, folder);
        }
        
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
    
    // Get parent folder and unique filename
    const folder = activeFile.parent;
    const fileInfo = await getUniqueFileName(objectName, folder);
    
    if (!fileInfo) {
        new Notice(t('cancelled'));
        return;
    }
    
    const finalObjectName = fileInfo.name;
    const newFileName = fileInfo.fileName;
    const newFilePath = fileInfo.filePath;
    
    try {
        // Create embed file FIRST
        const templateContent = await getTemplateContent(templateInfo);
        const content = templateContent.replace(/\${objectName}/g, finalObjectName);
        
        let newFile;
        try {
            newFile = await app.vault.create(newFilePath, content);
            console.log(`${t('fileEmbed')} ${newFilePath}`);
        } catch (createError) {
            console.error(`${t('errorCreatingFile')}:`, createError);
            new Notice(`${t('error')}: ${t('errorCreatingFile')} "${newFileName}". ${createError.message}`);
            return;
        }
        
        // Wait for file indexing
        await new Promise(resolve => setTimeout(resolve, 200));
        
        // Generate unique ID for embed
        const fileId = generateFileId();
        
        // Read current Excalidraw file content
        let excalidrawContent = await app.vault.read(activeFile);
        
        // Parse drawing JSON
        const drawingMatch = excalidrawContent.match(/## Drawing\n```json\n([\s\S]*?)\n```/);
        if (!drawingMatch) {
            new Notice(t('errorFileStructure'));
            return;
        }
        
        const drawingData = JSON.parse(drawingMatch[1]);
        
        // Calculate position - centered in current view
        const viewCenter = getViewCenter(excalidrawContent);
        let x = viewCenter.x - 200;
        let y = viewCenter.y - 150;
        
        // Cascade offset if embeds already exist
        if (drawingData.elements && drawingData.elements.length > 0) {
            const embedElements = drawingData.elements.filter(el => el.type === "image" && el.fileId);
            if (embedElements.length > 0) {
                const offset = (embedElements.length % 5) * 30;
                x += offset;
                y += offset;
            }
        }
        
        // Create image element for embed with status "saved"
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
            status: "saved",
            fileId: fileId,
            scale: [1, 1]
        };
        
        // Add element to drawing
        drawingData.elements = drawingData.elements || [];
        drawingData.elements.push(imageElement);
        
        // Manage Embedded Files section
        const embedEntry = `${fileId}: [[${finalObjectName}|100%]]`;
        
        const embeddedFilesRegex = /## Embedded Files\n([\s\S]*?)(?=\n%%|$)/;
        const embeddedFilesMatch = excalidrawContent.match(embeddedFilesRegex);
        
        if (embeddedFilesMatch) {
            let existingContent = embeddedFilesMatch[1].trim();
            
            const newContent = existingContent ? 
                `${existingContent}\n${embedEntry}` : 
                embedEntry;
            
            excalidrawContent = excalidrawContent.replace(
                embeddedFilesRegex, 
                `## Embedded Files\n${newContent}\n`
            );
        } else {
            const textElementsIndex = excalidrawContent.indexOf("## Text Elements");
            const drawingIndex = excalidrawContent.indexOf("## Drawing");
            const percentIndex = excalidrawContent.indexOf("%%");
            
            let insertPosition;
            
            if (percentIndex > -1 && percentIndex < drawingIndex) {
                insertPosition = percentIndex;
            } else if (drawingIndex > -1) {
                insertPosition = drawingIndex;
            } else {
                return;
            }
            
            excalidrawContent = 
                excalidrawContent.slice(0, insertPosition) + 
                `## Embedded Files\n${embedEntry}\n\n` +
                excalidrawContent.slice(insertPosition);
        }
        
        // Replace drawing JSON
        const newDrawingJson = JSON.stringify(drawingData, null, "\t");
        excalidrawContent = excalidrawContent.replace(
            /## Drawing\n```json\n[\s\S]*?\n```/,
            `## Drawing\n\`\`\`json\n${newDrawingJson}\n\`\`\``
        );
        
        // Save modifications
        await app.vault.modify(activeFile, excalidrawContent);
        
        // Force save to disk
        await app.vault.adapter.write(activeFile.path, excalidrawContent);
        
        // Wait for save to complete
        await new Promise(resolve => setTimeout(resolve, 1000));
        
        // PROGRAMMATIC TOGGLE to force Excalidraw to reload embeds
        try {
            console.log(t('attemptingToggle'));
            
            const activeLeaf = app.workspace.activeLeaf;
            if (activeLeaf && activeLeaf.view) {
                const view = activeLeaf.view;
                
                // Method 1: If it's a Markdown/Excalidraw view
                if (view.getViewType && (view.getViewType() === "markdown" || view.getViewType() === "excalidraw")) {
                    console.log(t('viewDetected'));
                    
                    const currentState = view.getState ? view.getState() : null;
                    
                    // Switch to source mode
                    if (view.setMode) {
                        await view.setMode('source');
                        console.log(t('toSourceMode'));
                        await new Promise(resolve => setTimeout(resolve, 200));
                        
                        // Back to preview mode
                        await view.setMode('preview');
                        console.log(t('toPreviewMode'));
                    } else if (view.setState && currentState) {
                        console.log(t('usingSetState'));
                        await view.setState(currentState, { history: false });
                    }
                }
                // Method 2: Close and reopen file
                else {
                    console.log(t('toggleViaReopen'));
                    
                    const tempNote = await app.vault.create(`temp-${Date.now()}.md`, "");
                    
                    await activeLeaf.openFile(tempNote);
                    await new Promise(resolve => setTimeout(resolve, 100));
                    
                    await activeLeaf.openFile(activeFile);
                    await new Promise(resolve => setTimeout(resolve, 100));
                    
                    await app.vault.delete(tempNote);
                }
                
                // Method 3: Trigger modification event
                if (app.workspace.trigger) {
                    console.log(t('triggeringEvent'));
                    app.workspace.trigger('file-open', activeFile);
                }
                
                console.log(t('toggleComplete'));
            }
        } catch (e) {
            console.error(t('toggleError'), e);
            
            // Fallback: simple refresh
            try {
                const activeLeaf = app.workspace.activeLeaf;
                if (activeLeaf) {
                    await activeLeaf.openFile(activeFile);
                }
            } catch (e2) {
                console.error(t('refreshError'), e2);
            }
        }
        
        // Success message
        const templateName = templateInfo.isDefault ? 
            t('defaultTemplate') : 
            templateInfo.displayName;
        new Notice(`${t('objectCreated')} "${finalObjectName}" ${t('createdWith')} ${templateName}!`);
        
        console.log(`${t('objectCreated')} ${finalObjectName} (Template: ${templateName})`);
        
    } catch (error) {
        console.error(`${t('error')}:`, error);
        new Notice(`${t('error')}: ${error.message}`);
    }
})();