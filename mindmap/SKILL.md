---
name: mindmap
description: >
  Generate Mermaid flowchart mindmaps (LR / left-to-right) from markdown file
  headings and embed them as PNG images. Trigger when the user wants to create
  visual mindmaps from markdown notes — phrases like "create mindmaps for this
  file", "generate mindmaps for headings", "make mindmaps from my notes", "add
  mindmap images", or any request to turn structured markdown content into
  colourful horizontal flowcharts with depth-based colouring. Does NOT trigger
  for general diagramming or unrelated visualisation requests.
---

# Mindmap

Generate `flowchart LR` (left-to-right) Mermaid mindmap PNGs from headings in a markdown file, colour-coded by hierarchy depth, on a dark background.

## Workflow

### 1. Read the file
Read the markdown file the user points to using `shell cat` or similar.

### 2. Identify headings without existing images
Parse the markdown to find all headings. For each heading, check whether an existing image reference (`![[…]]`) is present under that heading. Present the user with:
- Which headings **already have** images (skip these unless the user says to regenerate)
- Which headings are **missing** images
- Let the user choose which headings to generate mindmaps for

### 3. Parse content under each selected heading
For each heading the user selects:
- Extract everything under that heading until the next heading of equal or higher level
- Identify the **hierarchical structure**: bullet lists, sub-headings, and the natural tree
- Each line/item becomes a **node** in the mindmap
- The heading itself is the **root** (d0)

### 4. Generate the Mermaid `flowchart LR` diagram
Build a Mermaid diagram with this exact structure:

**Mermaid init config:**
```yaml
%%{init: { 'theme': 'base', 'themeVariables': { 'background': '#212121', 'primaryColor': '#4E5166', 'primaryBorderColor': '#6B6E87', 'primaryTextColor': '#fff', 'lineColor': '#5a5a5a', 'tertiaryColor': '#2a2a2a' } }}%%
```

**Colour palette by depth (cycles for deeper levels):**
| Depth | Fill | Stroke |
|-------|------|--------|
| d0 (root) | `#4E5166` | `#6B6E87` |
| d1 | `#424B54` | `#5C6570` |
| d2 | `#31423E` | `#425A53` |
| d3 | `#506341` | `#6A7E57` |
| d4 | `#4E5166` | `#6B6E87` (cycles to d0) |
| d5 | `#424B54` | `#5C6570` (cycles to d1) |
| d6 | `#31423E` | `#425A53` (cycles to d2) |
| … | cycles | cycles |

Always use `flowchart LR` direction. Write the diagram to a `.mmd` temp file.

### 5. Consider the content structure well

When converting content to a mindmap, think carefully about the natural hierarchy present in the original notes. Each bullet point or sub-topic typically represents a branch. Cases, statutes, and key concepts are usually children of the topics they belong to. The goal is a clean, well-structured mindmap that reflects the logical structure of the original notes, not just a flat list.

If there are case names, they should typically be at the deepest levels as leaves under their topic. If there are sub-topics, create appropriate intermediate nodes.

### 6. Render the PNG
Use `mmdc` to render the `.mmd` file to PNG:
```bash
mmdc -i <file.mmd> -o <output.png> --backgroundColor "#212121" -w 2400
```

Save the PNG to an **Assets/** folder relative to the markdown file's location (create it if it doesn't exist). Name the PNG after the heading (e.g., `## Illegality` → `Illegality.png`).

### 7. Embed the image in the markdown
Edit the markdown file to add `![[filename.png]]` on its own line right after the heading (before the existing content), using the `edit` tool.

### 8. Clean up
Remove the temporary `.mmd` file after rendering.

## Important notes

- Always use `flowchart LR` (never `mindmap` or any other diagram type)
- The background colour is always `#212121`
- Width should be `2400`px to give enough horizontal space for higher resolution output
- Node text should be concise — prefer short labels. If cases or concepts have long names, use spaces and quoted strings in Mermaid (e.g. `MyNode["Long node text here"]`)
- Use `classDef` and `class` syntax in Mermaid to assign depth-based colours
- If a heading's content has many sibling items at the same level, try to keep them balanced horizontally
- When naming nodes in Mermaid, use alphanumeric IDs (no spaces) and quoted display text: `NodeName["Display text"]`
