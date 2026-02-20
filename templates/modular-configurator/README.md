# Metabox Modular Configurator - Quick Start Template

This template provides a complete working configurator for component-assembly configurations using the `@3dsource/metabox-modular-configurator-api` package.

## Quick Start

1. **Copy this folder** to your project location
2. **Edit `index.html` line 382**: Replace `'YOUR-CONFIGURATOR-ID-HERE'` with your configurator ID
3. **Edit lines 388-391**: Uncomment and configure root component initialization
4. **Open `index.html`** in a browser using localhost or HTTPS (HTTP is not supported)

## What's Included

### Layout
- **50/50 split layout**: 3D viewport on left, controls on right
- **Responsive scrolling**: Right panel scrolls independently while viewport stays fixed
- **Critical CSS patterns**: Prevents layout shifts and viewport jitter

### Controls
- **Component type selectors**: Dropdowns for each component type (base, top, legs, etc.)
- **Configuration tree viewer**: Shows current component assembly with root highlighted
- **Material controls**: Buttons for each component's material slots (organized by component)
- **Environment selector**: Dropdown to change lighting/background
- **Export tools**: Screenshot and PDF generation buttons

### Architecture
- **Vue 3** via CDN (no build step required)
- **Shim class pattern** for API management
- **Event-driven state** from `configuratorDataUpdated` event
- **Standalone mode** (no embedded menu or overlay)
- **Modular API** for component-based configuration

## Key Differences from Basic Configurator

### API Package
Uses `@3dsource/metabox-modular-configurator-api` instead of `@3dsource/metabox-front-api`

### Commands
- `SetComponent(componentId, typeId, isRoot)` - Add/change components
- `SetComponentMaterial(componentId, slotId, materialId)` - Change component materials
- No `SetProduct` or `SetProductMaterial` (those are basic-only)

### State Structure
- `ModularConfiguratorEnvelope` instead of `ConfiguratorEnvelope`
- `configurationTree` - Array of current components in the configuration
- `componentsByComponentType` - Available components organized by type
- `componentTypes` - Available component types (base, top, legs, etc.)

### Initialization
**CRITICAL**: Must set a root component before adding children:
```javascript
// Set root component first
this.shim.setComponent('root-component-id', 'base-type-id', true);

// Then set environment
this.shim.setEnvironment('environment-id');

// Then add child components
this.shim.setComponent('child-id', 'type-id', false);
```

## Customization

### Configure Root Component
Lines 382-394 in `index.html`:
```javascript
integrateMetabox(
  'YOUR-CONFIGURATOR-ID-HERE',
  'embed3DSource',
  (api) => {
    this.shim.ready(api);

    // IMPORTANT: Set root component first
    this.shim.setComponent('root-component-id', 'base-type-id', true);
    this.shim.setEnvironment('environment-id');
  },
  { standalone: true }
);
```

**How to find the right IDs:**
1. Open the configurator in the Metabox admin panel
2. Inspect the component types - find which one has `isRootType: true`
3. Get a component ID from that root type
4. Get an environment ID

### Modify layout proportions
To change the 50/50 split (e.g., 60/40), edit the CSS (lines 42-45):
```css
.left-panel {
  flex: 0 0 60%;  /* 60% for viewport */
}
.right-panel {
  flex: 0 0 40%;  /* 40% for controls */
}
```

**Important**: Always use `flex: 0 0 X%` (no grow/shrink) to prevent layout shifts.

### Add custom component logic
Extend the Shim class with custom methods (after line 332):
```javascript
// In Shim class
removeComponent(componentId) {
  // Custom logic to remove a component
  // (API may provide a RemoveComponent command - check docs)
}

resetConfiguration() {
  // Custom logic to reset to default configuration
  this.setComponent('default-root-id', 'base-type-id', true);
}
```

### Style customization
All CSS is inline in the `<style>` block (lines 14-242). Modify colors, spacing, fonts, etc. as needed.

**Do not remove** the critical layout CSS (lines 14-56) - these patterns prevent viewport issues.

## API Reference

See [/metabox_configurator_modular_api.md](../../metabox_configurator_modular_api.md) for complete API documentation.

