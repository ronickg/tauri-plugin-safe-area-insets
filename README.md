# tauri-plugin-safe-area-insets

[![Crates.io](https://img.shields.io/crates/v/tauri-plugin-safe-area-insets.svg)](https://crates.io/crates/tauri-plugin-safe-area-insets)
[![npm](https://img.shields.io/npm/v/tauri-plugin-safe-area-insets.svg)](https://www.npmjs.com/package/tauri-plugin-safe-area-insets)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue.svg)](https://github.com/ahmet-cetinkaya/tauri-plugin-safe-area-insets)

A Tauri v2 plugin for accessing safe area insets on Android and iOS mobile platforms. This plugin provides system UI spacing information (status bar, navigation bar, notches, etc.) to help you design responsive mobile layouts.

## Platform Support

- **Android**: ✅ Fully supported
- **iOS**: ⚠️ Placeholder implementation (returns zeros)
- **Desktop**: ❌ Not supported (mobile-only plugin)

## Installation

### Rust

Add the plugin to your `Cargo.toml`:

```toml
[target.'cfg(any(target_os = "android", target_os = "ios"))'.dependencies]
tauri-plugin-safe-area-insets = "0.1"
```

### JavaScript

Install the JavaScript guest bindings:

```bash
npm add tauri-plugin-safe-area-insets
# or
pnpm add tauri-plugin-safe-area-insets
# or
yarn add tauri-plugin-safe-area-insets
# or
bun add tauri-plugin-safe-area-insets
```

## Usage

### Step 1: Configure Permissions

Create or update `src-tauri/capabilities/mobile.json`:

```json
{
  "$schema": "../gen/schemas/mobile-schema.json",
  "identifier": "mobile-capability",
  "description": "Capability for mobile platforms",
  "windows": ["main"],
  "platforms": ["iOS", "android"],
  "permissions": ["core:default", "safe-area-insets:default"]
}
```

### Step 2: Register Plugin in Rust

Register the plugin in your Tauri application (`src-tauri/src/lib.rs`):

```rust
#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .setup(move |app| {
            #[cfg(mobile)]
            {
                app.handle().plugin(tauri_plugin_safe_area_insets::init())?;
            }
            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Step 3: Apply Safe Area Insets in JavaScript

**Important**: Call `getInsets()` after the Tauri IPC is ready (e.g., after render or in a `setTimeout`).

#### Using CSS Variables (Recommended)

```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      .safe-area-inset {
        padding-top: var(--safe-area-inset-top, 0px);
        padding-bottom: var(--safe-area-inset-bottom, 0px);
        padding-left: var(--safe-area-inset-left, 0px);
        padding-right: var(--safe-area-inset-right, 0px);
      }
    </style>
  </head>
  <body>
    <div class="safe-area-inset">
      <h1>Your Content</h1>
    </div>

    <script type="module">
      import { getInsets } from "tauri-plugin-safe-area-insets";

      async function setupSafeAreaInsets() {
        const insets = await getInsets().catch(error => {
          console.warn("Failed to get safe area insets:", error);
          // Return fallback values on error
          return { top: 0, bottom: 0, left: 0, right: 0 };
        });

        document.documentElement.style.setProperty("--safe-area-inset-top", `${insets.top}px`);
        document.documentElement.style.setProperty("--safe-area-inset-bottom", `${insets.bottom}px`);
        document.documentElement.style.setProperty("--safe-area-inset-left", `${insets.left}px`);
        document.documentElement.style.setProperty("--safe-area-inset-right", `${insets.right}px`);
      }

      // Call after DOM is ready
      window.addEventListener("DOMContentLoaded", () => {
        setTimeout(setupSafeAreaInsets, 0);
      });
    </script>
  </body>
</html>
```

<details>
<summary><b>Framework Examples (React, Vue, Svelte, SolidJS, etc.)</b></summary>

The same pattern works with any UI framework. Use their lifecycle hooks to call `setupSafeAreaInsets()`:

**React:**

```tsx
import { useEffect } from "react";

function App() {
  useEffect(() => {
    setupSafeAreaInsets();
  }, []);

  return <div className="safe-area-inset">{/* Your content */}</div>;
}
```

**Vue:**

```vue
<script setup>
import { onMounted } from "vue";

onMounted(() => {
  setupSafeAreaInsets();
});
</script>

<template>
  <div class="safe-area-inset"><!-- Your content --></div>
</template>
```

**Svelte:**

```svelte
<script>
  import { onMount } from 'svelte';

  onMount(() => {
    setupSafeAreaInsets();
  });
</script>

<div class="safe-area-inset">
  <h1>Your Content</h1>
</div>
```

**SolidJS:**

```tsx
import { onMount } from "solid-js";

function App() {
  onMount(() => {
    setupSafeAreaInsets();
  });

  return <div class="safe-area-inset">{/* Your content */}</div>;
}
```

> Works with Angular, Preact, Qwik, Lit, Alpine.js, and other frameworks too!

</details>

#### Direct Style Application

```javascript
import { getInsets } from "tauri-plugin-safe-area-insets";

async function applySafeArea() {
  const insets = await getInsets().catch(error => {
    console.error("Failed to get insets:", error);
    return { top: 0, bottom: 0, left: 0, right: 0 };
  });

  console.log(insets); // e.g. { top: 47, bottom: 34, left: 0, right: 0 }

  // Apply insets to your UI
  document.body.style.paddingTop = `${insets.top}px`;
  document.body.style.paddingBottom = `${insets.bottom}px`;
  document.body.style.paddingLeft = `${insets.left}px`;
  document.body.style.paddingRight = `${insets.right}px`;
}

// Call after DOM is ready
window.addEventListener("DOMContentLoaded", () => {
  setTimeout(applySafeArea, 0);
});
```

## API Reference

### `getInsets()`

Returns the current safe area insets.

**Returns:** `Promise<Insets>`

```typescript
interface Insets {
  top: number; // Top inset in density-independent pixels (DIP)
  bottom: number; // Bottom inset in DIP
  left: number; // Left inset in DIP
  right: number; // Right inset in DIP
}
```

**Platform Behavior:**

- **Android**: Returns actual system bar insets converted to DIP
- **iOS**: Currently returns all zeros (placeholder)

## Development

### Prerequisites

- Rust 1.77.2+
- pnpm 9.11.0+
- Node.js
- Android SDK (for Android development)
- Xcode (for iOS development)

### Building

```bash
# Build JavaScript API
pnpm install
pnpm build

# Build Rust plugin
cargo build
```

### Testing

```bash
# Rust tests
cargo test

# Android unit tests
cd android
./gradlew test

# Android instrumented tests (requires device/emulator)
./gradlew connectedAndroidTest

# iOS tests
cd ios
swift test
```

### Running the Example

```bash
cd examples/tauri-app
pnpm install

# Run on Android
pnpm tauri dev --target android

# Run on iOS
pnpm tauri dev --target ios
```

## How It Works

The plugin bridges native platform APIs to JavaScript:

1. **Android**: Uses `WindowInsetsCompat` to query system bar insets, converting pixel values to device-independent pixels
2. **iOS**: Placeholder implementation that will query `UIViewController.view.safeAreaInsets` in future versions
3. **Rust**: Manages plugin lifecycle and routes commands to platform-specific implementations
4. **TypeScript**: Provides type-safe JavaScript API via Tauri's invoke system

## License

MIT OR Apache-2.0

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Acknowledgments

- Original plugin by [Ronald](https://github.com/ronald-dev)
- Documentation improvements by [Ahmet Çetinkaya](https://github.com/ahmet-cetinkaya)
