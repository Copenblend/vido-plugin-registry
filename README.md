# Vido Official Plugin Registry

The official plugin registry for [Vido](https://github.com/your-org/vido), the VS Code-styled video player for Windows.

## What Is This?

This repository hosts the `registry.json` file that Vido reads to populate the **Plugin Manager** with available plugins. When users browse or search for plugins in Vido, the app fetches this file to discover what's available for installation.

Plugins listed in this registry display a **verified badge** (blue checkmark) next to the publisher name in Vido's Plugin Manager, indicating they come from the official source.

## For Plugin Developers

Want to publish your plugin to the official registry? See [CONTRIBUTING.md](CONTRIBUTING.md) for the submission process.

Want to host your own private or community registry instead? See [HOSTING_YOUR_OWN_REGISTRY.md](HOSTING_YOUR_OWN_REGISTRY.md) for complete instructions.

## Registry URL

Vido fetches the registry from:

```
https://plugins.vido.app/registry/registry.json
```

This URL is built into Vido as the official registry endpoint. Plugins from this URL receive the verified badge automatically.

## Schema

The registry follows the schema defined in [registry.schema.json](registry.schema.json). See [HOSTING_YOUR_OWN_REGISTRY.md](HOSTING_YOUR_OWN_REGISTRY.md) for the full specification.

## License

This registry metadata is released under the [MIT License](LICENSE).
