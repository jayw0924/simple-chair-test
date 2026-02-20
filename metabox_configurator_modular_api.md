# Metabox Modular Configurator API

> **Package:** `@3dsource/metabox-modular-configurator-api`
> **Purpose:** TypeScript/JavaScript API for embedding a multi-component 3D configurator (Metabox Modular) into web apps. Assembles products from modular components with per-component materials, hierarchical component trees, and environment control.

## Quick Reference

```ts
// Install
npm install @3dsource/metabox-modular-configurator-api@latest --save

// Import
import {
  integrateMetabox,
  Communicator,
  ModularConfiguratorEnvelope,
  SetComponent,
  SetComponentMaterial,
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
} from '@3dsource/metabox-modular-configurator-api';
```

**CDN (prototyping only):**

```js
import { integrateMetabox, SetComponent, SetEnvironment, GetScreenshot, saveImage }
  from 'https://cdn.jsdelivr.net/npm/@3dsource/metabox-modular-configurator-api@latest/+esm';
```

> Pin a specific version for production instead of `@latest`.

---

## Key Differences from Basic Configurator

| Aspect | Basic (`metabox-front-api`) | Modular (`metabox-modular-configurator-api`) |
|--------|----------------------------|----------------------------------------------|
| **Product model** | Single monolithic product | Composed of multiple components |
| **Selection** | `SetProduct(productId)` | `SetComponent(componentId, typeId, isRoot)` |
| **Materials** | `SetProductMaterial(slotId, materialId)` | `SetComponentMaterial(componentId, slotId, materialId)` |
| **State type** | `ConfiguratorEnvelope` | `ModularConfiguratorEnvelope` |
| **State structure** | Flat (product + materials) | Hierarchical tree (root + children) |
| **Iframe URL** | `.../basic/{id}` | `.../modular/{id}` |

---

## Prerequisites

- Modern browser with ES6 modules (Chrome 61+, Firefox 60+, Safari 11+, Edge 79+)
- **HTTPS required** (Unreal Engine streaming mandate; HTTP URLs rejected)
- Valid Metabox **modular** configurator ID (UUID)
- Node.js >= 20, npm > 9 (for development)

---

## Core Integration

### `integrateMetabox()`

Embeds the modular configurator iframe and establishes the postMessage communication channel.

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
  state?: ConfiguratorState; // Initial component tree state
  domain?: string;           // Override domain (HTTPS only)
}
```

**Validation:**
- Throws if `configuratorId` is empty
- Throws if container element not found in DOM
- Throws if URL is not HTTPS
- Replaces existing iframe with ID `embeddedContent`

**Iframe URL format:** `https://{domain}/metabox-configurator/modular/{configuratorId}`

**Example:**

```ts
integrateMetabox(
  'modular-config-uuid',
  'my-container',
  (api) => {
    console.log('Modular configurator ready');
    // Set root component first (required)
    api.sendCommandToMetabox(new SetComponent('comp-1', 'base', true));
    api.sendCommandToMetabox(new SetEnvironment('env-1'));
  },
  {
    standalone: true,
    loadingImage: 'https://cdn.example.com/loader.gif',
  }
);
```

---

## System Terminology

| Term | Description |
|------|-------------|
| **component** | Modular product part/assembly (e.g., wheel, bumper, seat) |
| **componentId** | UUID identifying a component |
| **componentType** | Category of component (e.g., "wheel", "bumper") |
| **environment** | 3D scene/room for visualization |
| **environmentId** | UUID identifying an environment |
| **product** | Base 3D digital twin referenced by components |
| **productId** | UUID identifying a product |
| **externalId** | Custom SKU for e-commerce integration |
| **showcase** | Camera animation attached to a component |
| **slotId** | Material slot identifier on component/environment |
| **materialId** | UUID identifying a material |
| **configurationTree** | Hierarchical structure of assembled components |

---

## Type Definitions

### `Communicator`

```ts
interface Communicator {
  sendCommandToMetabox<T extends keyof ToMetaboxMessagePayloads>(
    command: CommandBase<T>
  ): void;
  addEventListener<K extends keyof FromMetaboxMessagePayloads>(
    eventType: K,
    listener: (data: FromMetaboxMessagePayloads[K]) => void
  ): void;
  removeEventListener<K extends keyof FromMetaboxMessagePayloads>(
    eventType: K,
    listener: (data: FromMetaboxMessagePayloads[K]) => void
  ): void;
}
```

