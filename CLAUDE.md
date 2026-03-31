# IWE Knowledge Graph CLI

This project uses `iwe` - a CLI tool for managing markdown document graphs with block references.

**All `iwe` commands must be run from the `seventeen-centuries/` directory** where the `.iwe/config.toml` is located.

## Quick Reference

### Project Setup

``` bash
iwe init                    # Initialize project (creates .iwe/config.toml)
```

### Creating Documents

``` bash
iwe new "Title"                           # Create from default template
iwe new "Title" --template meeting        # Use specific template
iwe new "Title" --content "Body text"     # With inline content
iwe new "Title" --edit                    # Open in $EDITOR after creation
iwe new "Title" --if-exists override      # Overwrite if exists
```

Template variables: `{{title}}`, `{{slug}}`, `{{today}}`, `{{id}}`, `{{content}}`

### Retrieving Documents

``` bash
iwe retrieve -k doc-key                   # Retrieve document with context
iwe retrieve -k key -d 2                  # Follow refs 2 levels deep
iwe retrieve -k key -c 2                  # Include 2 levels of parent context
iwe retrieve -k key -l                    # Include inline references
iwe retrieve -k key -b                    # Include backlinks
iwe retrieve -k key -f json               # Output as JSON
iwe retrieve -k key -f keys               # Output keys only
iwe retrieve --dry-run                    # Show doc count without content
```

### Finding Documents

``` bash
iwe find                                  # List all docs (sorted by popularity)
iwe find "query"                          # Fuzzy search title/key
iwe find --roots                          # Only root documents (no parents)
iwe find --refs-to key                    # Documents referencing this key
iwe find --refs-from key                  # Documents referenced by this key
iwe find -f keys                          # Output keys for piping
```

### Document Tree

``` bash
iwe tree                                  # Show full hierarchy
iwe tree -k doc-key                       # Tree from specific document
iwe tree -d 2                             # Limit depth to 2 levels
iwe tree -f keys                          # Output keys only
iwe tree -f json                          # Output as JSON
```

### Consolidating Documents

``` bash
iwe squash doc-key                        # Combine with linked content
iwe squash doc-key -d 3                   # Follow refs 3 levels deep
iwe squash doc-key > output.md            # Export to file
```

### Refactoring

**Extract section to new document:**

``` bash
iwe extract doc-key --list                # List sections with block numbers
iwe extract doc-key --section "Title"     # Extract by section title
iwe extract doc-key --block 2             # Extract by block number
iwe extract doc-key --dry-run             # Preview changes
```

**Inline referenced document:**

``` bash
iwe inline doc-key --list                 # List inclusion links
iwe inline doc-key --reference "ref-key"  # Inline by reference key
iwe inline doc-key --block 1              # Inline by block number
iwe inline doc-key --keep-target          # Keep target doc after inlining
iwe inline doc-key --as-quote             # Inline as blockquote
```

**Rename document:**

``` bash
iwe rename old-key new-key                # Rename and update all refs
iwe rename old-key new-key --dry-run      # Preview changes
```

**Delete document:**

``` bash
iwe delete doc-key                        # Delete with confirmation
iwe delete doc-key --force                # Skip confirmation
iwe delete doc-key --dry-run              # Preview changes
```

### Maintenance

``` bash
iwe normalize                             # Fix formatting across all docs
iwe stats                                 # Show graph statistics
iwe stats -f csv                          # Export stats as CSV
```

### Export & Visualization

``` bash
iwe export dot                            # Export as Graphviz DOT
iwe export dot -k doc-key -d 2            # Export subgraph
iwe export dot --include-headers          # Include section headers
iwe export dot | dot -Tpng -o graph.png   # Generate PNG
```

## Output Formats

Most commands support `-f/--format`:

- `markdown` (default): Human-readable with links
- `keys`: One key per line (for piping)
- `json`: Structured data

## Key Concepts

- **Document key**: Filename without .md extension
- **Block reference**: Inclusion link that embeds one document in another
- **Inline reference**: Regular markdown link within text
- **Root document**: Document with no incoming block references
- **Parent/Child**: Determined by block reference relationships
