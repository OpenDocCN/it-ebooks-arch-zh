# Shell 文件和解释器调用

## 文件扩展名

Tip

可执行文件应该没有扩展名（强烈建议）或者使用.sh 扩展名。库文件必须使用.sh 作为扩展名，而且应该是不可执行的。

当执行一个程序时，并不需要知道它是用什么语言编写的。而且 shell 脚本也不要求有扩展名。所以我们更喜欢可执行文件没有扩展名。

然而，对于库文件，知道其用什么语言编写的是很重要的，有时候会需要使用不同语言编写的相似的库文件。使用.sh 这样特定语言后缀作为扩展名，就使得用不同语言编写的具有相同功能的库文件可以采用一样的名称。

## SUID / SGID

Tip

SUID(Set User ID)和 SGID(Set Group ID)在 shell 脚本中是被禁止的。

shell 存在太多的安全问题，以致于如果允许 SUID/SGID 会使得 shell 几乎不可能足够安全。虽然 bash 使得运行 SUID 非常困难，但在某些平台上仍然有可能运行，这就是为什么我们明确提出要禁止它。

如果你需要较高权限的访问请使用 `sudo` 。

© Copyright . Created using [Sphinx](http://sphinx-doc.org/) 1.3.5.