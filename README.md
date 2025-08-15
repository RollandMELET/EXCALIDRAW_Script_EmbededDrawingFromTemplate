# Excalidraw Embed from Template Script

## The "Back of the Card" Problem - Solved

This script solves a specific workflow challenge in Obsidian's Excalidraw plugin, inspired by Zsolt's Visual Zettelkasten methodology demonstrated in his videos on [Visual Zettelkasten](https://www.youtube.com/watch?v=o49C8jQIsvs), [Block References](https://www.youtube.com/watch?v=yDZ6v7_OqZE), and [Hybrid Notes](https://www.youtube.com/watch?v=DbeLGGxvKZQ).

### The Problem
In Zsolt's Visual PKM approach, every visual element should have a "back of the card" - a markdown file containing the element's metadata, detailed information, and connections. Previously, creating these embedded objects required:

1. **Exit Excalidraw** - Break your visual flow
2. **Create a new markdown file** - Navigate file explorer  
3. **Apply a template** - Find and copy the right template
4. **Return to Excalidraw** - Context switch back
5. **Manually embed the file** - Drag or use `![[filename]]` syntax

**Result:** 5+ steps, multiple context switches, disrupted creative flow.

### The Solution
With this script: **One keyboard shortcut** creates everything instantly, without leaving Excalidraw.

Press `Cmd+Shift+O` → Select template → Name your object → Done! The embedded object appears immediately in your drawing with its backing markdown file already created.

## Current Version: v1.3.0 (Bilingual)
Released: August 15, 2025

## 🎯 Key Features

### ✨ True Visual PKM Workflow
- Create embedded objects **directly from Excalidraw** - no context switching
- Automatic "back of the card" markdown file generation
- Instant visual feedback - embeds appear immediately
- Perfect for Visual Zettelkasten methodology

### 🌍 Bilingual Support (v1.3.0)
- **Automatic language detection** (EN/FR)
- Uses Obsidian's locale settings
- Seamless experience for international users
- All messages and prompts in your language

### 🎨 Dynamic Template System
- **Interactive template selection** from your `Templates/` folder
- **Category support** via subfolders
- **Recently used templates** (last 5) for quick access
- **Favorite templates** marked with ⭐
- **Built-in fallback template** when no templates available

### 🔄 Smart Duplicate Handling
- Automatic detection of existing filenames
- **Intelligent suggestions**: name(1), name(2), name_v2, name_copy
- Clean interface using `utils.suggester()`
- Custom naming option available

## 📋 Prerequisites

- **Obsidian** v1.5.0 or higher
- **Excalidraw Plugin** v2.14.0 or higher (by Zsolt Viczian)
- `Templates/` folder in your vault (optional but recommended)

## 🚀 Installation

### Via Excalidraw Scripts Menu (Recommended)

1. Open any Excalidraw file in Obsidian
2. Click the **tools icon** in Excalidraw toolbar
3. Navigate to **Scripts** → **Downloaded** → **Install new**
4. Copy the entire content of `embed-object-from-template.md`
5. The script will appear in the "Downloaded" section with a custom cube icon

### Manual Installation

1. Copy `embed-object-from-template.md` to:
   ```
   YourVault/Excalidraw/Scripts/Downloaded/
   ```
2. Restart Obsidian or refresh scripts

## 🎮 Usage

### Basic Workflow

1. **Open** an Excalidraw drawing
2. **Launch** the script:
   - Via Scripts menu → Downloaded → embed-object-from-template
   - Or use your configured keyboard shortcut
3. **Select** a template from the list (or use default)
4. **Name** your workflow object
5. **Done!** Object is created and embedded instantly

### Keyboard Shortcut Setup

1. Settings → Hotkeys
2. Search for "Excalidraw: embed-object-from-template"
3. Assign shortcut (e.g., `Cmd+Shift+O` or `Ctrl+Shift+O`)

### Template Organization

Structure your templates in the `Templates/` folder:

```
Templates/
├── BasicCard.md
├── ProcessStep.md
├── Workflows/
│   ├── SimpleWorkflow.md
│   └── ComplexWorkflow.md
├── Concepts/
│   ├── Definition.md
│   └── Reference.md
└── DataStructure.md
```

Templates are automatically:
- Discovered and listed by category
- Sorted alphabetically
- Remembered after use (recent templates)

## ⚙️ Advanced Configuration

### Template Structure

Templates must be valid Excalidraw files containing:
- YAML header with `excalidraw-plugin: parsed`
- Standard Excalidraw sections
- `${objectName}` variable for dynamic content

### Minimal Template Example

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

## 🎯 Use Cases (Visual PKM)

### Visual Zettelkasten
Create atomic notes with visual representations. Each embedded object becomes a permanent note in your Zettelkasten with both visual and textual dimensions.

### Process Documentation
Build complex workflows visually while maintaining detailed documentation in the backing files. Perfect for standard operating procedures.

### Knowledge Graphs
Construct visual knowledge graphs where each node has rich metadata and connections stored in its markdown file.

### Project Planning
Design project structures visually with each component having detailed specifications in its "back of the card."

## 🔧 Troubleshooting

| Issue | Solution |
|-------|----------|
| Embed doesn't appear | ✅ Fixed in v1.3.0 with automatic toggle |
| "No active file" error | Open an Excalidraw drawing first |
| Templates not found | Check `Templates/` folder exists and contains valid files |
| Duplicate filename | Script automatically suggests alternatives |
| Creation error | Check folder permissions |

## 📊 Technical Architecture

### How It Works

1. **Template Selection**: Interactive menu with memory
2. **Name Input**: User prompt with duplicate detection
3. **File Creation**: Generate from template with variable substitution
4. **Excalidraw Integration**:
   - Generate unique fileId
   - Add to `## Embedded Files` section
   - Insert image element in JSON
   - Set status "saved" for immediate display
5. **Programmatic Toggle**: Force Excalidraw to refresh
6. **Success Notification**: Confirm creation

### The Innovation (v1.3.0)

The script implements multiple refresh strategies to ensure immediate display:
1. Toggle source/preview for Markdown views
2. setState() to force reload
3. File close/reopen via temporary file
4. Obsidian event triggering

This multi-strategy approach ensures compatibility across different Excalidraw states and view modes.

## 📈 Version History

### v1.3.0 (August 15, 2025) - **Current Stable**
- ✅ Bilingual support (EN/FR) with auto-detection
- ✅ Custom SVG icon for script menu
- ✅ Programmatic toggle for immediate display
- ✅ Final resolution of display bug
- ✅ International release ready

### v1.2.0 - v1.2.9
- Dynamic template selection system
- Advanced duplicate handling
- Multiple interface improvements
- Filename/embed synchronization

### v1.0.0
- Initial stable release
- Basic creation with fixed template

## 🌟 Why This Matters

This script bridges the gap between visual thinking and structured note-taking. It enables the Visual PKM methodology promoted by Zsolt without the friction of constant context switching. You can now build complex visual systems while maintaining the detailed metadata and connections that make Obsidian powerful.

## 🤝 Contributing

Contributions are welcome! 

- **Report bugs**: Via [GitHub issues](https://github.com/RollandMELET/EXCALIDRAW_Script_EmbededDrawingFromTemplate/issues)
- **Suggest improvements**: Pull requests accepted
- **Share templates**: Add examples to the Templates folder
- **Join the discussion**: [Obsidian Discord](https://discord.gg/obsidianmd) #excalidraw channel

## 📝 License

MIT License - See LICENSE file for details

## 👥 Credits

- **Rolland MELET** - Script Development & Visual PKM Implementation
- **Claude (Anthropic)** - Development assistance
- **Zsolt Viczian** - Excalidraw Plugin creator & Visual PKM inspiration
- **Obsidian Community** - Feedback and ideas

## 💬 Support

For questions or issues:
- Open an issue on [GitHub](https://github.com/RollandMELET/EXCALIDRAW_Script_EmbededDrawingFromTemplate)
- Contact: rm@360sc.io
- Discord: Find me in the Obsidian Discord #excalidraw channel

---

*Empowering Visual PKM in Obsidian - Create once, think visually, document thoroughly.*

**Version 1.3.0** - Bilingual Edition