### `ModularConfiguratorEnvelope`

The primary state object received via the `configuratorDataUpdated` event.

```ts
interface ModularConfiguratorEnvelope {
  /** Full configurator definition with components, types, and environments */
  configurator: InitialModularConfigurator;
  /** Hierarchical tree of assembled components with parent references */
  configurationTree: ComponentWithParent[];
  /** Map of component type -> currently selected component ID */
  componentsByComponentType: Record<string, string>;
  /** Map of component material selections */
  componentMaterialsIds: SelectedIds;
  /** Current environment UUID */
  environmentId: string;
  /** Map of environment material selections */
  environmentMaterialsIds: SelectedIds;
}
```

### `ModularConfiguratorComponent`

```ts
interface ModularConfiguratorComponent {
  id: string;
  componentType: ModularConfiguratorComponentType;
  name: string;
  position: number;
  product: Product;
  virtualSocketToConnectorsMap?: ModularConfiguratorVirtualSocketToConnector[];
  __typename: 'ModularConfiguratorComponent';
}
```

### `Material`

```ts
interface Material {
  id: string;
  title: string;
  externalId: string;
  thumbnailList: Thumbnail[];
  __typename: 'Material';
}
```

### `Environment`

```ts
interface Environment {
  id: string;
  title: string;
  udsEnabled: boolean;
  sunNorthYaw: number;
  udsHour: number;
  thumbnailList: Thumbnail[];
  slots: Slot[];
  __typename: 'Environment';
}
```

### Utility Types

```ts
type MimeType = 'image/png' | 'image/jpeg' | 'image/webp';
type ApplicationDeviceTypes = 'desktop' | 'tablet' | 'mobile';
```

---

## Command Classes

All commands are sent via: `api.sendCommandToMetabox(new CommandName(...))`

### Component & Environment Management

| Command | Parameters | Description |
|---------|-----------|-------------|
| `SetComponent` | `componentId: string, typeId: string, isRoot: boolean` | Add/select component in assembly |
| `SetComponentMaterial` | `componentId: string, slotId: string, materialId: string` | Apply material to component slot |
| `SetEnvironment` | `environmentId: string` | Change scene/environment |
| `SetEnvironmentMaterial` | `slotId: string, materialId: string` | Apply material to environment slot |

**Critical: Always set the root component first (`isRoot: true`), then add children.**

```ts
// Set root component (e.g., vehicle chassis)
api.sendCommandToMetabox(new SetComponent('chassis-001', 'base', true));

// Add child components
api.sendCommandToMetabox(new SetComponent('wheel-fr', 'wheel', false));
api.sendCommandToMetabox(new SetComponent('wheel-fl', 'wheel', false));
api.sendCommandToMetabox(new SetComponent('engine-001', 'engine', false));

// Apply materials per component
api.sendCommandToMetabox(new SetComponentMaterial('chassis-001', 'body', 'metallic-blue'));
api.sendCommandToMetabox(new SetComponentMaterial('wheel-fr', 'rim', 'chrome'));

// Set environment
api.sendCommandToMetabox(new SetEnvironment('studio-white'));
api.sendCommandToMetabox(new SetEnvironmentMaterial('floor', 'concrete-grey'));
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
    maxFov?: number;
    minFov?: number;
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
| `InitShowcase` | _(none)_ | Load animation for current component |
| `PlayShowcase` | _(none)_ | Start/resume playback |
| `PauseShowcase` | _(none)_ | Pause playback |
| `StopShowcase` | _(none)_ | Stop and reset |

### Measurement

| Command | Parameters | Description |
|---------|-----------|-------------|
| `ShowMeasurement` | _(none)_ | Display component dimensions |
| `HideMeasurement` | _(none)_ | Hide dimension overlay |

### Export & Media

| Command | Parameters | Description | Event |
|---------|-----------|-------------|-------|
| `GetScreenshot` | `format: MimeType, size?: {x, y}` | Render screenshot | `screenshotReady` |
| `GetPdf` | _(none)_ | Generate PDF | Server-side |
| `GetCallToActionInformation` | _(none)_ | Trigger CTA workflow | Backend redirect |

```ts
api.sendCommandToMetabox(new GetScreenshot('image/png', { x: 2048, y: 2048 }));
api.addEventListener('screenshotReady', (imageData) => {
  if (imageData) saveImage(imageData, `assembly-${Date.now()}.png`);
});
```

### Advanced Commands (Use with Caution)

| Command | Parameters | Description |
|---------|-----------|-------------|
| `UnrealCommand` | `command: object \| string` | Send raw command to Unreal Engine (bypasses validation) |
| `MetaboxConfig` | `appId: string, config: object` | Internal init (auto-sent, don't use) |

```ts
api.sendCommandToMetabox(new UnrealCommand({
  action: 'SetLightIntensity',
  lightId: 'keyLight',
  value: 2.5,
}));

