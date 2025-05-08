

## Summary

This guide walks you through building a simple Node.js CLI project that takes a Notion page URL, fetches its content via the Notion API, converts it to Markdown using the `notion-to-md` library, and saves it as an Obsidian-compatible `.md` file. You’ll create an internal Notion integration to obtain an access token, install and leverage the Notion SDK together with `notion-to-md`, post-process the output to include Obsidian frontmatter, and run the script locally.

## Prerequisites

* **Node.js & npm installed** on your machine.
* **A Notion account** and **Workspace Owner** access to create integrations ([Notion][1]).
* **Basic familiarity with JavaScript** and command-line operations.

## Step-by-Step Process

### 1. Create a Notion Internal Integration

1. Go to Notion’s Integrations page: [https://www.notion.com/my-integrations](https://www.notion.com/my-integrations).
2. Click **+ New integration**, give it a name, select your workspace, and set **Read content only** ([Notion][1]).
3. After creation, copy the **Internal Integration Token** (a bearer token) from the integration’s settings ([Notion][1]).

### 2. Initialize Your Node.js Project

```bash
mkdir notion-to-obsidian
cd notion-to-obsidian
npm init -y
npm install @notionhq/client notion-to-md
```

* This sets up a new project and installs the Notion SDK plus the `notion-to-md` converter ([GitHub][2]).

### 3. Extract the Notion Page ID

Notion page URLs embed a UUID after the last hyphen.

```js
const notionUrl = 'https://lunar-joke-35b.notion.site/Python-GIL-Global-Interpreter-Lock-a77f93b44f284160a5f163b8171b321d';
const pageId = notionUrl.split('-').pop().replace(/[^a-f0-9]/gi, '');
```

* This code isolates the 32-character page ID needed by the API ([Khaliulin.com][3]).

### 4. Write the Conversion Script (`index.js`)

```js
const { Client } = require('@notionhq/client');
const { NotionToMarkdown } = require('notion-to-md');
const fs = require('fs');

// 1. Initialize Notion client
const notion = new Client({ auth: process.env.NOTION_TOKEN });

// 2. Initialize converter
const n2m = new NotionToMarkdown({ notionClient: notion });

// 3. Main function
(async () => {
  const mdBlocks = await n2m.pageToMarkdown(pageId);
  let mdString = n2m.toMarkdownString(mdBlocks);
  
  // 4. Add Obsidian YAML frontmatter
  const frontmatter = `---
aliases: [Python GIL]
tags: [notion, obsidian]
---

`;
  mdString = frontmatter + mdString;

  // 5. Write to file
  fs.writeFileSync(`${pageId}.md`, mdString);
  console.log('Obsidian MD saved:', `${pageId}.md`);
})();
```

* Uses `pageToMarkdown` → `toMarkdownString` for nested blocks and code snippets ([GitHub][2]).
* Prepends a YAML frontmatter block for Obsidian compatibility.

### 5. Configure Environment & Run

1. In your shell, set the token:

   ```bash
   export NOTION_TOKEN="secret_XXXXXXXXXXXX"
   ```
2. Execute the script:

   ```bash
   node index.js
   ```

* This fetches the Notion page, converts, and writes `a77f93b44f284160a5f163b8171b321d.md` ([Khaliulin.com][3]).

## Next Steps & Enhancements

* **Handle Images & Attachments**: Download linked assets and rewrite image URLs to local paths inside Obsidian’s vault ([Khaliulin.com][3]).
* **Batch Processing**: Extend the script to accept multiple URLs or traverse a Notion database, exporting each row as a separate MD file ([npm][4]).
* **CLI Options**: Use packages like `commander` to parse arguments (e.g., `--output-dir`, `--tags`) for flexibility ([npm][4]).
* **Error Handling & Logging**: Add try/catch blocks and verbose logging to manage API rate limits and invalid page IDs.

With these steps, you now have a reproducible tool to bridge Notion content into your Obsidian workflow programmatically!

[1]: https://www.notion.com/help/create-integrations-with-the-notion-api?utm_source=chatgpt.com "Notion API integrations – Notion Help Center"
[2]: https://github.com/souvikinator/notion-to-md?utm_source=chatgpt.com "souvikinator/notion-to-md - GitHub"
[3]: https://khaliulin.com/posts/2022-11-12-convert-notion-to-markdown/?utm_source=chatgpt.com "Convert Notion Pages to Markdown | Khaliulin.com"
[4]: https://www.npmjs.com/package/notion-to-md/v/2.1.0?utm_source=chatgpt.com "notion-to-md - NPM"