### Key Commands (Modular-Specific)
- `SetComponent(componentId, typeId, isRoot)` - Add or change a component
- `SetComponentMaterial(componentId, slotId, materialId)` - Change component material
- `SetEnvironment(environmentId)` - Change environment
- `GetScreenshot(format, size)` - Export render image
- `GetPdf()` - Export PDF

### Key Events
- `configuratorDataUpdated` - Fired on state changes (use as source of truth)
  - Returns `ModularConfiguratorEnvelope` with `configurationTree`, `componentsByComponentType`, etc.
- `screenshotReady` - Fired when render is ready

## Troubleshooting

### Configurator not loading
- **Verify configurator ID is correct** - Should be a UUID format
- **Ensure you're using HTTPS or localhost** - HTTP is not supported (except localhost)
- **Check browser console for errors** - Look for CORS or network issues
- **Verify the configurator is modular** - Basic configurator IDs won't work with modular API

### Root component not set
- **Error: "Root component required"** - Must call `SetComponent` with `isRoot: true` before children
- **No components appearing** - Check that root component is set in the `apiReadyCallback`
- **Configuration tree empty** - Ensure `SetComponent` is called after API is ready

### Components not appearing in selectors
- **Check `componentsByComponentType`** - May be empty if configurator has no components
- **Verify component types are configured** - Check Metabox admin panel
- **Inspect `configuratorDataUpdated` event data** - Log to console to see what's available

### Materials not appearing
- **Check if component has material slots** - Some components may not have customizable materials
- **Verify slot has `enabledMaterials`** - Empty slots are filtered out
- **Check component ID is correct** - Material calls need the right component ID

### Layout issues
- **Viewport appears tiny** - Ensure `#embed3DSource` has `width: 100%; height: 100%`
- **Double scrollbars** - Ensure container has `overflow: hidden`, only right panel has `overflow: auto`
- **Layout shifts on load** - Ensure container has `position: absolute`
- **Panels not 50/50** - Ensure using `flex: 0 0 50%` (no grow/shrink)

### Controls not working
- **Verify commands are sent after `apiReadyCallback` fires** - Commands before callback will fail
- **Check that Shim `ready()` method is called** - Should be called in the callback
- **Ensure root component is set** - Must be set before other commands
- **Check console for errors** - Look for command/parameter issues

## Common Patterns

### Initialize with Default Configuration
```javascript
integrateMetabox(
  'configurator-id',
  'embed3DSource',
  (api) => {
    this.shim.ready(api);

    // Set root first
    this.shim.setComponent('base-id', 'base-type', true);

    // Set environment
    this.shim.setEnvironment('env-id');

    // Add default children
    this.shim.setComponent('top-id', 'top-type', false);
    this.shim.setComponent('legs-id', 'legs-type', false);

    // Set default materials
    this.shim.setComponentMaterial('base-id', 'slot-id', 'material-id');
  },
  { standalone: true }
);
```

### Handle Component Selection
```html
<select @change="onComponentSelected($event, componentType)">
  <option value="">-- Select --</option>
  <option v-for="component in availableComponents" :value="component.id">
    {{ component.title }}
  </option>
</select>
```

```javascript
onComponentSelected(event, componentType) {
  const componentId = event.target.value;
  if (componentId) {
    this.shim.setComponent(
      componentId,
      componentType.id,
      componentType.isRootType
    );
  }
}
```

### Display Component Tree
```html
<div v-for="item in shim.configurationTree" :key="item.component.id">
  <strong>{{ item.component.title }}</strong>
  <span v-if="item.isRoot">(Root)</span>
  <span>(Type: {{ item.componentType.title }})</span>
</div>
```

## Migration to Production

When you need Angular/TypeScript for production:
1. Follow the Production Mode instructions in [/CLAUDE.md](../../CLAUDE.md)
2. Use the Angular plugin system for proper architecture
3. Migrate the Shim class to an Angular service
4. Convert state to signals for reactive updates
5. Add TypeScript strict mode and proper typing using `ModularConfiguratorEnvelope` types

## Additional Resources

- [Metabox Modular API Documentation](../../metabox_configurator_modular_api.md)
- [CSS Patterns Documentation](../../docs/CSS_PATTERNS.md)
- [3DSource Documentation](https://docs.3dsource.com)
