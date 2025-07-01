<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/portabletext/portabletext/master/logo-white.svg?sanitize=true">
  <img alt="Portable Text Logo" src="https://raw.githubusercontent.com/portabletext/portabletext/master/logo.svg?sanitize=true">
</picture>

<div align="center">

<h1> Specification for Portable Text</h1>

<p>v0.0.1 - WORKING DRAFT</p>

</div>

> Portable Text is a JSON based rich text specification for modern content editing platforms.

Portable Text is an agnostic abstraction of rich text that can be serialized into pretty much any markup language, be it HTML, Markdown, SSML, XML, etc. It's designed to be efficient for real-time collaborative interfaces, and makes it possible to annotate rich text with additional data structures recursively.

Portable Text is built on the idea of rich text as an array of blocks, themselves arrays of children spans. Each block can have a style and a set of mark definitions, which describe data structures distributed on the children spans. Portable Text also allows for inserting arbitrary data objects in the array, only requiring `_type`-key. Portable Text also allows for custom content objects in the root array, enabling editing- and rendering environments to mix rich text with custom content types.

Example of a text block with some text annotated with some data:

```json
{
  "style": "normal",
  "_type": "block",
  "children": [
    {
      "_type": "span",
      "marks": ["a-key", "emphasis"],
      "text": "some text"
    }
  ],
  "markDefs": [
    {
      "_key": "a-key",
      "_type": "markType",
      "extraData": "some data"
    }
  ]
}
```

## Anatomy

Portable Text is a recursive composition of arrays and objects. In its simplest form it's an array of objects of a `type` with an array of `children`.

### Block

A block is what's typically recognized as a section of a text, e.g. a paragraph or a heading.

#### children (array)

Children is an array of spans or custom inline types that is contained within a block.

#### \_type (string)

All blocks must be of a type. The type makes it possible for a serializer to parse the contents of the block.

#### style (string)

Style typically describes a visual property for the whole block.

```json
{
  "style": "h1",
  "_type": "block",
  "children": [
    {
      "_type": "span",
      "text": "This is a heading"
    }
  ]
}
```

#### markDefs (array)

Mark definitions is an array of objects with a key, type and some data. Mark definitions are tied to `spans` by adding the referring `_key` in the `marks` array.

```json
{
  "markDefs": [
    {
      "_key": "some-random-key",
      "_type": "link",
      "href": "https://portabletext.org"
    },
    {
      "_key": "some-other-random-key",
      "_type": "comment",
      "text": "Change to https://",
      "author": "some-author-id"
    }
  ]
}
```

#### listItem

A block can be given the property `listItem` with a value that describes which kind of list it is. Typically `bullet`, `number`, `square` and so on. The list position is derived from the position the block has in the array and surrounding list items on the same level.

#### level

Level is used to express visual nesting and hierarchical structures between blocks in the array.

### span

A span is the standard way to express inline text within a block

#### \_type (string)

All children must be typed. The type makes it possible for a serializer to parse the contents of the child.

#### marks (array)

Marks are how we mark up inline text with additional data/features. Marks comes in two forms: Decorators and Annotations. Decorators are marks as simple string values, while Annotations are keys to a data structure. `marks` is therefore either an array of string values, or keys, corresponding to `markDefs`, with the same `_key`. See the examples below, or check out this [live CodeSandbox example with an inline image and a link](https://codesandbox.io/s/awesome-sammet-71gy8?file=/src/App.js):

Decorator example:

```json
[
  {
    "_type": "span",
    "marks": [],
    "text": "This is"
  },
  {
    "_type": "span",
    "marks": ["emphasis"],
    "text": "important"
  }
]
```

Annotation example:

```json
[
  {
    "_type": "block",
    "children": [
      {
        "_type": "span",
        "text": "Portable Text",
        "marks": ["<markDefId>"] // this corresponds to a `"_key"` in `markDefs`
      }
    ],
    "markDefs": [
      {
        "_type": "link",
        "_key": "<markDefId>", // this corresponds to a value in `children.marks`
        "href": "https://www.portabletext.org"
      }
    ]
  }
]
```

#### Text (string)

The text contents of the span.

### Custom blocks

Custom blocks are typically images (for inline images, see the marks section), code blocks, tables, video embeds, or any data structure. Custom blocks should be given a `_type`.

Examples of custom blocks:

```json
{
  "_type": "image",
  "asset": {
    "_ref": "some-asset-reference"
  }
}
```

```json
{
  "_type": "code",
  "language": "javascript",
  "code": "console.log(\"hello world\");"
}
```

## Serialization

### Tools

#### JavaScript

- [@portabletext/react](https://github.com/portabletext/react-portabletext)
- [@portabletext/react-pdf](https://github.com/portabletext/react-pdf-portabletext)
- [@portabletext/react-native](https://github.com/portabletext/react-native-portabletext)
- [@portabletext/svelte](https://github.com/portabletext/svelte-portabletext/)
- [@portabletext/vue](https://github.com/portabletext/vue-portabletext/)
- [@portabletext/to-html](https://github.com/portabletext/to-html/)
- [@portabletext/block-tools](https://www.npmjs.com/package/%40portabletext/block-tools)
- [@limitless-angular/sanity/portable-text](https://github.com/limitless-angular/limitless-angular/blob/main/libs/sanity/portabletext/README.md)
- [@sanity/block-content-to-hyperscript](https://www.npmjs.com/package/%40sanity/block-content-to-hyperscript)
- [@sanity/block-content-to-markdown](https://www.npmjs.com/package/%40sanity/block-content-to-markdown)
- [theisel/astro-portabletext](https://github.com/theisel/astro-portabletext)

#### PHP

- The `Sanity\BlockContent` class in [the sanity-php client](https://github.com/sanity-io/sanity-php)

#### Python

- [portabletext-html](https://github.com/otovo/python-portabletext-html)

#### C#

- The `SanityHtmlBuilder` class in [the Sanity LINQ client](https://github.com/oslofjord/sanity-linq)
- [Portable Text .NET](https://github.com/nhi/portable-text-dotnet), a C# HTML converter to Portable Text, also supporting custom types. (A graceful port of [`blockTools.htmlToBlocks()`](https://www.npmjs.com/package/@portabletext/block-tools#htmltoblockshtml-blockcontenttype-options-html-deserializer) to C#)

#### Dart/Flutter

- [Flutter Widget for Portable Text](https://pub.dev/packages/flutter_sanity_portable_text): Supports the entire spec of Portable Text with an easy to use Widget. It also includes support for **custom** blocks, marks, gestures, styles in an extensible manner. Use this package in conjunction with the [Sanity Client](https://pub.dev/packages/sanity_client) package for deserializing Portable Text in documents from Sanity.
- [flutter_portabletext](https://github.com/JobiJoba/flutter_portabletext)

---

Portable Text is presented by [Sanity.io](https://github.com/sanity-io)
