# Hosting Your Own Vido Plugin Registry

This guide provides complete, step-by-step instructions for creating and hosting your own Vido plugin registry. A custom registry lets you distribute plugins privately within a team, run a community registry, or test plugins locally during development.

---

## Table of Contents

1. [How Vido Registries Work](#how-vido-registries-work)
2. [Registry JSON Specification](#registry-json-specification)
3. [Option A: GitHub-Hosted Registry](#option-a-github-hosted-registry)
4. [Option B: Static Web Server Registry](#option-b-static-web-server-registry)
5. [Option C: Local File Registry (Development)](#option-c-local-file-registry-development)
6. [Adding Plugins to Your Registry](#adding-plugins-to-your-registry)
7. [Packaging a Plugin for Distribution](#packaging-a-plugin-for-distribution)
8. [Configuring Vido to Use Your Registry](#configuring-vido-to-use-your-registry)
9. [Registry Behavior Details](#registry-behavior-details)
10. [Troubleshooting](#troubleshooting)
11. [Complete Example](#complete-example)

---

## How Vido Registries Work

When a user opens the **Plugin Manager** in Vido, the app:

1. Reads the list of registry URLs from **Settings > Plugins > Custom Plugin Registry URLs**
2. For each URL, fetches `registry.json`:
   - **HTTP/HTTPS URLs**: Appends `/registry.json` to the URL if it doesn't already end with `.json`. For example, `https://my-registry.com/plugins` becomes `https://my-registry.com/plugins/registry.json`
   - **`file://` URLs**: Reads the file directly from the local path. The URL must point to the exact `registry.json` file. For example, `file:///C:/registries/my-registry/registry.json`
3. Parses the JSON into a list of available plugins
4. Displays them in the "Available" section of the Plugin Manager
5. Users can click **Install** to download and install any listed plugin

### Official vs. Custom Registries

- The **official Vido registry** (`https://plugins.vido.app/registry`) is always present and cannot be removed
- Plugins from the official registry display a **blue verified checkmark** next to the publisher name
- Plugins from custom registries do **not** display the verified badge — this is by design to distinguish officially reviewed plugins
- Users can add any number of custom registry URLs
- If the same plugin ID appears in multiple registries, the first registry wins (official registry is always first)

---

## Registry JSON Specification

A registry is a single `registry.json` file with this structure:

```json
{
  "name": "My Plugin Registry",
  "plugins": [
    {
      "id": "com.example.my-plugin",
      "displayName": "My Plugin",
      "description": "A short description of what the plugin does.",
      "author": "Author Name",
      "version": "1.0.0",
      "license": "MIT",
      "tags": ["video", "tools"],
      "downloadUrl": "https://example.com/releases/my-plugin-v1.0.0.zip",
      "iconUrl": "https://example.com/icons/my-plugin.png",
      "repository": "https://github.com/example/my-plugin",
      "lastUpdated": "2026-02-22"
    }
  ]
}
```

### Field Reference

#### Root Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Human-readable name for the registry. Displayed in Vido's Plugin Manager when showing where a plugin came from. |
| `plugins` | array | Yes | Array of plugin entry objects. Can be empty (`[]`). |

#### Plugin Entry Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | **Yes** | Unique plugin identifier. Must use reverse-domain naming (e.g. `com.author.plugin-name`). Must match the `id` field in the plugin's `plugin.json` manifest exactly. |
| `displayName` | string | **Yes** | User-facing name shown in the Plugin Manager. |
| `description` | string | **Yes** | Short description (1-2 sentences). Shown in search results and the plugin list. |
| `author` | string | **Yes** | Author or publisher name. |
| `version` | string | **Yes** | Current version available in this registry. Must use semantic versioning (e.g. `1.0.0`, `2.1.0-beta.1`). |
| `downloadUrl` | string | **Yes** | Direct HTTPS URL to the plugin `.zip` archive. Must be publicly downloadable without authentication. |
| `license` | string | No | SPDX license identifier (e.g. `MIT`, `Apache-2.0`, `GPL-3.0`). |
| `tags` | string[] | No | Array of lowercase tags for search and categorization. |
| `iconUrl` | string | No | URL to a square PNG icon (128×128 recommended). Displayed in the Plugin Manager. |
| `repository` | string | No | URL to the plugin's source code repository. |
| `lastUpdated` | string | No | ISO 8601 date of the last update (e.g. `2026-02-22`). |

### Validation Rules

- `id` must be unique within the registry
- `id` must contain only lowercase letters, numbers, dots, and hyphens
- `version` must follow semantic versioning (`MAJOR.MINOR.PATCH` with optional pre-release suffix)
- `downloadUrl` must be a valid URL — Vido will attempt to download the zip from this exact URL
- `plugins` array can be empty but must be present
- JSON must be valid (no trailing commas, no comments)

---

## Option A: GitHub-Hosted Registry

The simplest approach — host your `registry.json` in a GitHub repository and serve it via GitHub Pages or raw.githubusercontent.com.

### Step-by-Step

#### 1. Create a GitHub Repository

Create a new public repository (e.g. `my-vido-plugins`).

#### 2. Add `registry.json`

Create `registry.json` at the repository root:

```json
{
  "name": "My Team Plugin Registry",
  "plugins": []
}
```

Commit and push.

#### 3. Enable GitHub Pages (Recommended)

1. Go to your repository **Settings > Pages**
2. Under "Source", select **Deploy from a branch**
3. Select the **main** branch and **/ (root)** folder
4. Click **Save**
5. Wait for the deployment (usually < 1 minute)
6. Your registry URL will be: `https://your-username.github.io/my-vido-plugins`

> Vido will automatically append `/registry.json` to this URL, fetching `https://your-username.github.io/my-vido-plugins/registry.json`.

#### Alternative: Raw GitHub URL

If you don't want to enable GitHub Pages, use the raw content URL:

```
https://raw.githubusercontent.com/your-username/my-vido-plugins/main/registry.json
```

Since this URL already ends in `.json`, Vido will use it as-is.

#### 4. Configure Vido

In Vido, go to **Settings > Plugins > Custom Plugin Registry URLs** and add your URL:

- GitHub Pages: `https://your-username.github.io/my-vido-plugins`
- Raw URL: `https://raw.githubusercontent.com/your-username/my-vido-plugins/main/registry.json`

#### 5. Verify

Open the Plugin Manager in Vido. Your registry's plugins should appear in the "Available" section.

---

## Option B: Static Web Server Registry

Host `registry.json` on any static web server (Nginx, Apache, Caddy, S3, Azure Blob Storage, Netlify, Vercel, etc.).

### Step-by-Step

#### 1. Create the Registry File

Create a directory for your registry and add `registry.json`:

```json
{
  "name": "Acme Corp Internal Plugins",
  "plugins": []
}
```

#### 2. Deploy to Your Server

Place `registry.json` at a URL path of your choice. Examples:

| Hosting | Registry URL | Fetched URL |
|---------|-------------|-------------|
| Nginx | `https://plugins.acme.com` | `https://plugins.acme.com/registry.json` |
| S3 bucket | `https://my-bucket.s3.amazonaws.com/vido-plugins/registry.json` | Used as-is (already ends in `.json`) |
| Netlify | `https://vido-plugins.netlify.app` | `https://vido-plugins.netlify.app/registry.json` |

#### 3. Set CORS Headers (If Needed)

If your server requires CORS headers for cross-origin requests, add:

```
Access-Control-Allow-Origin: *
Content-Type: application/json
```

Most static hosting services (GitHub Pages, Netlify, S3 with public access) handle this automatically.

#### 4. Configure Vido

Add the registry URL in **Settings > Plugins > Custom Plugin Registry URLs**.

---

## Option C: Local File Registry (Development)

During plugin development, use a `file://` URL to serve plugins from a local directory. This avoids deploying to a server during the build-test cycle.

### Step-by-Step

#### 1. Create a Local Registry Directory

```powershell
mkdir C:\dev\my-vido-registry
```

#### 2. Create `registry.json`

Create `C:\dev\my-vido-registry\registry.json`:

```json
{
  "name": "Local Dev Registry",
  "plugins": [
    {
      "id": "com.myname.my-dev-plugin",
      "displayName": "My Dev Plugin",
      "description": "Plugin under development",
      "author": "Me",
      "version": "0.1.0",
      "downloadUrl": "file:///C:/dev/my-vido-registry/my-dev-plugin.zip"
    }
  ]
}
```

#### 3. Package Your Plugin as a Zip

After building your plugin:

```powershell
# Build the plugin
dotnet build -c Release

# Create the zip
$pluginDir = "C:\dev\my-plugin\bin\Release\net8.0"
$zipPath = "C:\dev\my-vido-registry\my-dev-plugin.zip"

# Copy plugin.json into the output if not already there
Copy-Item "C:\dev\my-plugin\plugin.json" $pluginDir

# Create the zip archive
Compress-Archive -Path "$pluginDir\*" -DestinationPath $zipPath -Force
```

#### 4. Configure Vido

Add the file URL in **Settings > Plugins > Custom Plugin Registry URLs**:

```
file:///C:/dev/my-vido-registry/registry.json
```

> **Important:** For `file://` URLs, you must specify the full path to `registry.json` — Vido does **not** append `/registry.json` for file URLs.

#### 5. Test the Install Flow

1. Open Vido's Plugin Manager
2. Your dev plugin should appear in the "Available" section
3. Click **Install** to test the full install flow from your local zip
4. When you update the plugin, increment the version in both `plugin.json` and `registry.json`, rebuild the zip, and click **Update** in the Plugin Manager

### Automating the Dev Loop

Create a PowerShell script to automate the build-package-test cycle:

```powershell
# build-and-package.ps1
param(
    [string]$PluginDir = ".",
    [string]$RegistryDir = "C:\dev\my-vido-registry",
    [string]$PluginId = "com.myname.my-dev-plugin"
)

# Build
dotnet build $PluginDir -c Release

# Gather output files
$outDir = Join-Path $PluginDir "bin\Release\net8.0"
$manifest = Join-Path $PluginDir "plugin.json"
$zipPath = Join-Path $RegistryDir "$PluginId.zip"

# Copy manifest into output
Copy-Item $manifest $outDir -Force

# Create zip
if (Test-Path $zipPath) { Remove-Item $zipPath }
Compress-Archive -Path "$outDir\*" -DestinationPath $zipPath

Write-Host "Plugin packaged to: $zipPath"
Write-Host "Refresh the Plugin Manager in Vido to pick up changes."
```

---

## Adding Plugins to Your Registry

### Step 1: Build and Zip the Plugin

The plugin zip must contain these files at the **root** level (not nested inside a subfolder):

```
my-plugin.zip
├── plugin.json          (required)
├── MyPlugin.dll         (required — the entry point DLL)
├── README.md            (optional)
└── Resources/           (optional)
    └── icon.png
```

To create the zip:

```powershell
# Assuming your plugin builds to bin/Release/net8.0/
$outDir = "C:\dev\my-plugin\bin\Release\net8.0"

# Copy plugin.json into the output directory
Copy-Item "C:\dev\my-plugin\plugin.json" $outDir

# Create the zip
Compress-Archive -Path "$outDir\*" -DestinationPath "my-plugin-v1.0.0.zip"
```

> **Important:** The zip contents must be at the root level. If Vido detects a single top-level folder wrapping everything, it will automatically unwrap it — but it's better practice to zip the files directly.

### Step 2: Host the Zip

Upload the zip to a permanent, publicly accessible URL. Recommended: GitHub Releases.

```
https://github.com/your-org/my-plugin/releases/download/v1.0.0/my-plugin-v1.0.0.zip
```

### Step 3: Add the Entry to `registry.json`

```json
{
  "id": "com.yourorg.my-plugin",
  "displayName": "My Plugin",
  "description": "Does something useful with videos.",
  "author": "Your Org",
  "version": "1.0.0",
  "license": "MIT",
  "tags": ["video", "tools"],
  "downloadUrl": "https://github.com/your-org/my-plugin/releases/download/v1.0.0/my-plugin-v1.0.0.zip",
  "iconUrl": "https://raw.githubusercontent.com/your-org/my-plugin/main/icon.png",
  "repository": "https://github.com/your-org/my-plugin",
  "lastUpdated": "2026-02-22"
}
```

### Step 4: Validate

1. Verify the JSON is syntactically valid: `python -m json.tool registry.json`
2. Verify `downloadUrl` is accessible (open it in a browser — it should start downloading)
3. Verify the `id` matches the plugin's `plugin.json` manifest `id` field exactly
4. Verify no duplicate `id` values exist in the `plugins` array

### Step 5: Commit and Push

```bash
git add registry.json
git commit -m "Add com.yourorg.my-plugin v1.0.0"
git push
```

If using GitHub Pages, the update will be live within ~1 minute.

---

## Packaging a Plugin for Distribution

This section covers the complete lifecycle of building a plugin and getting it into a registry.

### Plugin Project Structure

```
my-plugin/
├── MyPlugin.csproj
├── MyPlugin.cs           (implements IVidoPlugin)
├── plugin.json           (plugin manifest)
├── README.md             (displayed in Plugin Manager detail view)
└── Resources/
    └── icon.png          (24x24 sidebar icon, 16x16 file icons)
```

### `plugin.json` Manifest (Minimum)

```json
{
  "id": "com.yourname.my-plugin",
  "name": "my-plugin",
  "displayName": "My Plugin",
  "version": "1.0.0",
  "description": "What it does.",
  "author": "Your Name",
  "entryPoint": "MyPlugin.dll",
  "pluginClass": "MyPlugin.MyPlugin",
  "minVidoVersion": "0.1.0"
}
```

### Build Script

```powershell
# Clean and build
dotnet publish MyPlugin.csproj -c Release -o publish

# Copy manifest and resources
Copy-Item plugin.json publish/
Copy-Item README.md publish/ -ErrorAction SilentlyContinue
if (Test-Path Resources) { Copy-Item Resources publish/ -Recurse }

# Package
$version = "1.0.0"
Compress-Archive -Path "publish/*" -DestinationPath "my-plugin-v$version.zip" -Force
```

### Version Updates

When releasing a new version:

1. Update `version` in `plugin.json`
2. Build and create a new zip
3. Create a new GitHub release with the zip
4. Update `registry.json`:
   - Change `version` to the new version
   - Change `downloadUrl` to the new release URL
   - Change `lastUpdated` to today's date

---

## Configuring Vido to Use Your Registry

### Via the Settings UI

1. Open Vido
2. Go to **Settings** (gear icon in the Activity Bar, or press `Ctrl+,`)
3. Scroll to **Plugins** section
4. In **Custom Plugin Registry URLs**, enter your registry URL
5. The setting saves automatically

### URL Format Rules

| URL Type | Example | Notes |
|----------|---------|-------|
| HTTPS (base) | `https://my-registry.com/plugins` | Vido appends `/registry.json` automatically |
| HTTPS (direct) | `https://my-registry.com/plugins/registry.json` | Used as-is (already ends in `.json`) |
| Raw GitHub | `https://raw.githubusercontent.com/user/repo/main/registry.json` | Used as-is |
| GitHub Pages | `https://user.github.io/repo` | Vido appends `/registry.json` |
| Local file | `file:///C:/path/to/registry.json` | Must point to exact file. Use forward slashes. |

### Multiple Registries

You can add multiple registry URLs. Each URL goes on a separate line or as a comma-separated list (depending on the settings UI implementation). The official Vido registry is always present and always queried first.

**Plugin deduplication:** If the same plugin `id` appears in multiple registries, the first registry that lists it wins. Since the official registry is always first, official versions take precedence.

---

## Registry Behavior Details

### Fetch Timing

- Registries are fetched each time the user opens the Plugin Manager
- No background polling — data is fetched on demand only
- Failed fetches log a warning but don't block the UI

### Error Handling

- If a registry URL is unreachable, Vido logs a warning and skips it
- If `registry.json` contains invalid JSON, the entire registry is skipped
- Individual malformed plugin entries don't prevent other entries from loading
- The official registry is always queried regardless of errors in custom registries

### Caching

- No disk cache in the current version — each Plugin Manager open fetches fresh data
- Large registries should keep their JSON compact

### Install Process

When a user clicks **Install**:

1. Vido downloads the zip from `downloadUrl`
2. Extracts to `%APPDATA%/Vido/plugins/<plugin-id>/`
3. Validates the `plugin.json` manifest inside the zip
4. Loads and activates the plugin immediately
5. If activation fails, the plugin is marked as "Error" in the Plugin Manager

### Update Detection

- Vido compares the `version` field in the registry with the locally installed plugin's `plugin.json` version
- If the registry version is newer (semver comparison), an **Update** button appears
- Updating downloads the new zip, replaces the old files, and reactivates the plugin

---

## Troubleshooting

### Plugin doesn't appear in the Available section

1. **Check the URL**: Verify your registry URL is correct in Settings
2. **Check the JSON**: Validate `registry.json` is valid JSON (`python -m json.tool registry.json`)
3. **Check accessibility**: Open the full URL in a browser (e.g. `https://your-site.com/path/registry.json`) and verify the JSON loads
4. **Check the Output Log**: Open the bottom panel in Vido and look for warning messages about registry fetch failures
5. **Check `file://` paths**: For local registries, ensure you're using forward slashes and the path points to the exact `registry.json` file

### Plugin fails to install

1. **Check `downloadUrl`**: Open it in a browser — it should start downloading a `.zip` file immediately
2. **Check zip contents**: Extract the zip and verify `plugin.json` is at the root level (not inside a subfolder)
3. **Check `id` match**: The `id` in `registry.json` must exactly match the `id` in the plugin's `plugin.json`
4. **Check the Output Log**: Look for error messages about manifest validation or DLL loading failures

### CORS errors (web-hosted registries)

If your registry is on a custom web server and Vido can't fetch it, you may need to add CORS headers:

```
Access-Control-Allow-Origin: *
```

GitHub Pages, Netlify, and most static hosting services include this automatically.

### Local file registry not loading

- Ensure the URL uses three slashes: `file:///C:/path/to/registry.json` (not `file://C:/...`)
- Use forward slashes, not backslashes: `file:///C:/dev/registry.json` (not `file:///C:\dev\registry.json`)
- The path must point to `registry.json` directly — Vido does **not** append `/registry.json` for `file://` URLs

---

## Complete Example

Here's a full worked example of setting up a GitHub-hosted community registry with one plugin.

### 1. Create the Registry Repository

```bash
mkdir vido-community-plugins
cd vido-community-plugins
git init
```

### 2. Create `registry.json`

```json
{
  "name": "Community Vido Plugins",
  "plugins": [
    {
      "id": "io.github.janedoe.video-bookmarks",
      "displayName": "Video Bookmarks",
      "description": "Add named bookmarks to positions in a video for quick navigation.",
      "author": "Jane Doe",
      "version": "1.2.0",
      "license": "MIT",
      "tags": ["bookmarks", "navigation", "productivity"],
      "downloadUrl": "https://github.com/janedoe/vido-bookmarks/releases/download/v1.2.0/video-bookmarks-v1.2.0.zip",
      "iconUrl": "https://raw.githubusercontent.com/janedoe/vido-bookmarks/main/icon.png",
      "repository": "https://github.com/janedoe/vido-bookmarks",
      "lastUpdated": "2026-02-15"
    },
    {
      "id": "io.github.johndoe.playback-stats",
      "displayName": "Playback Stats",
      "description": "Track watch time, most-played videos, and playback statistics.",
      "author": "John Doe",
      "version": "0.5.0",
      "license": "Apache-2.0",
      "tags": ["statistics", "tracking"],
      "downloadUrl": "https://github.com/johndoe/vido-stats/releases/download/v0.5.0/playback-stats-v0.5.0.zip",
      "repository": "https://github.com/johndoe/vido-stats",
      "lastUpdated": "2026-01-30"
    }
  ]
}
```

### 3. Push to GitHub

```bash
git add .
git commit -m "Initial registry with 2 plugins"
git remote add origin https://github.com/your-org/vido-community-plugins.git
git push -u origin main
```

### 4. Enable GitHub Pages

Go to **Settings > Pages** → Deploy from **main** branch, root folder.

Your registry URL: `https://your-org.github.io/vido-community-plugins`

### 5. Add to Vido

In Vido Settings > Plugins > Custom Plugin Registry URLs, add:

```
https://your-org.github.io/vido-community-plugins
```

### 6. Verify

Open the Plugin Manager — you should see "Video Bookmarks" and "Playback Stats" in the Available section, listed as from "Community Vido Plugins" with no verified badge (since they're not from the official registry).
