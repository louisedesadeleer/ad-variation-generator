# Ad Variation Generator

When the user wants to generate ad variations from winning ads in Figma, use this skill. It reads headline data from a Google Sheet, identifies winning ad frames in Figma, and creates cloned variations with new headlines.

## When to use

- User says "create ad variations," "make variations of winning ads," "generate headline variants," "clone ads with new headlines," or "test new headlines on winning ads"
- User has a Figma file with winning ad frames and a Google Sheet with headline data

## What you need from the user

1. **Google Sheet URL** — containing ad headline data (angles, ad names, headlines, char counts)
2. **Figma file URL** — with winning ad frames to use as templates
3. **Number of variations** — per winner (default: 5)
4. **Figma channel** — the ClaudeTalkToFigma channel to join (ask if not already connected)

## Workflow

### Step 1: Connect & gather data

1. Join the Figma channel via `mcp__ClaudeTalkToFigma__join_channel` (if not already connected)
2. Read the Google Sheet using the Google Workspace CLI:
   - **First-time setup:** The user must authenticate before the CLI will work. Run `npx @googleworkspace/cli auth login` and follow the browser OAuth flow. If it fails, check that the user has a Google Cloud project with the Sheets API enabled and valid OAuth credentials configured.
   - Extract the spreadsheet ID from the URL (between `/d/` and `/edit`)
   - Fetch all data — use an open-ended range to avoid truncating rows:
     ```
     npx @googleworkspace/cli sheets spreadsheets values get --params '{"spreadsheetId": "<ID>", "range": "A:Z"}'
     ```
3. Get the Figma document info via `mcp__ClaudeTalkToFigma__get_document_info` to find all winner frames
4. Get detailed node info for each winner via `mcp__ClaudeTalkToFigma__get_nodes_info` to understand structure (headline text nodes, CTAs, layout)

### Step 2: Analyze the winners

For each winning ad frame, identify:
- **The headline text node** (largest font size, typically the main message)
- **The angle/style** of the headline (analogy, contrast, competitive, pain point, benefit, etc.)
- **The CTA text** and other supporting text nodes
- **Frame dimensions and position** for layout planning

### Step 3: Generate headline variations

For each winner, create headline variations that:
- **Match the original angle/style** — if the winner uses an analogy ("Like X for Y"), keep variations in analogy format. If it uses contrast ("X, not Y"), keep that structure.
- **Draw from the sheet data** — use the angles, themes, and language from the Google Sheet headlines as inspiration
- **Stay within character limits** — keep headlines punchy (under 35 chars for short headlines, under 50 for two-liners)
- **Align with Tella's positioning** — reference the CLAUDE.md for Tella one-liners, competitor angles, and customer pains
- **Mix headline formulas:**
  - Analogy: "Like [known tool], but for [use case]"
  - Contrast: "[Do this], not [that]"
  - Competitive: "[Competitor], but [advantage]"
  - Command: "[Action verb] + [benefit]"
  - Provocation: "Why [common behavior] when you can [better way]?"

### Step 4: Clone and populate in Figma

1. **Clone each winner frame** N times using `mcp__ClaudeTalkToFigma__clone_node`, positioning them in a row below the originals:
   - Row spacing: original frame height + 120px gap
   - Column spacing: frame width + 40px gap
   - Each winner's variations get their own row

2. **Find headline text nodes in every clone** using `mcp__ClaudeTalkToFigma__scan_text_nodes` on each clone individually. Figma node IDs are globally incremented, not per-frame, so you cannot predict IDs from one clone to another. Always scan each clone for its own text nodes.

3. **Update headline text** in each clone via `mcp__ClaudeTalkToFigma__set_text_content`

4. **Rename each clone** via `mcp__ClaudeTalkToFigma__rename_node` using format: `W[winner#] — V[variation#]`

### Step 5: Summarize

Present a clean summary table:
- Winner name -> original headline -> angle style
- Each variation with its new headline
- Note the layout in Figma (rows below originals)

## Tips for great variations

- **Don't just rephrase** — shift the frame. "Send videos, not emails" -> "One video beats ten emails" (same angle, different frame)
- **Use Loom bashing** for competitive angles — lean into the rivalry per Tella's positioning
- **Short > long** — the best ad headlines are scannable in under 2 seconds
- **Mix emotional and rational** — some variations should hit feelings ("Stop dreading edit day"), others hit logic ("3 min vs 3 hours")
- **Test specificity** — some variations should be specific ("Save 3hrs per video"), others broad ("Make videos effortlessly")

## Parallel execution tips

- Clone all frames for one winner in parallel (batch of 5 clone calls)
- Scan text nodes for all clones in parallel (one `scan_text_nodes` call per clone)
- Set text + rename in parallel for each clone (2 calls per clone)
