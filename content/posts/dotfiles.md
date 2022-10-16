+++
banner = ""
categories = ["Programming", "Development"]
date = 2022-05-29
description = "dotfiles を更新しました"
images = []
menu = ""
tags = ["dotfiles", "Zsh", "Vim", "Language Server Protocol", "Tmux"]
title = "Update dotfiles"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# dotfiles

dotfiles をアップデートしたので, 備忘録も兼ねて, 現時点での [my dotfiles](https://github.com/Korilakkuma/dotfiles) の構成をさらします.

ワンライナーインストール可能なように, Make (`/Makefile`) を使います (環境依存性が少なければいいので, Bourne Shell などでもいいでしょう).

```Makefile
DOTFILES_EXCLUDES := .DS_Store .git .gitmodules
DOTFILES_TARGET   := $(wildcard .??*)
DOTFILES_DIR      := $(PWD)
DOTFILES_FILES    := $(filter-out $(DOTFILES_EXCLUDES), $(DOTFILES_TARGET))

deploy            :
	@$(foreach val, $(DOTFILES_FILES), ln -sfnv $(abspath $(val)) $(HOME)/$(val);)

init              :
	bash ./etc/init/install.sh

clean             :
	@$(foreach val, $(DOTFILES_FILES), rm -rf $(HOME)/$(val);)
```

重要なのは, **デプロイ** (ホームディレクトリにシンボリックリンクを作成する処理) と**イニシャライズ** (インストールやビルド) に分けて実行可能にしておくことです.
なぜなら, デプロイは何度も実行する可能性がある, 一方で, イニシャライズはリポジトリをダウンロードしたときにしか実行しないからです.

`/etc` には, プラットフォームごと (現状は, mac のみですが ...) の初期化のフックとなるスクリプトが含まれており,

```bash
$ make init
```

このワンライナーコマンドで, プラットフォームに応じた初期化 (主に, アプリケーションのダウンロードとインストール) が実行されます.

```zsh
cwd=`dirname $0`

case $OSTYPE in
  darwin*)
    echo "Mac OS"
    source $cwd/mac.sh
    ;;
  linux*)
    echo "Linux"
    source $cwd/linux.sh
    ;;
  msys*)
    echo "Windows"
    # Not created ...
    source $cwd/windows.sh
    ;;
  *)
    echo "Not compatible"
    ;;
esac
```

# Zsh

以下, 3 つの設定ごとにファイルを分割し, `/.zsh/init.sh` でまとめて実行します.

```zsh
source $HOME/.zsh/env.sh
source $HOME/.zsh/basic.sh
source $HOME/.zsh/plugins.sh
```

- シェル環境変数の設定 (`/.zsh/env.sh`)
- Zsh の設定・シェル関数・alias など (`/.zsh/basic.sh`)
- プラグインインストール (`/.zsh/plugins.sh`)

`/.zsh/init.sh` は, `/.zshrc` を読み込んだときに実行されます.

```zsh
source $HOME/.zsh/init.sh
```

実は, これまでプラグインをまったく使っていなかったので, Zsh の設定が煩雑だったのですが, プライグインを導入したことで, 必要最低限の設定 (`setopt XXX`) になりました.

```zsh
bindkey -e

autoload -Uz compinit
compinit

setopt auto_cd
setopt auto_pushd
setopt cdable_vars
setopt correct
setopt extended_glob
setopt hist_ignore_all_dups
setopt hist_ignore_dups
setopt hist_ignore_space
setopt hist_reduce_blanks
setopt ignore_eof
setopt interactivecomments
setopt no_flow_control
setopt noautoremoveslash
setopt print_eight_bit
setopt prompt_subst
setopt pushd_ignore_dups
```

プラグインを導入するには, プラグインマネージャが必要になります. [Oh My Zsh](https://ohmyz.sh/) が有名なのかなと思いますが, プラグイン導入によって速さが犠牲になりすぎるのは嫌だったので, [パフォーマンスで優位](https://qiita.com/vintersnow/items/cef72aebb2d55a82f212)そうな, [zplug](https://github.com/zplug/zplug) を使うことにしました.

プラグイン自体も, テーマ (全体の外観), コマンド補完, シンタックスハイライトなど必要最低限の導入にしています.

```zsh
source $ZPLUG_HOME/init.zsh

# command
zplug "zsh-users/zsh-autosuggestions"
zplug "zsh-users/zsh-completions"
zplug "zsh-users/zsh-history-substring-search"
zplug "zsh-users/zsh-syntax-highlighting"

# `cd`
zplug "b4b4r07/enhancd"

# util
zplug "mafredri/zsh-async"

# theme, color
zplug "sindresorhus/pure"
zplug "chrissicool/zsh-256color"

# zplug manage zplug
zplug 'zplug/zplug', hook-build:'zplug --self-manage'

if ! zplug check;
then
  printf "Install? [y/N]: "  # no line break

  if read -q:
  then
    echo ""  # for line break
    zplug install
  fi
fi

zplug load --verbose
```

# Vim

Zsh と同様に, 設定ごとにファイルを分割します. 分割した設定ファイルは `/.vim/init.vim` で読み込まれます.

- Vim の基本設定 (`/.vim/basic.vim`)
- プラグインインストール (`/.vim/plugins.vim`)

今回の dotfiles の更新で最も大きな差分は, Vim の設定ファイルであり, 特に, **Language Server** を利用可能にするプラグインを導入したことによって, コード補完や定義位置ジャンプなどのために, **プログラミング言語ごとに導入していたプラグインを一掃**して, 設定ファイルをシンプルにすることができました.

カラースキーム, ファイラー, git 連携のプラグインなどで, プログラミング言語系のプラグインは LSP のみです.

```
call vundle#begin()

Plugin 'VundleVim/Vundle.vim'

Plugin 'prabirshrestha/vim-lsp'
Plugin 'mattn/vim-lsp-settings'
Plugin 'prabirshrestha/asyncomplete.vim'
Plugin 'prabirshrestha/asyncomplete-lsp.vim'
Plugin 'hrsh7th/vim-vsnip'
Plugin 'hrsh7th/vim-vsnip-integ'

Plugin 'editorconfig/editorconfig-vim'
Plugin 'nathanaelkane/vim-indent-guides'
Plugin 'alvan/vim-closetag'

Plugin 'tpope/vim-fugitive'
Plugin 'airblade/vim-gitgutter'

Plugin 'preservim/nerdtree'
Plugin 'vim-airline/vim-airline'
Plugin 'edkolev/tmuxline.vim'
Plugin 'tpope/vim-surround'

Plugin 'tomasr/molokai'

call vundle#end()
```

## Language Server Protocol (LSP)

Language Server Protocol (LSP) (の実装である [Language Server](https://langserver.org/)) を導入するメリットは, ずばり, これまで, それぞれのエディタや IDE が各々実装してきたコード補完やフォーマットを Language Server で一元管理することによって, これまで, エディタ・IDE ごと, プログラミング言語ごとにあった実装の関係 N : N が, 1 : N の関係で構成されることです (実際, Language Server を利用可能にするプラグインを導入したことで, 個々のプログラミング言語の Vim プラグインが不要になりました).

| C/C++ | TypeScript |
|-------|------------|
| ![C/C++ コード補完](https://user-images.githubusercontent.com/4006693/133930643-28332ed2-7f13-4d72-99b0-9a80ae824cb1.gif) | ![TypeScript コード補完](https://user-images.githubusercontent.com/4006693/133930646-067d9152-14a8-4be6-aa62-e59710a821ad.gif) |

コード補完や定義位置ジャンプといった機能は, プログラミング言語が変わっても本質的に同じです. Language Server を利用しない場合, エディタや IDE の機能とプログラミング言語特有の機能をひとつにして実装されるので, **密結合**になります.

Language Server Protocol は, プログラミング言語とエディタや IDE との間で交換される JSON-RPC 仕様と, それによって実現される機能を指しています. エディタや IDE など, **Language Server Client** から送信される, コード補完や定義位置ジャンプなどの固有の機能要求に対して, 演算を実行して応答を返す **Language Server** の 1 : N の関係で構成されます.

# Tmux

iTerm2 で Tmux を起動してコーディングするので, Tmux の設定ファイル (`/.tmux.conf`) もあります.

```
# 起動時のシェルを zsh にする
set-option -g default-shell /usr/local/bin/zsh

# 256 色表示できるようにする
set-option -g default-terminal screen-256color
set -g terminal-overrides 'xterm:colors=256'

# prefix キー を C-q に変更
set -g prefix C-q

# C-b のキーバインドを解除
unbind C-b

# ステータスバーをトップに配置する
set-option -g status-position top

# 左右のステータスバーの長さを決定する
set-option -g status-left-length 90
set-option -g status-right-length 90

# #P ペイン番号
# 最左に表示
set-option -g status-left '#H:[#P]'

# WiFi, バッテリー残量, 現在時刻
# 最右に表示
set-option -g status-right '#(wifi) #(battery --tmux) [%Y-%m-%d(%a) %H:%M]'

# ステータスバーを 1 sec 間隔で再描画
set-option -g status-interval 1

# センタリング
set-option -g status-justify centre

# ステータスバーの色
set-option -g status-bg "colour238"

# ステータスラインの文字色
set-option -g status-fg "colour255"

# ペインの移動 (Vim のキーバンドに変更)
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# ペインのリサイズ (Vim のキーバンドに変更)
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# ペインを垂直方向に分割
bind | split-window -h

# ペインを水平方向に分割
bind - split-window -v

# 番号基準値を変更
set-option -g base-index 1

# マウス操作を有効にする
set-option -g mouse on
bind -n WheelUpPane if-shell -F -t = "#{mouse_any_flag}" "send-keys -M" "if -Ft= '#{pane_in_mode}' 'send-keys -M' 'copy-mode -e'"

# コピーモードを設定する, Vim キーバインドを使う
setw -g mode-keys vi

# 選択
bind -T copy-mode-vi v send -X begin-selection

# 行選択
bind -T copy-mode-vi V send -X select-line

# 矩形選択
bind -T copy-mode-vi C-v send -X rectangle-toggle

# ヤンク
bind -T copy-mode-vi y send -X copy-selection

# 行ヤンク
bind -T copy-mode-vi Y send -X copy-line

# ペースト
bind-key C-p paste-buffer
```

# リファレンス

- [最強の dotfiles 駆動開発と GitHub で管理する運用方法](https://qiita.com/b4b4r07/items/b70178e021bef12cd4a2)
- [Langserver.org](https://langserver.org/)
- [LSP / LSIF](https://microsoft.github.io/language-server-protocol/implementors/servers/)
- [Vim をモダンな IDE に変える LSP の設定](https://mattn.kaoriya.net/software/vim/20191231213507.htm)
- [vim-lsp の導入コストを下げるプラグイン vim-lsp-settings を書いた。](https://qiita.com/mattn/items/e62b9f16bc487a271a7f)
- [tmuxを必要最低限で入門して使う](https://qiita.com/shin-ch13/items/9d207a70ccc8467f7bab)
