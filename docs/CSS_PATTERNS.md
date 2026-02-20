# Critical CSS Patterns for Metabox Configurator Layouts

This document explains the CSS patterns required for proper configurator embedding and viewport stability.

## The Problem

When embedding the Metabox configurator iframe, common CSS mistakes lead to:
- Layout shifts during page load
- Incorrect iframe dimensions
- Double scrollbars
- Viewport jitter when controls update
- Flexbox sizing conflicts

## The Solution

A specific set of CSS patterns that ensure viewport stability and proper iframe sizing.

---

## Pattern 1: Absolute Positioning Container

```css
.container {
  position: absolute;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
```

**Why:**
- `position: absolute` removes the container from normal document flow, preventing reflows
- `width: 100%; height: 100%` fills the viewport completely
- `overflow: hidden` prevents scrollbars on the container itself (scrolling happens inside panels)

**What happens without it:**
- Container height collapses if using default `position: static`
- Layout shifts occur when content loads
- Viewport size changes as DOM elements render

**Example of the problem:**
```css
/* BAD - causes layout shifts */
.container {
  display: flex;  /* No position, no explicit dimensions */
}
```

Result: Container height depends on content height, which changes as the configurator loads and controls render.

---

## Pattern 2: Fixed Flex Sizing

```css
.left-panel,
.right-panel {
  flex: 0 0 50%;
}
```

