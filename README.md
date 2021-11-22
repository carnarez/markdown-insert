# Module `markdown_insert`

Python-Markdown extension processing the `&[]()` insertion markers.

`pip install git+https://github.com/carnarez/markdown-insert` and refer to the brilliant
[`Python` implementation](https://github.com/Python-Markdown/markdown).

This was made to allow addition of external `Markdown` and/or HTML into the current
document. Since it is processed before any rendering happens it can be used to insert
any kind of \[text\] inputs; use/authorise with caution!

The marker is to be read "**insert \[** *line ranges (if provided, otherwise all lines)*
**\] from (** *file at this path* **)**."

**Example:**

```markdown
This is the external snippet located in `/wherever/snippet.md`.
```

```python
import markdown
provided = '''
**Snippet:**

&[1-10](/wherever/snippet.md)

Caption of the snippet.
'''.strip()
rendered = markdown.markdown(provided, extensions=[InsertExtension()])
expected = '''
**Snippet:**

This is the external snippet located in `/wherever/snippet.md`.

Caption of the snippet.
'''.strip()
assert rendered == expected
```

**Notes:**

- The marker needs to be on its own line, by itself. Any number of spacing character is
  allowed in front/back, but no other text.
- This implementation is rather simplistic, check
  [`markdown-include`](https://github.com/cmacmackin/markdown-include) for a more
  flexible handling of inserts (via a different syntax), or the
  [extension](https://facelessuser.github.io/pymdown-extensions/extensions/snippets/)
  from the [`pymdown` collection](https://facelessuser.github.io/pymdown-extensions/).

**Classes:**

- [`InsertPreprocessor`](#markdown_insertinsertpreprocessor)
- [`InsertExtension`](#markdown_insertinsertextension)

## Classes

### `markdown_insert.InsertPreprocessor`

Preprocessor to catch and replace the `&[]()` markers.

We are here abusing the `Markdown` link syntax; we need to run it *before* the regular
processing of the `Markdown` content.

**Methods:**

- [`expand_indices()`](#markdown_insertinsertpreprocessorexpand_indices)
- [`run()`](#markdown_insertinsertpreprocessorrun)

#### Constructor

```python
InsertPreprocessor()
```

#### Methods

##### `markdown_insert.InsertPreprocessor.expand_indices`

```python
expand_indices(ranges: str) -> typing.List[int]:
```

Expand a textual description of line range(s) to indices.

**Parameters:**

- `ranges` \[`str`\]: Line indices or range(s) to include, *i.e.*, `"1-4 7-10 22"`. Note
  the **lines are indexed from 1** (to make it more human-readable).

**Returns:**

- \[`typing.List[int]`\]: List of all indices to consider, 0-based for `Python`. The
  example from above would return: `[0, 1, 2, 3, 6, 7, 8, 9, 21]`.

**Decoration** via `@staticmethod`.

##### `markdown_insert.InsertPreprocessor.run`

```python
run(lines: typing.List[str]) -> typing.List[str]:
```

Overwritten method to process the input `Markdown` lines.

**Parameters:**

- `lines` \[`typing.List[str]`\]: `Markdown` content (split by `\n`).

**Returns:**

- \[`typing.List[str]`\]: Same list of lines, but processed (*e.g.*, containing the
  inserted content). The leading spacing -taken from the marker- is conserved for each
  inserted line.

**Notes:**

- *One per line!*
- The current implementation allows inserting *within triple-quoted blocks*.

### `markdown_insert.InsertExtension`

Extension proper, to be imported when calling for the `Markdown` renderer.

**Methods:**

- [`extendMarkdown()`](#markdown_insertinsertextensionextendmarkdown)

#### Constructor

```python
InsertExtension()
```

#### Methods

##### `markdown_insert.InsertExtension.extendMarkdown`

```python
extendMarkdown(md: Markdown):
```

Overwritten method to process the content.

**Parameters:**

- `md` \[`markdown.core.Markdown`\]: Internal `Markdown` object to process.

**Notes:**

Since we are abusing the `Markdown` link syntax the preprocessor needs to be called with
a high priority (100).
