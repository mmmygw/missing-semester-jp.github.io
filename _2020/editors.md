---
layout: lecture
title: "エディタ (Vim)"
date: 2020-01-15
ready: true
video:
  aspect: 56.25
  id: a6Q8Na575qc
---

文章を書くこととプログラミングは全く異なる営みです。
英語のライティングがまとまった文章を書く作業であるのに対し、
プログラミングではファイルを切り替え、コードを読み、辿り、そして編集します。
そのため、英語という言語に適したエディタとプログラム言語に適したエディタがある
のです。（例: Microsoft Word と Visual Studio Code）

プログラマとして我々は大半の時間をコードの編集に費やすため、
ニーズに合ったエディタに習熟することは非常に時間対効果が高いといえます。
以下に新しいエディタの使い方を学ぶ際の一般的な方法論を紹介します。

- チュートリアルを始める（例: 本講座 + 紹介資料）
- （たとえ一時的に生産性を損ねても）あらゆるテキスト編集にそのエディタを使うことに拘る
- 常により良い方法がないかを模索する。もしもっと良い方法がないかという考えが頭をよぎったら、それは必ず存在します。

上記の方法に従い新たなプログラムをエディタの使い方を学ぶ題材にするならば、
タイムラインはこのようになるでしょう。
まず１、２時間以内にエディタの基礎機能（ファイルを開いて編集する、保存・終了する、バッファを操作する）
を習得します。
２０時間も経てば元のエディタと同じくらいの速度で編集できるようになっているでしょう。
新しいエディタに関する十分な知識を得て、使い方を身体で覚えたならば、
その恩恵を受け膨大な時間を節約することができるようになっていきます。
モダンなテキストエディタは素晴らしく強力なツールです。
あなたは学べば学ぶほどより速い編集能力を身につけることができるでしょう。

# どのエディタを学ぶべきですか？

