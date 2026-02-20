# Metabox Basic Configurator API

> **Package:** `@3dsource/metabox-front-api`
> **Purpose:** TypeScript/JavaScript API for embedding a single-product 3D configurator (Metabox Basic) into web apps. Controls products, materials, environments, camera, screenshots, and more via iframe postMessage.

## Quick Reference

```ts
// Install
npm install @3dsource/metabox-front-api@latest --save

// Import
import {
  integrateMetabox,
  Communicator,
  ConfiguratorEnvelope,
  SetProduct,
  SetProductMaterial,
  SetEnvironment,
  SetEnvironmentMaterial,
  ShowEmbeddedMenu,
  ShowOverlayInterface,
  SetCamera,
  GetCamera,
  ResetCamera,
  ApplyZoom,
  InitShowcase,
  PlayShowcase,
  PauseShowcase,
  StopShowcase,
  ShowMeasurement,
  HideMeasurement,
  GetScreenshot,
  GetPdf,
  GetCallToActionInformation,
  UnrealCommand,
  saveImage,
} from '@3dsource/metabox-front-api';
```

**CDN (prototyping only):**

```js
import { integrateMetabox, SetProduct, SetProductMaterial, GetScreenshot, saveImage }
  from 'https://cdn.jsdelivr.net/npm/@3dsource/metabox-front-api@latest/+esm';
```

> Pin a specific version for production instead of `@latest`.

---

## Prerequisites

- Modern browser with ES6 modules (Chrome 61+, Firefox 60+, Safari 11+, Edge 79+)
- **HTTPS required** (Unreal Engine streaming mandate; HTTP URLs rejected)
- Valid Metabox configurator ID (UUID)
- Node.js >= 20, npm > 9 (for development)

---

## Core Integration

### `integrateMetabox()`

Embeds the configurator iframe and establishes the postMessage communication channel.

```ts
function integrateMetabox(
  configuratorId: string,
  containerId?: string,             // default: 'embed3DSource'
  apiReadyCallback: (api: Communicator) => void,
  config?: IntegrateMetaboxConfig
): void;
```

**Config options:**

```ts
interface IntegrateMetaboxConfig {
  standalone?: boolean;      // Disable built-in Metabox UI (see Standalone Mode)
  introImage?: string;       // URL for intro image
  introVideo?: string;       // URL for intro video
  loadingImage?: string;     // URL for loading spinner
  state?: ConfiguratorState; // Initial state
  domain?: string;           // Override domain (HTTPS only)
}
```

**Validation:**
- Throws if `configuratorId` is empty
- Throws if container element not found in DOM
- Throws if URL is not HTTPS
- Replaces existing iframe with ID `embeddedContent`

**Iframe URL format:** `https://{domain}/metabox-configurator/basic/{configuratorId}`

**Basic example:**

```ts
integrateMetabox(
  'a1b2c3d4-e5f6-7890-abcd-ef1234567890',
  'my-container',
  (api) => {
    console.log('Configurator ready');
    api.sendCommandToMetabox(new SetProduct('product-123'));
  },
  {
    standalone: true,
    loadingImage: 'https://cdn.example.com/loader.gif',
  }
);
```

### HTML Container Setup

```html
<div id="embed3DSource" style="width: 100%; height: 600px;"></div>
```

The container must have explicit dimensions. The configurator iframe fills the container.

---

## System Terminology

| Term | Description |
|------|-------------|
| **product** | 3D digital twin of a physical product |
| **productId** | UUID identifying a product |
| **environment** | 3D scene/room for product visualization |
| **environmentId** | UUID identifying an environment |
| **externalId** | Custom SKU/identifier for e-commerce integration |
| **showcase** | Camera animation sequence attached to a product |
| **slotId** | Material slot identifier on a product or environment |
| **materialId** | UUID identifying a material |

> **Note:** `component` / `componentId` / `componentType` are **modular configurator only** concepts. The basic configurator works with single products.

---

## Command Classes

All commands are sent via: `api.sendCommandToMetabox(new CommandName(...))`

### Product & Environment

| Command | Parameters | Description |
|---------|-----------|-------------|
| `SetProduct` | `productId: string` | Switch active product |
| `SetProductMaterial` | `slotId: string, materialId: string` | Apply material to product slot |
| `SetEnvironment` | `environmentId: string` | Change scene/environment |
| `SetEnvironmentMaterial` | `slotId: string, materialId: string` | Apply material to environment slot |

