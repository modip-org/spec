# Formatted Text

In various fields of the index, authors may write formatted text.

For best interoperability and simplicity, we use an XML-based syntax which is similar to a subset of HTML.

Motivating example:
```xml
This is an <b>example</b> mod. The word "<b>example</b>" is written in bold.

<h1>Blocks in my mod</h1>
<ul>
<li><b>Dostuffulator</b> - it does stuff.</li>
<li><b>Nitroglycerine</b> - it <big>explodes</big> when you click on it!</li>
</ul>

<h1>Recipes</h1>
<b>Dostuffulator</b>:
<img id="1234"/>
<b>Nitroglycerine</b>:
<img id="2345"/>
```

Since XML likes to work on documents, not fragments, the whole text is implicitly surrounded by `<?xml version="1.0" encoding="utf-8"?><root>...</root>`.
The point is that this boilerplate should not be included in every formatted text field.

# Inline and block contexts

Formatted text occurs in two contexts: inline contexts and block contexts.

Inline contexts include summary blurbs, names of people, and generally short pieces of content that are embedded in a larger piece.
Inline contexts only allow simple per-character formatting, such as bold or coloured text.

Block contexts are generally complete documents with their own structure, such as project descriptions.
Block contexts allow document structure elements, such as headings, lists, and large elements such as images.

*Unless specified otherwise, all elements are only valid in block contexts*

# Elements

## Simple formatting

Allowed in inline contexts.

* `<b>...</b>` - text within is bold
* `<i>...</i>` - text within is italic
* `<u>...</u>` - text within is underlined
* `<s>...</s>` - text within is struck through
* `<tt>...</tt>` - text within is monospace. Some viewers may also change the background and add a border, to indicate a code block.
* `<col c="#rrggbb">...</col>` - coloured text. Some viewers may alter the colour, if it conflicts with the background for example.
* `<a href="...">...</a>` - hyperlink. URL must be absolute. Links might not be allowed in all contexts.

## Block formatting

* `<h1>...</h1>` up to h6 - heading levels
* `<ul>...</ul>` - bullet list, list items are in `<li>...</li>`
* `<ol>...</ol>` - numbered list, list items are in `<li>...</li>`
* `<details><summary>Recipes</summary>...</details>` - a foldable content section (a "spoiler"). `summary` is optional - if omitted, it may default to something like "click here to open" (viewer-dependent). Note: `summary` must be the first child of `details` - there must not even be whitespace in between.
* `<br/>`, `<p/>` - as in HTML. Note that `<p>` is an empty element, as it represents a paragraph break, not a paragraph.

Note that line breaks and paragraph breaks are not allowed in inline context.

## Embedded content

* `<media id="..."/>` - display an image or video from the project. All content within the element is ignored (even `<vis>`). (TODO: what does the ID refer to?)
* `<youtube id="..."/>` - youtube embed

Unlike in HTML, media elements are considered block elements (i.e. they interrupt a paragraph and display on their own).

## Special

Allowed in inline contexts.

* `<invis>...</invis>` - may appear in any context. No content within is output, but tags are still processed.
* `<vis>...</vis>` - may appear in any context, and cancels the effect of a surrounding `<invis>`.

These are intended to allow forward compatibility. For example, if a `<script>` element is ever added, documents with scripts should use `<invis><script>...</script></invis>` so that older viewers do not render the script as text.

# Future extensibility

If XML parsing succeeds, but the document contains unknown tags, the client should transparently process the contents of the tags - it should behave as if the unknown start and end tags were not there.

For example, if the `<img>` tag was not present in the first version of this specification, "alt text" could be done by placing the alt text inside the image tag. v1 viewers would display the alt text and ignore the image; v2 viewers would display the image and ignore the alt text.

Anything wrapped in `<invis>` is not sent to the output. However, tags are still processed. Future extensions may include tags that override the effect of `<invis>`.
Anything wrapped in `<vis>` is not sent to the output. However, tags are still processed. Future extensions may include tags that override the effect of `<vis>`.

`<vis>` inside `<img>`, for example, is not displayed.

# Namespaces

Tags normally do not have namespace prefixes.

Future extensions to the protocol may define additional namespaces.

The `v1` prefix is implicitly defined as the same namespace as the default namespace (as unprefixed tags). So `<v1:b>this text is bold</v1:b>`.
Parsers are currently not required to implement namespaces. However, they must recognize unprefixed tags and `v1`-prefixed tags equivalently,
and they must ignore unknown prefixes (`<v2:b>this text is not bold</v2:b>`).

The parser MAY implement this by actually declaring the namespace on the root element. The namespace URI is unspecified. Or it may simply recognize the prefix.

If a document wishes to use additional namespace prefixes (for a future extension), it cannot declare them on the `<root>` element
because it has no access to that element. However, it can get the same effect by wrapping the entire document in `<vis>`, and
declaring the prefixes on the `<vis>` tag.

# Limits, error handling, edge cases

If XML parsing fails, the text should not be displayed. Do not make a "best guess". This is for reliability - it ensures that the document is parsed the same way on every client. However, clients are not required to detect documents which violate the following recommended maximums:

Any piece of formatted text should not be longer than 1048575 bytes (encoded as UTF-8).
Elements should not be nested more than 499 levels deep (including the root).
Element and attribute names should not be longer than 100 characters. Namespace prefixes should not be longer than 100 characters. (This means a qualified name may be up to 201 characters).

Note that unknown tags do not cause a parsing failure. Parsing failure only relates to the overall XML syntax. A document may parse correctly, but contain unknown tags or attributes.

Clients MUST allow text to be written inside CDATA sections - that is, they MUST NOT simply ignore CDATA sections.

# Security considerations

Incorrect parsing of any kind should not create a security vulnerability - the worst-case scenario should be ugly output. (No matter what the incorrect parsing is, the attacker could just create a correct document that parses the same way as the incorrect one)

Clients may crash when processing very large or deeply nested documents.
Documents should not be longer than 1048575 bytes (raw text, i.e. not including the root element) or contain nesting more than 499 levels deep (including the root element).
Clients should reject these unless they have been verified to not cause a problem.

It is well-known that XML DOCTYPE declarations can be used for DoS (Billion Laughs attack) and for file retrieval (XML External Entity attack).
Parsers MUST NOT accept DOCTYPE declarations. An attacker cannot write a DOCTYPE declaration at the beginning of the document, but they can write one in the middle (which is invalid) - even at the root level by surrounding it in `</root>...<root>`. Implementors should verify that these declarations are ignored or result in parsing errors.

A malicious project description may contain a hyperlink to, for example, `http://192.168.1.1/cgi-bin/ResetToFactoryDefaults?confirm=yes` - clicking on this link could (depending on the router model) reset your home router. This is an Internet-wide problem - not specific to MODIP - which could be solved by the router vendor by requiring a CSRF token, or by requiring the HTTP POST method. Systems which display links:

* MAY blacklist RFC1918 and RFC3927 IP addresses, IPv6 link-local addresses, and so on.
* MAY blacklist certain URL schemes.
* MUST NOT blacklist the `http` and `https` URL schemes.
* MUST NOT blacklist port numbers. (Sensitive ports like 25 should already be blocked by the user's browser)
* MAY blacklist the `file` URL scheme. [Note: This differs from the download recommendations in format_spec.md]

[a similar paragraph also appears in format_spec.md - should we combine these and put it in security_considerations.md?]
