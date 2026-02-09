# GitHub Sync Test Course

This repository is a test course for the [moodle-local_githubsync](https://github.com/PSA-Corporate-Learning-Branch/moodle-local_githubsync) Moodle plugin. It demonstrates the expected repository structure and serves as a reference for course developers building content that syncs from GitHub into Moodle.

## How It Works

The GitHub Sync plugin lets you author Moodle course content as HTML files in a GitHub repository. The repository becomes the **single source of truth** — content is synced into Moodle automatically, and any direct edits made in Moodle will be overwritten on the next sync.

Syncing can happen four ways:

- **Manual sync** from the course's GitHub Sync settings page in Moodle
- **Scheduled task** (hourly cron) for courses with auto-sync enabled
- **Webhook** triggered instantly on every `git push`
- **CLI** for bulk or per-course syncing

The plugin uses content hashing to detect changes — only modified files are updated on each sync. Files removed from the repo are automatically hidden (not deleted) in the course.

## Repository Structure

```
course.yaml                              # Course-level metadata
assets/
  css/
    custom.css                           # Custom stylesheets
  images/
    placeholder.png                      # Image assets
  js/                                    # JavaScript files (if needed)
sections/
  01-introduction/
    section.yaml                         # Section metadata
    01-welcome.html                      # Page activity (default)
    02-course-overview.html              # Page activity
    03-notice.html                       # Label activity (via front matter)
  02-module-one/
    section.yaml
    01-lesson-one.html                   # Page activity
    03-summary.html                      # Page activity
    04-external-resource.html            # URL activity (via front matter)
```

### Naming Conventions

- **Sections** live inside `sections/` and use the pattern `NN-name/` (e.g., `01-introduction`, `02-module-one`)
- **Activities** within each section use the pattern `NN-name.html` (e.g., `01-welcome.html`, `02-course-overview.html`)
- Numeric prefixes control the **display order** and are stripped from the name shown in Moodle — `02-course-overview.html` becomes "Course Overview"
- Gaps in numbering are fine (e.g., `01`, `03`, `04` with no `02`)

## Developing a Course

### Step 1: Create the Course Configuration

Create a `course.yaml` in the repository root with course metadata:

```yaml
fullname: "My Course Title"
shortname: "MYCOURSE01"
summary: "A brief description of the course."
format: topics
visible: true
```

| Field       | Description                                        |
|-------------|----------------------------------------------------|
| `fullname`  | The full course name displayed in Moodle           |
| `shortname` | A unique short identifier for the course           |
| `summary`   | HTML or plain text course description              |
| `format`    | Moodle course format (`topics`, `weeks`, etc.)     |
| `visible`   | `true` or `false` — whether the course is visible  |

All fields are optional. If `course.yaml` is omitted, course metadata is not modified during sync.

### Step 2: Create Sections

Create numbered directories inside `sections/`. Each directory becomes a Moodle course section (topic).

```
sections/
  01-getting-started/
  02-core-concepts/
  03-advanced-topics/
```

Optionally add a `section.yaml` inside each section directory:

```yaml
title: "Getting Started"
summary: "<p>This section covers the basics you need to know.</p>"
visible: true
```

If no `section.yaml` is present, the section title is derived from the directory name (numeric prefix stripped, hyphens replaced with spaces, title-cased).

### Step 3: Create Activities

Add `.html` files inside each section directory. Each file becomes a Moodle activity.

#### Page Activities (default)

Any `.html` file without front matter (or with `type: page`) becomes a **Page** activity — a full content page students click into to read.

```html
<h2>Welcome to the Course</h2>
<p>This is your first lesson. Content supports standard HTML including Bootstrap classes.</p>
<div class="alert alert-info">
  <strong>Tip:</strong> You can use Bootstrap components in your content.
</div>
```

#### Label Activities

Labels display content directly on the course page without requiring a click-through. Add `type: label` in YAML front matter:

```html
---
type: label
---
<div class="alert alert-warning">
  <strong>Important:</strong> Complete the previous section before continuing.
</div>
```

#### URL Activities

URL activities link to an external resource. Add `type: url` and a `url` field in front matter:

```html
---
type: url
name: "Moodle Documentation"
url: "https://docs.moodle.org"
---
<p>Visit the official Moodle documentation for more information.</p>
```

### Front Matter Reference

Front matter is optional YAML at the top of any `.html` file, delimited by `---` markers:

| Field     | Description                                              | Default                     |
|-----------|----------------------------------------------------------|-----------------------------|
| `type`    | Activity type: `page`, `label`, or `url`                 | `page`                      |
| `name`    | Override the activity name                               | Derived from filename       |
| `url`     | External URL (required when `type: url`)                 | —                           |
| `visible` | Show or hide the activity (`true`/`false`)               | `true`                      |

### Step 4: Add Assets

Place static assets in the `assets/` directory. The directory structure is preserved when uploaded to Moodle's file storage.

```
assets/
  css/
    custom.css
  images/
    diagram.png
    photo.jpg
  js/
    interactive.js
```

#### Allowed File Types

| Category    | Extensions                                          |
|-------------|-----------------------------------------------------|
| Stylesheets | `.css`                                              |
| Scripts     | `.js`                                               |
| Images      | `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.ico`, `.bmp` |
| Fonts       | `.woff`, `.woff2`, `.ttf`, `.eot`, `.otf`           |
| Media       | `.pdf`, `.mp3`, `.mp4`, `.webm`, `.ogg`             |
| Data        | `.json`, `.txt`, `.csv`, `.xml`                     |

**Note:** `.svg` and `.php` files are blocked for security reasons.

#### Referencing Assets in Content

Reference assets using relative paths from the repository root. The plugin automatically rewrites these to Moodle `pluginfile.php` URLs during sync.

```html
<link rel="stylesheet" href="assets/css/custom.css">
<img src="assets/images/diagram.png" alt="Architecture diagram" width="400">
```

Both `assets/...` and `../assets/...` path formats are supported.

### Step 5: Connect to Moodle

1. In Moodle, navigate to the target course and open **GitHub Sync** in the course settings
2. Enter the **Repository URL** (e.g., `https://github.com/your-org/your-course-repo`)
3. Enter a **GitHub Personal Access Token** with `repo` scope (or `public_repo` for public repos)
4. Set the **Branch** (defaults to `main`)
5. Optionally enable **Auto-sync** for hourly automatic updates
6. Click **Sync from GitHub** to perform the initial sync

### Step 6: Set Up Webhooks (Optional)

For instant sync on every push:

1. In your GitHub repo, go to **Settings > Webhooks > Add webhook**
2. Set the Payload URL to `https://your-moodle-site.com/local/githubsync/webhook.php`
3. Set Content type to `application/json`
4. Enter the webhook secret (must match the Moodle admin setting)
5. Select **Just the push event**

## Content Authoring Tips

- Write **HTML fragments**, not full documents — no `<html>`, `<head>`, or `<body>` tags needed
- **Bootstrap 5** classes are available since Moodle uses Bootstrap (e.g., `alert`, `card`, `badge`)
- All HTML content is **sanitized** by Moodle's HTMLPurifier — `<script>` tags and event handlers will be stripped
- Use **semantic HTML** for accessibility (`<h2>`, `<h3>`, `<ul>`, `<table>`, etc.)
- Content removed from the repo is **hidden** in Moodle, not deleted — this preserves student activity data
- Keep YAML configuration simple — the plugin's parser supports basic `key: value` pairs, quoted strings, and booleans

## Example Workflow

```
1.  Clone the course repository
2.  Create or edit HTML files in sections/
3.  Add any images, CSS, or other assets to assets/
4.  Commit and push to the configured branch
5.  The plugin syncs changes into Moodle (via webhook, cron, or manual trigger)
6.  Review the course in Moodle to verify
```