// String-based command (if supported)
api.sendCommandToMetabox(new UnrealCommand('SetSunTime 14:30'));
```

---

## Event Handling

Register via `api.addEventListener(eventName, handler)`. Remove via `api.removeEventListener(eventName, handler)`.

### Available Events

| Event | Payload Type | Description |
|-------|-------------|-------------|
| `configuratorDataUpdated` | `ModularConfiguratorEnvelope` | Component tree/material/environment state changed |
| `ecomConfiguratorDataUpdated` | `EcomConfigurator` | CTA config loaded (label, URL) |
| `viewportReady` | `boolean` | 3D viewport ready/not ready |
| `showcaseStatusChanged` | `ShowCaseStatus` | `'playing'` / `'paused'` / `'stopped'` |
| `statusMessageChanged` | `string \| null` | Loading/progress message |
| `screenshotReady` | `string \| null` | Base64 screenshot data |
| `getCameraResult` | `CameraCommandPayload` | Response to `GetCamera` command |
| `videoResolutionChanged` | `{width: number\|null, height: number\|null}` | Stream resolution changed |

### Key Event Examples

```ts
// State sync - primary event for building custom UI
api.addEventListener('configuratorDataUpdated', (data: ModularConfiguratorEnvelope) => {
  console.log('Environment:', data.environmentId);
  console.log('Component tree:', data.configurationTree);
  console.log('Components by type:', data.componentsByComponentType);
  console.log('Component materials:', data.componentMaterialsIds);
  updateComponentTree(data.configurationTree);
  updateMaterialSelectors(data.componentMaterialsIds);
});

// Loading state
api.addEventListener('viewportReady', (isReady: boolean) => {
  if (isReady) {
    hideLoadingSpinner();
    enableComponentSelectors();
  }
});

api.addEventListener('statusMessageChanged', (message: string | null) => {
  document.getElementById('status-text')!.textContent = message || '';
});

