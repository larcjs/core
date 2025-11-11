# LARC Core

[![Version](https://img.shields.io/npm/v/@larcjs/core.svg)](https://www.npmjs.com/package/@larcjs/core)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-production--ready-brightgreen.svg)](CHANGELOG.md)

> **Lightweight Asynchronous Relay Core** — The PAN (Page Area Network) messaging bus implementation

LARC Core provides the foundational messaging infrastructure for building loosely-coupled, event-driven web applications. It implements the PAN (Page Area Network) protocol, enabling seamless communication between components, iframes, workers, and tabs.

## Features

- 🚀 **Zero build required** — Drop-in `<pan-bus>` element, communicate via CustomEvents
- 🔌 **Loose coupling** — Components depend on topic contracts, not imports
- 🌐 **Universal** — Works with vanilla JS, Web Components, React, Lit, Vue, iframes
- 📬 **Rich messaging** — Pub/sub, request/reply, retained messages, cross-tab mirroring
- 🎯 **Lightweight** — ~12KB minified, no dependencies
- ⚡ **Performance** — 300k+ messages/second, zero memory leaks
- 🔒 **Security** — Built-in message validation and sanitization

## Quick Start

### Installation

```bash
npm install @larcjs/core
```

### CDN Usage (No Build Required)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <!-- Load the autoloader -->
  <script type="module" src="https://unpkg.com/@larcjs/core/src/pan.js"></script>
</head>
<body>
  <!-- The pan-bus is automatically created -->
  <script>
    // Publish a message
    document.dispatchEvent(new CustomEvent('pan:publish', {
      detail: {
        topic: 'greeting.message',
        payload: { text: 'Hello, PAN!' }
      }
    }));

    // Subscribe to messages
    document.addEventListener('pan:message', (e) => {
      if (e.detail.topic === 'greeting.message') {
        console.log('Received:', e.detail.payload.text);
      }
    });
  </script>
</body>
</html>
```

### Module Usage

```javascript
import { PanBus } from '@larcjs/core';

// Create a bus instance
const bus = new PanBus();

// Subscribe to a topic
bus.subscribe('user.login', (message) => {
  console.log('User logged in:', message.payload);
});

// Publish a message
bus.publish('user.login', { userId: 123, name: 'Alice' });

// Request/reply pattern
const response = await bus.request('user.get', { id: 123 });
console.log('User data:', response);
```

## Core Components

### `<pan-bus>`

The central message hub that routes all PAN messages.

```html
<pan-bus id="myBus" mirror="false"></pan-bus>
```

**Attributes:**
- `mirror` — Enable cross-tab message mirroring (default: false)
- `debug` — Enable debug logging (default: false)

### `<pan-client>`

Simplifies publishing and subscribing for components.

```html
<pan-client id="client"></pan-client>

<script>
  const client = document.getElementById('client');

  client.subscribe('data.changed', (msg) => {
    console.log('Data updated:', msg.payload);
  });

  client.publish('data.request', { id: 42 });
</script>
```

## Message Patterns

### Publish/Subscribe

```javascript
// Publisher
bus.publish('notifications.new', {
  type: 'info',
  message: 'Welcome!'
});

// Subscriber
bus.subscribe('notifications.new', (msg) => {
  showNotification(msg.payload);
});
```

### Request/Reply

```javascript
// Responder
bus.subscribe('user.get', async (msg) => {
  const user = await fetchUser(msg.payload.id);
  return { ok: true, user };
});

// Requester
const result = await bus.request('user.get', { id: 123 });
if (result.ok) {
  console.log('User:', result.user);
}
```

### Retained Messages (State)

```javascript
// Publish with retain flag
bus.publish('app.state', { theme: 'dark' }, { retain: true });

// Late subscribers immediately receive the retained message
bus.subscribe('app.state', (msg) => {
  applyTheme(msg.payload.theme);
});
```

## Topic Conventions

LARC uses hierarchical topic naming:

- `${resource}.list.get` — Request list of items
- `${resource}.list.state` — Current list state (retained)
- `${resource}.item.select` — User selected an item
- `${resource}.item.get` — Request single item
- `${resource}.item.save` — Save an item
- `${resource}.item.delete` — Delete an item
- `${resource}.changed` — Item(s) changed notification
- `${resource}.error` — Error occurred

Example:

```javascript
// Request list of products
await bus.request('products.list.get', {});

// Subscribe to product selection
bus.subscribe('products.item.select', (msg) => {
  loadProductDetails(msg.payload.id);
});

// Save a product
await bus.request('products.item.save', {
  item: { id: 1, name: 'Widget', price: 9.99 }
});
```

## Cross-Tab Communication

Enable the `mirror` attribute to sync messages across browser tabs:

```html
<pan-bus mirror="true"></pan-bus>
```

Only non-sensitive topics should be mirrored. Use topic filters:

```javascript
bus.setMirrorFilter((topic) => {
  // Don't mirror authentication tokens
  return !topic.startsWith('auth.');
});
```

## TypeScript Support

Full TypeScript definitions are included:

```typescript
import type { PanMessage, PanBus } from '@larcjs/core';

interface UserPayload {
  id: number;
  name: string;
}

const bus = new PanBus();

bus.subscribe<UserPayload>('user.login', (msg) => {
  console.log(msg.payload.name); // Type-safe!
});
```

## Architecture

```
┌─────────────────────────────────────────────────┐
│                   Application                    │
├─────────────┬─────────────┬─────────────────────┤
│  Component  │  Component  │     Component       │
│      A      │      B      │         C           │
└──────┬──────┴──────┬──────┴──────┬──────────────┘
       │             │             │
       └─────────────┼─────────────┘
                     │
              ┌──────▼──────┐
              │  <pan-bus>  │  ← Central Message Hub
              └──────┬──────┘
                     │
       ┌─────────────┼─────────────┐
       │             │             │
┌──────▼──────┐ ┌───▼────┐ ┌──────▼──────┐
│   Worker    │ │ iframe │ │ Other Tabs  │
└─────────────┘ └────────┘ └─────────────┘
```

## Related Packages

- **[@larcjs/components](https://github.com/larcjs/larc-components)** — UI components built on LARC Core
- **[@larcjs/devtools](https://github.com/larcjs/larc-devtools)** — Chrome DevTools for debugging PAN messages
- **[@larcjs/examples](https://github.com/larcjs/larc-examples)** — Demo applications and examples

## Documentation

- [API Reference](docs/API_REFERENCE.md)
- [LARC Specification](docs/LARC_SPEC.v0.md)
- [Roadmap](docs/LARC_ROADMAP.md)
- [Full Documentation](https://larcjs.github.io/larc-site/)

## Browser Support

- ✅ Chrome/Edge 90+
- ✅ Firefox 88+
- ✅ Safari 14+
- ✅ Opera 76+

## Performance

- **Throughput:** 300,000+ messages/second
- **Latency:** <1ms per message (local)
- **Memory:** Zero leaks, constant memory usage
- **Bundle size:** ~12KB minified

## Contributing

Contributions are welcome! Please see our [Contributing Guide](CONTRIBUTING.md).

## License

MIT © Chris Robison

## Support

- 📖 [Documentation](https://larcjs.github.io/larc-site/)
- 💬 [Discussions](https://github.com/larcjs/larc-core/discussions)
- 🐛 [Issue Tracker](https://github.com/larcjs/larc-core/issues)
