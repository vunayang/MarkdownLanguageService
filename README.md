Markdown Language Service for Visual Studio
===

This project adds support for Markdown **editing** and **live preview** in Visual Studio.

## Hosting Markdown preview

You may embed the Markdown preview in your component:

1. Add reference to the [Microsoft.VisualStudio.Markdown.Platform](https://devdiv.visualstudio.com/DevDiv/_artifacts/feed/vs-impl/NuGet/Microsoft.VisualStudio.Markdown.Platform) package
1. Create instance of `Microsoft.VisualStudio.Markdown.Platform.PreviewBuilder`
1. Set its properties based on your needs; Markdown content may be provided through the `TextBuffer` property, or set with the `UpdateContentAsync` call.
1. Call `Build()` which returns `IMarkdownPreview` instance. Place its `VisualElement` within your WPF tree.

See documentation of `PreviewBuilder` for more details

Example usage for preview synchronized with a text editor:
```
var markdownPreview = new PreviewBuilder()
{
    JoinableTaskContext = this.joinableTaskContext,
    TextBuffer = this.textView.TextBuffer,
    TextDocumentFactoryService = this.textDocumentFactoryService,
    TextView = this.textView
}
.Build();
hostControl.Child = markdownPreview.VisualElement;
```

Example usage for preview of arbitrary document using custom rendering rules:
```
var markdownPreview = new PreviewBuilder()
{
    JoinableTaskContext = this.joinableTaskContext,
    DocumentPath = @"C:\temp\document.md", // This is used only to facilitate relative links. This does not open the file.
    MarkdownPipeline = customMarkdownPipeline
}
.Build();
// Set custom content using UpdateContentAsync
markdownPreview.UpdateContentAsync(System.IO.File.ReadAllText(@"C:\temp\document.md"), ScrollHint.None);
hostControl.Child = markdownPreview.VisualElement;
```

---

## Supported features

Preview

unicode Ⓐℬć and emoji 🚀👍

Typing improvements for lists, such as:

Unordered lists
- one
- two
    - Nested item one
         - Nested item two
-three

Ordered lists
1. One
2. Two
3. Three

Task list
- [ ] one
- [x] two
- [x] three


### Highlighted tags

Formatted text, including *emphasis*, **bold**, ~~strikethrough~~, ^superscript^,  ~subscript~

### Links

- [Header](#supported-features) and [another header](#coming-soon)
- [External URI](https://en.wikipedia.org/wiki/Markdown) - opens in web browser
- [Relative file path](Directory.Build.props) and [another one](impl\Markdown.Platform\MarkdownPackage.cs)
- ![Image](https://upload.wikimedia.org/wikipedia/commons/4/48/Markdown-mark.svg)

> blockquote may extend multiple lines
it continues until it reaches
the end of the paragraph

> Another quoted paragraph must begin
with its own blockquote symbol

<!---
html style comment
-->

`inline code`

```csharp
while(true)
{
    Console.WriteLine("Sample Text");
}
```

HTML <b>elements</b> such as <script>javascript</script> and <pre>other html elements</pre> <br />

---
Horizontal rule

## Not implemented

Options to configure behavior of editor and preview

Smart indent

Brace completion and overtyping

Structure Guides

## Spellchecking

Language service performs spellchecking of literal spans (atypo) as well as *various atypo* **emphasis atypo** ~~spans atypo~~ ^atypo^
it also checks within [link labels atypo](atypo) and

> blockquotes with atypo

but does not check within `inline code blocks atypo`, <a href="atypo">html code</a>, [link targets](atypo) yaml front matter as well as

```
multiline code blocks atypo
```

