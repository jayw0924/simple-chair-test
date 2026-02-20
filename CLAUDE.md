# Metabox Configurator Project

## Project Modes

This project supports two development modes. Claude will automatically detect which mode to use based on project state and user intent.

### Mode Detection

**Quick Prototype Mode** - Use when:
- No `package.json` exists in the project
- User says "demo", "prototype", "quick", "test", "example"
- User asks to "start fresh" or "create a simple page"
- Project is in early exploration phase

**Production Mode** - Use when:
- `package.json` with Angular dependencies exists
- User mentions "production", "app", "application", "deploy"
- TypeScript strict mode is required
- Testing and build pipeline are needed

**If unclear**, ask the user: "Would you like a quick HTML prototype or a full Angular application?"

---

## Quick Prototype Mode

Use ready-to-use HTML templates for instant demos with zero build configuration.

### Starting a New Prototype

1. **Choose the right template:**
   - `/templates/basic-configurator/` - For single-product configurators (`@3dsource/metabox-front-api`)
   - `/templates/modular-configurator/` - For component-assembly configurators (`@3dsource/metabox-modular-configurator-api`)

2. **Copy template files to project root**

3. **Update configurator ID:**
   - Basic: Edit line 330 in `index.html`
   - Modular: Edit line 382 in `index.html`
   - Replace `'YOUR-CONFIGURATOR-ID-HERE'` with actual UUID

4. **For modular: Initialize root component:**
   - Edit lines 388-391 in `index.html`
   - Uncomment and configure root component setup
   - Must set root component before children

5. **Open in browser** (localhost or HTTPS required)

### Template Architecture

**Shim Class Pattern:**
- Stores the `Communicator` API reference
- Manages event listeners
- Provides methods to send commands
- Exposes reactive state for Vue.js

**Critical CSS Layout Patterns:**
```css
/* Container must be position absolute with explicit dimensions */
.container {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  overflow: hidden;
}

/* Equal 50/50 split with no flex grow/shrink */
.left-panel,
.right-panel {
  flex: 0 0 50%;
}

/* Left panel contains 3D viewport */
.left-panel {
  height: 100%;
}

/* Right panel scrolls independently */
.right-panel {
  overflow: auto;
}

/* Embed container fills left panel completely */
#embed3DSource {
  width: 100%;
  height: 100%;
}
```

**Why these patterns matter:**
- `position: absolute` prevents layout shifts during load
- `flex: 0 0 50%` ensures exact 50/50 split (no flex grow/shrink)
- `overflow: hidden` on container prevents double scrollbars
- `overflow: auto` on right panel enables independent scrolling
- Explicit dimensions on `#embed3DSource` ensure iframe fills space

See [/docs/CSS_PATTERNS.md](./docs/CSS_PATTERNS.md) for detailed explanations of all patterns.

### API Integration Patterns

**Basic Configurator:**
```javascript
integrateMetabox(
  'configurator-id',
  'embed3DSource',
  (api) => {
    // 1. Store API reference
    this.api = api;

    // 2. Register event listeners FIRST
    api.addEventListener('configuratorDataUpdated', (data) => {
      this.updateState(data);
    });

    // 3. Hide default UI (for custom interfaces)
    api.sendCommandToMetabox(new ShowEmbeddedMenu(false));
    api.sendCommandToMetabox(new ShowOverlayInterface(false));

    // 4. Initialize state (optional)
    api.sendCommandToMetabox(new SetProduct('product-id'));
    api.sendCommandToMetabox(new SetEnvironment('env-id'));
  },
  { standalone: true }
);
```

**Modular Configurator:**
```javascript
integrateMetabox(
  'configurator-id',
  'embed3DSource',
  (api) => {
    // 1-2. Same as basic

    // 3. Hide default UI
    api.sendCommandToMetabox(new ShowEmbeddedMenu(false));
    api.sendCommandToMetabox(new ShowOverlayInterface(false));

    // 4. CRITICAL: Set root component FIRST
    api.sendCommandToMetabox(new SetComponent('root-id', 'base', true));

    // 5. Set environment
    api.sendCommandToMetabox(new SetEnvironment('env-id'));

    // 6. Add child components
    api.sendCommandToMetabox(new SetComponent('child-id', 'type', false));
  },
  { standalone: true }
);
```

### Common Pitfalls to Avoid

1. **Commands before callback** - Always wait for `apiReadyCallback` before sending commands
2. **No root component (modular)** - Must set root with `isRoot: true` before children
3. **Missing dimensions** - Container must have explicit width/height
4. **HTTP instead of HTTPS** - Configurator requires HTTPS (except localhost)
5. **Rapid command spam** - Debounce material/product changes
6. **Wrong package** - Basic uses `@3dsource/metabox-front-api`, Modular uses `@3dsource/metabox-modular-configurator-api`
7. **Removing critical CSS** - Do not remove the 7 critical CSS patterns (see CSS_PATTERNS.md)
8. **Flex-grow/shrink** - Always use `flex: 0 0 X%` for panels, never `flex: 1`