// Screenshot
api.addEventListener('screenshotReady', (imageData: string | null) => {
  if (imageData) saveImage(imageData, `assembly-${Date.now()}.png`);
});
```

---

## Standalone Mode

Disables all built-in Metabox UI (menus, overlays, template logic). Use when building a fully custom modular interface.

### When to Use

- Building fully custom component selection UX
- Integrating into existing design systems
- Need complete control over assembly flow

### Behavior Changes

| Feature | Default Mode | Standalone Mode |
|---------|-------------|-----------------|
| Right sidebar menu | Visible | Hidden |
| Viewport overlays | Visible | Hidden |
| Template logic | Active | Disabled |
| API control | Partial | Full |
| Component tree management | Shared | Full control |
| Event system | Available | Available |

### Implementation

```ts
integrateMetabox(
  'modular-config-id',
  'embed3DSource',
  (api) => {
    // 1. Listen for state changes first
    api.addEventListener('configuratorDataUpdated', (data) => {
      updateCustomComponentTree(data.configurationTree);
      updateMaterialSelectors(data.componentMaterialsIds);
    });

    // 2. Set root component (MANDATORY - nothing renders without this)
    api.sendCommandToMetabox(new SetComponent('chassis-001', 'base', true));

    // 3. Set environment
    api.sendCommandToMetabox(new SetEnvironment('studio-white'));

    // 4. Build component tree
    api.sendCommandToMetabox(new SetComponent('wheel-fl', 'wheel', false));
    api.sendCommandToMetabox(new SetComponent('wheel-fr', 'wheel', false));

    // 5. Apply materials
    api.sendCommandToMetabox(new SetComponentMaterial('chassis-001', 'body', 'metallic-blue'));
  },
  { standalone: true }
);
```

**Critical notes:**
- Standalone starts with a blank viewport - you **must** send `SetComponent(id, type, true)` and `SetEnvironment` to initialize
- Always set the root component **before** children
- All events and commands work normally
- HTTPS and parameter validation still enforced

---

## Building a Custom UI (Pattern)

```ts
integrateMetabox('config-id', 'container', (api) => {
  // 1. Listen for state changes
  api.addEventListener('configuratorDataUpdated', (data: ModularConfiguratorEnvelope) => {
    updateComponentSelector(data.componentsByComponentType);
    updateMaterialOptions(data.componentMaterialsIds);
  });

  // 2. Hide default UI
  api.sendCommandToMetabox(new ShowEmbeddedMenu(false));
  api.sendCommandToMetabox(new ShowOverlayInterface(false));

  // 3. Set root component
  api.sendCommandToMetabox(new SetComponent('root-id', 'base', true));

  // 4. Set environment
  api.sendCommandToMetabox(new SetEnvironment('env-id'));

  // 5. Add children and materials
  api.sendCommandToMetabox(new SetComponent('child-id', 'wheel', false));
  api.sendCommandToMetabox(new SetComponentMaterial('root-id', 'body', 'material-id'));
});
```

---

## E-Commerce CTA Integration

E-Com configurators add a call-to-action button. Configuration is done in Metabox admin:
1. **CTA Button Label** - Custom text (e.g., "Request Quote", "Add to Cart")
2. **Callback URL** - HTTP POST endpoint receiving configuration JSON (includes full BOM)

**Flow:** User clicks CTA -> Metabox POSTs config JSON -> Your backend processes -> Returns `{ redirectUrl }` -> User redirected

```ts
api.sendCommandToMetabox(new GetCallToActionInformation());
```

---

## Component Tree Concept

The modular configurator uses a hierarchical tree where one root component anchors the assembly:

```
Root (chassis, isRoot: true)
  ├── Child (wheel-front-left, isRoot: false)
  ├── Child (wheel-front-right, isRoot: false)
  └── Child (engine, isRoot: false)
```

```ts
// Correct order - root FIRST
api.sendCommandToMetabox(new SetComponent('chassis', 'base', true));   // Root
api.sendCommandToMetabox(new SetComponent('wh-fl', 'wheel', false));   // Child
api.sendCommandToMetabox(new SetComponent('wh-fr', 'wheel', false));   // Child
api.sendCommandToMetabox(new SetComponent('engine', 'engine', false)); // Child

// WRONG - no root set, nothing will render
api.sendCommandToMetabox(new SetComponent('wh-fl', 'wheel', false));
```

---

## Troubleshooting

### Configurator Not Loading
- Verify **modular** configurator ID (not basic) is correct
- Ensure container element exists in DOM **before** calling `integrateMetabox`
- Confirm HTTPS (required, except localhost)
- Check container has non-zero dimensions (set explicit width/height via CSS)
- Check browser console for CORS/X-Frame-Options errors (domain allowlist in Metabox settings)

### Commands Not Working
- Only send commands inside `apiReadyCallback` (not before)
- **Set root component first** with `isRoot: true` - nothing renders without it
- Verify component/material/environment IDs from `configuratorDataUpdated` event data
- Ensure `typeId` matches available component types in your configurator

### TypeScript Errors

```ts
// Correct import path
import type { Communicator, ModularConfiguratorEnvelope }
  from '@3dsource/metabox-modular-configurator-api';

// NOT from /types subpath
```

**Required tsconfig settings:**
```json
{
  "compilerOptions": {
    "moduleResolution": "node",
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

### Component Tree Issues
- Always set root component (`isRoot: true`) before any children
- Verify component IDs match what's available in the configurator data
- Check `configurationTree` in the `configuratorDataUpdated` event for current hierarchy
