# Contributing to the Vido Official Plugin Registry

Thank you for your interest in publishing a plugin to the official Vido registry! This guide walks you through the submission process.

## Prerequisites

Before submitting, your plugin must:

1. **Build and run** against the current stable version of Vido
2. **Include a valid `plugin.json` manifest** with all required fields
3. **Be packaged as a `.zip` archive** hosted at a publicly accessible HTTPS URL (GitHub Releases recommended)
4. **Not contain malware, adware, or intentionally harmful code**

## Submission Process

### 1. Prepare Your Plugin

Ensure your plugin zip contains:
- `plugin.json` (valid manifest)
- Your plugin DLL(s)
- Any resource files (icons, etc.)
- Optional: `README.md`

### 2. Host Your Plugin Zip

Create a GitHub release (or other permanent HTTPS host) with your `.zip` file. The URL must be:
- Publicly accessible (no authentication required)
- A direct download link (not a page that requires clicking "Download")
- Permanent (release assets, not branch artifacts that expire)

**GitHub Release Example:**
```
https://github.com/your-username/your-plugin/releases/download/v1.0.0/your-plugin-v1.0.0.zip
```

### 3. Fork This Repository

1. Fork `vido-plugin-registry` on GitHub
2. Clone your fork locally

### 4. Add Your Plugin Entry

Edit `registry.json` and add your plugin to the `plugins` array:

```json
{
  "name": "Vido Official Plugin Registry",
  "plugins": [
    {
      "id": "com.yourname.your-plugin",
      "displayName": "Your Plugin Name",
      "description": "A short description of what your plugin does.",
      "author": "Your Name",
      "version": "1.0.0",
      "license": "MIT",
      "tags": ["category1", "tag2"],
      "downloadUrl": "https://github.com/your-username/your-plugin/releases/download/v1.0.0/your-plugin-v1.0.0.zip",
      "iconUrl": "https://raw.githubusercontent.com/your-username/your-plugin/main/icon.png",
      "repository": "https://github.com/your-username/your-plugin",
      "lastUpdated": "2026-02-22"
    }
  ]
}
```

**Required fields:** `id`, `displayName`, `description`, `author`, `version`, `downloadUrl`

**Optional fields:** `license`, `tags`, `iconUrl`, `repository`, `lastUpdated`

### 5. Validate Your Entry

Before submitting, verify:
- [ ] `id` uses reverse-domain naming (e.g. `com.author.plugin-name`) and matches your `plugin.json`
- [ ] `version` follows semantic versioning (e.g. `1.0.0`)
- [ ] `downloadUrl` is a direct HTTPS link to a `.zip` file
- [ ] `downloadUrl` is publicly accessible (test in an incognito browser window)
- [ ] The JSON is valid (no trailing commas, proper quoting)
- [ ] Your entry doesn't duplicate an existing plugin `id`

### 6. Submit a Pull Request

1. Commit your change to your fork
2. Open a Pull Request against the main `vido-plugin-registry` repository
3. In the PR description, include:
   - A link to your plugin's source repository
   - A brief description of what the plugin does
   - Confirmation that you've tested it with the current Vido version

### 7. Review

A maintainer will review your submission. We check:
- Manifest validity
- Download URL accessibility
- Basic functionality (we'll install and test the plugin)
- No malicious behavior

## Updating Your Plugin

To publish a new version:

1. Build your updated plugin
2. Create a new GitHub release with the updated `.zip`
3. Fork the registry again (or update your existing fork)
4. Update your entry in `registry.json`:
   - Change `version` to the new version
   - Update `downloadUrl` to point to the new release
   - Update `lastUpdated` to today's date
5. Submit a new Pull Request

## Plugin ID Guidelines

- Use reverse-domain style: `com.yourname.plugin-name` or `io.github.username.plugin-name`
- Use lowercase letters, numbers, dots, and hyphens only
- The ID must be globally unique and must match the `id` in your plugin's `plugin.json`
- Once published, your plugin ID cannot be changed

## Questions?

Open an issue in this repository if you have questions about the submission process.
