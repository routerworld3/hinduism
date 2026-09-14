Confirmed by testing: `--strict` does **not** catch a file you forget to add to `nav`. More on that below.

## Add a page to an existing section

Three steps. Say you're adding a Madhvācārya page under Schools of Vedānta.

**1.** Create `docs/madhvacharya.md`, following your existing convention:

```markdown
---
title: "Madhvacharya and Dvaita Vedanta"
description: "Dualism, the eternal distinction between soul and God, and Madhva's critique of Advaita."
---

Your content here...
```

**2.** Add one line to `nav:` in `mkdocs.yml`, in the position you want it to appear:

```yaml
  - Schools of Vedanta:
      - hindu-philosophy-schools.md
      - adi-shankaracharya.md
      - ramanujacharya.md
      - madhvacharya.md          # ← new
```

**3.** Add a card to `docs/index.md` under that section's `<div class="grid cards" markdown>` block:

```markdown
-   __Madhvācārya and Dvaita Vedānta__

    ---

    Strict dualism — the soul and God remain eternally distinct. Madhva's
    reading of "tat tvam asi" against both Śaṅkara and Rāmānuja.

    [Read about Madhva](madhvacharya.md)
```

Step 3 is manual because the homepage is hand-written prose, not generated. Skip it and the page still works — it just won't be on the homepage.

Then `git add docs/madhvacharya.md mkdocs.yml docs/index.md && git commit && git push`.

## Add a new section

Just a new key with children. Order in the file is the order in the sidebar:

```yaml
nav:
  - Home: index.md
  - Foundations:
      - hindu-text-timeline.md
      - monotheism-and-divine-forms.md
  - Schools of Vedanta:
      - hindu-philosophy-schools.md
      - adi-shankaracharya.md
      - ramanujacharya.md
  - Practice and ritual:          # ← new section
      - puja-and-worship.md
      - festivals.md
  - Modern traditions:
      - swaminarayan-philosophy.md
      - dada-bhagwan-akram-vignan.md
  - Stories and cosmology:
      - brahma-vishnu-shiva-stories.md
```

Nothing else changes. No folder needed — section names live in `mkdocs.yml`, filenames stay flat in `docs/`. You'd also add a matching `## Practice and ritual` heading and card grid to `index.md`.

## The gotcha now that `nav` is explicit

Before, MkDocs auto-discovered every file. Now `nav` is the source of truth. I tested this: drop a `.md` into `docs/` and forget the nav line, and MkDocs logs it at **INFO** level, which `--strict` ignores. The build goes green, the page deploys, it's reachable by URL and findable in search — but there's no link to it anywhere. Easy to lose a page for weeks.

Add this to the bottom of `mkdocs.yml` to turn that into a hard failure:

```yaml
validation:
  nav:
    omitted_files: warn
  links:
    absolute_links: warn
    unrecognized_links: warn
```

Verified: with this in place, a file missing from `nav` aborts `mkdocs build --strict`. So CI now tells you instead of silently orphaning the page.

## Two details worth knowing

**Nav titles** come from the `title:` frontmatter in each file — that's why your eight existing pages already show proper names. To override just in the sidebar without touching the file:

```yaml
      - Madhva: madhvacharya.md
```

**Cross-page links** use the relative `.md` path, not the URL: `[see Rāmānuja](ramanujacharya.md)`. MkDocs rewrites it to `../ramanujacharya/` at build time, so the same link also works when browsing the repo on GitHub. With `unrecognized_links: warn` above, a typo in that path now fails the build instead of shipping a dead link.

Your workflow after each change stays the same: `mkdocs serve` to eyeball it, `mkdocs build --strict` to catch what your eyes won't, then push.
