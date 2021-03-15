---
layout: lecture
title: "Potpourri"
date: 2020-01-29
ready: true
video:
  aspect: 56.25
  id: JZDt-PRq0uo
---

## 目次

- [キーボード割り当て変更](#keyboard-remapping)
- [デーモン](#daemon)
- [FUSE](#fuse)
- [バックアップ](#backups)
- [API](#api)
- [Common command-line flags/patterns](#common-command-line-flagspatterns)
- [Window managers](#window-managers)
- [VPN](#vpn)
- [Markdown](#markdown)
- [Hammerspoon (desktop automation on macOS)](#hammerspoon-desktop-automation-on-macos)
- [Booting + Live USBs](#booting--live-usbs)
- [Docker, Vagrant, VMs, Cloud, OpenStack](#docker-vagrant-vms-cloud-openstack)
- [Notebook programming](#notebook-programming)
- [GitHub](#github)

## キーボード割り当て変更

プログラマにとって、キーボードは主要な入力方法です。コンピュータについて様々なカスタマイズができるように、キーボードについても様々な設定ができます。（そして、それには設定する価値があります。）

最も基本的な変更は、キーボードのキー割り当てを変更することです。
この変更は、通常、あるキーが押されるたびに、このイベントに割り込み、別のキーに対応する別のイベントに置き換えます。例を以下に示します。
- Caps LockをCtrlやEscapeに割り当て変更します。Caps Lockは非常に便利な位置にありますが、ほとんど使われていないので、私たち（インストラクター）はこの設定を強く推奨します。
- PrtScを音楽の再生/一時停止キーに割り当て変更します。ほとんどのOSは再生/一時停止キーの機能を持っています。
- CtrlとMeta（WindowsまたはCommand）を入れ替えます。

また、キーを任意のコマンドに対応付けることもできます。これは、頻繁に行うタスクを実行させたい時に便利です。ここでは、何らかのソフトウェアが特定のキーの組み合わせを待ち受け、その入力イベントが検出されるたびに何らかのスクリプトを実行します。
- 新しいターミナルやブラウザのウィンドウを開く。
- 長いメールアドレスやMITのID番号など、特定のテキストを入力する。
- コンピュータやディスプレイをスリープ状態にする。

さらに複雑な割り当て変更も可能です。
- 連続したキー入力の割り当てを変更できます。例えば、Shiftを5回押すとCaps Lockがトグルされる、といったように設定できます。
- キーの「タップ」（押し離し）と「ホールド」（押しっぱなし）で割り当てを変更できます。例えば、Caps Lockを素早くタップするとEscに割り当てられ、ホールドして修飾キーとして使用するとCtrlに割り当てられる、といったことができます。
- キーボードやソフトウェア固有の割り当て変更設定を持つこと。

このトピックを開始するためのいくつかのソフトウェアリソースを紹介します。
- macOS - [karabiner-elements](https://pqrs.org/osx/karabiner/), [skhd](https://github.com/koekeishiya/skhd), [BetterTouchTool](https://folivora.ai/)
- Linux - [xmodmap](https://wiki.archlinux.org/index.php/Xmodmap), [Autokey](https://github.com/autokey/autokey)
- Windows - コントロールパネル, [AutoHotkey](https://www.autohotkey.com/), [SharpKeys](https://www.randyrants.com/category/sharpkeys/)
- QMK - キーボードがカスタムファームウェアをサポートしている場合は、[QMK](https://docs.qmk.fm/)を使ってキーボードのハードウェア自体を設定し、そのキーボードを使用しているどのマシンでも割り当て変更を動作するようにできます。

## デーモン

デーモンという言葉は新しい言葉のように感じるかもしれませんが、あなたはすでにデーモンという概念になじんでいるでしょう。
ほとんどのコンピュータには、ユーザが起動して操作するのを待つのではなく、常にバックグラウンドで実行されている一連のプロセスがあります。
これらのプロセスはデーモンと呼ばれ、デーモンとして実行されるプログラムは、そのことを示すために `d` で終わることが多いです。
例えば、SSHデーモンである `sshd` は SSHのリクエストを待ち受けて、リモートユーザがログインに必要な資格を持っているかどうかを確認するプログラムです。

Linuxでは、`systemd`（システムデーモン）がデーモンプロセスを実行、設定するための最も一般的な方法です。
現在実行中のデーモンを一覧表示するために `systemctl status` を実行することができます。
ほとんどのデーモンはなじみがないように感じるかもしれませんが、ネットワークの管理、DNSクエリの解決、GUIの表示など、システムのコア部分を担当しています。
Systemd は `enable`, `disable`, `start`, `stop`, `restart`, あるいはサービスの `status` をチェックするために `systemctl` コマンドとやりとりをすることができます（これらは `systemctl` コマンドです）。

さらに興味深いことに、`systemd` は新しいデーモン（またはサービス）を設定し、有効にするための、とても扱いやすいインタフェースを持っています。
以下はシンプルなPythonアプリケーションを実行するためのデーモンの例です。
詳細は省略しますが、ほとんどのフィールドは自己説明的なものです。

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Custom App
After=network.target

[Service]
User=foo
Group=foo
WorkingDirectory=/home/foo/projects/mydaemon
ExecStart=/usr/bin/local/python3.7 app.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

また、特定の頻度でプログラムを実行したいだけならば、独自にデーモンを構築する必要はなく、システムがすでに実行しているデーモンである [`cron`](https://www.man7.org/linux/man-pages/man8/cron.8.html) を使うことで、スケジューリングされたタスクを実行することができます。

## FUSE
現代のソフトウェアシステムは通常、小さなブロック構造を組み合わせて構成されています。
オペレーティングシステムには、ファイルシステムがどのような操作をサポートするかという共通規格があるため、異なるファイルシステムのバックエンドの使用がサポートされます。
例えば、ファイルを作成するために `touch` を実行すると、`touch` はそのファイルの作成のためにカーネルにシステムコールを発行し、カーネルは適切なファイルシステムコールを発行して指定されたファイルを作成します。
注意点として、UNIXファイルシステムは伝統的にカーネルモジュールとして実装されており、カーネルのみがファイルシステムコールの発行を許されているということです。

[FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace) (Filesystem in User Space) を使うと、ファイルシステムをユーザプログラムで実装することができます。
ユーザはファイルシステムコールのためにユーザ空間のコードを実行し、FUSEは必要なシステムコールをカーネルインタフェースへと「橋渡し」します。
これは事実上、ユーザがファイルシステムコールに対して任意の機能を実装できることを意味します。

例えば、FUSEを使うと、仮想ファイルシステムで操作を実行するときはいつでも、その操作をSSH経由でリモートマシンに転送し、そこで実行し、その出力を自分に返すことができます。
このようにして、ローカルのプログラムは、実際にはリモートサーバにあるファイルを、あたかも自分のコンピュータにあるかのように扱えます。
これは実質的に `sshfs` が行っていることです。

FUSEファイルシステムの興味深い例をいくつか紹介します。
- [sshfs](https://github.com/libfuse/sshfs) - SSH経由のリモートファイル/フォルダをローカルで開きます。
- [rclone](https://rclone.org/commands/rclone_mount/) - DropboxやGoogle Drive、Amazon S3やGoogle Cloud Storageなどのクラウドストレージサービスをマウントして、ローカルでデータを開きます。
- [gocryptfs](https://nuetzlich.net/gocryptfs/) - 暗号化オーバーレイファイルシステムです。ファイルは暗号化されて保存されますが、ファイルシステムがマウントされるとマウントポイントではプレーンテキストとして表示されます。
- [kbfs](https://keybase.io/docs/kbfs) - End-to-endで暗号化された分散ファイルシステムです。個人フォルダ、共有フォルダ、パブリックフォルダを持つことができます。
- [borgbackup](https://borgbackup.readthedocs.io/en/stable/usage/mount.html) - データ重複除去、圧縮、暗号化されたバックアップをマウントして、簡単に参照できるようにします。

## バックアップ

バックアップされていないデータは、いつか永遠に消えてしまうかもしれません。
データをコピーするのは簡単ですが、データを確実にバックアップするのは難しいものです。
ここでは、バックアップの基本的な方法と、いくつかのアプローチの落とし穴について説明します。

まず、同じディスク内でデータをコピーすることはバックアップとは言えません。なぜなら、ディスクはすべてのデータの単一障害点だからです。
同様に、自宅の外付けドライブも、火事や強盗などで失われる可能性があるため、バックアップ手段としては不十分です。むしろ、オフサイトでバックアップをとることをお勧めします。

オンラインストレージでの同期もバックアップではありません。例えば、DropboxやGDriveは便利な手段ですが、データが消去されたり破損したりすると、その変更が伝播してしまいます。
同じ理由で、RAIDのようなディスクのミラーリングもバックアップではありません。データが削除されたり、破損したり、ランサムウェアで暗号化されたりした時に役に立たないからです。

優れたバックアップ手段の中核となる機能は、バージョン管理、重複排除、セキュリティです。
バージョン管理されたバックアップでは、変更履歴にアクセスして効率的にファイルを復元することができます。
効率的なバックアップ手段は、データの重複排除を使用することで増分の変更のみを保存し、ストレージのオーバーヘッドを削減します。
セキュリティに関しては、データを読み取ったり、さらに重要なこととして、すべてのデータや関連するバックアップを削除したりするために、他人が何を知っていたり持っていたりする必要があるのかを点検すべきです。
最後に、バックアップを盲目的に信用するのはよくない考えで、バックアップを使ってデータを復旧できるかどうかを定期的に確認する必要があります。

バックアップは、コンピュータ内のローカルファイルだけに留まりません。
ウェブアプリケーションの大幅な増加に伴い、大量のデータがクラウドに保存されるようになりました。
例えば、ウェブメール、ソーシャルメディアの写真、ストリーミングサービスの音楽プレイリスト、オンラインドキュメントなどは、対応するアカウントにアクセスできなくなると消えてしまいます。
このような情報のオフラインコピーを持つことが望ましく、これらのデータを取得して保存するために有志が作ったオンラインツールが提供されています。

より詳しい説明は、2019年の講義ノート [バックアップ](/2019/backups) を見てみてください。

## API

この授業では、コンピュータをより効率的に使って _ローカル_ なタスクをこなすことについて多くのことを説明してきましたが、これらの学びの多くは、より広いインターネットにも当てはまることがわかります。
インターネット上のほとんどのサービスは、そのデータにプログラムでアクセスできる「API」を持っています。
例えば、アメリカ政府は天気予報を取得するためのAPIを持っており、これを使えば自分のシェルで簡単に天気予報を取得することができます。

これらのAPIのほとんどは、似たフォーマットを持っています。
APIは構造化されたURLであり、多くの場合、`api.service.com`をルートとしており、パスとクエリパラメータは、読みたいデータや実行したいアクションを示します。
例えば、米国の気象データの場合、特定の場所の予報を取得するには、https://api.weather.gov/points/42.3604,-71.094 に GET リクエスト（例えば `curl`を使用）を発行します。
レスポンス自体には、その地域の特定の予報を得ることができる他のURLのまとまりが含まれています。
通常、レスポンスはJSONとしてフォーマットされており、これを[`jq`](https://stedolan.github.io/jq/)のようなツールに通すことで、関心のある情報に変換することができます。

APIによっては認証が必要なものがありますが、この認証は通常ある種の秘密の _トークン_ が用いられ、これをリクエストに含める必要があります。
APIのドキュメントを読んで、探している特定のサービスが何を使用しているかを確認する必要がありますが、「[OAuth](https://www.oauth.com/)」はよく使われるプロトコルです。
本質的には、OAuthは特定のサービスで「自分のように振る舞う」ことができるトークンを与え、特定の目的のためにのみ使用できるようにする方法です。
このトークンは _秘密_ のものであり、あなたのトークンにアクセスした人は、 _あなたの_ アカウントでトークンが許可したことは何でもできるということを覚えておいてください。

[IFTTT](https://ifttt.com/)は、APIのアイデアを中心としたウェブサイトとサービスです。膨大な数のサービスとの統合を提供し、それらのサービスからほぼ任意の方法でイベントを連鎖させることができます。
ぜひ一度見てみてください。

## Common command-line flags/patterns

Command-line tools vary a lot, and you will often want to check out their `man` pages before using them.
They often share some common features though that can be good to be aware of:

 - Most tools support some kind of `--help` flag to display brief usage instructions for the tool.
 - Many tools that can cause irrevocable change support the notion of a "dry run" in which they only print what they _would have done_, but do not actually perform the change. Similarly, they often have an "interactive" flag that will prompt you for each destructive action.
 - You can usually use `--version` or `-V` to have the program print its own version (handy for reporting bugs!).
 - Almost all tools have a `--verbose` or `-v` flag to produce more verbose output. You can usually include the flag multiple times (`-vvv`) to get _more_ verbose output, which can be handy for debugging. Similarly, many tools have a `--quiet` flag for making it only print something on error.
 - In many tools, `-` in place of a file name means "standard input" or "standard output", depending on the argument.
 - Possibly destructive tools are generally not recursive by default, but support a "recursive" flag (often `-r`) to make them recurse.
 - Sometimes, you want to pass something that _looks_ like a flag as a normal argument. For example, imagine you wanted to remove a file called `-r`. Or you want to run one program "through" another, like `ssh machine foo`, and you want to pass a flag to the "inner" program (`foo`). The special argument `--` makes a program _stop_ processing flags and options (things starting with `-`) in what follows, letting you pass things that look like flags without them being interpreted as such: `rm -- -r` or `ssh machine --for-ssh -- foo --for-foo`.

## Window managers

Most of you are used to using a "drag and drop" window manager, like what comes with Windows, macOS, and Ubuntu by default.
There are windows that just sort of hang there on screen, and you can drag them around, resize them, and have them overlap one another.
But these are only one _type_ of window manager, often referred to as a "floating" window manager.
There are many others, especially on Linux.
A particularly common alternative is a "tiling" window manager.
In a tiling window manager, windows never overlap, and are instead arranged as tiles on your screen, sort of like panes in tmux.
With a tiling window manager, the screen is always filled by whatever windows are open, arranged according to some _layout_.
If you have just one window, it takes up the full screen.
If you then open another, the original window shrinks to make room for it (often something like 2/3 and 1/3).
If you open a third, the other windows will again shrink to accommodate the new window.
Just like with tmux panes, you can navigate around these tiled windows with your keyboard, and you can resize them and move them around, all without touching the mouse.
They are worth looking into!

## VPN

VPNs are all the rage these days, but it's not clear that's for [any good reason](https://gist.github.com/joepie91/5a9909939e6ce7d09e29).
You should be aware of what a VPN does and does not get you.
A VPN, in the best case, is _really_ just a way for you to change your internet service provider as far as the internet is concerned.
All your traffic will look like it's coming from the VPN provider instead of your "real" location, and the network you are connected to will only see encrypted traffic.

While that may seem attractive, keep in mind that when you use a VPN, all you are really doing is shifting your trust from you current ISP to the VPN hosting company.
Whatever your ISP _could_ see, the VPN provider now sees _instead_.
If you trust them _more_ than your ISP, that is a win, but otherwise, it is not clear that you have gained much.
If you are sitting on some dodgy unencrypted public Wi-Fi at an airport, then maybe you don't trust the connection much, but at home, the trade-off is not quite as clear.

You should also know that these days, much of your traffic, at least of a sensitive nature, is _already_ encrypted through HTTPS or TLS more generally.
In that case, it usually matters little whether you are on a "bad" network or not -- the network operator will only learn what servers you talk to, but not anything about the data that is exchanged.

Notice that I said "in the best case" above.
It is not unheard of for VPN providers to accidentally misconfigure their software such that the encryption is either weak or entirely disabled.
Some VPN providers are malicious (or at the very least opportunist), and will log all your traffic, and possibly sell information about it to third parties.
Choosing a bad VPN provider is often worse than not using one in the first place.

In a pinch, MIT [runs a VPN](https://ist.mit.edu/vpn) for its students, so that may be worth taking a look at. Also, if you're going to roll your own, give [WireGuard](https://www.wireguard.com/) a look.

## Markdown

There is a high chance that you will write some text over the course of
your career. And often, you will want to mark up that text in simple
ways. You want some text to be bold or italic, or you want to add
headers, links, and code fragments. Instead of pulling out a heavy tool
like Word or LaTeX, you may want to consider using the lightweight
markup language [Markdown](https://commonmark.org/help/).

You have probably seen Markdown already, or at least some variant of it.
Subsets of it are used and supported almost everywhere, even if it's not
under the name Markdown. At its core, Markdown is an attempt to codify
the way that people already often mark up text when they are writing
plain text documents. Emphasis (*italics*) is added by surrounding a
word with `*`. Strong emphasis (**bold**) is added using `**`. Lines
starting with `#` are headings (and the number of `#`s is the subheading
level). Any line starting with `-` is a bullet list item, and any line
starting with a number + `.` is a numbered list item. Backtick is used
to show words in `code font`, and a code block can be entered by
indenting a line with four spaces or surrounding it with
triple-backticks:

    ```
    code goes here
    ```

To add a link, place the _text_ for the link in square brackets,
and the URL immediately following that in parentheses: `[name](url)`.
Markdown is easy to get started with, and you can use it nearly
everywhere. In fact, the lecture notes for this lecture, and all the
others, are written in Markdown, and you can see the raw Markdown
[here](https://raw.githubusercontent.com/missing-semester/missing-semester/master/_2020/potpourri.md).



## Hammerspoon (desktop automation on macOS)

[Hammerspoon](https://www.hammerspoon.org/) is a desktop automation framework
for macOS. It lets you write Lua scripts that hook into operating system
functionality, allowing you to interact with the keyboard/mouse, windows,
displays, filesystem, and much more.

Some examples of things you can do with Hammerspoon:

- Bind hotkeys to move windows to specific locations
- Create a menu bar button that automatically lays out windows in a specific layout
- Mute your speaker when you arrive in lab (by detecting the WiFi network)
- Show you a warning if you've accidentally taken your friend's power supply

At a high level, Hammerspoon lets you run arbitrary Lua code, bound to menu
buttons, key presses, or events, and Hammerspoon provides an extensive library
for interacting with the system, so there's basically no limit to what you can
do with it. Many people have made their Hammerspoon configurations public, so
you can generally find what you need by searching the internet, but you can
always write your own code from scratch.

### Resources

- [Getting Started with Hammerspoon](https://www.hammerspoon.org/go/)
- [Sample configurations](https://github.com/Hammerspoon/hammerspoon/wiki/Sample-Configurations)
- [Anish's Hammerspoon config](https://github.com/anishathalye/dotfiles-local/tree/mac/hammerspoon)

## Booting + Live USBs

When your machine boots up, before the operating system is loaded, the
[BIOS](https://en.wikipedia.org/wiki/BIOS)/[UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface)
initializes the system. During this process, you can press a specific key
combination to configure this layer of software. For example, your computer may
say something like "Press F9 to configure BIOS. Press F12 to enter boot menu."
during the boot process. You can configure all sorts of hardware-related
settings in the BIOS menu. You can also enter the boot menu to boot from an
alternate device instead of your hard drive.

[Live USBs](https://en.wikipedia.org/wiki/Live_USB) are USB flash drives
containing an operating system. You can create one of these by downloading an
operating system (e.g. a Linux distribution) and burning it to the flash drive.
This process is a little bit more complicated than simply copying a `.iso` file
to the disk. There are tools like [UNetbootin](https://unetbootin.github.io/)
to help you create live USBs.

Live USBs are useful for all sorts of purposes. Among other things, if you
break your existing operating system installation so that it no longer boots,
you can use a live USB to recover data or fix the operating system.

## Docker, Vagrant, VMs, Cloud, OpenStack

[Virtual machines](https://en.wikipedia.org/wiki/Virtual_machine) and similar
tools like containers let you emulate a whole computer system, including the
operating system. This can be useful for creating an isolated environment for
testing, development, or exploration (e.g. running potentially malicious code).

[Vagrant](https://www.vagrantup.com/) is a tool that lets you describe machine
configurations (operating system, services, packages, etc.) in code, and then
instantiate VMs with a simple `vagrant up`. [Docker](https://www.docker.com/)
is conceptually similar but it uses containers instead.

You can also rent virtual machines on the cloud, and it's a nice way to get instant
access to:

- A cheap always-on machine that has a public IP address, used to host services
- A machine with a lot of CPU, disk, RAM, and/or GPU
- Many more machines than you physically have access to (billing is often by
the second, so if you want a lot of computing for a short amount of time, it's
feasible to rent 1000 computers for a couple of minutes)

Popular services include [Amazon AWS](https://aws.amazon.com/), [Google
Cloud](https://cloud.google.com/),[ Microsoft Azure](https://azure.microsoft.com/),
[DigitalOcean](https://www.digitalocean.com/).

If you're a member of MIT CSAIL, you can get free VMs for research purposes
through the [CSAIL OpenStack
instance](https://tig.csail.mit.edu/shared-computing/open-stack/).

## Notebook programming

[Notebook programming
environments](https://en.wikipedia.org/wiki/Notebook_interface) can be really
handy for doing certain types of interactive or exploratory development.
Perhaps the most popular notebook programming environment today is
[Jupyter](https://jupyter.org/), for Python (and several other languages).
[Wolfram Mathematica](https://www.wolfram.com/mathematica/) is another notebook
programming environment that's great for doing math-oriented programming.

## GitHub

[GitHub](https://github.com/) is one of the most popular platforms for
open-source software development. Many of the tools we've talked about in this
class, from [vim](https://github.com/vim/vim) to
[Hammerspoon](https://github.com/Hammerspoon/hammerspoon), are hosted on
GitHub. It's easy to get started contributing to open-source to help improve
the tools that you use every day.

There are two primary ways in which people contribute to projects on GitHub:

- Creating an
[issue](https://help.github.com/en/github/managing-your-work-on-github/creating-an-issue).
This can be used to report bugs or request a new feature. Neither of these
involves reading or writing code, so it can be pretty lightweight to do.
High-quality bug reports can be extremely valuable to developers. Commenting on
existing discussions can be helpful too.
- Contribute code through a [pull
request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).
This is generally more involved than creating an issue. You can
[fork](https://help.github.com/en/github/getting-started-with-github/fork-a-repo)
a repository on GitHub, clone your fork, create a new branch, make some changes
(e.g. fix a bug or implement a feature), push the branch, and then [create a
pull
request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request).
After this, there will generally be some back-and-forth with the project
maintainers, who will give you feedback on your patch. Finally, if all goes
well, your patch will be merged into the upstream repository. Often times,
larger projects will have a contributing guide, tag beginner-friendly issues,
and some even have mentorship programs to help first-time contributors become
familiar with the project.
