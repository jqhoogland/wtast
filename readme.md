# ![wtast][logo]

**W**iki**t**ext **A**bstract **S**yntax **T**ree.

***

**wtast** is a specification for representing [wikitext]() in a [syntax
tree][syntax-tree].

**Note**: this is not (yet) affiliated with the [unified]() ecosystem though it is scaffolded on top of the [mdast]() specification for markdown and implements **[unist][]**.


This document is currently under development. *It is not yet stable.*

## Contents

*   [Introduction](#introduction)
    *   [Where this specification fits](#where-this-specification-fits)
*   [Nodes](#nodes)
    *   [`Parent`](#parent)
    *   [`Literal`](#literal)
    *   [`Root`](#root)
    *   [`Paragraph`](#paragraph)
    *   [`Heading`](#heading)
    *   [`ThematicBreak`](#thematicbreak)
    *   [`List`](#list)
    *   [`ListItem`](#listitem)
    *   [`HTML`](#html)
    *   [`Definition`](#definition)
    *   [`Text`](#text)
    *   [`Emphasis`](#emphasis)
    *   [`Strong`](#strong)
    *   [`Link`](#link)
    *   [`Image`](#image)
    *   [`LinkReference`](#linkreference)
    *   [`ImageReference`](#imagereference)
    *   [Footnotes](#footnotes)
*   [Mixin](#mixin)
    *   [`Resource`](#resource)
    *   [`Association`](#association)
    *   [`Reference`](#reference)
    *   [`Alternative`](#alternative)
*   [Enumeration](#enumeration)
    *   [`referenceType`](#referencetype)
*   [Content model](#content-model)
    *   [`FlowContent`](#flowcontent)
    *   [`Content`](#content)
    *   [`ListContent`](#listcontent)
    *   [`PhrasingContent`](#phrasingcontent)
    *   [`StaticPhrasingContent`](#staticphrasingcontent)
    *   [`TransparentContent`](#transparentcontent)
*   [Glossary](#glossary)
*   [List of utilities](#list-of-utilities)
*   [References](#references)
*   [Security](#security)
*   [Related](#related)
<!-- *   [Contribute](#contribute) -->
*   [Acknowledgments](#acknowledgments)
*   [License](#license)

## Introduction

This document defines a format for representing [wikitext][] as an [abstract
syntax tree][syntax-tree].
This specification is written in a [Web IDL][webidl]-like grammar.

### Where this specification fits

wtast extends [unist][], a format for syntax trees, to benefit from its
[ecosystem of utilities][utilities].

wtast relates to [JavaScript][] in that it has a rich [ecosystem of
utilities][list-of-utilities] for working with compliant syntax trees in
JavaScript.
However, wtast is not limited to JavaScript and can be used in other programming
languages.

wtast relates to the [unified][] and [remark][] projects in that wtast is based off [mdast]() (though it is currently still unaffiliated).

## Nodes

### `Parent`

```idl
interface Parent <: UnistParent {
  children: [WtastContent]
}
```

**Parent** ([**UnistParent**][dfn-unist-parent]) represents an abstract
interface in wtast containing other nodes (said to be [*children*][term-child]).

Its content is limited to only other [**wtast content**][dfn-wtast-content].

### `Literal`

```idl
interface Literal <: UnistLiteral {
  value: string
}
```

**Literal** ([**UnistLiteral**][dfn-unist-literal]) represents an abstract
interface in wtast containing a value.

Its `value` field is a `string`.

### `Root`

```idl
interface Root <: Parent {
  type: "root"
}
```

**Root** ([**Parent**][dfn-parent]) represents a document.

**Root** can be used as the [*root*][term-root] of a [*tree*][term-tree], never
as a [*child*][term-child].
Its content model is **not** limited to [**flow**][dfn-flow-content] content,
but instead can contain any [**wtast content**][dfn-wtast-content] with the
restriction that all content must be of the same category.

### `Paragraph`

```idl
interface Paragraph <: Parent {
  type: "paragraph"
  children: [PhrasingContent]
}
```

**Paragraph** ([**Parent**][dfn-parent]) represents a unit of discourse dealing
with a particular point or idea.

**Paragraph** can be used where [**content**][dfn-content] is expected.
Its content model is [**phrasing**][dfn-phrasing-content] content.

For example, the following wikitext:

```markdown
Alpha bravo charlie.
```

Yields:

```js
{
  type: 'paragraph',
  children: [{type: 'text', value: 'Alpha bravo charlie.'}]
}
```

### `Heading`

```idl
interface Heading <: Parent {
  type: "heading"
  depth: 1 <= number <= 6
  children: [PhrasingContent]
}
```

**Heading** ([**Parent**][dfn-parent]) represents a heading of a section.

**Heading** can be used where [**flow**][dfn-flow-content] content is expected.
Its content model is [**phrasing**][dfn-phrasing-content] content.

A `depth` field must be present.
A value of `1` is said to be the highest rank and `6` the lowest.

For example, the following wikitext:

```wikitext
= Alpha =
```

Yields:

```js
{
  type: 'heading',
  depth: 1,
  children: [{type: 'text', value: 'Alpha'}]
}
```

### `ThematicBreak`

```idl
interface ThematicBreak <: Node {
  type: "thematicBreak"
}
```

**ThematicBreak** ([**Node**][dfn-node]) represents a thematic break, such as a
scene change in a story, a transition to another topic, or a new document.

**ThematicBreak** can be used where [**flow**][dfn-flow-content] content is
expected.
It has no content model.

For example, the following wikitext:

```wikitext
----
```

Yields:

```js
{type: 'thematicBreak'}
```


### `List`

```idl
interface List <: Parent {
  type: "list"
  ordered: boolean?
  definition: boolean?
  start: number?
  spread: boolean?
  children: [ListContent]
}
```

**List** ([**Parent**][dfn-parent]) represents a list of items.

**List** can be used where [**flow**][dfn-flow-content] content is expected.
Its content model is [**list**][dfn-list-content] content.

An `ordered` field can be present.
It represents that the items have been intentionally ordered (when `true`), or
that the order of items is not important (when `false` or not present).

A `definition` field can be present.
It represents that items represent a `definition` list. 

A `start` field can be present.
It represents, when the `ordered` field is `true`, the starting number of the
list.

A `spread` field can be present.
It represents that one or more of its children are separated with a blank line
from its [siblings][term-sibling] (when `true`), or not (when `false` or not
present).

For example, the following wikitext:

```wikitext
# foo
```

Yields:

```js
{
  type: 'list',
  ordered: true,
  definition: false,
  start: 1,
  spread: false,
  children: [{
    type: 'listItem',
    spread: false,
    children: [{
      type: 'paragraph',
      children: [{type: 'text', value: 'foo'}]
    }]
  }]
}
```

### `ListItem`

```idl
interface ListItem <: Parent {
  type: "listItem"
  spread: boolean?
  children: [FlowContent]
}
```

**ListItem** ([**Parent**][dfn-parent]) represents an item in a
[**List**][dfn-list].

**ListItem** can be used where [**list**][dfn-list-content] content is expected.
Its content model is [**flow**][dfn-flow-content] content.

A `spread` field can be present.
It represents that the item contains two or more [*children*][term-child]
separated by a blank line (when `true`), or not (when `false` or not present).

For example, the following wikitext:

```markdown
* bar
```

Yields:

```js
{
  type: 'listItem',
  spread: false,
  children: [{
    type: 'paragraph',
    children: [{type: 'text', value: 'bar'}]
  }]
}
```

### `HTML`

```idl
interface HTML <: Literal {
  type: "html"
}
```

**HTML** ([**Literal**][dfn-literal]) represents a fragment of raw [HTML][].

**HTML** can be used where [**flow**][dfn-flow-content] or
[**phrasing**][dfn-phrasing-content] content is expected.
Its content is represented by its `value` field.

HTML nodes do not have the restriction of being valid or complete HTML
([\[HTML\]][html]) constructs.

For example, the following wikitext:

```wikitext
<div>
```

Yields:

```js
{type: 'html', value: '<div>'}
```

### `Code`

```idl
interface Code <: Literal {
  type: "code"
  lang: string?
  meta: string?
}
```

**LeadingSpaceBlock** ([**Literal**][dfn-literal]) represents a block of monospaced text that preserves the original formatting but converts links such.

**LeadingSpaceBlock** can be used where [**flow**][dfn-flow-content] content is expected.
Its content is represented by its `value` field.

For example, the following wikitext:

```markdown
 This is preceded by a space.
 As is this.
```

Yields:

```js
{
  type: 'leadingSpaceBlock',
  lang: null,
  meta: null,
  value: 'This is preceded by a space.\nAs is this'
}
```

### `Text`

```idl
interface Text <: Literal {
  type: "text"
}
```

**Text** ([**Literal**][dfn-literal]) represents everything that is just text.

**Text** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content is represented by its `value` field.

For example, the following wikitext:

```wikitext
Alpha bravo charlie.
```

Yields:

```js
{type: 'text', value: 'Alpha bravo charlie.'}
```

### `Emphasis`

```idl
interface Emphasis <: Parent {
  type: "emphasis"
  children: [TransparentContent]
}
```

**Emphasis** ([**Parent**][dfn-parent]) represents stress emphasis of its
contents.

**Emphasis** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is [**transparent**][dfn-transparent-content] content.

For example, the following wikitext:

```wikitext
''alpha'' ''bravo''
```

Yields:

```js
{
  type: 'paragraph',
  children: [
    {
      type: 'emphasis',
      children: [{type: 'text', value: 'alpha'}]
    },
    {type: 'text', value: ' '},
    {
      type: 'emphasis',
      children: [{type: 'text', value: 'bravo'}]
    }
  ]
}
```

### `Strong`

```idl
interface Strong <: Parent {
  type: "strong"
  children: [TransparentContent]
}
```

**Strong** ([**Parent**][dfn-parent]) represents strong importance, seriousness,
or urgency for its contents.

**Strong** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is [**transparent**][dfn-transparent-content] content.

For example, the following wikitext:

```wikitext
'''alpha''' '''''bravo'''''
```

Yields:

```js
{
  type: 'paragraph',
  children: [
    {
      type: 'strong',
      children: [{type: 'text', value: 'alpha'}]
    },
    {type: 'text', value: ' '},
    {
      type: 'strong',
      children: [
        {
          type: 'emphasis',
          children: [{type: 'text', value: 'bravo'}]
        }
      ]
    }
  ]
}
```


### `Break`

```idl
interface Break <: Node {
  type: "break"
}
```

**Break** ([**Node**][dfn-node]) represents a line break, such as in poems or
addresses.

**Break** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
It has no content model.

For example, the following wikitext:

```markdown
foo··
bar
```

Yields:

```js
{
  type: 'paragraph',
  children: [
    {type: 'text', value: 'foo'},
    {type: 'break'},
    {type: 'text', value: 'bar'}
  ]
}
```

### `Link`

```idl
interface Link <: Parent {
  type: "link"
  children: [StaticPhrasingContent]
}

Link includes Resource
```

**Link** ([**Parent**][dfn-parent]) represents a hyperlink.

**Link** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is [**static phrasing**][dfn-static-phrasing-content] content.

**Link** includes the mixin [**Resource**][dfn-mxn-resource].

For example, the following wikitext:

```markdown
[https://example.com "bravo")
```

Yields:

```js
{
  type: 'link',
  url: 'https://example.com',
  title: 'bravo'
}
```

### `Image`

```idl
interface Image <: Node {
  type: "image"
}

Image includes Resource
Image includes Alternative
```

**Image** ([**Node**][dfn-node]) represents an image.

**Image** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
It has no content model, but is described by its `alt` field.

**Image** includes the mixins [**Resource**][dfn-mxn-resource] and
[**Alternative**][dfn-mxn-alternative].

TODO: ...

### `LinkReference`

```idl
interface LinkReference <: Parent {
  type: "linkReference"
  children: [StaticPhrasingContent]
}

LinkReference includes Reference
```

**LinkReference** ([**Parent**][dfn-parent]) represents a hyperlink through
association, or its original source if there is no association.

**LinkReference** can be used where [**phrasing**][dfn-phrasing-content] content
is expected.
Its content model is [**static phrasing**][dfn-static-phrasing-content] content.

**LinkReference** includes the mixin [**Reference**][dfn-mxn-reference].

**LinkReferences** should be associated with a [**Definition**][dfn-definition].

TODO: ...

Yields:

```js
{
  type: 'linkReference',
  identifier: 'bravo',
  label: 'Bravo',
  referenceType: 'full',
  children: [{type: 'text', value: 'alpha'}]
}
```

### `ImageReference`

```idl
interface ImageReference <: Node {
  type: "imageReference"
}

ImageReference includes Reference
ImageReference includes Alternative
```

**ImageReference** ([**Node**][dfn-node]) represents an image through
association, or its original source if there is no association.

**ImageReference** can be used where [**phrasing**][dfn-phrasing-content]
content is expected.
It has no content model, but is described by its `alt` field.

**ImageReference** includes the mixins [**Reference**][dfn-mxn-reference] and
[**Alternative**][dfn-mxn-alternative].

**ImageReference** should be associated with a [**Definition**][dfn-definition].

For example, the following wikitext:

TODO: ...

Yields:

```js
{
  type: 'imageReference',
  identifier: 'bravo',
  label: 'bravo',
  referenceType: 'full',
  alt: 'alpha'
}
```

## Mixin

### `Resource`

```idl
interface mixin Resource {
  url: string
  title: string?
}
```

**Resource** represents a reference to resource.

A `url` field must be present.
It represents a URL to the referenced resource.

A `title` field can be present.
It represents  advisory information for the resource, such as would be
appropriate for a tooltip.

### `Association`

```idl
interface mixin Association {
  identifier: string
  label: string?
}
```

**Association** represents an internal relation from one node to another.

An `identifier` field must be present.
It can match another node.
`identifier` is a source value: character escapes and character references are
*not* parsed.
Its value must be normalized.

A `label` field can be present.
`label` is a string value: it works just like `title` on a link or a `lang` on
code: character escapes and character references are parsed.

To normalize a value, collapse markdown whitespace (`[\t\n\r ]+`) to a space,
trim the optional initial and/or final space, and perform case-folding.

Whether the value of `identifier` (or normalized `label` if there is no
`identifier`) is expected to be a unique identifier or not depends on the type
of node including the **Association**.
An example of this is that they should be unique on
[**Definition**][dfn-definition], whereas multiple
[**LinkReference**][dfn-link-reference]s can be non-unique to be associated with
one definition.

### `Reference`

```idl
interface mixin Reference {
  referenceType: string
}

Reference includes Association
```

**Reference** represents a marker that is [**associated**][dfn-mxn-association]
to another node.

A `referenceType` field must be present.
Its value must be a [**referenceType**][dfn-enum-reference-type].
It represents the explicitness of the reference.

### `Alternative`

```idl
interface mixin Alternative {
  alt: string?
}
```

**Alternative** represents a node with a fallback

An `alt` field should be present.
It represents equivalent content for environments that cannot represent the
node as intended.

## Enumeration

### `referenceType`

```idl
enum referenceType {
  "shortcut" | "collapsed" | "full"
}
```

**referenceType** represents the explicitness of a reference.

*   **shortcut**: the reference is implicit, its identifier inferred from its
    content
*   **collapsed**: the reference is explicit, its identifier inferred from its
    content
*   **full**: the reference is explicit, its identifier explicitly set

## Content model

```idl
type MdastContent = FlowContent | ListContent | PhrasingContent
```

Each node in wtast falls into one or more categories of **Content** that group
nodes with similar characteristics together.

### `FlowContent`

```idl
type FlowContent =
  Blockquote | Code | Heading | HTML | List | ThematicBreak | Content
```

**Flow** content represent the sections of document.

### `Content`

```idl
type Content = Definition | Paragraph
```

**Content** represents runs of text that form definitions and paragraphs.

### `ListContent`

```idl
type ListContent = ListItem
```

**List** content represent the items in a list.

### `PhrasingContent`

```idl
type PhrasingContent = Link | LinkReference | StaticPhrasingContent
```

**Phrasing** content represent the text in a document, and its markup.

### `StaticPhrasingContent`

```idl
type StaticPhrasingContent =
  Break | Emphasis | HTML | Image | ImageReference | InlineCode | Strong | Text
```

**StaticPhrasing** content represent the text in a document, and its
markup, that is not intended for user interaction.

### `TransparentContent`

The **transparent** content model is derived from the content model of its
[parent][dfn-parent].
Effectively, this is used to prohibit nested links (and link references).


#### `FootnoteDefinition`

```idl
interface FootnoteDefinition <: Parent {
  type: "footnoteDefinition"
  children: [FlowContent]
}

FootnoteDefinition includes Association
```

**FootnoteDefinition** ([**Parent**][dfn-parent]) represents content relating
to the document that is outside its flow.

**FootnoteDefinition** can be used where [**flow**][dfn-flow-content] content is
expected.
Its content model is also [**flow**][dfn-flow-content] content.

**FootnoteDefinition** includes the mixin
[**Association**][dfn-mxn-association].

**FootnoteDefinition** should be associated with
[**FootnoteReferences**][dfn-footnote-reference].

For example, the following wikitext:

TODO: ...

Yields:

```js
{
  type: 'footnoteDefinition',
  identifier: 'alpha',
  label: 'alpha',
  children: [{
    type: 'paragraph',
    children: [{type: 'text', value: 'bravo and charlie.'}]
  }]
}
```

#### `FootnoteReference`

```idl
interface FootnoteReference <: Node {
  type: "footnoteReference"
}

FootnoteReference includes Association
```

**FootnoteReference** ([**Node**][dfn-node]) represents a marker through
association.

**FootnoteReference** can be used where [**phrasing**][dfn-phrasing-content]
content is expected.
It has no content model.

**FootnoteReference** includes the mixin [**Association**][dfn-mxn-association].

**FootnoteReference** should be associated with a
[**FootnoteDefinition**][dfn-footnote-definition].

For example, the following wikitext:

TODO: ...

Yields:

```js
{
  type: 'footnoteReference',
  identifier: 'alpha',
  label: 'alpha'
}
```

#### `Table`

```idl
interface Table <: Parent {
  type: "table"
  align: [alignType]?
  children: [TableContent]
}
```

**Table** ([**Parent**][dfn-parent]) represents two-dimensional data.

**Table** can be used where [**flow**][dfn-flow-content] content is expected.
Its content model is [**table**][dfn-table-content] content.

The [*head*][term-head] of the node represents the labels of the columns.

An `align` field can be present.
If present, it must be a list of [**alignType**s][dfn-enum-align-type].
It represents how cells in columns are aligned.

For example, the following wikitext:

TODO: ...

Yields:

```js
{
  type: 'table',
  align: ['left', 'center'],
  children: [
    {
      type: 'tableRow',
      children: [
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'foo'}]
        },
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'bar'}]
        }
      ]
    },
    {
      type: 'tableRow',
      children: [
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'baz'}]
        },
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'qux'}]
        }
      ]
    }
  ]
}
```

#### `TableRow`

```idl
interface TableRow <: Parent {
  type: "tableRow"
  children: [RowContent]
}
```

**TableRow** ([**Parent**][dfn-parent]) represents a row of cells in a table.

**TableRow** can be used where [**table**][dfn-table-content] content is
expected.
Its content model is [**row**][dfn-row-content] content.

If the node is a [*head*][term-head], it represents the labels of the columns
for its parent [**Table**][dfn-table].

For an example, see [**Table**][dfn-table].

#### `TableCell`

```idl
interface TableCell <: Parent {
  type: "tableCell"
  children: [PhrasingContent]
}
```

**TableCell** ([**Parent**][dfn-parent]) represents a header cell in a
[**Table**][dfn-table], if its parent is a [*head*][term-head], or a data
cell otherwise.

**TableCell** can be used where [**row**][dfn-row-content] content is expected.
Its content model is [**phrasing**][dfn-phrasing-content] content excluding
[**Break**][dfn-break] nodes.

For an example, see [**Table**][dfn-table].

#### `ListItem` (GFM)

```idl
interface ListItemGfm <: ListItem {
  checked: boolean?
}
```

In GFM, a `checked` field can be present.
It represents whether the item is done (when `true`), not done (when `false`),
or indeterminate or not applicable (when `null` or not present).

#### `FlowContent` (GFM)

```idl
type FlowContentGfm = FootnoteDefinition | Table | FlowContent
```

#### `TableContent`

```idl
type TableContent = TableRow
```

**Table** content represent the rows in a table.

#### `RowContent`

```idl
type RowContent = TableCell
```

**Row** content represent the cells in a row.

#### `ListContent` (GFM)

```idl
type ListContentGfm = ListItemGfm
```

#### `StaticPhrasingContent` (GFM)

```idl
type StaticPhrasingContentGfm =
  FootnoteReference | Delete | StaticPhrasingContent
```

#### `FlowContent` (frontmatter)

```idl
type FlowContentFrontmatter = FrontmatterContent | FlowContent
```

### Footnotes

The following interfaces are found with footnotes (pandoc).
Note that pandoc also uses [**FootnoteReference**][dfn-footnote-reference]
and [**FootnoteDefinition**][dfn-footnote-definition], but since
[GFM now supports footnotes][gfm-footnote], their definitions were moved to the
[GFM][gfm-section] section

#### `Footnote`

```idl
interface Footnote <: Parent {
  type: "footnote"
  children: [PhrasingContent]
}
```

**Footnote** ([**Parent**][dfn-parent]) represents content relating to the
document that is outside its flow.

**Footnote** can be used where [**phrasing**][dfn-phrasing-content] content is
expected.
Its content model is also [**phrasing**][dfn-phrasing-content] content.

For example, the following wikitext:

TODO: ...

Yields:

```js
{
  type: 'footnote',
  children: [{type: 'text', value: 'alpha bravo'}]
}
```

#### `StaticPhrasingContent` (footnotes)

```idl
type StaticPhrasingContentFootnotes = Footnote | StaticPhrasingContent
```
## Glossary

See the [unist glossary][glossary].

## List of utilities

See the [unist list of utilities][utilities] for more utilities.

## References
*   **unist**:
    [Universal Syntax Tree][unist].
    T. Wormer; et al.
*   **mdast**:
    [Markdown Abstract Syntax Tree][mdast].
    T. Wormer; et al.
*   **Markdown**:
    [Markdown][].
    J. Gruber.
*   **CommonMark**:
    [CommonMark][].
    J. MacFarlane; et al.
*   **GFM**:
    [GitHub Flavored Markdown][gfm].
    GitHub.
*   **HTML**:
    [HTML Standard][html],
    A. van Kesteren; et al.
    WHATWG.
*   **CSSTEXT**:
    [CSS Text][css-text],
    CSS Text, E. Etemad, K. Ishii.
    W3C.
*   **JavaScript**:
    [ECMAScript Language Specification][javascript].
    Ecma International.
*   **YAML**:
    [YAML Ain’t Markup Language][yaml],
    O. Ben-Kiki, C. Evans, I. döt Net.
*   **Web IDL**:
    [Web IDL][webidl],
    C. McCormack.
    W3C.

## Security

As wtast can contain HTML and be used to represent HTML, and improper use of
HTML can open you up to a [cross-site scripting (XSS)][xss] attack, improper use
of wtast is also unsafe.

When transforming to HTML (typically through [**hast**][hast]), always be
careful with user input and use [`hast-util-santize`][sanitize] to make the hast
tree safe.

## Related

*   [mdast]()
    - Markdown Abstract Syntax Tree format
*   [hast](https://github.com/syntax-tree/hast)
    — Hypertext Abstract Syntax Tree format
*   [nlcst](https://github.com/syntax-tree/nlcst)
    — Natural Language Concrete Syntax Tree format
*   [xast](https://github.com/syntax-tree/xast)
    — Extensible Abstract Syntax Tree

## Acknowledgments

This is based (much of it verbatim) on [mdast](). A huge shoutout to everyone on [this list](https://github.com/syntax-tree/mdast#Acknowledgments)

## License

[CC-BY-4.0][license] © [Jesse Hoogland][author]

<!-- Definitions -->

[mdast]: https://github.com/syntax-tree/mdast

[health]: https://github.com/syntax-tree/.github

[contributing]: https://github.com/syntax-tree/.github/blob/HEAD/contributing.md

[support]: https://github.com/syntax-tree/.github/blob/HEAD/support.md

[coc]: https://github.com/syntax-tree/.github/blob/HEAD/code-of-conduct.md

[awesome]: https://github.com/syntax-tree/awesome-syntax-tree

[ideas]: https://github.com/syntax-tree/ideas

[license]: https://creativecommons.org/licenses/by/4.0/

[author]: https://wooorm.com

[logo]: logo.svg

[releases]: https://github.com/syntax-tree/mdast/releases

[latest]: https://github.com/syntax-tree/mdast/releases/tag/4.0.0

[dfn-node]: https://github.com/syntax-tree/unist#node

[dfn-unist-parent]: https://github.com/syntax-tree/unist#parent

[dfn-unist-literal]: https://github.com/syntax-tree/unist#literal

[dfn-parent]: #parent

[dfn-literal]: #literal

[dfn-code]: #code

[dfn-inline-code]: #inlinecode

[dfn-list]: #list

[dfn-table]: #table

[dfn-break]: #break

[dfn-link-reference]: #linkreference

[dfn-image-reference]: #imagereference

[dfn-footnote-reference]: #footnotereference

[dfn-definition]: #definition

[dfn-footnote-definition]: #footnotedefinition

[term-tree]: https://github.com/syntax-tree/unist#tree

[term-child]: https://github.com/syntax-tree/unist#child

[term-sibling]: https://github.com/syntax-tree/unist#sibling

[term-root]: https://github.com/syntax-tree/unist#root

[term-head]: https://github.com/syntax-tree/unist#head

[dfn-mxn-resource]: #resource

[dfn-mxn-association]: #association

[dfn-mxn-reference]: #reference

[dfn-mxn-alternative]: #alternative

[dfn-enum-align-type]: #aligntype

[dfn-enum-reference-type]: #referencetype

[dfn-wtast-content]: #content-model

[dfn-flow-content]: #flowcontent

[dfn-frontmatter-content]: #frontmattercontent

[dfn-content]: #content

[dfn-list-content]: #listcontent

[dfn-table-content]: #tablecontent

[dfn-row-content]: #rowcontent

[dfn-phrasing-content]: #phrasingcontent

[dfn-static-phrasing-content]: #staticphrasingcontent

[dfn-transparent-content]: #transparentcontent

[gfm-section]: #gfm

[gfm-footnote]: https://github.blog/changelog/2021-09-30-footnotes-now-supported-in-markdown-fields/

[list-of-utilities]: #list-of-utilities

[unist]: https://github.com/syntax-tree/unist

[syntax-tree]: https://github.com/syntax-tree/unist#syntax-tree

[yaml]: https://yaml.org

[html]: https://html.spec.whatwg.org/multipage/

[css-text]: https://drafts.csswg.org/css-text/

[css-left]: https://drafts.csswg.org/css-text/#valdef-text-align-left

[css-right]: https://drafts.csswg.org/css-text/#valdef-text-align-right

[css-center]: https://drafts.csswg.org/css-text/#valdef-text-align-center

[javascript]: https://www.ecma-international.org/ecma-262/9.0/index.html

[webidl]: https://heycam.github.io/webidl/

[markdown]: https://daringfireball.net/projects/markdown/

[commonmark]: https://commonmark.org

[gfm]: https://github.github.com/gfm/

[glossary]: https://github.com/syntax-tree/unist#glossary

[utilities]: https://github.com/syntax-tree/unist#list-of-utilities

[unified]: https://github.com/unifiedjs/unified

[remark]: https://github.com/remarkjs/remark

[xss]: https://en.wikipedia.org/wiki/Cross-site_scripting

[hast]: https://github.com/syntax-tree/hast

[sanitize]: https://github.com/syntax-tree/hast-util-sanitize

[wikitext]: https://en.wikipedia.org/wiki/Help:Wikitext