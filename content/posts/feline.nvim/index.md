---
author: "im"
title: "feline.nvim: is the order a cool and fast status bar?"
date: "2022-02-12"
tags: [
    "Neovim",
    "feline.nvim"
]
share: true
---

## 序

私の Neovim は起動時間が長い。 180 ms もかかっていた。
1 秒もかかっていないので、気にするほどのことでもないのかもしれないが、
生の Neovim が 5ms 程度で起動することを考えると、速さを追い求めたくなってしまう。
速さにこそ本質がある。

起動・動作の高速化のためにいくつかのことを行ってみたが、
ここではステータスバーを [vim-airline](https://github.com/vim-airline/vim-airline) から
[feline.nvim](https://github.com/feline-nvim/feline.nvim) に変更してみた。 

もともと airline はなんかかっこいいという程度の理由で使っており、
大した設定もしていなかった。よく意味もわからず表示していたものもあり、今回はこれを見直すことにした。

feline.nvim は lua で書かれたプラグインで、 vimscript で書かれたものよりも高速に動作することが期待できる。
lua で書かれたステータスバープラグインは他にもあるが、 なんとなく feline.nvim を使うことにした。

結果として、 Neovim の起動時間が半分程度になり（ 118ms 程度 → 64ms 程度）、
なかなか良い見た目のステータスバーとなった。

本稿では、 vim-airline と feline.nvim の起動時間の比較と、私の設定を紹介する。

## 環境

### PC

- OS: Ubuntu 20.04.3 LTS
- CPU: Intel(R) Core(TM) i7-9700K CPU @ 3.60GHz
- RAM: DIMM DDR4 Synchronous 2133 MHz x2 (32GB)

### Neovim

- NVIM v0.6.1
- feline.nvim v0.4.3

## feline.nvim の導入

[README](https://github.com/feline-nvim/feline.nvim) に書かれている通りにインストールする。
私の場合は、 vim-plug を使用しているので、以下のようにインストールした。

```vim
Plug 'feline-nvim/feline.nvim'
```

ステータスバーに表示されている内容は以下の通りである：

- Vim mode (NORMAL とか INSERT とか)
- File type (python とか cpp とか)
- File name
- Git branch, add, change, remove
- Diagnostics
- File OS 
- Cursor position and percentage in file

{{< figure src="figures/feline.nvim.png" class="center" width="100%" >}}

## feline.nvim の設定

プラグインの設定は lua で記述する必要がある。
README のリンクにあった[この例](https://github.com/ibhagwan/nvim-lua/blob/main/lua/plugins/feline.lua)を参考に設定を記述した。

しかし、この例では Diagnostics の情報を builtin lsp からとるようになっており、私が使用している coc.nvim には対応していない。
そのため、 coc.nvim の Diagnostics の情報をとってくる部分は自分で書く必要があった。[^lua]
[^lua]: lua を触るのは初めてだったので、よくわからずに書いた。

実装をみる限りでは、builtin lsp の各 severity の個数をとってきて表示しているようなので、
これを coc.nvim から取得できるように実装すれば良い。

実装すべきなのは以下の 2 つ:

- 各 severity の個数の表示
    - (info, warning, error, hint) について、その個数を表示する。
- 表示が必要かどうかの判定
    - もし、 diagnostics の情報がとれない、あるいは、個数が 0 であれば何も表示しない。

coc.nvim の diagnostics の情報は、辞書型の変数 `b:coc_diagnostic_info` に格納されている。
よって、この変数を lua で読み取るような関数を実装する。

```lua
-- ある severity の diagnostics が存在するかを判定
local function diagnostics_exist(severity)
    local info = vim.b.coc_diagnostic_info
    if (not info) then
        return false
    else
        local count = info[severity]
        if count > 0 then
            return true
        else
            return false
        end
    end
end

-- ある severity の diagnostics の個数を返す
local lsp_get_diag = function(str)
    local info = vim.b.coc_diagnostic_info
    if (not info) then
        return ''
    else
        local count = info[str]
        if (not count) then
            return ' 0 '
        else
            return ' '..count..' '
        end
    end
end
```

以上の設定は[ここ](https://github.com/Abemii/dotfiles/blob/master/nvim/cfgs/feline.lua)にある。

## 起動時間の比較

ここでは vim-airline と feline.nvim を読み込んだときの起動時間を比較する。
使用した `init.vim` は次の通り。

- vim-airline

```vim
filetype off

call plug#begin('/tmp/nvim/plugged_airline')
Plug 'vim-airline/vim-airline'
call plug#end()

set termguicolors

filetype plugin indent on
```

- feline.nvim

```vim
filetype off

call plug#begin('/tmp/nvim/plugged_feline')
Plug 'feline-nvim/feline.nvim'
call plug#end()

set termguicolors

lua << EOF
require('feline').setup()
EOF

filetype plugin indent on
```

それぞれ `:PlugInstall` を行った後、 10 回ずつ起動して `startuptime` を記録し、その起動時間の平均をとった。

```bash
# log startuptime
$ repeat 10 nvim -u init_feline.vim --startuptime /tmp/startuptime_feline.log
$ repeat 10 nvim -u init_airline.vim --startuptime /tmp/startuptime_airline.log

# calculate average
$ cat /tmp/startuptime_airline.log | grep "NVIM STARTED" | awk '{m+=$1} END{print m/NR;}'   
57.5417
$ cat /tmp/startuptime_feline.log | grep "NVIM STARTED" | awk '{m+=$1} END{print m/NR;}'
19.2784
```

確かに、 feline.nvim のほうがだいぶ起動時間が短いことがわかる。

## 結

本稿では、 vim-airline から feline.nvim に変更してみた結果を紹介した。
また、 feline.nvim で coc.nvim の diagnostics 情報を読み取る方法も書いた。

## 参考

- feline.nvim
    - [久しぶりにNeovimの設定を見直してみる｜kyoh86](https://zenn.dev/kyoh86/articles/8f6d070bc4be20)
    - [feline.nvim を使ってみる](https://scrapbox.io/tamago324vim/feline.nvim_%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B)
    - [Config by iBhagwan](https://github.com/ibhagwan/nvim-lua/blob/main/lua/plugins/feline.lua)
- lua
    - [NeovimとLua｜hituzi no sippo](https://zenn.dev/hituzi_no_sippo/articles/871c06cdbc45b53181e3)
    - [Getting started using Lua in Neovim](https://github.com/willelz/nvim-lua-guide-ja/blob/master/README.ja.md)
- 私の設定: https://github.com/Abemii/dotfiles/blob/master/nvim/cfgs/feline.lua
