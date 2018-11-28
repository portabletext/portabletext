# Specification for Portable Text

v.0.0.1 - WORKING DRAFT

> Portable Text is a JSON based rich text specification for modern content editing platforms.

Portable Text is an agnostic abstraction of rich text that can be serialized into pretty much any markup language, be it HTML, Markdown, SSML, XML, etc. It's designed to be efficiant for real-time collaborative interfaces, and makes it possible to annotate rich text with additional data structures recursively. 

Portable Text is built on the idea of rich text as an array of blocks, themselves arrays of children spans. Each block can have a style and a set of mark definitions, which describe data structures distributed on the children spans. Portable Text also allows for inserting arbitary data objects in the array, only requiring `_type`-key. Portable Text also allows for custom content objects in the root array, enabling editing- and rendering environments  to mix rich text with custom content types.

Example of a text block with some text annotated with some data:

```json
{
  "style": "normal",
  "_type": "block",
  "children": [
    {
      "_type": "span",
      "marks": [
        "a-key",
        "emphasis"
      ],
      "text": "some text"
    }
  ],
  "markDefs": [
    {
      "_key": "a-key",
      "_type": "markType",
      "extraData": "some data"
    }
  ],
}
```

## Anatomy

Portable Text is a recursive composition of arrays and objects. In its simplest form it's an array of objects of a `type` with an array of `children`. 

### Block

A block is what's typically recognized as a section of a text, e.g. a paragraph or a heading.

#### children (array)

Children is an array of spans or custom inline types that is contained within a block.

#### _type (string)

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

Mark definitions is an array of objects with a key, type and some data. Mark definitions are tied to `spans` by adding the refering `_key` in the `marks` array.

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

Level is used to express visual nesting and hiearchical structures between blocks in the array. 

### span

A span is the standard way to express inline text within a block

#### _type (string)

All children must be of a type. The type makes it possible for a serializer to parse the contents of the child.

#### marks (array)

An array of decorators or keys that corrensponds with those in the block mark definitions (markDefs). A decorator is a string that describes some arbitrary feature of the span:

```json
  [
    {
      "_type": "span",
      "marks": [],
      "text": "This is"
    },
    {
      "_type": "span",
      "marks": [
        "emphasis"
      ],
      "text": "important"
    }
  ]
```

#### Text (string)

The text contents of the span.

### Custom blocks

Custom blocks is typically images, code blocks, tables, video embeds, or any data structure. Custom blocks should be given a `_type`. 

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

* [block-content-toolkit](https://github.com/sanity-io/sanity/tree/next/packages/%40sanity/block-tools)
* [block-content-to-hyperscript](https://github.com/sanity-io/block-content-to-hyperscript)
* [block-content-to-html](https://github.com/sanity-io/block-content-to-html)
* [block-content-to-react](https://github.com/sanity-io/block-content-to-react)
* [block-content-to-markdown](https://github.com/sanity-io/block-content-to-markdown)
* [block-content-to-vue](https://github.com/rdunk/sanity-blocks-vue-component)

#### PHP

* The `Sanity\BlockContent` class in [the sanity-php client](https://github.com/sanity-io/sanity-php)

#### C#

* The `SanityHtmlBuilder` class in [the Sanity LINQ client](https://github.com/oslofjord/sanity-linq)
