# Metabox Basic Configurator - Quick Start Template

This template provides a complete working configurator for single-product configurations using the `@3dsource/metabox-front-api` package.

## Quick Start

1. **Copy this folder** to your project location
2. **Edit `index.html` line 330**: Replace `'YOUR-CONFIGURATOR-ID-HERE'` with your configurator ID
3. **Open `index.html`** in a browser using localhost or HTTPS (HTTP is not supported)

That's it! Your configurator should load and display controls automatically.

## What's Included

### Layout
- **50/50 split layout**: 3D viewport on left, controls on right
- **Responsive scrolling**: Right panel scrolls independently while viewport stays fixed
- **Critical CSS patterns**: Prevents layout shifts and viewport jitter

### Controls
- **Product selector**: Dropdown to switch between products
- **Material controls**: Buttons for each material slot (seat, frame, etc.)
- **Environment selector**: Dropdown to change lighting/background
- **Environment materials**: Buttons to customize environment materials (if available)
- **Export tools**: Screenshot and PDF generation buttons
- **Call-to-action**: Optional e-commerce integration button

### Architecture
- **Vue 3** via CDN (no build step required)
- **Shim class pattern** for API management
- **Event-driven state** from `configuratorDataUpdated` event
- **Standalone mode** (no embedded menu or overlay)

## Customization

### Change the configurator ID
Line 330:
```javascript
integrateMetabox(
  'YOUR-CONFIGURATOR-ID-HERE',  // <-- Replace this
  'embed3DSource',
  (api) => { ... }
);
```

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

### Add custom buttons
1. Add HTML in the right panel Vue template (after line 346)
2. Add a method to the Shim class (after line 283)
3. Call the method from your button: `@click="shim.yourMethod()"`

Example:
```javascript
// In Shim class
sendCustomCommand() {
  this.api.sendCommandToMetabox(new SomeCommand(params));
}
```

```html
<!-- In right panel -->
<button @click="shim.sendCustomCommand()">Custom Action</button>
```

### Style customization
All CSS is inline in the `<style>` block (lines 14-189). Modify colors, spacing, fonts, etc. as needed.

**Do not remove** the critical layout CSS (lines 14-56) - these patterns prevent viewport issues.

### Add loading/intro images
Add to the `integrateMetabox` options (line 266):
```javascript
{
  standalone: true,
  introImage: 'https://your-domain.com/intro.png',
  loadingImage: 'https://your-domain.com/loading.png',
}
```

## API Reference

See [/metabox_configurator_basic_api.md](../../metabox_configurator_basic_api.md) for complete API documentation.

### Key Commands
- `SetProduct(productId)` - Switch products
- `SetProductMaterial(slotId, materialId)` - Change product material
- `SetEnvironment(environmentId)` - Change environment
- `SetEnvironmentMaterial(slotId, materialId)` - Change environment material
- `GetScreenshot(format, size)` - Export render image
- `GetPdf()` - Export PDF

### Key Events
- `configuratorDataUpdated` - Fired on state changes (use as source of truth)
- `screenshotReady` - Fired when render is ready
- `ecomConfiguratorDataUpdated` - Fired with e-commerce data

## Troubleshooting

### Configurator not loading
- **Verify configurator ID is correct** - Should be a UUID format
- **Ensure you're using HTTPS or localhost** - HTTP is not supported (except localhost)
- **Check browser console for errors** - Look for CORS or network issues
- **Verify the configurator exists** - Test the ID in the Metabox admin panel

### Layout issues
- **Viewport appears tiny** - Ensure `#embed3DSource` has `width: 100%; height: 100%`
- **Double scrollbars** - Ensure container has `overflow: hidden`, only right panel has `overflow: auto`
- **Layout shifts on load** - Ensure container has `position: absolute`
- **Panels not 50/50** - Ensure using `flex: 0 0 50%` (no grow/shrink)

### Controls not working
- **Verify API commands are sent after `apiReadyCallback` fires** - Commands before callback will fail silently
- **Check that Shim `ready()` method is called** - Should be called in the callback
- **Ensure event listeners are registered before sending commands** - Register in `addApiEventListeners()`
- **Check console for errors** - Look for command/parameter issues

### Materials not appearing
- **Check if slot has `enabledMaterials`** - Empty slots are filtered out
- **Verify product has material slots** - Some products may not have customizable materials
- **Check `configuratorDataUpdated` event data** - Inspect what's returned in the browser console

### Screenshot/PDF not working
- **Ensure configurator is fully loaded** - Wait for `configuratorDataUpdated` event
- **Check browser console for errors** - Look for screenshot/PDF generation errors
- **Verify screenshot size** - Large sizes may fail or timeout
- **Check popup blockers** - PDF generation may be blocked by browser

## Migration to Production

When you need Angular/TypeScript for production:
1. Follow the Production Mode instructions in [/CLAUDE.md](../../CLAUDE.md)
2. Use the Angular plugin system for proper architecture
3. Migrate the Shim class to an Angular service
4. Convert state to signals for reactive updates
5. Add TypeScript strict mode and proper typing

## Additional Resources

- [Metabox Basic API Documentation](../../metabox_configurator_basic_api.md)
- [CSS Patterns Documentation](../../docs/CSS_PATTERNS.md)
- [3DSource Documentation](https://docs.3dsource.com)
