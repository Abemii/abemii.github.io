---
author: "im"
title: "Is the order a partial formatter for python?"
date: "2022-01-24"
tags: [
    "Neovim",
    "black-macchiato",
    "Python",
    "black"
]
---
> Python コードはブラックであるべきだ、そうコーヒーのように。
> しかしときたま、ある人が不可解な理由でミルクを入れるように言ってくることがある。
> カフェラテは飲めないから、結局妥協してカフェマキアートに落ち着くのだ。

([black-macchiato README](https://github.com/wbolster/black-macchiato) より引用、邦訳。)

## Black is awesome

Python のフォーマッタとしては、以下の3つのツールが人気らしい。[^1]
[^1]: [Pythonのコードフォーマッターについての個人的ベストプラクティス - Qiita](https://qiita.com/sin9270/items/85e2dab4c0144c79987d)

- [autopep8](https://github.com/hhatto/autopep8)
- [yapf](https://github.com/google/yapf)
- [black](https://github.com/psf/black)

なかでも black は特に最近人気があるらしく、自分も使わせてもらっている。

black の良い点として、カスタマイズ性が低さが挙げられる。
これにより、誰が使用しても標準的と考えられるスタイルに修正することができるため、
プロジェクト内でスタイルを統一するのも容易であろう。

しかし、その思想の強さゆえか、ファイルの一部のみの修正は許されていない。
他の（自分の変更箇所ではない）部分は変更したくないなどの理由により、
ファイルの一部（例えば選択範囲）のみをフォーマットしたいといった要望は
自分のみならず多くあると思われるが、それは叶わないようである。

本稿では、（冒涜的かもしれないが）これを叶えるツール [black-macchiato](https://github.com/wbolster/black-macchiato) の Neovim への導入方法と使い方について書く。

## black-macchiato の導入 (Neovim)

black-macchiato は Vim 用のプラグインが公開されているため、それを使うのが良いと思われる。[^2]
[^2]: [GitHub - smbl64/vim-black-macchiato: Vim plugin for black-macchiato integration](https://github.com/smbl64/vim-black-macchiato)

### インストール

まず、 black-macchiato 本体を Neovim 用の仮想環境にインストールする。

```bash
source activate neovim  # virtual env for neovim
pip install black-macchiato
```

Neovim とのインテグレーションには、上述の通り、 vim-black-macchiato を用いる。
基本的には README 通りにインストールすれば問題ない。
自分は [vim-plug](https://github.com/junegunn/vim-plug) を使っているため、次のようにインストールした。

```vim
Plug 'smbl64/vim-black-macchiato'
```

### 設定

こちらも、ほとんど README 通りに設定した。

```vim
let g:black_macchiato_path = fnamemodify( g:python3_host_prog, ':h') . '/black-macchiato'
autocmd FileType python xmap <silent> <buffer> <Leader>f <plug>(BlackMacchiatoSelection)
autocmd FileType python nmap <silent> <buffer> <Leader>f <plug>(BlackMacchiatoCurrentLine)
```

このとき以下の2点について注意した。
- black-macchiato の実行ファイルのパスを指定する
- [coc.nvim](https://github.com/neoclide/coc.nvim) のマップと競合しないようにする

まず、 black-macchiato の実行ファイルのパスの指定については、
上記の仮想環境に black-macchiato がインストールされているとすると、

```bash
<virtual env>/bin/
|-- python
|-- black-macchiato
|-- ...
```

のように配置されているはずなので、 `g:python3_host_prog` 変数から、
そのディレクトリ (`<virtual env>/bin/`) を取得することで、ハードコードを避けられる。

次に、 coc.nvim のマッピングとの競合についてだが、
自分は coc.nvim のフォーマットに関するマップを次のように設定している。

```vim
" Formatting selected code.
xmap <leader>f <Plug>(coc-format-selected)
nmap <leader>f <Plug>(coc-format-selected)
xmap <leader>F <Plug>(coc-format)
nmap <leader>F <Plug>(coc-format)
```

先程の black-macchiato に関するマッピングは、この記述の後に置かないと coc-format-selected のマップにより上書きされてしまう（逆に言えば、 coc-format-selected についてのマップを black-macchiato で上書きできれば良い）。
一つのファイルに設定を書いている場合は単に後に書けば良いだけだが、
複数のファイルに分けて設定を記述している場合は読み込み順について注意が必要だと思われる。

また、読み込み順について、プラグインの読み込みが上記設定の読み込みよりも後になることがあり、
設定が上書きされてしまうことがあるようである。（実際、私の環境では後になっていた）。
そこで、次のようにプラグインの記述を変更してしまうのもありかもしれない。[^3]
[^3]: [g:black_macchiato_path setting is overwritten by plugin · Issue #1 · smbl64/vim-black-macchiato · GitHub](https://github.com/smbl64/vim-black-macchiato/issues/1)

```diff
--- a/plugin/black-macchiato.vim
+++ b/plugin/black-macchiato.vim
@@ -1,7 +1,5 @@
-let g:black_macchiato_path = "black-macchiato"
-
 function s:RunBlackMacchiato() range
-    let cmd = g:black_macchiato_path
+    let cmd = get(g:, 'black_macchiato_path', 'black-macchiato')
     if !executable(cmd)
         echohl ErrorMsg
         echom "black-macchiato not found!"
```

## 動作確認

![動作確認](/images/BlackMacchiato.gif)
