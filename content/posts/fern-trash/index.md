---
author: "im"
title: "fern.vim でファイルを削除できない問題の解決"
date: "2022-03-27"
tags: [
    "Neovim",
    "fern.vim",
]
---
## fern.vim

[fern.vim](https://github.com/lambdalisue/fern.vim) は Vim のファイルエクスプローラプラグインの一つである。
以前までは NerdTree を使用していたが、最近 fern.vim に移行してみた。
キーマップを NerdTree に合わせることができることもあり、使い勝手が良い。

## ファイルを削除できない

しかし使う中で、ファイルを消すことができないという問題に気づいた。
ファイルを消すキーマップは、デフォルトでは、 `Shift-D` となっている。
`Shift-D` を打鍵すると、ウインドウ下部に以下のメッセージが表示され、
`y` で削除できるはずだった。

```
The following 1 files will be trashed
/path/to/file
Are you sure to continue (Y[es]/no): 
```

しかし、何も起きない。

## :h fern

困ったときは help に頼るべきだ。

help によれば、 fern でファイルを削除する方法は 2 つある。
- fern-action-trash: ファイルを「ゴミ箱」に移す。
- fern-action-remove: ファイルを消去する（`rm` と同じ）。

`Shift-D` は前者を実行するが、そのためには、 以下のように CLI のゴミ箱コマンドが必要であるらしい。

```
*<Plug>(fern-action-trash=)*
	Open a prompt to ask if fern can send the cursor node or marked nodes
	to the system trash-bin. It uses the following implementations to send
	the node(s) into system trash-bin.
	OS		Requirement~
	macOS:		osascript (OS builtin)
	Windows:	PowerShell (OS builtin)
	Linux:		trash-cli or gomi (Users need to install)
			https://github.com/andreafrancia/trash-cli
			https://github.com/b4b4r07/gomi
```

Ubuntu では、通常どおり `apt-get install` で trash-cli をインストールすることができる。

```sh
apt-get update
apt-get install trash-cli
```

## 結果

trash-cli がインストールされた状態で Neovim を起動し直すと、
確かにファイルを削除できるようになった。

```
1 items are trashed
```

## まとめ

困ったらまず help を見よう。
