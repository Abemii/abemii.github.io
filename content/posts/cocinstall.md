---
author: "im"
title: "Is the order a well-managed CocInstall?"
date: "2022-01-16"
tags: [
    "Neovim",
    "coc.nvim",
]
---
## 導入

coc.nvim の拡張を管理する方法について。

自分は、Neovim の設定をすべて dotfiles レポジトリで管理している。
そのため、新しく環境構築をする必要がある場合、
```bash
./setup.sh
```
を実行するだけですべてがインストールされる。

coc の拡張についても `setup.sh` 中に

```bash
nvim +'CocInstall coc-xxx coc-yyy' +qa
```
などと書いていた。
（通常の拡張のインストール方法と同じように）

これはこれで良いのだが、 Neovim 関連の設定が `.config/nvim/` の外に出てしまっているのが気持ち悪いし、
ここに追加したとしても、このスクリプトを実行するのは最初の一度きりなので、使い勝手が悪かった。
（そして何よりも、ここに追加するのを忘れることが多かった）

この拡張のリストをどこかで管理しておいて、 
Neovim を起動する度にインストールされているかチェックし、
もしインストールされていなければ自動的にインストールしてほしい。

## 解決方法

これは `init.vim` に以下のように記述することで解決可能である。[^1]
[^1]: coc-settings.json のほうに記述する方法もあるらしいが、試していない。

```vim
let g:coc_global_extensions = [
    \'coc-xxx',
    \'coc-yyy',
    \]
```

Neovim の起動時にこの設定が読み込まれ、もしインストールされていなければ自動的にインストールしてくれる。
嬉しいね。

## 参考文献

- [https://github.com/neoclide/coc.nvim/issues/560:title]
- [https://dev.classmethod.jp/articles/coc-extension-install-by-init-settings/:title]
