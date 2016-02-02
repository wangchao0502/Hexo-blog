title: 如何开发Sublime插件
date: 2016-02-02 12:42:29
tags: 
- 其它
- Sublime
---

## Sublime自带配置功能

### .sublime-settings
JSON格式，供插件或sublime调用的配置文件。例如Preferences.sublime-settings(sublime的全局配置)，WakaTime.sublime-settings(插件的配置和信息，例如api_key)

```js
{
	"color_scheme": "Packages/Devastate/Devastate.tmTheme",
	"font_size": 12,
	"ignored_packages":
	[
		"Vintage"
	],
	"theme": "Centurion.sublime-theme",
	"word_wrap": true,
	"wrap_width": 120
}
```
```js
{
	"api_key": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

### .sublime-completions
输入自动提示，自动补全功能
插件实例：Bootstrap 3 Snippets, Javascript Completions

```js
{
  "scope": "source.js",

  "completions":
  [
    {
      "trigger": "new Date()\tDate",
      "contents": "new Date()"
    },
    {
      "trigger": "length\tDate",
      "contents": "length"
    },
    {
      "trigger": "valueOf()\tDate",
      "contents": "valueOf()"
    }
  ]
}
```

```js
{
	"scope": "source.js",
	"completions": [
		{
			"trigger": "description-Date",
			"contents": "/*\n\tDescription:\n\tCreates a JavaScript Date instance that represents a single moment in time. Date objects are based on a time value that is the number of milliseconds since 1 January, 1970 UTC.\n\n\tSyntax:\n\tnew Date();\n\tnew Date(value);\n\tnew Date(dateString);\n\tnew Date(year, month, day, hour, minute, second, millisecond);\n*/"
		},
		{
			"trigger": "description-Date.now()",
			"contents": "/*\n\tDescription:\n\tThe Date.now() method returns the number of milliseconds elapsed since 1 January 1970 00:00:00 UTC.\n\n\tSyntax:\n\tvar timeInMs = Date.now();\n*/"
		}
	]
}
```

### .sublime-snippets
代码自动补全，但是没有提示框。文件支持XML的CDATA格式。

```xml
<snippet>
	<content><![CDATA[
<form action="${1}" method="${2:POST}" class="form-inline" role="form">

	${3:<div class="form-group">
		<label class="sr-only" for="">label</label>
		<input type="email" class="form-control" id="" placeholder="Input field">
	</div>}

	${0}

	<button type="submit" class="btn btn-primary">${4:Submit}</button>
</form>
]]></content>
	<tabTrigger>bs3-form:inline</tabTrigger>
</snippet>
```
注：.sublime-completions文件和.sublime-snippets文件都不支持生成动态内容。比如在我的snippets插入当前时间。如果想要插入动态内容请使用这个插件：[Smart Snippets](https://github.com/BoundInCode/SMART-Snippets)

### .sublime-commands
声明commands，一个command对应一个python class
args对应这个class中run方法的参数

这个command可以在.sublime-keymap中引用，来定义快捷键
也可以直接在sublime的console中执行 `` control + ` `` view.run_command('')

**Sublime提供了三种类型的command**

- Text Commands提供了对当前View对象(就是正在编辑的文件)内容的访问。
- Window Commands提供里对当前编辑器Window对象的引用。
- Application Commands不提供对任何window或者文件的引用，而且也很少用到。

```js
[
	{
		"caption": "Emmet: Expand Abbreviation",
		"command": "run_emmet_action",
		"args": {
			"action": "expand_abbreviation"
		}
	},

	{
		"caption": "Emmet: Wrap With Abbreviation",
		"command": "wrap_as_you_type"
	},

	{
		"caption": "Emmet: Balance (outward)",
		"command": "run_emmet_action",
		"args": {
			"action": "balance_outward"
		}
	}
]
```

### .sublime-keymap
sublime快捷键设置。例如Default (OSX).sublime-keymap

```js
[
    { "keys": ["f5"], "command": "add_time" }
]
```


### .sublime-menu
自定义菜单

- Main.sublime-menu 顶部菜单
- Side Bar.sublime-menu 右键操作左侧Side Bar菜单
- Context.sublime-menu 右键操作文件菜单

```js
[
    {
        "caption": "Preferences",
        "mnemonic": "n",
        "id": "preferences",
        "children":
        [
            {
                "caption": "WakaTime",
                "mnemonic": "W",
                "id": "wakatime-settings",
                "children":
                [
                    {
                        "command": "open_file", "args":
                        {
                            "file": "${packages}/WakaTime/WakaTime.sublime-settings"
                        },
                        "caption": "Settings – Default"
                    },
                    {
                        "command": "open_file", "args":
                        {
                            "file": "${packages}/User/WakaTime.sublime-settings"
                        },
                        "caption": "Settings – User"
                    }
                ]
            }
        ]
    }
]

```