```ts
api.sendCommandToMetabox(new SetProduct('product-uuid'));
api.sendCommandToMetabox(new SetProductMaterial('body', 'red-metallic'));
api.sendCommandToMetabox(new SetEnvironment('studio-env'));
```

### UI Control

| Command | Parameters | Description |
|---------|-----------|-------------|
| `ShowEmbeddedMenu` | `visible: boolean` | Show/hide right sidebar |
| `ShowOverlayInterface` | `visible: boolean` | Show/hide viewport controls |

```ts
// Hide default UI for custom interface
api.sendCommandToMetabox(new ShowEmbeddedMenu(false));
api.sendCommandToMetabox(new ShowOverlayInterface(false));
```

### Camera Control

| Command | Parameters | Description |
|---------|-----------|-------------|
| `SetCamera` | `camera: CameraCommandPayload` | Set camera position, rotation, FOV, restrictions |
| `GetCamera` | _(none)_ | Request current camera state (returns via `getCameraResult` event) |
| `ResetCamera` | _(none)_ | Reset to default position |
| `ApplyZoom` | `zoom: number` | Zoom delta (positive = in, negative = out) |

```ts
api.sendCommandToMetabox(new ApplyZoom(0.5));    // zoom in
api.sendCommandToMetabox(new ApplyZoom(-0.25));   // zoom out
api.sendCommandToMetabox(new ResetCamera());

api.sendCommandToMetabox(new SetCamera({
  fov: 45,
  mode: 'orbit',
  position: { x: 100, y: 100, z: 100 },
  rotation: { horizontal: 0, vertical: 0 },
  restrictions: {
    maxDistanceToPivot: 500,
    minDistanceToPivot: 50,
    maxHorizontalRotation: 180,
    minHorizontalRotation: -180,
    maxVerticalRotation: 90,
    minVerticalRotation: -90,
  },
}));

// Get current camera state
api.sendCommandToMetabox(new GetCamera());
api.addEventListener('getCameraResult', (camera) => {
  console.log('Current camera:', camera);
});
```

**`CameraCommandPayload` interface:**

```ts
interface CameraCommandPayload {
  fov?: number;                    // Field of view in degrees
  mode?: 'fps' | 'orbit';         // fps = free movement, orbit = pivot-based (typical)
  position?: { x: number; y: number; z: number };
  rotation?: {
    horizontal: number;            // Yaw (left/right) in degrees
    vertical: number;              // Pitch (up/down) in degrees
  };
  restrictions?: {
    maxDistanceToPivot?: number;   // orbit mode
    minDistanceToPivot?: number;   // orbit mode
    maxFov?: number;               // fps mode
    minFov?: number;               // fps mode
    maxHorizontalRotation?: number;
    minHorizontalRotation?: number;
    maxVerticalRotation?: number;
    minVerticalRotation?: number;
  };
}
```

- Only provide restrictions you want to enforce; omitted values keep current settings.
- `ResetCamera` restores defaults from the product/environment template.
- In orbit mode, distance limits are relative to the orbit pivot.

### Showcase / Animation

| Command | Parameters | Description |
|---------|-----------|-------------|
| `InitShowcase` | _(none)_ | Load animation for current product |
| `PlayShowcase` | _(none)_ | Start/resume playback |
| `PauseShowcase` | _(none)_ | Pause playback |
| `StopShowcase` | _(none)_ | Stop and reset |

### Measurement

| Command | Parameters | Description |
|---------|-----------|-------------|
| `ShowMeasurement` | _(none)_ | Display product dimensions |
| `HideMeasurement` | _(none)_ | Hide dimension overlay |

### Export & Media

| Command | Parameters | Description | Event |
|---------|-----------|-------------|-------|
| `GetScreenshot` | `format: MimeType, size?: {x, y}` | Render screenshot | `screenshotReady` |
| `GetPdf` | _(none)_ | Generate PDF | Server-side |
| `GetCallToActionInformation` | _(none)_ | Trigger CTA workflow | Backend redirect |

```ts
// Take screenshot
api.sendCommandToMetabox(new GetScreenshot('image/png', { x: 1920, y: 1080 }));
api.addEventListener('screenshotReady', (imageData) => {
  if (imageData) {
    saveImage(imageData, `product-${Date.now()}.png`);
  }
});
```

`MimeType` = `'image/png' | 'image/jpeg' | 'image/webp'`

### Advanced Commands (Use with Caution)

