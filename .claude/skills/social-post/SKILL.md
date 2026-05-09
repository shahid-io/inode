---
name: social-post
description: Generate a paper-and-ink carousel HTML post for the inode project, focused on story and meaning rather than feature pitches. Use when the user wants to share what the project is FOR, who it's FOR, or why it exists — not what it does or what's coming next.
---

# Social post generator (story-style)

## What this skill is for

Generate a 1080×1080 carousel HTML in `social/` for posts about *meaning* —
what the tool exists for, who it's for, why anyone would care. The reader
is the protagonist; the project is the supporting character.

## What this skill is NOT for

- **Dev logs / debugging stories** → write manually using `social/day-2-debug-and-ship.html` as the model
- **Feature launches / roadmaps** → use `social/post-templates.html` as the model
- **Technical breakdowns** → not the right tone

If the user asks for one of those, decline this skill and offer the
manual path instead.

## Process

1. **Get the angle.** Ask the user in one sentence who the reader is —
   not the demographic ("developers"), the *moment* ("the dev who lost a
   stripe key at 11pm"). Skip if they've already given it. If they say
   "you decide," default to: *the engineer who has 7 untitled notes apps,
   none trusted.*

2. **Read the design system.** Open `social/post-templates.html` and
   `social/day-2-debug-and-ship.html`. Reuse verbatim:
   - The entire `<style>` block (CSS tokens, post canvas, typography)
   - The `<svg width="0" height="0">` defs block (logo, sketch shapes)
   - The download script at the bottom (PDF + ZIP + per-post PNG)

   Do not invent new fonts or colors. The brand is consistent.

3. **Pick page count.** 10–14 pages, default 12. Match the post's depth.

4. **Draft the arc.** A story arc that has worked:
   1. cover — name the reader in their moment
   2. the relatable pain — specific, not abstract
   3. the pattern — why this keeps happening
   4. the shift — what changed (often: AI / embeddings)
   5. the admission — why this was built (use first person sparingly)
   6. the principle — local-first / encrypted / yours
   7. the how (plain words) — embeddings as memory aids, LLM as interpreter
   8. the honest tradeoff — what it doesn't do
   9. for-whom — concrete reader profiles
   10. the promise — what you stop worrying about
   11. the invitation — low-pressure CTA (try / watch / follow)

   Combine or split as needed. One idea per page.

5. **Write the HTML.** Self-contained file. Include:
   - 1080×1080 pages with paper-and-ink aesthetic
   - Header: logo + a short caption
   - Footer: handle (`@binary.semaphore`), page counter (`07 / 12`), tags
   - Three download buttons up top: PDF, ZIP, and per-post PNG under each
   - Reuse the SVG defs (logo, sketch-box, sketch-underline, etc.)

6. **Filename.** Slug from the cover line:
   `social/<short-slug>.html`. Examples: `for-the-ones-who-lose-things.html`,
   `your-second-brain-can-be-yours.html`. Date is implicit in git.

7. **Tell the user the path** and remind them no build step is needed —
   open the HTML in a browser to preview, click the buttons to export.

## Voice

- The reader is the protagonist. Not the project, not the founder.
- First person sparingly — when used, it must be a confession or admission,
  not a brag.
- No marketing voice. Banned: "10x", "supercharge", "unleash", "game-changer".
- Concrete > abstract. *"the .env file you can't find at midnight"* beats
  *"knowledge management."*
- Brand names (Slack, Notion, 1Password) appear only when they make a
  sentence more honest. Never as targets to dunk on.
- Cite real numbers and real code only when they earn their place. This
  isn't a dev log.
- No emojis. The hand-drawn marks (✓, ✗, →) inside the SVGs are enough.

## Project facts you can draw on

- inode is a CLI knowledge base. Save anything (notes, API keys, decisions,
  commands, URLs); ask in plain English; get the right thing back.
- Stack today: Go · SQLite + sqlite-vec · Ollama (local LLM + embeddings) ·
  AES-256-GCM at rest · OS keychain for keys.
- Ollama is the default — fully local, fully free. Voyage / Claude API
  paths exist but are optional.
- This project is being built at zero cost. Local-first is not an
  aesthetic choice; it's the architecture.
- Phase 1 (v0.1.0) shipped. Phase 2 (multi-user / hosted) is hypothetical
  and not the subject of these posts.

## Reference files
- `social/post-templates.html` — launch carousel; design system source of truth
- `social/day-2-debug-and-ship.html` — most recent carousel; current download UX
- `README.md` — neutral-voice product description
