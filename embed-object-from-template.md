/*
Script Excalidraw pour créer et intégrer des objets depuis un template
Version Finale 1.0.0
Author: Rolland MELET & Claude Code
Date: 2025-08-14
*/

// Configuration
const TEMPLATE_PATH = "Templates/MonTemplateObjet.md";
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

// Obtenir le contenu du template
async function getTemplate() {
    try {
        const templateFile = app.vault.getAbstractFileByPath(TEMPLATE_PATH);
        if (templateFile) {
            const content = await app.vault.read(templateFile);
            // Vérifier si c'est un fichier Excalidraw
            if (!content.includes("excalidraw-plugin: parsed")) {
                return TEMPLATE_DEFAULT;
            }
            return content;
        }
    } catch (e) {
        // Utiliser le template par défaut en cas d'erreur
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
    // Demander le nom de l'objet
    const objectName = await utils.inputPrompt("Nom de l'objet de workflow:");
    if (!objectName) {
        return;
    }
    
    // Obtenir le fichier actif
    const activeFile = app.workspace.getActiveFile();
    if (!activeFile) {
        new Notice("Aucun fichier actif");
        return;
    }
    
    // Créer le nouveau fichier depuis le template
    const folder = activeFile.parent;
    const newFileName = `${objectName}.md`;
    const newFilePath = folder ? `${folder.path}/${newFileName}` : newFileName;
    
    try {
        // Créer le fichier avec le contenu du template Excalidraw
        const template = await getTemplate();
        const content = template.replace(/\${objectName}/g, objectName);
        const newFile = await app.vault.create(newFilePath, content);
        
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
        
        new Notice(`✅ Objet "${objectName}" créé et intégré!`);
        
    } catch (error) {
        new Notice(`Erreur: ${error.message}`);
    }
})();