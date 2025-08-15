# Product Requirements Document
## Excalidraw Embed from Template Script

**Version:** 1.3.0  
**Date:** August 15, 2025  
**Author:** Rolland MELET & Claude (Anthropic)  
**Status:** Production Ready - Bilingual Edition

## 1. Executive Summary

This script enables seamless creation of embedded Excalidraw objects with backing markdown files ("back of the card") directly from within Excalidraw, following Zsolt Viczian's Visual PKM methodology.

### Core Problem
Visual PKM requires every visual element to have detailed metadata in a backing file. Previously, this required 5+ manual steps and constant context switching between Excalidraw and Obsidian's file explorer.

### Solution
One keyboard shortcut creates both the visual embed and its backing markdown file instantly, without leaving Excalidraw.

## 2. User Stories

### Primary User Story
**As a** Visual PKM practitioner using Obsidian and Excalidraw,  
**I want to** create embedded objects with backing files using a single keyboard shortcut,  
**So that** I can maintain my visual flow while building comprehensive knowledge systems.

### Secondary User Stories

**Template Selection**  
**As a** user with different object types,  
**I want to** choose from multiple templates when creating objects,  
**So that** I can quickly create structured objects for different purposes.

**Language Support**  
**As an** international user,  
**I want** the script to detect and use my language automatically,  
**So that** I can work in my native language without configuration.

## 3. Functional Requirements

### 3.1 Core Functionality
- **Single-action creation**: One keyboard shortcut triggers the entire workflow
- **In-context execution**: All operations happen within Excalidraw view
- **Immediate visual feedback**: Embedded object appears instantly
- **Automatic file generation**: Backing markdown file created with template content

### 3.2 Template System
- **Dynamic discovery**: Scan `Templates/` folder for available templates
- **Category support**: Organize templates in subfolders
- **Recent templates**: Remember last 5 used templates
- **Fallback template**: Built-in default when no templates available
- **Variable substitution**: Replace `${objectName}` in templates

### 3.3 User Interface
- **Interactive selection**: Clean menu for template choice
- **Smart suggestions**: Propose alternatives for duplicate names
- **Bilingual support**: Auto-detect language (EN/FR)
- **Custom icon**: Distinctive cube icon in scripts menu

### 3.4 File Management
- **Smart positioning**: Center embed in current viewport
- **Cascade offset**: Offset multiple embeds for visibility
- **Duplicate handling**: Automatic name suggestions (name(1), name_v2, etc.)
- **Same folder creation**: Files created in active drawing's folder

## 4. Technical Specifications

### 4.1 Excalidraw Integration
```javascript
// Required modifications to Excalidraw file:
1. Add entry to "## Embedded Files" section
   Format: fileId: [[ObjectName|100%]]

2. Insert image element in JSON with:
   - type: "image"
   - fileId: unique 40-character hash
   - status: "saved"
   - Centered position based on viewport

3. Force refresh via programmatic toggle
```

### 4.2 Language Detection
```javascript
const getUserLanguage = () => {
    const lang = moment?.locale() || 
                 navigator.language || 
                 app.locale || 
                 'en';
    return lang.startsWith('fr') ? 'fr' : 'en';
};
```

### 4.3 Refresh Strategy
Multiple fallback methods ensure immediate display:
1. Toggle source/preview mode
2. setState() for view refresh
3. File close/reopen sequence
4. Obsidian event triggering

## 5. Non-Functional Requirements

### Performance
- **Creation time**: < 2 seconds from trigger to visible embed
- **Template scanning**: Cached for repeated use
- **Memory usage**: Minimal footprint, cleanup after execution

### Compatibility
- **Obsidian**: v1.5.0+
- **Excalidraw Plugin**: v2.14.0+
- **Platform**: Cross-platform (Windows, Mac, Linux)

### Usability
- **Zero configuration**: Works out of the box
- **Intuitive workflow**: Follows Obsidian conventions
- **Error recovery**: Graceful handling with user feedback

## 6. Success Metrics

### Adoption Metrics
- Script installations via Excalidraw menu
- GitHub stars and forks
- Community feedback and contributions

### Usage Metrics
- Objects created per session
- Template variety usage
- Error rate < 1%

### Quality Metrics
- Immediate display success rate > 99%
- User-reported bugs resolved within 48 hours
- Community template contributions

## 7. Implementation Phases

### Phase 1: Core Functionality ✅
- Basic object creation
- Fixed template support
- Manual refresh required

### Phase 2: Template System ✅
- Dynamic template selection
- Category organization
- Recent templates memory

### Phase 3: Polish & International ✅
- Bilingual support (EN/FR)
- Custom icon
- Programmatic refresh
- Production stability

### Phase 4: Future Vision
- Multi-language support (ES, DE, etc.)
- Template sharing marketplace
- AI-suggested templates
- Batch object creation

## 8. Technical Debt & Known Limitations

### Current Limitations
- Language support limited to EN/FR
- Template variables limited to `${objectName}`
- Maximum 5 recent templates remembered

### Technical Debt
- Refresh mechanism uses multiple fallbacks (could be simplified when Excalidraw API improves)
- Template validation is basic (checks for header only)

## 9. Security Considerations

- No external API calls
- All operations local to vault
- No sensitive data handling
- Template content sandboxed

## 10. Documentation Requirements

### User Documentation
- README.md in English and French
- Installation guide with screenshots
- Template creation guide
- Troubleshooting section

### Developer Documentation
- Code comments in English
- JSDoc for all functions
- Architecture diagram
- Contribution guidelines

## 11. Release Criteria

### v1.3.0 Release Checklist
- ✅ Bilingual support functional
- ✅ Custom icon displayed
- ✅ Programmatic refresh working
- ✅ Documentation in both languages
- ✅ GitHub repository prepared
- ✅ Community announcement drafted

## 12. Support & Maintenance

### Support Channels
- GitHub Issues for bug reports
- Discord #excalidraw channel for discussions
- Direct email for critical issues

### Maintenance Plan
- Monthly review of issues
- Quarterly compatibility testing
- Annual major version consideration

---

*This PRD represents the current state of the Excalidraw Embed from Template Script, focusing on solving the "back of the card" workflow challenge in Visual PKM.*