**Why:**
- `flex: 0 0 50%` means:
  - `0` = no flex grow (won't expand beyond 50%)
  - `0` = no flex shrink (won't contract below 50%)
  - `50%` = base size is exactly 50% of container
- This creates a perfect 50/50 split that never changes

**What happens without it:**
- Using `flex: 1` causes panels to grow/shrink based on content
- Viewport width changes as controls render
- 3D iframe gets resized unexpectedly, causing re-renders

**Example of the problem:**
```css
/* BAD - causes viewport jitter */
.left-panel,
.right-panel {
  flex: 1;  /* Panels resize based on content */
}
```

Result: As buttons/dropdowns load in the right panel, flexbox recalculates space, causing the left panel (viewport) to resize.

---

## Pattern 3: Full-Height Left Panel

```css
.left-panel {
  height: 100%;
}
```

**Why:**
- Ensures the left panel (containing the 3D viewport) fills the entire vertical space
- Prevents vertical gaps or overflow

**What happens without it:**
- Panel height depends on content, which is minimal (just the embed div)
- Viewport appears squashed or doesn't fill space

**Example of the problem:**
```css
/* BAD - viewport doesn't fill space */
.left-panel {
  /* No explicit height */
}
```

Result: The panel collapses to the height of its content (`#embed3DSource`), which has no intrinsic height without this pattern.

---

## Pattern 4: Independent Scrolling Right Panel

```css
.right-panel {
  overflow: auto;
  padding: 20px;
}
```

**Why:**
- `overflow: auto` enables scrolling only when content exceeds panel height
- Keeps 3D viewport fixed while controls scroll
- Padding provides spacing for readability

**What happens without it:**
- Entire page scrolls, causing 3D viewport to scroll out of view
- Or, no scrolling occurs and content is cut off

**Example of the problem:**
```css
/* BAD - entire page scrolls */
body {
  overflow: auto;  /* Scrolls everything including viewport */
}
.right-panel {
  overflow: visible;  /* No independent scrolling */
}
```

Result: When controls exceed viewport height, the entire page scrolls, moving the 3D viewport off-screen.

---

## Pattern 5: Full-Size Embed Container

```css
#embed3DSource {
  width: 100%;
  height: 100%;
  background-color: #e0e0e0;
}
```

**Why:**
- `width: 100%; height: 100%` fills the parent (left panel) completely
- `background-color` provides a loading placeholder before iframe renders
- The iframe injected by `integrateMetabox()` respects these dimensions

**What happens without it:**
- Iframe gets default dimensions (300x150px in most browsers)
- Viewport appears tiny or incorrectly sized
- `integrateMetabox()` cannot determine proper iframe size

**Example of the problem:**
```css
/* BAD - tiny viewport */
#embed3DSource {
  /* No explicit dimensions */
}
```

Result: The iframe defaults to 300x150px, creating a tiny unusable viewport.

---

## Pattern 6: Body Reset

```css
body {
  margin: 0;
  overflow: hidden;
  height: 100vh;
}
```

**Why:**
- `margin: 0` removes default browser margins that cause gaps
- `overflow: hidden` prevents page-level scrollbars (panels handle their own scrolling)
- `height: 100vh` ensures body fills viewport height

**What happens without it:**
- White margins appear around the container
- Double scrollbars (page + panel) create confusion
- Container doesn't fill full viewport

**Example of the problem:**
```css
/* BAD - has margins and scrollbars */
body {
  /* Uses browser defaults: margin: 8px, overflow: visible */
}
```

Result: 8px white border around the page, scrollbars on both body and right panel.

---

## Pattern 7: Box-Sizing Reset

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

**Why:**
- Ensures padding and borders are included in element dimensions
- Prevents layout shifts when adding padding/borders to elements
- Makes calculations predictable (50% means exactly 50%, including padding)

**What happens without it:**
- Adding padding to panels changes their effective width
- `flex: 0 0 50%` plus padding exceeds 50%, breaking the layout
- Elements overflow containers unexpectedly

**Example of the problem:**
```css
/* BAD - no box-sizing reset */
/* Browser default is box-sizing: content-box */

.right-panel {
  flex: 0 0 50%;
  padding: 20px;  /* Adds 40px to width! */
}
```

Result: Right panel is now 50% + 40px wide, breaking the 50/50 split and causing horizontal overflow.

---

## Complete Minimal Template

Here's a minimal HTML template with all critical patterns:

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    /* Pattern 7: Box-sizing reset */
    *,*::before,*::after {
      box-sizing: border-box;
    }

    /* Pattern 6: Body reset */
    body {
      margin: 0;
      overflow: hidden;
      height: 100vh;
    }

    /* Pattern 1: Absolute positioning container */
    .container {
      position: absolute;
      width: 100%;
      height: 100%;
      display: flex;
      overflow: hidden;
    }

    /* Pattern 2: Fixed flex sizing */
    .left-panel, .right-panel {
      flex: 0 0 50%;
    }

    /* Pattern 3: Full-height left panel */
    .left-panel {
      height: 100%;
    }

    /* Pattern 4: Independent scrolling right panel */
    .right-panel {
      overflow: auto;
      padding: 20px;
    }

    /* Pattern 5: Full-size embed container */
    #embed3DSource {
      width: 100%;
      height: 100%;
      background-color: #e0e0e0;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="left-panel">
      <div id="embed3DSource"></div>
    </div>
    <div class="right-panel">
      <!-- Controls here -->
    </div>
  </div>
</body>
</html>
```

---

## Customization Guidelines

### Changing Panel Proportions

To adjust the split (e.g., 60/40 instead of 50/50):

```css
.left-panel { flex: 0 0 60%; }
.right-panel { flex: 0 0 40%; }
```

Always use `flex: 0 0 X%` (no grow/shrink) to prevent layout shifts.

### Changing to Vertical Stack (Mobile)

For mobile, stack panels vertically:

```css
@media (max-width: 768px) {
  .container {
    flex-direction: column;
  }
  .left-panel, .right-panel {
    flex: 0 0 50%; /* Each takes 50% of vertical space */
  }
}
```

Or make viewport take 60% and controls 40%:

```css
@media (max-width: 768px) {
  .container {
    flex-direction: column;
  }
  .left-panel {
    flex: 0 0 60%;
  }
  .right-panel {
    flex: 0 0 40%;
  }
}
```

### Full-Screen Viewport Mode

To make the viewport fill the entire page (no control panel):

```css
.left-panel {
  flex: 0 0 100%;
}
.right-panel {
  display: none;
}
```

Or overlay controls on top of the viewport:

```css
.container {
  position: absolute;
  width: 100%;
  height: 100%;
  /* No flex display */
}

.left-panel {
  position: absolute;
  width: 100%;
  height: 100%;
}

.right-panel {
  position: absolute;
  top: 20px;
  right: 20px;
  width: 300px;
  max-height: calc(100% - 40px);
  overflow: auto;
  background: white;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}
```

### Adding a Header/Footer

If you need a header or footer outside the configurator:

```html
<body>
  <header style="height: 60px; padding: 20px; background: #333; color: white;">
    My Configurator
  </header>

  <div class="container" style="height: calc(100vh - 120px); top: 60px;">
    <!-- Configurator panels here -->
  </div>

  <footer style="height: 60px; padding: 20px; background: #333; color: white; position: absolute; bottom: 0; width: 100%;">
    Footer content
  </footer>
</body>
```

Adjust container height to account for header/footer heights.

---

## Troubleshooting

### Symptom: Viewport is tiny (300x150px)

**Cause**: `#embed3DSource` doesn't have explicit dimensions

**Fix**: Add to CSS:
```css
#embed3DSource {
  width: 100%;
  height: 100%;
}
```

### Symptom: Double scrollbars appear

**Cause**: Both container and right panel have scrolling enabled

**Fix**: Ensure container has `overflow: hidden` and only right panel has `overflow: auto`:
```css
.container {
  overflow: hidden; /* No scrolling here */
}
.right-panel {
  overflow: auto; /* Scrolling only here */
}
```

### Symptom: Layout shifts when page loads

**Cause**: Container is `position: static` (default)

**Fix**: Use `position: absolute`:
```css
.container {
  position: absolute;
  width: 100%;
  height: 100%;
}
```

### Symptom: Panels aren't exactly 50/50

**Cause 1**: Using `flex: 1` or similar flex-grow values

**Fix**: Use `flex: 0 0 50%`:
```css
.left-panel,
.right-panel {
  flex: 0 0 50%; /* No grow/shrink */
}
```

**Cause 2**: Padding without `box-sizing: border-box`

**Fix**: Add universal box-sizing reset:
```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

### Symptom: Viewport scrolls out of view

**Cause**: Entire page scrolls instead of just the control panel

**Fix**: Add `overflow: hidden` to body and container, `overflow: auto` only to right panel:
```css
body {
  overflow: hidden;
}
.container {
  overflow: hidden;
}
.right-panel {
  overflow: auto;
}
```

### Symptom: White gap around the page

**Cause**: Default browser body margin

**Fix**: Reset body margin:
```css
body {
  margin: 0;
}
```

### Symptom: Viewport doesn't fill left panel height

**Cause**: Left panel has no explicit height, or embed div has no dimensions

**Fix**: Ensure both have 100% height:
```css
.left-panel {
  height: 100%;
}
#embed3DSource {
  width: 100%;
  height: 100%;
}
```

### Symptom: Viewport jitters/resizes as controls load

**Cause**: Panels using `flex: 1` or other flex-grow values

**Fix**: Use fixed flex sizing:
```css
.left-panel,
.right-panel {
  flex: 0 0 50%; /* Fixed at 50%, no resize */
}
```

---

## Why These Patterns Work Together

The seven patterns form a cohesive system:

1. **Box-sizing reset** ensures dimensions are predictable
2. **Body reset** removes default browser interference
3. **Absolute container** prevents layout shifts
4. **Fixed flex sizing** ensures panels never resize
5. **Full-height left panel** fills vertical space
6. **Independent scrolling** keeps viewport fixed
7. **Full-size embed** ensures iframe fills space

Each pattern addresses a specific issue, and together they create a stable, predictable layout that works consistently across browsers.

## Testing Your Implementation

To verify your CSS is correct:

1. **Open in browser** - Does the viewport fill the left half exactly?
2. **Resize window** - Does the layout stay stable?
3. **Add/remove controls** - Does the viewport stay the same size?
4. **Scroll controls** - Does only the right panel scroll?
5. **Check for double scrollbars** - Should only see one scrollbar (right panel)
6. **Inspect element** - Is `#embed3DSource` 50% of viewport width and 100% of viewport height?
7. **Check for white gaps** - Should be no margins around the page

If any test fails, review the pattern that addresses that issue.

## Additional Resources

- [Metabox Basic Configurator Template](../templates/basic-configurator/)
- [Metabox Modular Configurator Template](../templates/modular-configurator/)
- [MDN: CSS Flexible Box Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout)
- [MDN: box-sizing](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing)