| Command | Parameters | Description |
|---------|-----------|-------------|
| `UnrealCommand` | `command: object` | Send raw command to Unreal Engine (bypasses validation) |
| `MetaboxConfig` | `appId: string, config: object` | Internal init (auto-sent, don't use) |

```ts
api.sendCommandToMetabox(new UnrealCommand({
  type: 'SetLightIntensity',
  value: 2.5,
}));
```

---

## Event Handling

Register via `api.addEventListener(eventName, handler)`. Remove via `api.removeEventListener(eventName, handler)`.

### Available Events

| Event | Payload Type | Description |
|-------|-------------|-------------|
| `configuratorDataUpdated` | `ConfiguratorEnvelope` | Product/material/environment state changed |
| `ecomConfiguratorDataUpdated` | `EcomConfigurator` | CTA config loaded (label, URL) |
| `viewportReady` | `boolean` | 3D viewport ready/not ready |
| `showcaseStatusChanged` | `ShowCaseStatus` | `'playing'` / `'paused'` / `'stopped'` |
| `statusMessageChanged` | `string \| null` | Loading/progress message |
| `screenshotReady` | `string \| null` | Base64 screenshot data |
| `getCameraResult` | `CameraCommandPayload` | Response to `GetCamera` command |
| `pdfGenerated` | `string` | PDF generation complete |
| `videoResolutionChanged` | `{width: number\|null, height: number\|null}` | Stream resolution changed |

### Key Event Examples

```ts
// State sync - primary event for building custom UI
api.addEventListener('configuratorDataUpdated', (data: ConfiguratorEnvelope) => {
  console.log('Product:', data.productId);
  console.log('Materials:', data.productMaterialsIds);
  console.log('Environment:', data.environmentId);
  updateProductSelector(data.productId);
  updateMaterialSwatches(data.productMaterialsIds);
});

// Loading state
api.addEventListener('viewportReady', (isReady: boolean) => {
  if (isReady) {
    hideLoadingOverlay();
    enableUserControls();
  }
});

api.addEventListener('statusMessageChanged', (message: string | null) => {
  document.getElementById('loader-text')!.textContent = message || 'Loading...';
});

// Showcase control
api.addEventListener('showcaseStatusChanged', (status: ShowCaseStatus) => {
  const playButton = document.getElementById('play-btn')!;
  playButton.textContent = status === 'playing' ? 'Pause' : 'Play';
});
```

---

## Utility Functions

### `saveImage(imageUrl: string, filename: string): void`

Downloads a base64-encoded image to the user's device. Creates a temporary `<a>` element with `download` attribute.

```ts
api.addEventListener('screenshotReady', (imageData: string | null) => {
  if (imageData) {
    saveImage(imageData, `config-${Date.now()}.png`);
  }
});
```

---

## Standalone Mode

Disables all built-in Metabox UI (menus, overlays, template logic). Use when building a fully custom interface.

```ts
integrateMetabox(
  'configurator-id',
  'embed3DSource',
  (api: Communicator) => {
    api.sendCommandToMetabox(new SetProduct('your-product-id'));
    api.sendCommandToMetabox(new SetEnvironment('your-environment-id'));
  },
  { standalone: true }
);
```

**Key points:**
- Nothing renders until you send `SetProduct` / `SetEnvironment` commands
- All events still fire normally
- You control everything through the API
- Runtime validation (HTTPS, required params) still applies

---

## Building a Custom UI (Pattern)

```ts
integrateMetabox('config-id', 'container', (api) => {
  // 1. Listen for state changes
  api.addEventListener('configuratorDataUpdated', (data) => {
    updateUI(data);
  });

  // 2. Hide default UI
  api.sendCommandToMetabox(new ShowEmbeddedMenu(false));
  api.sendCommandToMetabox(new ShowOverlayInterface(false));

  // 3. Set initial state
  api.sendCommandToMetabox(new SetProduct('product-id'));
  api.sendCommandToMetabox(new SetEnvironment('environment-id'));
  api.sendCommandToMetabox(new SetProductMaterial('slot-id', 'material-id'));
});
```

---

## E-Commerce CTA Integration

E-Com configurators add a call-to-action button. Configuration is done in Metabox admin:
1. **CTA Button Label** - Custom text (e.g., "Get Quote", "Add to Cart")
2. **Callback URL** - HTTP POST endpoint receiving configuration JSON

**Flow:** User clicks CTA -> Metabox POSTs config JSON -> Your backend processes -> Returns `{ redirectUrl }` -> User redirected

```ts
// Trigger CTA from custom button
api.sendCommandToMetabox(new GetCallToActionInformation());

// Listen for CTA config to display button label
api.addEventListener('ecomConfiguratorDataUpdated', (ecomData) => {
  ctaButton.textContent = ecomData.label;
});
```

---

## Framework Integration Notes

### React

```tsx
import { useEffect, useRef, useState, useCallback } from 'react';
import {
  integrateMetabox, Communicator, ConfiguratorEnvelope,
  SetProduct, SetProductMaterial, GetScreenshot, saveImage
} from '@3dsource/metabox-front-api';

function MetaboxConfigurator({ configuratorId }: { configuratorId: string }) {
  const containerRef = useRef<HTMLDivElement>(null);
  const apiRef = useRef<Communicator | null>(null);
  const containerId = useRef(`metabox-${Math.random().toString(36).substr(2, 9)}`);

  useEffect(() => {
    if (!containerRef.current) return;
    containerRef.current.id = containerId.current;
    let mounted = true;

    integrateMetabox(configuratorId, containerId.current, (api) => {
      if (!mounted) return;
      apiRef.current = api;

      api.addEventListener('configuratorDataUpdated', (data: ConfiguratorEnvelope) => {
        if (mounted) console.log('State updated:', data);
      });

      api.addEventListener('screenshotReady', (imageData: string) => {
        if (mounted && imageData) saveImage(imageData, `screenshot-${Date.now()}.png`);
      });
    });

    return () => { mounted = false; };
  }, [configuratorId]);

  const changeProduct = useCallback((productId: string) => {
    apiRef.current?.sendCommandToMetabox(new SetProduct(productId));
  }, []);

  return <div ref={containerRef} style={{ width: '100%', height: '600px' }} />;
}
```

**Key considerations:**
- Generate unique container IDs to avoid conflicts
- Track `mounted` flag to prevent updates after unmount
- Store `api` in a ref (not state) to avoid re-renders

### Angular

```ts
import { Component, input, OnInit, output, signal } from '@angular/core';
import {
  Communicator, ConfiguratorEnvelope, integrateMetabox,
  SetProduct, SetProductMaterial, SetEnvironment,
  ShowEmbeddedMenu, ShowOverlayInterface, GetScreenshot, saveImage
} from '@3dsource/metabox-front-api';

@Component({
  selector: 'app-metabox-configurator',
  template: `<div [id]="containerId()" style="width:100%; height:600px;"></div>`,
})
export class MetaboxConfiguratorComponent implements OnInit {
  configuratorId = input.required<string>();
  stateChange = output<ConfiguratorEnvelope>();

  containerId = signal(`metabox-${Math.random().toString(36).substring(2, 9)}`);
  private api: Communicator | null = null;

  ngOnInit(): void {
    integrateMetabox(this.configuratorId(), this.containerId(), (api) => {
      this.api = api;

      api.addEventListener('configuratorDataUpdated', (data: ConfiguratorEnvelope) => {
        this.stateChange.emit(data);
      });

      api.addEventListener('screenshotReady', (imageData: string | null) => {
        if (imageData) saveImage(imageData, `screenshot-${Date.now()}.png`);
      });
    });
  }

  changeProduct(productId: string): void {
    this.api?.sendCommandToMetabox(new SetProduct(productId));
  }

  applyMaterial(slotId: string, materialId: string): void {
    this.api?.sendCommandToMetabox(new SetProductMaterial(slotId, materialId));
  }
}
```

---

## Troubleshooting

### Configurator Not Loading
- Verify configurator ID is correct and accessible
- Ensure container element exists in DOM **before** calling `integrateMetabox`
- Confirm HTTPS (required, except localhost)
- Check container has non-zero dimensions

### Commands Not Working
- Only send commands inside `apiReadyCallback` (not before)
- Verify product/material/environment IDs are correct UUIDs
- Check browser console for errors

### Events Not Firing
- Register listeners immediately in `apiReadyCallback`
- Verify event name spelling (case-sensitive)

### Screenshot/PDF Issues
- Wait for scene to fully load (`viewportReady` event) before capturing
- Ensure viewport is visible and has non-zero dimensions
- Use reasonable dimensions (e.g., `{ x: 1920, y: 1080 }`)

---

## Best Practices

1. **Error handling** - Wrap command sends in try/catch; listen for errors
2. **Debounce rapid changes** - Don't spam material/product changes
3. **State management** - Use `configuratorDataUpdated` as single source of truth
4. **Responsive container** - Use `width: 100%; height: 60vh; min-height: 400px;`
5. **Loading feedback** - Show loading UI until `viewportReady` fires with `true`
