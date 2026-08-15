---
name: document-browser
description: |-
  Create or update a professional live-linked document browser dashboard for a SharePoint document library, using real metadata fields, Knowledge Explorer header styling, visible refresh, reliable SharePoint Open/Download links, and a 3-card desktop layout.

  Use when the user says:
    - "create a document browser"
    - "make a live dashboard for this document library"
    - "build a Knowledge Management Database dashboard"
    - "use the document browser skill on this library"
    - "make sure the dashboard is live linked"
---
# Document Browser

## When to use
Use this skill when the user wants a professional document browser dashboard for a SharePoint document library or folder. The dashboard must browse documents using real metadata, search, filters, cards, read-only Open links, Download links, and live-linked data whenever supported.

## Inputs
- Target SharePoint document library URL, list ID, or selected/current library.
- Optional folder path inside the library.
- Optional approval rule, default: show approved documents only when content approval is enabled.
- Optional dashboard file name and destination folder.
- Optional requested filters.
- Optional reference dashboard for styling. If none is supplied, use the Knowledge Explorer header/design pattern.

## Steps
1. **Resolve and inspect**
   - Resolve the target library ID and web URL.
   - Retrieve the full schema and use internal field names only.
   - Identify approval status, Document Title, name/path fields, description/abstract, metadata filters, Document Owner, contacts, author/editor, created/modified dates.

2. **Title and owner rules**
   - Prefer **Document Title** for card titles. Match `Document Title`, `documentTitle`, and `Document_x0020_Title`.
   - Fall back to `Title`, then file name.
   - Show **Document Owner** name directly below the card title. Match `documentOwner` / `Document Owner`; fall back to Author only if no owner exists.

3. **Live-linked build**
   - Use LiveData/list binding for supported library roots or list sources.
   - Include `ka-livedata-manifest` and runtime code that reads `window.__LD_RESULTS__`.
   - Include a visible **Refresh live data** button wired to `refresh()`.
   - Don't claim live-linked unless newly added/updated source documents appear after refresh without regenerating HTML.

4. **Design requirements**
   - Match Knowledge Explorer styling by default: navy `#03274A`, blue `#164C80`, yellow `#FFD633`, pale blue `#CAD8E5`, line `#D7E2EC`, sand `#F7F5F3`, black `#161616`, muted `#5A5A57`.
   - Use Aptos / Segoe UI / Arial.
   - Use rounded navy hero, yellow translucent circle, short kicker label, large white title, pale lead text, optional inline SVG illustration.
   - Use exactly 3 document cards per desktop row: `grid-template-columns: repeat(3, minmax(0, 1fr))`; collapse to 2 then 1.

5. **Dashboard features**
   - Search across Document Title/title, filename, abstract/description, metadata, Document Owner, contacts, author/editor.
   - Filter using real fields. Prefer Practice, Service Area, Industry Sectors, Market, Regions, Lead Office, Document Type, Year created when present.
   - Cards include title, Document Owner below title, abstract preview with hover/focus popup, metadata pills, dates, Open and Download actions.

6. **Open and Download link rules — critical**
   - In SharePoint HTML preview the dashboard runs inside `about:srcdoc`; **do not use `location.origin`** for Open or Download because it may resolve to `null`.
   - Define a constant site URL from the resolved web URL, e.g. `var SITE='https://tenant.sharepoint.com/sites/site';`.
   - Do not render `href="#"`. If no path is available, render a disabled text control instead.
   - Build links from a decoded server-relative SharePoint path.
   - Resolve path in this order: `FileRef` / `File_x0020_Ref` / `ServerRelativeUrl` / `Path`; then `EncodedAbsUrl` parsed to `pathname`; then `FileDirRef + '/' + FileLeafRef`.
   - Use `safeDecode()` repeatedly before encoding route parameters so `Documents%2520Browser` becomes `Documents Browser`, then re-encodes only once.
   - **Verified working Open pattern for this site:** `SITE + '/_layouts/15/Doc.aspx?sourcedoc=' + encodeURIComponent(serverRelativePath) + '&file=' + encodeURIComponent(fileName) + '&action=embedview&mobileredirect=true'`.
   - The `serverRelativePath` must be decoded first, then encoded exactly once in the `sourcedoc` route parameter.
   - Do **not** replace this with the direct file URL `?web=1&action=embedview` unless it has been manually tested in the target dashboard after saving.
   - Do **not** use `download.aspx?SourceUrl=...&transform=pdf` for Open; it can fail with File Not Found when spaces are double-encoded.
   - Download: `SITE + '/_layouts/15/download.aspx?SourceUrl=' + encodeURIComponent(serverRelativePath)`.
   - Add `target="_blank"`, `rel="noopener"`, and `data-interception="off"`.
   - Never use PDF transform links for Open; never append `download=1`; validate no `%2520`, no `null/sites`, no `href="#"`, and no absolute URL inside `SourceUrl`.

7. **Validate before saving**
   - Confirm live manifest/source exists.
   - Confirm visible refresh control exists and is wired.
   - Confirm 3-column desktop grid exists.
   - Confirm Document Title is first title source and Document Owner appears below title.
   - Confirm Knowledge Explorer-style header colors/structure are present.
   - Confirm Open uses the verified `Doc.aspx?sourcedoc=...&file=...&action=embedview&mobileredirect=true` pattern; confirm Download uses `SITE`, robust source-path fallback, and neither link renders `#`, `null/sites`, PDF transform, direct untested `?web=1&action=embedview`, or double-encoded `%2520`.

8. **Save**
   - Create or update an `.html` file in the requested destination.
   - Prefer saving HTML dashboards to a Reports library or the location requested by the user unless updating an explicitly selected existing file.
   - Provide the file link.

## Output format
Respond with:
- Link to the dashboard.
- Whether it is fully live-linked.
- Note that it uses 3 desktop columns, Document Title, Document Owner below title, Knowledge Explorer styling, visible refresh, and robust decoded server-relative Open/Download links.
- Any limitation or workaround.

## Honesty
If live binding can't target the requested folder or view, say so and offer a workaround. Don't invent working links or data.
