# Metabox Configurator Project - Quick Start Guide

A dual-mode workflow for Metabox configurator projects: instant HTML prototypes OR full Angular production apps.

## 📋 Table of Contents

- [What This Is](#what-this-is)
- [Quick Start](#quick-start)
- [Project Setup](#project-setup)
- [Using the Templates](#using-the-templates)
- [Initial Prompt Examples](#initial-prompt-examples)
- [Plan Mode vs Direct Execution](#plan-mode-vs-direct-execution)
- [Complete Example](#complete-example)
- [Mode Detection](#mode-detection)
- [File Reference](#file-reference)
- [Troubleshooting](#troubleshooting)

---

## What This Is

This project provides **two ways** to build Metabox configurator interfaces:

### 🚀 Quick Prototype Mode
- **Zero build configuration** - Copy template, edit ID, open in browser
- **Vue 3 via CDN** - No npm, no webpack, no build step
- **Ready in 30 seconds** - Perfect for demos, testing, rapid iteration
- **Proven patterns** - 50/50 layout, Shim class, critical CSS

### 🏗️ Production Mode
- **Full Angular application** - TypeScript, testing, routing
- **Signal-based state** - Reactive configurator state management
- **Type-safe** - Strict TypeScript with proper API types
- **Production-ready** - Build optimization, deployment pipeline

**The magic**: Same project folder, same `CLAUDE.md` file, mode detected automatically.

---

## Quick Start

### For Quick Demos (30 seconds)

```bash
# 1. Copy template files to your project
cp -r templates/ my-project/
cp CLAUDE.md my-project/

# 2. Tell Claude what you want
"Create a quick demo for Metabox configurator ID: abc-123-def (basic type)"

# 3. Open the generated file in browser
python3 -m http.server 8000
# Open http://localhost:8000
```

Done! ✅

### For Production Apps

```bash
# 1. Copy template files to your project
cp -r templates/ my-project/
cp CLAUDE.md my-project/

# 2. Tell Claude what you want
"Create a production Angular app for Metabox configurator ID: abc-123-def"

# 3. Follow Angular development workflow
npm install
npm start
```

---

## Project Setup

### Minimal Setup (Required)

Copy these 3 things to your new project folder:

```
my-configurator-project/
├── CLAUDE.md                  # Project instructions - tells Claude which mode to use
├── templates/                 # Ready-to-use HTML templates
│   ├── basic-configurator/
│   │   ├── index.html
│   │   └── README.md
│   └── modular-configurator/
│       ├── index.html
│       └── README.md
└── docs/                      # CSS pattern documentation
    └── CSS_PATTERNS.md
```

### Recommended (Optional)

Include API documentation for reference:

```
my-configurator-project/
├── ... (files above)
├── metabox_configurator_basic_api.md      # Basic configurator API reference
└── metabox_configurator_modular_api.md    # Modular configurator API reference
```

### Setup Commands

```bash
# Create project folder
mkdir my-configurator-project
cd my-configurator-project

# Copy essential files from this repo
cp /path/to/prompt-fix/CLAUDE.md .
cp -r /path/to/prompt-fix/templates .
cp -r /path/to/prompt-fix/docs .

# Optional: Copy API documentation
cp /path/to/prompt-fix/metabox_configurator_basic_api.md .
cp /path/to/prompt-fix/metabox_configurator_modular_api.md .
```

---

## Using the Templates

### Basic Configurator Template

**Use for**: Single-product configurators (e.g., chairs, tables, single items)

**Features**:
- Product selector dropdown
- Material buttons for each slot
- Environment selector
- Export (screenshot/PDF) buttons
- 50/50 split layout

**Package**: `@3dsource/metabox-front-api`

**Commands**: `SetProduct`, `SetProductMaterial`, `SetEnvironment`

### Modular Configurator Template

**Use for**: Component-assembly configurators (e.g., kitchens, modular furniture)

**Features**:
- Component type selectors
- Configuration tree viewer
- Material controls per component
- Root component initialization
- Export buttons

**Package**: `@3dsource/metabox-modular-configurator-api`

**Commands**: `SetComponent`, `SetComponentMaterial`, `SetEnvironment`

---

## Initial Prompt Examples

### Scenario 1: Quick Prototype/Demo

**Goal**: Test a configurator quickly, iterate on materials, demo to client

**Prompt**:
```
Create a quick demo for Metabox configurator ID: 624b6c49-3fc8-4c4d-8e6e-5221355afd68

It's a basic configurator (single product type). I want to test it quickly.
```

**What Happens**:
1. ✅ Claude detects "quick demo" → Quick Prototype Mode
2. ✅ Copies `/templates/basic-configurator/index.html` to project root
3. ✅ Updates line 330 with your configurator ID
4. ✅ Tells you to open in browser

**Result**: Working HTML file, ready to open, no build step

---

### Scenario 2: Production Application

**Goal**: Build a customer-facing configurator with routing, testing, deployment

**Prompt**:
```
I need to build a production configurator app with Angular for Metabox configurator ID: abc-123-def-456

This will be a full application with routing, testing, and deployment. Using a basic configurator.
```

**What Happens**:
1. ✅ Claude detects "production app" + "Angular" → Production Mode
2. ✅ Triggers `default-stack@3dsource` plugin
3. ✅ Routes to `angular` skill (new project) or `angular-legacy` (existing)
4. ✅ Creates Angular project with services, components, signals
5. ✅ Sets up TypeScript strict mode, testing infrastructure

**Result**: Full Angular project, production-ready architecture

---

### Scenario 3: Modular Configurator

**Goal**: Quick demo of a component-assembly configurator (kitchen, modular system)

**Prompt**:
```
Create a demo page for modular Metabox configurator ID: xyz-789-abc

It's a kitchen configurator with base cabinets, doors, and drawers. I need to quickly test material combinations.
```

**What Happens**:
1. ✅ Claude detects "demo" + "quickly test" → Quick Prototype Mode
2. ✅ Detects "modular configurator" → Uses modular template
3. ✅ Copies `/templates/modular-configurator/index.html`
4. ✅ Updates configurator ID (line 382)
5. ✅ Reminds you to set root component (lines 388-391)

**Result**: Working modular configurator demo

---

### Scenario 4: Plan Mode (Review First)

**Goal**: See the approach before implementation, clarify requirements

**Prompt**:
```
/plan

I have a Metabox modular configurator for office desks. I need a demo page where I can:
- Select desk base style
- Add legs and tops
- Change wood and metal materials
- Export renders

Configurator ID: abc-123-def
```

**What Happens**:
1. ✅ Claude enters Plan Mode
2. ✅ Analyzes requirements → Quick Prototype Mode (demo page)
3. ✅ Detects modular configurator
4. ✅ Creates plan explaining:
   - Copy modular template
   - Update configurator ID
   - Configure root component
   - What files will be created
5. ✅ Presents plan for your approval
6. ✅ After approval, executes the plan

**Result**: Clear plan first, then quick execution

---

## Plan Mode vs Direct Execution

### Use Plan Mode When:

✅ You want to see the approach before code is written
✅ Requirements are complex or have multiple possible approaches
✅ You're making significant changes to existing code
✅ You want to clarify details before starting

**Example**:
```
/plan

Create a configurator interface with custom animations and state persistence
```

### Skip Plan Mode When:

✅ You just want a quick demo (clear, simple intent)
✅ You're using a template as-is
✅ Requirements are straightforward

**Example**:
```
Quick prototype for configurator abc-123-def (basic type)
```

---

## Complete Example

Let's walk through creating a new configurator demo from scratch.

### Step 1: Create Project Folder

```bash
mkdir my-chair-configurator
cd my-chair-configurator
```

### Step 2: Copy Files

```bash
# Copy essential files
cp /path/to/prompt-fix/CLAUDE.md .
cp -r /path/to/prompt-fix/templates .
cp -r /path/to/prompt-fix/docs .

# Optional: API docs
cp /path/to/prompt-fix/metabox_configurator_basic_api.md .
```

**Your folder now**:
```
my-chair-configurator/
├── CLAUDE.md
├── metabox_configurator_basic_api.md
├── templates/
└── docs/
```

### Step 3: Open Claude

In your Claude session (working directory: `my-chair-configurator/`):

**Your Prompt**:
```
Create a demo page for basic Metabox configurator: 624b6c49-3fc8-4c4d-8e6e-5221355afd68

I want to test product switching and materials quickly.
```

### Step 4: Claude Does Its Thing

Claude will:
1. Read `CLAUDE.md`
2. Detect "demo" + "quickly" → **Quick Prototype Mode**
3. Copy `templates/basic-configurator/index.html` → `./index.html`
4. Replace line 330: `'624b6c49-3fc8-4c4d-8e6e-5221355afd68'`
5. Respond: "✅ Created `index.html` - Open it in your browser (localhost or HTTPS)"

### Step 5: Test It

```bash
# Start a local server
python3 -m http.server 8000

# Open browser
open http://localhost:8000
```

**You should see**:
- Left: Clean 3D viewport (no overlay menu)
- Right: Controls (product selector, materials, environment, export buttons)

### Step 6: Iterate

Want to customize? Just ask:

```
Add a fullscreen button and change the split to 60/40
```

Claude will edit the HTML, update CSS, add the button. Refresh browser. Done.

---

## Mode Detection

The magic happens in `CLAUDE.md`. Here's how Claude decides which mode:

### Quick Prototype Mode Triggers

Claude uses **Quick Prototype Mode** when:

✅ No `package.json` exists in the project
✅ User says: "demo", "prototype", "quick", "test", "example"
✅ User asks to: "start fresh", "create a simple page"
✅ Project is in early exploration phase

**Keywords that trigger prototype mode**:
- "quick demo"
- "test page"
- "prototype"
- "simple example"
- "try out"
- "rapid iteration"

### Production Mode Triggers

Claude uses **Production Mode** when:

✅ `package.json` with Angular dependencies exists
✅ User mentions: "production", "app", "application", "deploy"
✅ TypeScript strict mode is required
✅ Testing and build pipeline are needed

**Keywords that trigger production mode**:
- "production app"
- "full application"
- "with routing and tests"
- "deploy to production"
- "customer-facing"

### Ambiguous Cases

If Claude isn't sure, it will ask:

**Your Prompt**:
```
Help me set up a Metabox configurator for my client
```

**Claude Asks**:
```
Would you like a quick HTML prototype or a full Angular application?

- Quick prototype: Copy template, edit ID, open in browser (ready in ~1 minute)
- Angular app: Full TypeScript project with routing, testing, services (production-ready)
```

You clarify, Claude proceeds.

---

## File Reference

### Core Files

| File | Purpose | Required |
|------|---------|----------|
| `CLAUDE.md` | Project instructions, mode detection | ✅ Yes |
| `templates/basic-configurator/` | Single-product template | ✅ Yes |
| `templates/modular-configurator/` | Component-assembly template | ✅ Yes |
| `docs/CSS_PATTERNS.md` | Critical CSS documentation | ✅ Yes |
| `metabox_configurator_basic_api.md` | Basic API reference | Recommended |
| `metabox_configurator_modular_api.md` | Modular API reference | Recommended |

### Template Features

#### Basic Configurator (`templates/basic-configurator/index.html`)

- **Lines**: 409 total
- **Configurator ID**: Line 330 (replace `'YOUR-CONFIGURATOR-ID-HERE'`)
- **Critical CSS**: Lines 14-56 (do not remove!)
- **Shim Class**: Lines 218-322
- **Vue App**: Lines 324-340

**Controls included**:
- Product selector dropdown
- Material buttons (all slots)
- Environment selector
- Environment material buttons (if available)
- Export buttons (screenshot, PDF)
- Call-to-action button (e-commerce integration)

#### Modular Configurator (`templates/modular-configurator/index.html`)

- **Lines**: 482 total
- **Configurator ID**: Line 382 (replace `'YOUR-CONFIGURATOR-ID-HERE'`)
- **Root Component Init**: Lines 388-391 (uncomment and configure)
- **Critical CSS**: Lines 14-56 (do not remove!)
- **Shim Class**: Lines 270-373
- **Vue App**: Lines 375-397

**Controls included**:
- Component type selectors (dropdowns per type)
- Configuration tree viewer (shows current components)
- Material controls per component
- Environment selector
- Export buttons (screenshot, PDF)

---

## Troubleshooting

### Configurator Not Loading

**Symptom**: Blank viewport or "Loading..." forever

**Causes & Fixes**:
1. **Wrong configurator ID** → Verify UUID format, check Metabox admin
2. **HTTP instead of HTTPS** → Use localhost or HTTPS (HTTP is blocked)
3. **Wrong configurator type** → Basic template with modular ID (or vice versa)
4. **CORS/network error** → Check browser console

### Default Menu Still Shows

**Symptom**: Overlay menu appears in left viewport (duplicate controls)

**Fix**: Templates now include these commands (should be automatic):
```javascript
this.api.sendCommandToMetabox(new ShowEmbeddedMenu(false));
this.api.sendCommandToMetabox(new ShowOverlayInterface(false));
```

If still showing, verify these are in the `ready()` method.

### Layout Issues

**Symptom**: Viewport tiny, double scrollbars, panels not 50/50

**Check**:
1. `#embed3DSource` has `width: 100%; height: 100%`
2. Container has `position: absolute`
3. Panels use `flex: 0 0 50%` (NOT `flex: 1`)
4. Container has `overflow: hidden`
5. Only right panel has `overflow: auto`

See `docs/CSS_PATTERNS.md` for detailed explanations.

### Controls Not Working

**Symptom**: Buttons/dropdowns don't change configurator

**Check**:
1. Commands sent **after** `apiReadyCallback` fires
2. Event listeners registered in `addApiEventListeners()`
3. For modular: Root component set first
4. Browser console for errors

### Modular: No Components Showing

**Symptom**: Component selectors are empty or configurator won't load

**Check**:
1. Root component is set in `apiReadyCallback` (lines 388-391)
2. `isRoot: true` parameter on first `SetComponent` call
3. Root component ID is valid for the configurator
4. Browser console for `configuratorDataUpdated` event data

---

## Best Practices

### For Quick Prototypes

✅ **DO**:
- Use templates as-is for fastest results
- Keep critical CSS patterns intact
- Test on localhost first
- Iterate in the HTML file directly

❌ **DON'T**:
- Remove critical CSS patterns
- Change `flex: 0 0 50%` to `flex: 1`
- Send commands before `apiReadyCallback`
- Use HTTP (must be HTTPS or localhost)

### For Production Apps

✅ **DO**:
- Follow Angular plugin guidance
- Use signals for configurator state
- Implement proper error handling
- Add loading states and fallbacks
- Write tests for configurator interactions

❌ **DON'T**:
- Store API reference in components (use service)
- Use `any` types for API interactions
- Skip cleanup in `ngOnDestroy`
- Track state manually (use events)

---

## Migration Path

Start with a prototype, then migrate to production:

### 1. Prototype Phase
```bash
# Quick demo for client feedback
"Create a quick demo for configurator abc-123"
# Result: index.html, ready to test
```

### 2. Client Approval
```bash
# Open index.html, share with client
# Iterate on materials, products, layout
```

### 3. Production Phase
```bash
# Once approved, migrate to Angular
"Convert this prototype to a production Angular app"
# Result: Full Angular project, same functionality
```

Claude will:
- Create Angular project structure
- Migrate Shim class → Angular service
- Convert state to signals
- Add TypeScript types
- Set up testing infrastructure
- Preserve all configurator logic

---

## Additional Resources

- **CSS Patterns**: See `docs/CSS_PATTERNS.md` for detailed CSS explanations
- **Basic API**: See `metabox_configurator_basic_api.md` for API reference
- **Modular API**: See `metabox_configurator_modular_api.md` for API reference
- **3DSource Docs**: [https://docs.3dsource.com](https://docs.3dsource.com)

---

## License

This template system is provided as-is for use with Metabox configurator projects.

## Support

For issues with:
- **Templates**: Check this README and `docs/CSS_PATTERNS.md`
- **Metabox API**: See API documentation files or 3DSource support
- **Angular integration**: Follow plugin guidance in CLAUDE.md

---

**Ready to start?** Copy `CLAUDE.md`, `templates/`, and `docs/` to your project, then tell Claude what you want to build!