## Sublime插件开发

**Examples**

- delete_word.py Deletes a word to the left or right of the cursor
- duplicate_line.py Duplicates the current line
- goto_line.py Prompts the user for input, then updates the selection
- font.py Shows how to work with settings
- mark.py Uses add_regions() to add an icon to the gutter
- trim_trailing_whitespace.py Modifies a buffer just before its saved

### delete_word.py
```python
import sublime, sublime_plugin

def clamp(xmin, x, xmax):
    if x < xmin:
        return xmin
    if x > xmax:
        return xmax
    return x;

class DeleteWordCommand(sublime_plugin.TextCommand):
    def find_by_class(self, pt, classes, forward):
        if forward:
            delta = 1
            end_position = self.view.size()
            if pt > end_position:
                pt = end_position
        else:
            delta = -1
            end_position = 0
            if pt < end_position:
                pt = end_position

        while pt != end_position:
            if self.view.classify(pt) & classes != 0:
                return pt
            pt += delta

        return pt

    def expand_word(self, view, pos, classes, forward):
        if forward:
            delta = 1
        else:
            delta = -1
        ws = ["\t", " "]

        if forward:
            if view.substr(pos) in ws and view.substr(pos + 1) in ws:
                classes = sublime.CLASS_WORD_START | sublime.CLASS_PUNCTUATION_START | sublime.CLASS_LINE_END
        else:
            if view.substr(pos - 1) in ws and view.substr(pos - 2) in ws:
                classes = sublime.CLASS_WORD_END | sublime.CLASS_PUNCTUATION_END | sublime.CLASS_LINE_START

        return sublime.Region(pos, self.find_by_class(pos + delta, classes, forward))

    def run(self, edit, forward = True, sub_words = False):
        if forward:
            classes = sublime.CLASS_WORD_END | sublime.CLASS_PUNCTUATION_END | sublime.CLASS_LINE_START
            if sub_words:
                classes |= sublime.CLASS_SUB_WORD_END
        else:
            classes = sublime.CLASS_WORD_START | sublime.CLASS_PUNCTUATION_START | sublime.CLASS_LINE_END | sublime.CLASS_LINE_START
            if sub_words:
                classes |= sublime.CLASS_SUB_WORD_START

        new_sels = []
        for s in reversed(self.view.sel()):
            if s.empty():
                new_sels.append(self.expand_word(self.view, s.b, classes, forward))

        sz = self.view.size()
        for s in new_sels:
            self.view.sel().add(sublime.Region(clamp(0, s.a, sz),
                clamp(0, s.b, sz)))

        self.view.run_command("add_to_kill_ring", {"forward": forward})

        if forward:
            self.view.run_command('right_delete')
        else:
            self.view.run_command('left_delete')

```

### duplicate_line.py

```python
import sublime, sublime_plugin

class DuplicateLineCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        for region in self.view.sel():
            if region.empty():
                line = self.view.line(region)
                line_contents = self.view.substr(line) + '\n'
                self.view.insert(edit, line.begin(), line_contents)
            else:
                self.view.insert(edit, region.begin(), self.view.substr(region))
```

### goto_line.py

```python
import sublime, sublime_plugin

class PromptGotoLineCommand(sublime_plugin.WindowCommand):

    def run(self):
        self.window.show_input_panel("Goto Line:", "", self.on_done, None, None)
        pass

    def on_done(self, text):
        try:
            line = int(text)
            if self.window.active_view():
                self.window.active_view().run_command("goto_line", {"line": line} )
        except ValueError:
            pass

class GotoLineCommand(sublime_plugin.TextCommand):

    def run(self, edit, line):
        # Convert from 1 based to a 0 based line number
        line = int(line) - 1

        # Negative line numbers count from the end of the buffer
        if line < 0:
            lines, _ = self.view.rowcol(self.view.size())
            line = lines + line + 1

        pt = self.view.text_point(line, 0)

        self.view.sel().clear()
        self.view.sel().add(sublime.Region(pt))

        self.view.show(pt)
```

