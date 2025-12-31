+++
title = "Release 25.01 Highlights"
date = 2025-01-03T00:01:00Z
type = "post"
description = "25.01 版本的主要亮点。"
in_search_index = true
+++

新年伊始，Helix 发布了新版本！欢迎使用 25.01 版本。这是一个规模庞大的版本，包含了来自 171 位
贡献者的诸多改进。
感谢所有为此次发布做出贡献的人！

初次接触 Helix？
Helix 是一款模式文本编辑器，内置支持多重选择、语言服务器协议 (LSP)、tree-sitter，并提供对调
试适配器协议 (DAP) 的实验性支持。

此次重大版本更新带来了许多重要的改进，让我们一起来看看其中的亮点。

## 代码补全

在 25.01 版本中，代码补全功能进行了两项重大更新。首先，代码补全现在支持路径：

{{ asciinema(id="path-completion", width="94", height="25") }}

Helix 现在在插入文本时可以检测到可能的路径，并根据当前工作目录建议绝对路径或相对路径。
这对于像 Nix 这样路径使用频繁的语言尤其有用。

路径补全功能尤其令人兴奋，因为它是第一个不受 LSP 驱动的补全功能。添加路径补全功能的代码
重构了部分代码库，这些代码库之前假定补全功能只能来自语言服务器。这应该会使将来添加新的补
全源变得更加容易。

另一个重大变化是代码片段补全功能的行为——目前仅由语言服务器提供——25.01 版本新增了对代码
片段制表位的支持。

{{ asciinema(id="snippet-tabstops", width="94", height="25") }}

在上面的录音中，rust-analyzer 为 `add` 函数建议了一段代码补全，其中包含一些函数参数的
占位符。以前，Helix 在按下 Enter 键接受补全时会清除这些占位符。现在，按下 Enter 键会将
光标跳转到第一个制表位。可以使用 Tab 键切换到下一个制表位。输入任何其他字符都会替换占位符
文本。

## 诊断


从历史上看，LSP 诊断信息一直显示在编辑器的右上角，并右对齐。虽然这种显示方式简单明了，
但在终端窗口较小或语言服务器发送的诊断消息较长时，可能会难以阅读。25.01 版本新增了一种
将诊断信息 _inline_ 显示的方式。

{{ asciinema(id="inline-diagnostics", width="94", height="25") }}

内联诊断功能利用内部虚拟文本系统在文档中诊断信息所在的位置渲染诊断结果。诊断消息会显示在
代码行的末尾或相应的代码行之间。

由于我们仍在调整显示效果并修复错误，因此内联诊断功能目前默认禁用，您可以通过以下配置启用它：

```toml
# ~/.config/helix/config.toml
[editor]
# Minimum severity to show a diagnostic after the end of a line:
end-of-line-diagnostics = "hint"

[editor.inline-diagnostics]
# Minimum severity to show a diagnostic on the primary cursor's line.
# Note that `cursor-line` diagnostics are hidden in insert mode.
cursor-line = "error"
# Minimum severity to show a diagnostic on other lines:
# other-lines = "error"
```

将 `end-of-line-diagnostics`、`cursor-line` 或 `other-lines` 设置为除 `"disable"` 
之外的任何值都可以启用内联诊断功能。

## 表格式选择器

选择器 UI 组件是 Helix 的核心组件：它是一种高效且易于阅读的方式，
可以帮助用户快速跳转到感兴趣的文件和位置，例如 LSP 诊断信息和
符号。在 25.01 版本中，选择器组件进行了重大改进，现在项目以表格
形式显示。

{{ asciinema(id="tabular-pickers", width="94", height="25") }}

像文件选择器（`<space>f`）这样只需要一列的简单选择器保持不变。而包含多
个信息字段的选择器现在会在结果窗格顶部显示列名。行仍然对应于要选择的各个项目，
但现在有了列，可以将不同项目之间相同类型的内容水平对齐。例如，诊断信息选择器
现在有三列——“严重性”、“代码”和“消息”——而 LSP 符号选择器有两列——“类型”和“名称”。

列可以单独过滤：默认情况下会过滤一列，其他列可以使用查询语法进行过滤：`%<列名> 过滤文本`。
例如，现在可以通过搜索 `%severity error` 将诊断信息选择器过滤到只显示错误。过滤文本支
持模糊匹配，并且可以使用前缀指定列名，因此在诊断信息选择器中搜索 `%s e` 的效果与搜
索 `%severity error` 相同。

{{ asciinema(id="interactive-global-search", width="94", height="25") }}

与此同时，`global_search` 命令（`<space>/`）也进行了重构，使其能够根据选择器的查询
动态更新。这使得可以交互式地在代码库中搜索正则表达式。然后，可以使用类似 `%path filename` 的
查询单独过滤文件名。

## 宏按键绑定

25.01 版本已初步支持以宏形式编写的按键绑定。宏按键绑定可以通过在输入字符串前加上“@”符号来编写。

```toml
# Change the `<space>y` keybinding to yank to the `a` register instead of the
# system clipboard:
[keys.normal]
space.y = "@\"ay"
```

宏键绑定使用与通过 `q` 录制并使用 `Q` 执行的宏相同的底层架构，因此上述操作的行为就如同您依
次输入了以下内容：

1. `"`：打开寄存器选择弹出窗口
2. `a`：选择“a”寄存器
3. `y`：将选定内容复制到该寄存器

宏弥补了键绑定方面的不足：像寄存器选择弹出窗口这样的组件的输入操作并未作为命令公开，因此像选
择寄存器这样的操作原本无法进行绑定。

宏键绑定的初始支持限制了其在命令序列中的使用：您目前还无法将宏与常规命令组合使用。例如，您可
以将上述示例中的 `y` 替换为其对应的命令 `yank`，形成类似 `["@\"a", "yank"]` 的序列，
但目前尚不支持这种用法。

## 注释

25.01 版本改进了评论功能，使编写行内注释（例如内嵌文档）的体验更加流畅。

{{ asciinema(id="25.01-comment-improvements", width="94", height="25") }}

现在，行注释默认可以自动延续：在插入模式下按 Enter 键，或者在普通模式下光标位于行注释上
时按 `o`/`O` 键，都会扩展该注释，并插入当前的注释标记前缀和末尾的空格。现在，按下 Enter 键
时会自动删除末尾的空格，因此可以使用 `<ret><ret>` 在内联 Markdown 文档中插入段落之间的换
行符。

`join_selections` 和 `join_selections_space` 命令（`J` 和 `A-J`）也得到了改进，现在
合并行注释时会自动删除注释标记。这使得编辑现有注释或调整段落之间的间距更加方便。

## 总结

一如既往，以上只是 25.01 版本的一些亮点。更多详情请查看完整的
[更新日志]。

欢迎加入 [Matrix 空间][matrix] 讨论使用和开发方面的问题，并在 [GitHub 仓库][helix-git] 关注 Helix 的开发进展。

[更新日志]: https://github.com/helix-editor/helix/blob/master/CHANGELOG.md#2501-2025-01-03
[helix-git]: https://github.com/helix-editor/helix/
[matrix]: https://matrix.to/#/#helix-community:matrix.org
