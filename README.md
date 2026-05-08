# granola-obsidian-sync

Automatically syncs [Granola](https://www.granola.ai) meeting notes into an [Obsidian](https://obsidian.md) vault — with optional Claude AI enrichment that adds frameworks, key takeaways, and deeper analysis to every note.

Works on **macOS and Windows**. Works for any Obsidian vault, any course names, any meeting categories.

## What it does

- **Class meetings** → appended as sessions to `Classes/[ClassName].md`
- **All other meetings** → individual files in `Meetings/[category]/YYYY-MM-DD - Title.md`, auto-categorized by Granola folder name or title keywords
- **Smart updates** — if you edit notes in Granola later, only the AI block refreshes; your personal `### My Notes` section is always preserved
- **No duplicates** — each meeting is tracked by `granola_id`; re-runs are safe
- **Optional enrichment** — add an Anthropic API key to get a `## What This Was Really About` synthesis block appended to every note (frameworks, key takeaways, connections)

## Folder taxonomy

Non-class meetings are automatically sorted using two layers:

1. **Granola folder → Obsidian folder** — configure `_GRANOLA_FOLDER_MAP` in the script
2. **Title keyword rules** — configure `_TITLE_CATEGORY_RULES` in the script

Default fallback: `Meetings/Other/`

---

## Setup

### 1. Dependencies

**macOS**
```bash
pip3 install httpx
```

**Windows** (PowerShell or Command Prompt)
```
pip install httpx
```

Python 3.8+ required. Download from [python.org](https://python.org) if needed.

---

### 2. Granola auth

The script reads your Granola token automatically — no configuration needed, just be logged into Granola.

| Platform | Auth file location |
|----------|--------------------|
| macOS | `~/Library/Application Support/Granola/supabase.json` |
| Windows | `%APPDATA%\Granola\supabase.json` |

---

### 3. Configure your vault paths

Edit the paths near the top of `granola_obsidian.py`:

```python
OBSIDIAN_CLASSES  = Path.home() / "Documents/Obsidian Vault/Classes"
OBSIDIAN_MEETINGS = Path.home() / "Documents/Obsidian Vault/Meetings"
```

`Path.home()` resolves to your user home directory on both platforms (`C:\Users\You` on Windows, `/Users/you` on Mac). Adjust the subfolder to match where your vault lives.

---

### 4. Configure class matching

Edit `CLASS_PATTERNS` to match your course titles as they appear in Granola:

```python
CLASS_PATTERNS = [
    ("cs 221",    "CS221"),       # matches any title containing "cs 221"
    ("econ 102",  "Econ102"),     # matches any title containing "econ 102"
    # add your courses here
]
```

The matched name must correspond to a `.md` file in your `Classes/` folder.

---

### 5. Optional: Claude enrichment

Create a `.env` file next to the script:
```
ANTHROPIC_API_KEY=sk-ant-...
```

When set, every synced note gets a rich synthesis block with frameworks, takeaways, and connections to outside sources. Uses `claude-sonnet-4-5` (configurable via `ENRICH_MODEL`).

---

### 6. Schedule automatic daily runs

#### macOS — launchd

```bash
# Keep the script outside ~/Desktop and ~/Documents (macOS TCC blocks background access there)
mkdir -p ~/.granola-obsidian
cp granola_obsidian.py ~/.granola-obsidian/
cp .env ~/.granola-obsidian/        # if using enrichment

# Install the launchd agent (runs at 9 PM daily)
cp launchd/com.granola-obsidian.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.granola-obsidian.plist
```

Check logs:
```bash
tail -f ~/Library/Logs/granola-obsidian.log
```

Run manually:
```bash
python3 granola_obsidian.py
```

#### Windows — Task Scheduler

1. Copy the script to a permanent location, e.g. `C:\Users\You\granola-obsidian\`
2. Copy `.env` to the same folder (if using enrichment)
3. Import the task (run PowerShell as Administrator):

```powershell
Register-ScheduledTask `
  -Xml (Get-Content "task-scheduler\granola-obsidian.xml" -Raw) `
  -TaskName "granola-obsidian" `
  -Force
```

Or open **Task Scheduler** → **Action** → **Import Task…** → select `task-scheduler\granola-obsidian.xml`.

> Edit the `<WorkingDirectory>` path in the XML to match where you placed the script before importing.

Check logs:
```
%USERPROFILE%\AppData\Local\granola-obsidian\granola-obsidian.log
```

Run manually:
```powershell
python "%USERPROFILE%\granola-obsidian\granola_obsidian.py"
```

---

## Note format

### Class files (`Classes/ClassName.md`)

```markdown
## 2026-04-16 — Session Title from Granola

<!-- granola_id: abc123 updated_at: 2026-04-16T21:00:00 -->

[AI-generated notes from Granola]

## What This Was Really About

[Claude synthesis — only if ANTHROPIC_API_KEY is set]

---
### My Notes

*(your personal notes — never overwritten)*

---
```

### Meeting files (`Meetings/[category]/YYYY-MM-DD - Title.md`)

```markdown
---
granola_id: abc123
updated_at: 2026-04-16T21:00:00
title: "Meeting Title"
created_at: 2026-04-16
tags:
  - person/first-last
  - source/category
  - topic/work
---

# Meeting Title
**With:** Person Name

## Notes
[AI-generated notes from Granola]

## What This Was Really About
[Claude synthesis — only if ANTHROPIC_API_KEY is set]

---
### My Notes

*(your personal notes — never overwritten)*

---
```

---

## Extending categorization

Add keyword rules to `_TITLE_CATEGORY_RULES` in the script:

```python
_TITLE_CATEGORY_RULES = [
    ("board meeting",  "Work/Board"),
    ("1:1",            "Work/1on1s"),
    ("investor",       "Fundraising"),
    ("product review", "Product"),
    # more specific patterns first
]
```

Add Granola folder mappings to `_GRANOLA_FOLDER_MAP`:

```python
_GRANOLA_FOLDER_MAP = {
    "personal": "Personal",
    "clients":  "Work/Clients",
}
```

---

## Credits

Techniques adapted from [dannymcc/Granola-to-Obsidian](https://github.com/dannymcc/Granola-to-Obsidian).