プログラマはテキストエディタに関して[様々な選択肢](https://ja.wikipedia.org/wiki/%E3%82%A8%E3%83%87%E3%82%A3%E3%82%BF%E6%88%A6%E4%BA%89)があります。
どのエディタが現在最も人気でしょうか？
[Stack Overflow
での調査](https://insights.stackoverflow.com/survey/2019/#development-environments-and-tools)
によると、
[Visual Studio Code](https://code.visualstudio.com/) が全体で最も人気があり、
[Vim](https://www.vim.org/) はCLI（コマンドラインインターフェース）のエディタの中で最も支持を集めています。
（ただし、Stack Overflow のユーザは全プログラマを代表するものではないため調査結果にはバイアスが含まれています。)

## Vim
この講座では全てのインストラクションに Vim を用います。
Vim には長い歴史があります。
Vim は元々 Vi（1976）として開発され、今日まで開発が継続されてきました。
Vim はいくつかの素晴らしいアイデアが背景にあります。そのため、
多くのツールが Vim の挙動をエミュレートするモードを採用しています
（VSCode でも、１４０万のユーザが [Vim emulation for VS code](https://github.com/VSCodeVim/Vim) をインストールしています）。
つまり、たとえ他のエディタを最終的に使うことになるとしても、Vim は間違いなく学ぶ価値があるといえるでしょう。

Vim の機能全てを５０分の講座で紹介し尽くすことは到底できません、
そこで本講座は Vim の哲学とその基礎、より高度な機能の一部を紹介するに留めようと思います。
講座の終わりでは Vim に更に習熟するための情報源をまとめて紹介します。

# Vim の哲学
プログラミングをする際、我々は殆どの時間をコードを書くことではなく、読むことと編集に費やします。
このため Vim は **モーダル** エディタとして設計されており、
テキストの挿入と編集にはそれぞれ異なるモードが割り当てられています。
また、Vim はプログラマブル（Vimscript や Python 等他のプログラミング言語で拡張機能を開発可能）
であり、覚えやすいキーストロークやコマンドは全て組み合わせて実行することができます。
Vim はマウスの使用を徹底して避けます。何故ならマウスでの操作はキーボードのそれと比較して余りにも遅いからです。
更に Vim は矢印キーの使用を避けます、これもキーボード上で必要な移動量が大きすぎるためです。
これらの思想から、思考のスピードで編集することが可能なエディタが完成しました。

# モーダル編集
Vim は多くのプログラマはまとまった長いテキストを書くよりも、
コードを読み、ファイルを辿り、そして僅かに編集を加える
という作業に大半の時間を費やしているという事実に着眼して設計されています。
このため、Vim は複数の操作モードを備えています。

- **標準モード**：ファイル上を移動し、編集を加える
- **挿入モード**：テキストを挿入する
- **置換モード**：テキストを置換する
- **ビジュアルモード**：テキストをブロック選択する
- **コマンドラインモード**：コマンドを実行する

キーストロークは操作モード毎に異なる機能が割り当てられています。
挿入モードでの x は単に文字 x の挿入を行いますが、標準モードではカーソル下の文字を消去し、
ビジュアルモードでは選択範囲を消去します。

初期設定では、Vim の画面左下に現在のモードが表示されます。
デフォルトのモードは標準モードです。
基本的には、標準モードと挿入モードを行き来することになります。

`<ESC>` (Escキー) を押すと現在のモードから標準モードに切り替えることができます。
標準モードからは、`i` で挿入モード、`R` で置換モード、`v` でビジュアルモード、`V` でビジュアルブロックモード、
`<C-v>`（Ctrl-V、`^V` とも表記）、コマンドラインモードに `:` で入ります。

Vim で編集する際は `<ESC>` を頻繁に使うため、
Caps Lock キーを Escape に置換することを検討してみてもいいでしょう
（[macOS
instructions](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)）。

# 基本
## テキストを挿入する
標準モードから、`i` を押下して挿入モードに入ります。
挿入モードでは、Vim は他のエディタと殆ど同じような挙動をしているでしょう。
そこで `<ESC>` を押すとまた標準モードに戻ってきます。
これが Vim によるファイル編集を始める全ての基本になります。
（挿入モードで大半の編集作業を行わないよう注意し、速やかに標準モードに戻るようにして下さい。）

## Buffers, tabs, and windows

Vim maintains a set of open files, called "buffers". A Vim session has a number
of tabs, each of which has a number of windows (split panes). Each window shows
a single buffer. Unlike other programs you are familiar with, like web
browsers, there is not a 1-to-1 correspondence between buffers and windows;
windows are merely views. A given buffer may be open in _multiple_ windows,
even within the same tab. This can be quite handy, for example, to view two
different parts of a file at the same time.

By default, Vim opens with a single tab, which contains a single window.

## Command-line

Command mode can be entered by typing `:` in Normal mode. Your cursor will jump
to the command line at the bottom of the screen upon pressing `:`. This mode
has many functionalities, including opening, saving, and closing files, and
[quitting Vim](https://twitter.com/iamdevloper/status/435555976687923200).

- `:q` quit (close window)
- `:w` save ("write")
- `:wq` save and quit
- `:e {name of file}` open file for editing
- `:ls` show open buffers
- `:help {topic}` open help
    - `:help :w` opens help for the `:w` command
    - `:help w` opens help for the `w` movement

# Vim's interface is a programming language

The most important idea in Vim is that Vim's interface itself is a programming
language. Keystrokes (with mnemonic names) are commands, and these commands
_compose_. This enables efficient movement and edits, especially once the
commands become muscle memory.

## Movement

You should spend most of your time in Normal mode, using movement commands to
navigate the buffer. Movements in Vim are also called "nouns", because they
refer to chunks of text.

- Basic movement: `hjkl` (left, down, up, right)
- Words: `w` (next word), `b` (beginning of word), `e` (end of word)
- Lines: `0` (beginning of line), `^` (first non-blank character), `$` (end of line)
- Screen: `H` (top of screen), `M` (middle of screen), `L` (bottom of screen)
- Scroll: `Ctrl-u` (up), `Ctrl-d` (down)
- File: `gg` (beginning of file), `G` (end of file)
- Line numbers: `:{number}<CR>` or `{number}G` (line {number})
- Misc: `%` (corresponding item)
- Find: `f{character}`, `t{character}`, `F{character}`, `T{character}`
    - find/to forward/backward {character} on the current line
    - `,` / `;` for navigating matches
- Search: `/{regex}`, `n` / `N` for navigating matches

## Selection

Visual modes:

- Visual
- Visual Line
- Visual Block

Can use movement keys to make selection.

## Edits

Everything that you used to do with the mouse, you now do with the keyboard
using editing commands that compose with movement commands. Here's where Vim's
interface starts to look like a programming language. Vim's editing commands
are also called "verbs", because verbs act on nouns.

- `i` enter Insert mode
    - but for manipulating/deleting text, want to use something more than
    backspace
- `o` / `O` insert line below / above
- `d{motion}` delete {motion}
    - e.g. `dw` is delete word, `d$` is delete to end of line, `d0` is delete
    to beginning of line
- `c{motion}` change {motion}
    - e.g. `cw` is change word
    - like `d{motion}` followed by `i`
- `x` delete character (equal do `dl`)
- `s` substitute character (equal to `xi`)
- Visual mode + manipulation
    - select text, `d` to delete it or `c` to change it
- `u` to undo, `<C-r>` to redo
- `y` to copy / "yank" (some other commands like `d` also copy)
- `p` to paste
- Lots more to learn: e.g. `~` flips the case of a character

## Counts

You can combine nouns and verbs with a count, which will perform a given action
a number of times.

- `3w` move 3 words forward
- `5j` move 5 lines down
- `7dw` delete 7 words

## Modifiers

You can use modifiers to change the meaning of a noun. Some modifiers are `i`,
which means "inner" or "inside", and `a`, which means "around".

- `ci(` change the contents inside the current pair of parentheses
- `ci[` change the contents inside the current pair of square brackets
- `da'` delete a single-quoted string, including the surrounding single quotes

# Demo

Here is a broken [fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz)
implementation:

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print('fizz')
        if i % 5 == 0:
            print('fizz')
        if i % 3 and i % 5:
            print(i)

def main():
    fizz_buzz(10)
```

We will fix the following issues:

- Main is never called
- Starts at 0 instead of 1
- Prints "fizz" and "buzz" on separate lines for multiples of 15
- Prints "fizz" for multiples of 5
- Uses a hard-coded argument of 10 instead of taking a command-line argument

{% comment %}
- main is never called
  - `G` end of file
  - `o` open new line below
  - type in "if __name__ ..." thing
- starts at 0 instead of 1
  - search for `/range`
  - `ww` to move forward 2 words
  - `i` to insert text, "1, "
  - `ea` to insert after limit, "+1"
- newline for "fizzbuzz"
  - `jj$i` to insert text at end of line
  - add ", end=''"
  - `jj.` to repeat for second print
  - `jjo` to open line below if
  - add "else: print()"
- fizz fizz
  - `ci'` to change fizz
- command-line argument
  - `ggO` to open above
  - "import sys"
  - `/10`
  - `ci(` to "int(sys.argv[1])"
{% endcomment %}

See the lecture video for the demonstration. Compare how the above changes are
made using Vim to how you might make the same edits using another program.
Notice how very few keystrokes are required in Vim, allowing you to edit at the
speed you think.

# Customizing Vim

Vim is customized through a plain-text configuration file in `~/.vimrc`
(containing Vimscript commands). There are probably lots of basic settings that
you want to turn on.

We are providing a well-documented basic config that you can use as a starting
point. We recommend using this because it fixes some of Vim's quirky default
behavior. **Download our config [here](/2020/files/vimrc) and save it to
`~/.vimrc`.**

Vim is heavily customizable, and it's worth spending time exploring
customization options. You can look at people's dotfiles on GitHub for
inspiration, for example, your instructors' Vim configs
([Anish](https://github.com/anishathalye/dotfiles/blob/master/vimrc),
[Jon](https://github.com/jonhoo/configs/blob/master/editor/.config/nvim/init.vim) (uses [neovim](https://neovim.io/)),
[Jose](https://github.com/JJGO/dotfiles/blob/master/vim/.vimrc)). There are
lots of good blog posts on this topic too. Try not to copy-and-paste people's
full configuration, but read it, understand it, and take what you need.

# Extending Vim

There are tons of plugins for extending Vim. Contrary to outdated advice that
you might find on the internet, you do _not_ need to use a plugin manager for
Vim (since Vim 8.0). Instead, you can use the built-in package management
system. Simply create the directory `~/.vim/pack/vendor/start/`, and put
plugins in there (e.g. via `git clone`).

Here are some of our favorite plugins:

- [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim): fuzzy file finder
- [ack.vim](https://github.com/mileszs/ack.vim): code search
- [nerdtree](https://github.com/scrooloose/nerdtree): file explorer
- [vim-easymotion](https://github.com/easymotion/vim-easymotion): magic motions

We're trying to avoid giving an overwhelmingly long list of plugins here. You
can check out the instructors' dotfiles
([Anish](https://github.com/anishathalye/dotfiles),
[Jon](https://github.com/jonhoo/configs),
[Jose](https://github.com/JJGO/dotfiles)) to see what other plugins we use.
Check out [Vim Awesome](https://vimawesome.com/) for more awesome Vim plugins.
There are also tons of blog posts on this topic: just search for "best Vim
plugins".

# Vim-mode in other programs

Many tools support Vim emulation. The quality varies from good to great;
depending on the tool, it may not support the fancier Vim features, but most
cover the basics pretty well.

## Shell

If you're a Bash user, use `set -o vi`. If you use Zsh, `bindkey -v`. For Fish,
`fish_vi_key_bindings`. Additionally, no matter what shell you use, you can
`export EDITOR=vim`. This is the environment variable used to decide which
editor is launched when a program wants to start an editor. For example, `git`
will use this editor for commit messages.

## Readline

Many programs use the [GNU
Readline](https://tiswww.case.edu/php/chet/readline/rltop.html) library for
their command-line interface. Readline supports (basic) Vim emulation too,
which can be enabled by adding the following line to the `~/.inputrc` file:

```
set editing-mode vi
```

With this setting, for example, the Python REPL will support Vim bindings.

## Others

There are even vim keybinding extensions for web
[browsers](http://vim.wikia.com/wiki/Vim_key_bindings_for_web_browsers) - some
popular ones are
[Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en)
for Google Chrome and [Tridactyl](https://github.com/tridactyl/tridactyl) for
Firefox. You can even get Vim bindings in [Jupyter
notebooks](https://github.com/lambdalisue/jupyter-vim-binding).

# Advanced Vim

Here are a few examples to show you the power of the editor. We can't teach you
all of these kinds of things, but you'll learn them as you go. A good
heuristic: whenever you're using your editor and you think "there must be a
better way of doing this", there probably is: look it up online.

## Search and replace

`:s` (substitute) command ([documentation](http://vim.wikia.com/wiki/Search_and_replace)).

- `%s/foo/bar/g`
    - replace foo with bar globally in file
- `%s/\[.*\](\(.*\))/\1/g`
    - replace named Markdown links with plain URLs

## Multiple windows

- `:sp` / `:vsp` to split windows
- Can have multiple views of the same buffer.

## マクロ

- `q{character}` to start recording a macro in register `{character}`
- `q` to stop recording
- `@{character}` replays the macro
- Macro execution stops on error
- `{number}@{character}` executes a macro {number} times
- Macros can be recursive
    - first clear the macro with `q{character}q`
    - record the macro, with `@{character}` to invoke the macro recursively
    (will be a no-op until recording is complete)
- Example: convert xml to json ([file](/2020/files/example-data.xml))
    - Array of objects with keys "name" / "email"
    - Use a Python program?
    - Use sed / regexes
        - `g/people/d`
        - `%s/<person>/{/g`
        - `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
        - ...
    - Vim commands / macros
        - `Gdd`, `ggdd` delete first and last lines
        - Macro to format a single element (register `e`)
            - Go to line with `<name>`
            - `qe^r"f>s": "<ESC>f<C"<ESC>q`
        - Macro to format a person
            - Go to line with `<person>`
            - `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`
        - Macro to format a person and go to the next person
            - Go to line with `<person>`
            - `qq@pjq`
        - Execute macro until end of file
            - `999@q`
        - Manually remove last `,` and add `[` and `]` delimiters

# 参考

- `vimtutor` はVimと共にインストールされるチュートリアルです。もしVimが既にインストールされている環境ならば、シェルで `vimtutor` を実行することができます。
- [Vim Adventures](https://vim-adventures.com/) is a game to learn Vim
- [Vim Tips Wiki](http://vim.wikia.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/) has various Vim tips
- [Vim Golf](http://www.vimgolf.com/) は Vimの[コードゴルフ](https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%BC%E3%83%89%E3%82%B4%E3%83%AB%E3%83%95)場です
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/titles/dnvim2/) (book)

- `vimtutor` is a tutorial that comes installed with Vim - if Vim is installed, you should be able to run `vimtutor` from your shell
- [Vim Adventures](https://vim-adventures.com/) is a game to learn Vim
- [Vim Tips Wiki](http://vim.wikia.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/) has various Vim tips
- [Vim Golf](http://www.vimgolf.com/) is [code golf](https://en.wikipedia.org/wiki/Code_golf), but where the programming language is Vim's UI
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/titles/dnvim2/) (book)

# 練習
1. `vimtutor` を完了させる。[80x24](https://ja.wikipedia.org/wiki/VT100)(80 columns by 24 lines) ターミナルウィンドウで行うのが良いでしょう。
1. 我々の [basic vimrc](/2020/files/vimrc) を`~/.vimrc` に保存し、ファイル全体を（*Vim を使って*）読み、Vimがデフォルト設定とどのように挙動や見た目が異なるかを確認して下さい。
1. プラグインのインストール、設定を行う
   1. プラグインのディレクトリを作ります `mkdir -p ~/.vim/pack/vendor/start`
   1. プラグインをダウンロードする `cd ~/.vim/pack/vendor/start; gitclone https://github.com/ctrlpvim/ctrlp.vim`
   1. [プラグインのドキュメント](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md)
      を読みます。Vim を開いて、Vimコマンドで `:CtrlP` を使ってファイルを見つけてみましょう。プロジェクトディレクトリに移動し、Vimを開いて `:CtrlP` を入力します。
   1. [configuration](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md#basic-options) を参考に、`~/.vimrc` へ設定を追加してみましょう。`Ctrl-P` で CtrlP を開けるようにします。
1. Vim の訓練に、[Demo](#demo) をあなたのマシンで復習しましょう。
1. 次の月から__全ての__テキスト編集にVimを使います。何かが非行率または "もっと良い方法があるはずだ" と感じる度に方法を検索しましょう。きっと良い方法があるはずです。
1. 他のツールを Vim のキーバインディングを使うように設定してみましょう。
1. `~/.vimrc` を更にカスタマイズしましょう。そして好みのプラグインを入れましょう。
1. （発展）XMLをJSON [example file](/2020/files/example-data.xml) にVimおマクロを使って変換してみましょう。躓いたら、[マクロ](#マクロ) セクションがヒントになります。


1. Complete `vimtutor`. Note: it looks best in a
   [80x24](https://en.wikipedia.org/wiki/VT100) (80 columns by 24 lines)
   terminal window.
1. Download our [basic vimrc](/2020/files/vimrc) and save it to `~/.vimrc`. Read
   through the well-commented file (using Vim!), and observe how Vim looks and
   behaves slightly differently with the new config.
1. Install and configure a plugin:
   [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim).
   1. Create the plugins directory with `mkdir -p ~/.vim/pack/vendor/start`
   1. Download the plugin: `cd ~/.vim/pack/vendor/start; git clone
      https://github.com/ctrlpvim/ctrlp.vim`
   1. Read the
      [documentation](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md)
      for the plugin. Try using CtrlP to locate a file by navigating to a
      project directory, opening Vim, and using the Vim command-line to start
      `:CtrlP`.
    1. Customize CtrlP by adding
       [configuration](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md#basic-options)
       to your `~/.vimrc` to open CtrlP by pressing Ctrl-P.
1. To practice using Vim, re-do the [Demo](#demo) from lecture on your own
   machine.
1. Use Vim for _all_ your text editing for the next month. Whenever something
   seems inefficient, or when you think "there must be a better way", try
   Googling it, there probably is. If you get stuck, come to office hours or
   send us an email.
1. Configure your other tools to use Vim bindings (see instructions above).
1. Further customize your `~/.vimrc` and install more plugins.
1. (Advanced) Convert XML to JSON ([example file](/2020/files/example-data.xml))
   using Vim macros. Try to do this on your own, but you can look at the
   [macros](#macros) section above if you get stuck.
