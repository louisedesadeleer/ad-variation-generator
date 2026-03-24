# Ad Variation Generator

Generate ad headline variations from winning ads in Figma. Reads your product marketing context, identifies winning ad frames, and creates cloned variations with new headlines directly in your Figma file.

## When to use

- User says "create ad variations," "make variations of winning ads," "generate headline variants," "clone ads with new headlines," or "test new headlines on winning ads"

## What you need from the user

Ask for these three things before starting:

1. **Product marketing document** (required) — a doc, page, or file describing the product's positioning, value props, target audience, and key messaging. This is what the generated headlines will draw from.
2. **Figma file URL** (required) — the Figma file containing the winning ad frames to use as templates.
3. **Copy guidelines** (optional) — a Google Sheet, doc, or list with headline examples, character limits, or copy rules. If not provided, default to a **max 25 characters** per headline.

Also ask:
- **Number of variations** per winner (default: 5)
- Whether they've completed the one-time setup below (Figma plugin + Google Sheets CLI)

## One-time setup

Before running this skill for the first time, the user needs two things connected: the Figma plugin and (if using Google Sheets for copy guidelines) the Google Workspace CLI.

### Figma plugin: ClaudeTalkToFigma

This plugin lets Claude read and manipulate Figma files directly.

1. **Install the plugin** — open Figma, go to `Resources` (Shift+I) > `Plugins` tab > search for "ClaudeTalkToFigma" > click `Install`
2. **Run the plugin** — open the Figma file with your ads, then go to `Plugins` > `ClaudeTalkToFigma` > `Run`. The plugin panel will open showing a channel name (e.g., `abc123`).
3. **Share the channel** — give Claude the channel name so it can connect via `mcp__ClaudeTalkToFigma__join_channel`. The channel stays active as long as the plugin panel is open.
4. **Keep Figma open** — the plugin communicates in real time, so the file must stay open with the plugin running throughout the session.

If the plugin isn't found, it may need to be installed from the Figma Community page. Search "Claude Talk To Figma" on figma.com/community.

### Google Workspace CLI (only if using a Google Sheet)

This lets Claude read headline data from Google Sheets. Skip this if you're providing copy guidelines as plain text or a doc.

1. **Create a Google Cloud project** (if you don't have one):
   - Go to https://console.cloud.google.com/projectcreate
   - Name it anything (e.g., "Marketing Tools")
   - Click `Create`

2. **Enable the Google Sheets API**:
   - Go to https://console.cloud.google.com/apis/library/sheets.googleapis.com
   - Make sure your project is selected in the top dropdown
   - Click `Enable`

3. **Create OAuth credentials**:
   - Go to https://console.cloud.google.com/apis/credentials
   - Click `+ Create Credentials` > `OAuth client ID`
   - If prompted to configure the consent screen, select `External`, fill in the app name (anything), your email, and save
   - For application type, select `Desktop app`, name it anything, click `Create`
   - Download the JSON file — save it somewhere you'll remember

4. **Authenticate the CLI**:
   ```bash
   npx @googleworkspace/cli auth login --keys-file /path/to/your/downloaded-credentials.json
   ```
   This opens a browser window for Google OAuth. Sign in and grant access.

5. **Verify it works**:
   ```bash
   npx @googleworkspace/cli sheets spreadsheets get --params '{"spreadsheetId": "SOME_SHEET_ID"}'
   ```
   Replace `SOME_SHEET_ID` with any sheet ID from a URL (the part between `/d/` and `/edit`).

## Workflow

### Step 1: Read product marketing context

Read the product marketing document the user provided. Extract:
- Core value proposition and positioning
- Target audience and their pain points
- Key differentiators vs competitors
- Existing taglines, one-liners, or messaging frameworks
- Tone of voice (casual, professional, provocative, etc.)

This context drives all headline generation. Keep it loaded throughout the session.

### Step 2: Read copy guidelines (if provided)

If the user provided a Google Sheet or doc with copy guidelines:
- Extract the spreadsheet ID from the URL (the part between `/d/` and `/edit`)
- Fetch all data with an open-ended range:
  ```
  npx @googleworkspace/cli sheets spreadsheets values get --params '{"spreadsheetId": "<ID>", "range": "A:Z"}'
  ```
- Identify: headline examples, character limits per format, angle categories, any naming conventions

If no copy guidelines were provided, use these defaults:
- Max headline length: **25 characters**
- No specific angle constraints — derive angles from the product marketing doc

### Step 3: Connect to Figma & analyze winners

1. Join the Figma channel via `mcp__ClaudeTalkToFigma__join_channel`
2. Get document info via `mcp__ClaudeTalkToFigma__get_document_info` to find all winner frames
3. Get detailed node info for each winner via `mcp__ClaudeTalkToFigma__get_nodes_info`

For each winning ad frame, identify:
- **The headline text node** (largest font size, typically the main message)
- **The angle/style** of the headline (analogy, contrast, competitive, pain point, benefit, etc.)
- **The CTA text** and other supporting text nodes
- **Frame dimensions and position** for layout planning

### Step 4: Generate headline variations

For each winner, create headline variations that:
- **Match the original angle/style** — if the winner uses an analogy ("Like X for Y"), keep variations in analogy format. If it uses contrast ("X, not Y"), keep that structure.
- **Draw from the product marketing doc** — use the positioning, pain points, and differentiators as raw material
- **Respect character limits** — from the copy guidelines, or 25 chars max by default
- **Mix headline formulas:**
  - Analogy: "Like [known tool], but for [use case]"
  - Contrast: "[Do this], not [that]"
  - Competitive: "[Competitor], but [advantage]"
  - Command: "[Action verb] + [benefit]"
  - Provocation: "Why [common behavior] when you can [better way]?"

### Step 5: Clone and populate in Figma

1. **Clone each winner frame** N times using `mcp__ClaudeTalkToFigma__clone_node`, positioning them in a row below the originals:
   - Row spacing: original frame height + 120px gap
   - Column spacing: frame width + 40px gap
   - Each winner's variations get their own row

2. **Find headline text nodes in every clone** using `mcp__ClaudeTalkToFigma__scan_text_nodes` on each clone individually. Figma node IDs are globally incremented, not per-frame, so you cannot predict IDs from one clone to another. Always scan each clone for its own text nodes.

3. **Update headline text** in each clone via `mcp__ClaudeTalkToFigma__set_text_content`

4. **Rename each clone** via `mcp__ClaudeTalkToFigma__rename_node` using format: `W[winner#] — V[variation#]`

### Step 6: Summarize

Present a clean summary table:
- Winner name → original headline → angle style
- Each variation with its new headline and character count
- Note the layout in Figma (rows below originals)

## Tips for great variations

- **Don't just rephrase** — shift the frame. "Send videos, not emails" → "One video beats ten emails" (same angle, different frame)
- **Short > long** — the best ad headlines are scannable in under 2 seconds
- **Mix emotional and rational** — some variations should hit feelings ("Stop dreading edit day"), others hit logic ("3 min vs 3 hours")
- **Test specificity** — some variations should be specific ("Save 3hrs per video"), others broad ("Make videos effortlessly")

## Parallel execution tips

- Clone all frames for one winner in parallel (batch of 5 clone calls)
- Scan text nodes for all clones in parallel (one `scan_text_nodes` call per clone)
- Set text + rename in parallel for each clone (2 calls per clone)