### font.py
```python
import sublime, sublime_plugin

class IncreaseFontSizeCommand(sublime_plugin.ApplicationCommand):
    def run(self):
        s = sublime.load_settings("Preferences.sublime-settings")
        current = s.get("font_size", 10)

        if current >= 36:
            current += 4
        elif current >= 24:
            current += 2
        else:
            current += 1

        if current > 128:
            current = 128
        s.set("font_size", current)

        sublime.save_settings("Preferences.sublime-settings")

class DecreaseFontSizeCommand(sublime_plugin.ApplicationCommand):
    def run(self):
        s = sublime.load_settings("Preferences.sublime-settings")
        current = s.get("font_size", 10)
        # current -= 1

        if current >= 40:
            current -= 4
        elif current >= 26:
            current -= 2
        else:
            current -= 1

        if current < 8:
            current = 8
        s.set("font_size", current)

        sublime.save_settings("Preferences.sublime-settings")

class ResetFontSizeCommand(sublime_plugin.ApplicationCommand):
    def run(self):
        s = sublime.load_settings("Preferences.sublime-settings")
        s.erase("font_size")

        sublime.save_settings("Preferences.sublime-settings")
```

### mark.py
```python
import sublime, sublime_plugin

class SetMarkCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        mark = [s for s in self.view.sel()]
        self.view.add_regions("mark", mark, "mark", "dot",
            sublime.HIDDEN | sublime.PERSISTENT)

class SwapWithMarkCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        old_mark = self.view.get_regions("mark")

        mark = [s for s in self.view.sel()]
        self.view.add_regions("mark", mark, "mark", "dot",
            sublime.HIDDEN | sublime.PERSISTENT)

        if len(old_mark):
            self.view.sel().clear()
            for r in old_mark:
                self.view.sel().add(r)

class SelectToMarkCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        mark = self.view.get_regions("mark")

        num = min(len(mark), len(self.view.sel()))

        regions = []
        for i in range(num):
            regions.append(self.view.sel()[i].cover(mark[i]))

        for i in range(num, len(self.view.sel())):
            regions.append(self.view.sel()[i])

        self.view.sel().clear()
        for r in regions:
            self.view.sel().add(r)

class DeleteToMark(sublime_plugin.TextCommand):
    def run(self, edit):
        self.view.run_command("select_to_mark")
        self.view.run_command("add_to_kill_ring", {"forward": False})
        self.view.run_command("left_delete")

```

### trim\_trailing_whitespace.py
```python
import sublime, sublime_plugin

class TrimTrailingWhiteSpaceCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        trailing_white_space = self.view.find_all("[\t ]+$")
        trailing_white_space.reverse()
        for r in trailing_white_space:
            self.view.erase(edit, r)

class TrimTrailingWhiteSpace(sublime_plugin.EventListener):
    def on_pre_save(self, view):
        if view.settings().get("trim_trailing_white_space_on_save") == True:
            view.run_command("trim_trailing_white_space")

class EnsureNewlineAtEofCommand(sublime_plugin.TextCommand):
    def run(self, edit):
        if self.view.size() > 0 and self.view.substr(self.view.size() - 1) != '\n':
            self.view.insert(edit, self.view.size(), "\n")

class EnsureNewlineAtEof(sublime_plugin.EventListener):
    def on_pre_save(self, view):
        if view.settings().get("ensure_newline_at_eof_on_save") == True:
            if view.size() > 0 and view.substr(view.size() - 1) != '\n':
                view.run_command("ensure_newline_at_eof")
```

### 第三方Javascript支持
[SublimeJs](https://github.com/75team/SublimeJS)

## 插件提交到Package Control
1. 插件的代码需要托管到Github／Bitbucket上，或者支持ssl的服务器。一下我们默认托管到了Github上。
2. 需要有一个Tag，这个tag要符合semantic规范。为了显得我们的专业比较成熟，我没有使用v0.0.1而是v1.0.0。
3. 需要fork一份sublime的官方代码[wbond/package_control_channel](https://github.com/wbond/package_control_channel)。
4. 修改里边的一个json文件，把我们的插件信息写进去。注意：用tab而不是空格缩紧，否则会不通过。
5. 提交一个pull-request。
6. 等待通过审核。

## Links

- [Sublime Text Unofficial Documentation](http://docs.sublimetext.info/en/latest/index.html)
- [Sublime Text 3 Documentation](http://www.sublimetext.com/docs/3/index.html)
- [Sublime Api Reference](http://www.sublimetext.com/docs/3/api_reference.html)
- [Submitting a package](https://packagecontrol.io/docs/submitting_a_package#Step_8)
- [Package Control](https://packagecontrol.io)
- [SublimeJs](https://github.com/75team/SublimeJS)
- [Travis CI持续集成](https://travis-ci.com)
- [阿里妈妈翻译的Sublime API文档](http://mux.alimama.com/posts/549)
- [自己开发的sublime插件](https://github.com/wangchao0502/MatrixList-Snippets)