### When to Migrate to Production Mode

Move to Angular when you need:
- TypeScript strict mode and type safety
- Unit/integration testing infrastructure
- State management (NgRx, signals)
- Routing and multi-page flows
- Build optimization and tree-shaking
- Production deployment pipeline

---

## Production Mode (Angular + TypeScript)

For production applications, use Angular with the plugin system.

### Plugins (Autoloaded)

Two Claude plugins are active and autoloaded for this project:

- **`default-stack@3dsource`** — Detects whether this is a new or existing Angular project and routes to the correct Angular skill automatically.
- **`angular@3dsource`** — Provides Angular development standards:
  - `angular` skill: Angular v20+ (standalone, zoneless, signals-first, OnPush, SCSS)
  - `angular-legacy` skill: Preserves existing project patterns for older Angular versions

**Follow the plugin guidance for all Angular code.** The `default-stack` plugin will determine which Angular skill applies based on the project state.

### Angular + Configurator Conventions

- Store the `Communicator` API reference in an Angular service, not directly in components
- Expose configurator state as signals from the service (convert event callbacks to signals)
- Components should consume configurator state reactively via the service
- Use `OnPush` change detection (enforced by the Angular plugin) — signal-based state ensures proper updates
- Clean up event listeners when services/components are destroyed

### Code Quality

- Follow all rules from the active Angular plugin (`angular` or `angular-legacy` depending on project)
- TypeScript strict mode — use the types exported from the Metabox API package
- No `any` types for Metabox API interactions — use `Communicator`, `ConfiguratorEnvelope` / `ModularConfiguratorEnvelope`, and command class types

---

## API Documentation

This project includes Metabox API documentation as markdown files. **Read and follow them when working with the configurator API:**

- [Basic Configurator API](./metabox_configurator_basic_api.md) — Single-product configurator (`@3dsource/metabox-front-api`)
- [Modular Configurator API](./metabox_configurator_modular_api.md) — Multi-component configurator (`@3dsource/metabox-modular-configurator-api`)

When implementing configurator features:
1. Read the API doc first to understand available commands, events, and types
2. Use the correct package imports (basic vs modular have different package names and APIs)
3. Follow the patterns documented in the API reference, especially:
   - Initialize via `integrateMetabox()` and only send commands inside the `apiReadyCallback`
   - Use `configuratorDataUpdated` event as the single source of truth for UI state
   - Hide default UI (`ShowEmbeddedMenu(false)`, `ShowOverlayInterface(false)`) when building custom interfaces

---

## Development Rules (All Modes)

### Configurator Integration Patterns

- **API reference is authoritative** — Always check the API doc before implementing or modifying configurator interactions. Do not guess command names, event names, or parameter signatures.
- **Commands only in callback** — Never send commands to the configurator before the `apiReadyCallback` fires. Store the `api` reference and guard all command calls.
- **Event-driven state** — Build UI state from `configuratorDataUpdated` events, not from tracking commands sent. The event payload is the source of truth.
- **HTTPS required** — The configurator requires HTTPS. Account for this in dev environment setup (localhost is exempt).
- **Debounce user interactions** — Rapid material/product changes should be debounced before sending to the API.
- **Critical CSS patterns** — When creating layouts, always use the 7 critical CSS patterns documented in [CSS_PATTERNS.md](./docs/CSS_PATTERNS.md). These patterns prevent layout shifts, viewport jitter, and double scrollbars.

### Template Usage

When a user asks for a prototype or demo:

1. **Determine configurator type** (basic vs modular) - ask if unclear
2. **Copy the appropriate template** to project root or requested location
3. **Update configurator ID** in the HTML file
4. **For modular: Configure root component initialization**
5. **Inform user** of the file location and what to do next (open in browser)

DO NOT:
- Create Angular projects when the user wants a quick prototype
- Modify the critical CSS patterns without understanding their purpose
- Remove the Shim class pattern (it's proven to work)
- Create complex build configurations for simple demos

### Troubleshooting Common Issues

**Layout Problems:**
- Check that all 7 critical CSS patterns are present (see CSS_PATTERNS.md)
- Verify `#embed3DSource` has `width: 100%; height: 100%`
- Ensure container has `position: absolute`
- Confirm panels use `flex: 0 0 50%` (not `flex: 1`)

**Configurator Not Loading:**
- Verify configurator ID is correct (UUID format)
- Ensure using HTTPS or localhost (not HTTP)
- Check browser console for errors
- Verify API package matches configurator type (basic vs modular)

**Controls Not Working:**
- Ensure commands sent after `apiReadyCallback` fires
- Verify event listeners registered before sending commands
- For modular: Check that root component is set first
- Check browser console for command errors

**Viewport Issues:**
- If tiny (300x150px): Add dimensions to `#embed3DSource`
- If jittery: Use `flex: 0 0 50%` instead of `flex: 1`
- If scrolls out of view: Add `overflow: auto` only to right panel
- If double scrollbars: Add `overflow: hidden` to container

See template README files for detailed troubleshooting guides.
