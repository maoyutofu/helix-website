+++
title = "Release 25.07 Highlights"
date = 2025-07-15T00:01:00Z
type = "post"
description = "25.07 版本的主要亮点。"
in_search_index = true
+++

期待已久的 25.07 版本终于发布了！此次版本更新替换了 Helix 的一个主要核心组件，并新增
了许多引人注目的功能。共有 195 位贡献者参与了此次版本更新。衷心感谢所有为此次发布做出贡献的人！

您是 Helix 的新用户吗？Helix 是一款模式文本编辑器，内置支持多重选择、语言服务器协议 (LSP)、
Tree-sitter，并提供对调试适配器协议 (DAP) 的实验性支持。

请做好准备；接下来的版本说明会涉及一些技术细节，我们将介绍我们新的 Tree-sitter 绑定库 `tree-house`。
在深入探讨技术细节之前，让我们先来看看一些更亮眼的功能。

## 文件资源管理器

{{ asciinema(id="file-explorer", width="94", height="25") }}

25.07 版本新增了文件浏览器功能，可通过 `<space>e` 快捷键访问。该文件浏览器是一个“选择器”
（picker），是 Helix 的核心 UI 组件之一，类似于 [telescope](https://github.com/nvim-telescope/telescope.nvim)。
与其他大多数选择器一样，您可以在选项中进行模糊搜索。按下 Enter 键选择目录会打开该目录下的
新文件浏览器，选择文件则会打开该文件。这对于以层级结构查看目录非常有用。相比之下，常用的文
件浏览器（`<space>f`）会递归地显示目录内容。对于大型项目，这个新的文件浏览器可以提供更精确的文件浏览体验。

## LSP 文档颜色

语言服务器协议 (LSP) 规范的一个引人注目的功能是文档颜色请求（[Document Color Request](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#textDocument_documentColor)）。
此请求允许客户端（Helix）询问语言服务器（例如 `tailwindcss-language-server` 或 `vscode-css-language-server`）
文档中哪些范围对应于 RGB 颜色。

在 25.07 版本中，Helix 现在会向语言服务器请求文档颜色，并以内联方式显示颜色样本（一个小方框）。这与 LSP 的
内嵌提示功能（用于显示类型）类似，只不过这里显示的是颜色。

{{ asciinema(id="lsp-document-colors", width="94", height="25") }}

## 新的命令模式功能

命令模式（`:`）用于执行可输入的命令。例如，`:write` 就是一个可输入的命令，它可以接受一个可选参数。`:quit` 也是一个命令，它不接受任何参数。

命令模式的语法比较复杂。它应该足够简单，以便像 `:write path/to/doc.md` 这样的常用操作易于输入。但它也需要提供转义空格的机制，例如 `:write 'a b.txt'`。对于某些命令，拥有自定义或可扩展的语法也很有用，例如 `:run-shell-command <复杂的特定于 shell 的命令>`。

25.07 版本彻底重写了用于解析和表示参数以及提供命令行补全的所有代码。这修复了许多解析和补全方面的错误，例如尝试补全文件名中包含空格的文件，并引入了两个新功能：_flags_ 和 _expansions_。

### Flags

Flags 的功能与你在 shell 命令中传递的  Flags 类似。它们用于处理需要对命令行为进行细微修改的情况。

目前，flags 仅用于少数命令：`:write` 系列命令（所有以 `:write` 开头的命令）和 `:sort`。

25.07 版本移除了 `:rsort` 命令，并将其替换为 `:sort --reverse`，或简写为 `:sort -r`。此外，所有 `:write` 命令现在都接受 `--no-format` flag。通常情况下，如果当前文档已配置为自动格式化，你会希望对其进行格式化，但有时也需要将文件原样写入。这些都是使用  flags 的绝佳示例：你无需为了调整一些小细节而创建额外的命令。

Flags 会显示在可输入命令的信息框中，并且长版本（例如 `--reverse`）可以自动补全。

### Expansions

Expansions 功能引入了一种特殊的语法来插入值。这些 expansions 功能大多沿用了 [Kakoune](https://github.com/mawww/kakoune) 的扩展概念，并进行了一些细微的调整。

基于当前编辑器状态的变量可以写成 `%{variable_name}` 的形式。`%{buffer_name}` 会打印当前聚焦文档的名称（与状态栏中显示的名称一致），而 `%{cursor_line}` 会打印主光标所在行的行号（从 1 开始计数）。

可以使用 `%sh{...}` 扩展来执行 shell 命令。结合上述变量 expansions  和新的简单 `:echo` 命令（该命令会将内容打印到状态栏），以下命令可以将当前行的 git blame 信息打印到状态栏：

```
:echo %sh{git blame -L %{cursor_line},+1 %{buffer_name}}
```

变量名和 expansion 类型（例如用于 shell 命令的 `sh` 或用于 Unicode 的 `u`）都可以自动补全。

### 可扩展解析

最初重写此代码的原因是为了改进命令行解析方式。在 25.07 版本中，命令模式能够更好地解析和补全文件名，支持标志和扩展，并且允许在命令行解析过程中切换到其他解析方法。

可输入的命令 `:set-option` 和 `:toggle-option` 现在使用 [`serde_json` 的流式反序列化器](https://docs.rs/serde_json/latest/serde_json/struct.StreamDeserializer.html) 来解析复杂的配置值，例如列表。像 `:run-shell-command` 和 `:pipe` 这样的 shell 命令不再尝试解析命令名称之后的命令行部分。因此，您无需费力猜测如何同时满足 Helix 的解析规则和 shell 的解析规则——避免了令人头疼的引号转义问题。

## Tree-house

在此发布周期中，我们替换了用于与 Tree-sitter 交互的库，添加了全新构建的库，并移除了官方绑定以及 Helix 中的许多旧代码。

本文的其余部分将讨论 Tree-sitter 和 Tree-house 的详细信息。想要了解 Helix 25.01.1 版本以来的更多更改细节？请查看[更新日志]以获取完整的代码更改列表。

#### Tree-sitter

不熟悉 [Tree-sitter](https://github.com/tree-sitter/tree-sitter)？简而言之，它是一个用于生成和使用快速、容错的解析器的框架。您可以在 `grammar.js` 文件中通过 [语法 DSL](https://tree-sitter.github.io/tree-sitter/creating-parsers/2-the-grammar-dsl.html) 编写解析规则，然后使用 `tree-sitter` 命令行工具生成和测试解析器。

编辑器等工具可以使用您定义的解析器，结合 Tree-sitter C 库或特定语言的绑定，来解析语法树并对其进行操作。如何使用语法树完全取决于您的想象力！语言服务器可以使用 Tree-sitter 作为其解析器，像 [Difftastic](https://github.com/Wilfred/difftastic) 这样的差异比较工具可以生成语法感知的差异，像 [Codebook](https://github.com/blopker/codebook) 这样的语言服务器可以进行语法感知的拼写检查。甚至 [GitHub 也使用 Tree-sitter](https://docs.github.com/en/repositories/working-with-files/using-files/navigating-code-on-github) 进行代码导航和某些语言的语法高亮显示。

处理解析树的一个非常强大的工具是 _Tree-sitter 查询_。查询是一种对子树进行模式匹配并_捕获_节点以供将来使用的方法。对于编辑器，您可以使用一个通常称为 `highlights.scm` 的查询来捕获像 Rust 关键字这样的树节点，以便根据当前主题高亮显示节点的文本。

与语法树一样，查询的应用也仅受限于您的想象力。我们目前在 Helix 中使用查询来实现语法高亮、缩进和文本对象（识别函数、参数等）。未来，代码折叠、拼写检查和代码导航也可以使用 Tree-sitter 查询。

#### Helix 的历史

Helix 在其首次公开发布之前就依赖于 Tree-sitter 进行语法高亮，它通过官方的 Rust 绑定库（即 [`tree-sitter`](https://crates.io/crates/tree-sitter) crate）调用 C 库。`tree-sitter` crate 封装了 C 库，并且是相当底层的库。我们还需要一个高亮器，这由另一个 crate 提供：`tree-sitter-highlight`。

[`tree-sitter-highlight`](https://crates.crates.io/crates/tree-sitter-highlight) 提供了一个语法高亮器，它接受某种语言的查询和文档文本进行高亮，并且可以迭代生成高亮事件。Helix 可以在渲染可见文档时使用这些高亮迭代器。`tree-sitter-highlight` 可以开箱即用，对于我们简单的用例（例如一次性高亮整个文档），`tree-sitter-highlight` 就足够了。

`tree-sitter-highlight` 的问题在于它无法增量工作。创建一个新的高亮迭代器意味着需要完全重新解析文档并重新分析查询。这是浪费的，因为 Tree-sitter 可以重用查询。此外，Tree-sitter 的解析可以增量进行：您可以将旧的语法树提供给 Tree-sitter，它将更快地解析新版本的文档。

因此，Helix 早期的语法高亮器是 `tree-sitter-highlight` 的一个分支，灵感来自于 Tree-sitter 在 [Atom 编辑器](https://github.com/atom/atom) 中的使用，它将解析后的树（`Syntax` 类型）和 `tree_sitter::Query` 从高亮器中分离出来。理想情况下，我们希望有一天能将这个高亮器提取到它自己的 crate 中，以便与其他工具轻松共享。

然而，这个高亮器变得难以维护。修复长期存在的 bug 的工作量太大，或者与该高亮器的设计完全不兼容，而且代码难以理解。 #### 引入 Tree-house

在此版本中，我们用一个新的库 [`tree-house`](https://github.com/helix-editor/tree-house) 替换了之前的语法高亮组件。Tree-house 是我们基于早期语法高亮组件的经验从零开始编写的。

Tree-house 保留了那些行之有效的设计，例如将解析与查询分离，并在解析过程中确定注入点。同时，它也摒弃了那些效果不佳的设计，例如将高亮结果暴露为 `Iterator`。Tree-house 的代码被分解成更小、更易于理解的组件，并修复了我们之前无法解决的长期存在的 bug。Tree-house 还为未来的改进（例如并行解析）奠定了基础。

Tree-house 的主要优势在于它对一项名为“_injections_”的功能的强大处理能力。

#### Injections, a tree of trees

“_Injections_”是 `tree-sitter-highlight` 中的一个概念。注入查询用于捕获应该“切换”到另一种语言的节点。例如，在 Markdown 中，您可以使用如下所示的代码块：

    ```rust
    println!("Hello, world!")
    ```

您可能会期望代码块中的内容被高亮显示为 Rust 代码。其工作原理是：使用 Markdown Tree-sitter 
解析器解析此 Markdown 文档的全部文本。Markdown 的 `injections.scm` 查询会告诉 Helix，
此代码块的内容应被视为 Rust 代码。然后，Helix 会对文档的该范围运行 Rust Tree-sitter 解析
器，并创建语法树。

接下来，当需要高亮显示此文档时，Markdown 的 `highlights.scm` 查询会定义 Markdown 部分的
高亮规则。而当我们到达文档的 Rust“层”时，Rust 的 `highlights.scm` 会接管高亮显示工作。

虽然这是一个简单的示例，但注入功能即使在复杂情况下也能稳定运行。例如，此版本新增了对 Rust 文档
注释中 Markdown 注入的支持，从而实现了如下所示的深度嵌套注入：

```rust
/// A type that parses **stuff**
///
/// This is a doc comment, so it should have _Markdown_ highlighting
/// on top of the regular comment highlights.
///
/// # Heading 1
///
/// Know what we can do with Markdown? Inject Rust!
///
///     println!("Hello, world!");
pub struct Parser(/* ... */);
```

在这样的 Rust 文件中，覆盖整个文档的根层自然是 Rust 语言。然后，每个文档行注释（`///`）
后面的内容都会被解析为 Markdown 文档，这些内容会被“合并”处理——这意味着这些范围会被整体视
为一个 Markdown 层。而在其中嵌套的缩进代码块则会被视为代码围栏，形成另一个 Rust 层。

在内部，Tree-house 将这种“层”的概念表示为一棵树。整体的 `Syntax` 类型有一个根层，对应于
文件类型，并在其下包含所有注入的子层。子层本身也可以注入其他层，以此类推。因此，这些层构成了
一棵树。每个层都会被解析，从而拥有自己的语法树，形成一种“树中树”的结构。

#### Incremental injections

早在 [22.03 版本发布说明](@/news/2022-03-28-release-22.03-highlights.md) 中就讨论过注入功能，该版本增加了对“组合注入”的支持，例如 Markdown 注释。同年晚些时候发布的 [22.12 版本](@/news/2022-12-06-release-22.12-highlights.md) 引入了“增量注入”。这项改进减少了对包含大量注入的文档进行重新解析和重新运行注入查询所产生的冗余工作。切换到 Tree-house 后，增量注入功能得到了进一步增强，现在只有实际发生更改的注入层才会被重新解析，并且只有这些层的注入查询才会被重新运行。

为了更直观地理解其工作原理，想象一个大型 Markdown 列表。Markdown Tree-sitter 解析器实际上分为两部分：一部分用于块级语法，例如代码块；另一部分用于“inline”语法，例如粗体、斜体和 inline 代码。Markdown 解析器会在列表项等情况下注入“inline Markdown”解析器，因此，一个大型 Markdown 列表意味着每个列表项都会有数千个“inline”解析器的小型注入。

切换到 Tree-house 后，在大型列表中的某个列表项内进行编辑只会导致根层和被编辑的“inline”层被重新解析并重新运行注入查询——这是所需的最少工作量。

#### Locals

`tree-sitter-highlight` 中的另一个有用概念是“_locals_”（locals）。`locals.scm` 是一个查询文件，用
于标记那些应该将其高亮样式应用于同一 _scope_ 内所有后续“_references_”的节点。

想象一个简单的 Rust 函数：

```rust
fn add(a: usize, b: usize) -> usize {
    a + b
}
```

此函数的参数 `a` 和 `b` 应该被高亮显示为参数——高亮颜色可能因主题而异。为了跟踪这些信息，locals 查询会捕获函数参
数（例如 `a` 和 `b`）以及 _scope_（在本例中为函数体）的节点。_scope_ 内的任何  _references_（同样由 loc
als 查询捕获）都应该继
承其定义的语法高亮。

Tree-house 对局部变量的处理方式与众不同，解决了 Helix 中一个长期存在的 bug。由于语法高亮器仅对屏幕上可见的小范
围代码运行，因此当函数定义超出屏幕范围时，局部变量的语法高亮就会消失。

{{ asciinema(id="viewport-locals-before", width="94", height="25") }}

请注意，当参数超出可视范围时，参数 `slice`、`char_idx` 和 `n` 的参数高亮（下划线）会消失。

使用 Tree-house，变量定义会在解析时被追踪，并以树状结构（类似于注入）存储，以便快速查找。因此，代码的当前视
图不会产生任何影响。无论定义是否在可视范围内，参数都会被正确高亮显示。

{{ asciinema(id="viewport-locals-after", width="94", height="25") }}

现在，无论您在函数中执行到哪个位置，`slice`、`char_idx` 和 `n` 这三个参数都会保持高亮显示。

#### 全面支持注入

Tree-house 带来的一个显著的体验提升是，Tree-house 的 `Syntax` 类型（对应于已解析的文档）
提供了流畅处理注入的功能。`Syntax` 的结构是树状的，因此查找注入层的时间复杂度是对数级的，而
不是对所有层进行完整扫描。

在此基础上，`TreeCursor` 类型（与 Tree-sitter C 库中的 `TreeCursor` 类型类似）可以使用
与 Tree-sitter 的 `TreeCursor` API 几乎相同的 API 在注入层之间移动。新的 `QueryIter` 类
型允许在文档的所有注入层上运行任何查询，而不仅仅是高亮显示。

利用注入功能改进所有基于 Tree-sitter 的功能，将带来更一致的跨语言体验。HTML `<script>` 标签
内的注释标记和文本对象应该遵循 JavaScript 规则，而不是 HTML 规则。Markdown 代码块内的缩进应
该遵循您正在编写的语言的规则，而不是 Markdown 的规则。这些功能尚未合并或发布，但最终所有基于 
Tree-sitter 的 Helix 功能都应该像高亮显示一样保持一致的行为。

## 总结

以上是 25.07 版本的一些亮点，以及对我们 Tree-sitter 集成的深入介绍。查看完整的[更新日志]了解更多详情。

欢迎在 [Matrix 空间][matrix]讨论使用和开发方面的问题，并在 [GitHub 仓库][helix-git]关注 Helix 的开发进展。

[更新日志]: https://github.com/helix-editor/helix/blob/master/CHANGELOG.md#2507-2025-07-15
[helix-git]: https://github.com/helix-editor/helix/
[matrix]: https://matrix.to/#/#helix-community:matrix.org
