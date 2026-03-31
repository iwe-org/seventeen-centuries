# Seventeen Centuries

A knowledge graph connecting three philosophical texts across seventeen centuries—from Marcus Aurelius (170 AD) to Machiavelli (1513) to Nietzsche (1886)—exploring how ideas of virtue, power, and morality evolved through time.

## Overview

This project transforms philosophical texts into an interconnected knowledge graph using [iwe](https://github.com/iwe-org/iwe), a CLI tool for managing markdown document graphs with inclusion links.

The graph is **hierarchical** with **polyhierarchy support**—documents can belong to multiple parent documents simultaneously. A concept like "virtue" can appear under both a book index and a thematic category, reflecting how knowledge naturally connects across domains.

The graph contains **1,200+ documents** including:

- **838 text fragments** from three books
- **340+ concept files** linking philosophical ideas, historical figures, and themes
- **Category documents** organizing concepts into navigable groupings

## Highlights

Some concept files reveal how the same idea carries different meanings across philosophers:

**[virtue.md](graph/virtue.md)** — For Marcus Aurelius, virtue is acting according to nature and reason, serving the common good as naturally as the eye sees. Machiavelli inverts this: a prince who acts entirely virtuously will be ruined among so much evil. Nietzsche warns against becoming enslaved to one's own virtues, noting that every virtue inclines toward stupidity.

## Source Texts

All source texts are public domain editions from [Standard Ebooks](https://standardebooks.org/):

| Book                                                                                                             | Index                                  | Author              | Fragments |
| ---------------------------------------------------------------------------------------------------------------- | -------------------------------------- | ------------------- | --------- |
| [Beyond Good and Evil](https://standardebooks.org/ebooks/friedrich-nietzsche/beyond-good-and-evil/helen-zimmern) | [bge.md](graph/bge.md)                 | Friedrich Nietzsche | 296       |
| [Meditations](https://standardebooks.org/ebooks/marcus-aurelius/meditations/george-long)                         | [meditations.md](graph/meditations.md) | Marcus Aurelius     | 515       |
| [The Prince](https://standardebooks.org/ebooks/niccolo-machiavelli/the-prince/w-k-marriott)                      | [prince.md](graph/prince.md)           | Niccolò Machiavelli | 27        |


## Categories

Concepts are organized into thematic categories:

- [philosophers](graph/philosophers.md) - Thinkers discussed across the texts
- [virtues](graph/virtues.md) - Qualities of character and excellence
- [moral-systems](graph/moral-systems.md) - Different ethical frameworks
- [nietzschean-concepts](graph/nietzschean-concepts.md) - Ideas central to Nietzsche
- [power-dynamics](graph/power-dynamics.md) - Hierarchy, domination, social stratification
- [religion](graph/religion.md) - Religious concepts and critiques
- [politics](graph/politics.md) - Forms of government, political theory
- [ancient-cultures](graph/ancient-cultures.md) - Civilizations referenced as examples
- [psychology](graph/psychology.md) - Mental phenomena, drives, emotions
- [philosophical-schools](graph/philosophical-schools.md) - Named traditions and movements

## How the Graph Was Constructed

The knowledge graph was built through a multi-stage pipeline, transforming raw XHTML source files into an interconnected web of addressable markdown documents.

### Stage 1: Fragment Extraction

The first stage converts each book from Standard Ebooks XHTML format into atomic markdown fragments—one file per aphorism, passage, or chapter.

#### Beyond Good and Evil

Nietzsche's work consists of 296 numbered aphorisms across 9 parts, plus a preface and epilogue poem.

**Source structure:**

```
source/friedrich-nietzsche_beyond-good-and-evil_helen-zimmern/src/epub/text/
├── preface.xhtml
├── part-1.xhtml through part-9.xhtml
├── from-the-heights.xhtml (epilogue poem)
└── endnotes.xhtml (ignored)
```

**Extraction rules:**

1.  Parse each `part-X.xhtml` file
2.  Extract each `<section id="chapter-X-Y">` as one aphorism
3.  Get aphorism number from `<h3 epub:type="ordinal">` (handles sub-numbered entries like 65a, 73a)
4.  Zero-pad to 3 digits: `001.md`, `065a.md`, `146.md`
5.  Strip endnote reference tags entirely
6.  Convert `<em>` and `<i>` to `*text*` markdown
7.  Convert `<blockquote>` to `>` markdown quotes
8.  Preserve foreign language text (strip `xml:lang` attributes)
9.  Separate paragraphs with blank lines

**Fragment format:**

``` markdown
# 146

He who fights with monsters should be careful lest he thereby become a monster. And if thou gaze long into an abyss, the abyss will also gaze into thee.
```

#### Meditations

Marcus Aurelius's work consists of 12 books containing ~486 individual passages (meditations).

**Source structure:**

```
source/marcus-aurelius_meditations_george-long/src/epub/text/
├── book-1.xhtml through book-12.xhtml
└── endnotes.xhtml (ignored)
```

**Extraction rules:**

1.  Parse each `book-X.xhtml`
2.  Extract all `<p>` tags in order (each paragraph = one passage)
3.  Skip final paragraph if it's a location note (e.g., "Among the Quadi at the Granua")
4.  Strip endnote reference tags
5.  Convert emphasis tags to markdown
6.  Number passages sequentially per book
7.  Filename format: `BBB-PPP.md` (book-passage, zero-padded)

**Fragment format:**

``` markdown
# III.7

Never value anything as profitable to thyself which shall compel thee to break thy promise, to lose thy self-respect, to hate any man...
```

#### The Prince

Machiavelli's work consists of a dedication and 26 chapters.

**Source structure:**

```
source/niccolo-machiavelli_the-prince_w-k-marriott/src/epub/text/
├── dedication.xhtml
├── chapter-1.xhtml through chapter-26.xhtml
└── endnotes.xhtml (ignored)
```

**Extraction rules:**

1.  Parse `dedication.xhtml` → `00-dedication.md`
2.  Parse each `chapter-X.xhtml` → `XX.md` (zero-padded)
3.  Extract chapter number from `<h2>` (Roman numeral)
4.  Extract chapter title from `<p epub:type="title">`
5.  Heading format: `# VII. Title Here`
6.  Strip endnote references, convert emphasis to markdown

**Fragment format:**

``` markdown
# VII. Concerning New Principalities Which Are Acquired Either by the Arms of Others or by Good Fortune

Those who solely by good fortune become princes from being private citizens have little trouble in rising, but much in keeping atop...
```

### Stage 2: Entity Extraction

With ~845 fragments extracted, the next stage identifies philosophical concepts, historical figures, and themes within each fragment, creating entity files and inline links.

**Entity types extracted:**

1.  **Philosophical concepts** - Abstract ideas, technical terms ([Will to Power](graph/will-to-power.md), Logos, Virtù)
2.  **Named persons/figures** - Historical figures, philosophers ([Spinoza](graph/spinoza.md), [Caesar](graph/caesar.md), [Plato](graph/plato.md))
3.  **Themes/topics** - Recurring subjects ([morality](graph/morality.md), [power](graph/power.md), [death](graph/death.md), [nature](graph/nature.md))

**Process:**

1.  For each book directory, list all fragment files
2.  Run an LLM subagent on each fragment to extract entities as structured JSON
3.  Collect all entities across fragments, deduplicate by slug
4.  Create one `.md` file per unique entity with backlinks to mentioning fragments
5.  Update fragment text with inline links to entity files

**Entity file format:**

``` markdown
# Will to Power

## Mentioned In

- [013](013.md)
- [022](022.md)
- [036](036.md)
```

**Fragment modification:**

Before:

``` markdown
# 13

...life itself is *Will to Power*; self-preservation is only one of the indirect...
```

After:

``` markdown
# 13

...life itself is [Will to Power](will-to-power.md); [self-preservation](self-preservation.md) is only one of the indirect...
```

Each fragment was limited to 3-7 most significant entities to avoid over-linking.

### Stage 3: Flattening and Merging

Initially, each book existed in its own subdirectory with independent entity files. The flattening stage merged everything into a single flat directory, handling naming collisions and combining overlapping concepts.

**Before (nested structure):**

```
graph/
├── prince/
│   ├── prince.md          (index)
│   ├── 01.md ... 26.md    (fragments)
│   ├── virtue.md          (concept)
│   └── cesare-borgia.md   (concept)
├── meditations/
│   ├── meditations.md     (index)
│   ├── 001-001.md ...     (fragments)
│   ├── virtue.md          (concept)
│   └── stoicism.md        (concept)
└── bge/
    ├── bge.md             (index)
    ├── 001.md ... 296.md  (fragments)
    ├── virtue.md          (concept)
    └── will-to-power.md   (concept)
```

**After (flat structure):**

```
graph/
├── prince.md              (index)
├── meditations.md         (index)
├── bge.md                 (index)
├── prince-01.md           (fragment)
├── meditations-001-001.md (fragment)
├── bge-001.md             (fragment)
├── virtue.md              (merged concept)
├── cesare-borgia.md       (unique concept)
└── will-to-power.md       (unique concept)
```

**Step 1: Prefix all fragments**

Fragment files received a book prefix to avoid collisions:

| Book                 | Original Pattern            | New Pattern                               |
| -------------------- | --------------------------- | ----------------------------------------- |
| The Prince           | `01.md`, `00-dedication.md` | `prince-01.md`, `prince-00-dedication.md` |
| Meditations          | `001-001.md`                | `meditations-001-001.md`                  |
| Beyond Good and Evil | `001.md`, `065a.md`         | `bge-001.md`, `bge-065a.md`               |


All internal links were updated accordingly.

**Step 2: Restructure book index files**

Each book index was updated to have two sections—Fragments and Concepts:

``` markdown
# Beyond Good and Evil

## Fragments

[bge-001](bge-001.md)

[bge-002](bge-002.md)

## Concepts

[will-to-power](will-to-power.md)

[will-to-truth](will-to-truth.md)
```

**Step 3: Merge overlapping concept files**

Concepts appearing in multiple books were merged into single files with book-specific sections:

Before (3 separate files):

``` markdown
# prince/virtue.md
## Mentioned In
- [prince-15](prince-15.md)

# meditations/virtue.md
## Mentioned In
- [meditations-009-043](meditations-009-043.md)

# bge/virtue.md
## Mentioned In
- [bge-041](bge-041.md)
```

After (1 merged file):

``` markdown
# Virtue

## Mentioned In

### The Prince
- [prince-15](prince-15.md)

### Meditations
- [meditations-009-043](meditations-009-043.md)

### Beyond Good and Evil
- [bge-041](bge-041.md)
```

**Overlapping concepts that were merged:** [virtue](graph/virtue.md), [soul](graph/soul.md), [plato](graph/plato.md), [socrates](graph/socrates.md), [truth](graph/truth.md), [nature](graph/nature.md), [gods](graph/gods.md), [epicurus](graph/epicurus.md), [cruelty](graph/cruelty.md), [free-will](graph/free-will.md)

**Statistics after flattening:**

| Category                       | Count |
| ------------------------------ | ----- |
| Total files                    | 1,183 |
| The Prince fragments           | 27    |
| The Prince concepts            | 20    |
| Meditations fragments          | 515   |
| Meditations concepts           | 20    |
| Beyond Good and Evil fragments | 296   |
| Beyond Good and Evil concepts  | 310   |
| Overlapping concepts merged    | 10    |


### Stage 4: Adding Categories

With hundreds of atomic concept documents, categories provide thematic entry points for navigation and discovery.

**Methodology:**

1.  **Survey existing concepts** - Review all concept documents (excluding numbered fragments)
2.  **Identify natural groupings** - Look for entity types, conceptual domains, thematic clusters, and hierarchical relationships. Some categories like [philosophers](graph/philosophers.md) and [religion](graph/religion.md) are directly mentioned in the source texts themselves, making them natural organizing principles
3.  **Balance specificity and coverage** - Categories should be specific enough to be meaningful but broad enough to contain 5+ documents
4.  **Allow multiple membership** - Unlike folders, a concept like [Socrates](graph/socrates.md) can belong to both [philosophers](graph/philosophers.md) and [ancient-greeks](graph/ancient-greeks.md)

**Categories emerged from the content:**

| Category                                                | Rationale                                             |
| ------------------------------------------------------- | ----------------------------------------------------- |
| [philosophers](graph/philosophers.md)                   | Core thinkers discussed or referenced in the texts    |
| [people](graph/people.md)                               | Historical figures who are not primarily philosophers |
| [moral-systems](graph/moral-systems.md)                 | Different ethical frameworks and moral typologies     |
| [nietzschean-concepts](graph/nietzschean-concepts.md)   | Ideas originating from or central to Nietzsche        |
| [power-dynamics](graph/power-dynamics.md)               | Hierarchy, domination, social stratification          |
| [virtues](graph/virtues.md)                             | Qualities of character and excellence                 |
| [religion](graph/religion.md)                           | Religious concepts, institutions, critiques           |
| [politics](graph/politics.md)                           | Forms of government, political theory                 |
| [ancient-cultures](graph/ancient-cultures.md)           | Civilizations and peoples referenced as examples      |
| [epistemology](graph/epistemology.md)                   | Knowledge, truth, reasoning, perspective              |
| [psychology](graph/psychology.md)                       | Mental phenomena, drives, emotions                    |
| [self-development](graph/self-development.md)           | Personal transformation and self-overcoming           |
| [will-concepts](graph/will-concepts.md)                 | Variations on will as a philosophical concept         |
| [metaphysics](graph/metaphysics.md)                     | Nature of reality, existence, being                   |
| [aesthetics](graph/aesthetics.md)                       | Art, beauty, tragedy, music                           |
| [philosophical-schools](graph/philosophical-schools.md) | Named traditions and movements                        |
| [states-of-being](graph/states-of-being.md)             | Existential conditions and experiences                |
| [paradoxes](graph/paradoxes.md)                         | Tensions and dialectical concepts                     |


**Implementation:**

Each category becomes a document containing inclusion links to its member concepts:

``` markdown
# Philosophers

[plato](plato.md)

[socrates](socrates.md)

[aristotle](aristotle.md)
```

For example, the [philosophers](graph/philosophers.md) category includes [Plato](graph/plato.md), [Socrates](graph/socrates.md), [Aristotle](graph/aristotle.md), and dozens more.

The blank lines between links are required by iwe syntax for inclusion links (block references).

### Stage 5: Writing Summaries

The final stage adds summaries to concept files that appear across multiple books, highlighting how the same idea carries different meanings for different philosophers.

For merged concepts like `virtue.md`, an LLM analyzed the referenced fragments from each book and synthesized a summary showing the contrasting perspectives:

``` markdown
# Virtue

## Summary

Virtue carries vastly different meanings across these works. For Marcus Aurelius,
virtue is acting according to nature and reason, serving the common good as
naturally as the eye sees, requiring no external reward. Machiavelli inverts this,
arguing that a prince who acts entirely virtuously will be ruined among so much
evil; what appears virtuous may destroy while apparent vice may preserve. Nietzsche
warns against becoming enslaved to one's own virtues, noting that every virtue
inclines toward stupidity and that moral philosophy has made virtue tedious through
its ponderous advocates.

## Mentioned In
...
```

These summaries transform simple backlink indexes into valuable comparative analyses, revealing how philosophical concepts evolved and were contested across traditions.

## Usage

Requires [iwe](https://github.com/iwe-org/iwe) CLI.

``` bash
iwe find                           # List all documents
iwe retrieve -k virtue -b          # Get [virtue](graph/virtue.md) with backlinks
iwe tree -k bge                    # Show BGE document tree
iwe find --refs-to will-to-power   # Find references to [will-to-power](graph/will-to-power.md)
iwe stats                          # Graph statistics
```

## Related Projects

- [iwe](https://github.com/iwe-org/iwe) - CLI for markdown document graphs
- [Standard Ebooks](https://standardebooks.org/) - Source texts
