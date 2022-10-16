+++
banner = ""
categories = ["Programming", "Audio"]
date = 2022-05-14
description = "WebAssembly でオーディオ処理を実装する"
images = []
menu = ""
tags = ["WebAssembly", "WebAssembly Text format", "WAT", "Web Audio API"]
title = "Audio Processing by WebAssembly"
disable_comments = false
disable_profile = true
disable_widgets = false
+++

# WebAssembly

**WebAssembly** とは, スタックマシンのための**仮想命令セットアーキテクチャ** (**Virtual ISA**: **V**irtual **I**nstruction **S**et **A**rchitecture) のことで, 「仮想」と命名されているように, 特定の物理的なハードウェアを前提として設計にはなっていません (逆に, 一般的な ISA は特定のハードウェアに依存します). 厳密には異なるかもしれませんが, Java の経験のある方であれば, JVM に近いイメージかもしれません (命名がややこしいのですが, WebAssembly は Virtual ISA でアセンブリ言語ではありません. WebAssembly におけるアセンブリ言語相当の言語が後述する WAT です).

つまり, Web ブラウザだけでなく, Node.js や組み込み環境でも実行可能です. Web ブラウザであれば, モダンブラウザすべてが対応しています (ただ, デバッガーなどが豊富なのは Firefox や Chrome になります).

## WebAssembly のメリット

### 実行パフォーマンスの改善

おそらく, WebAssembly で処理を実装する最大の理由となるでしょう. しかしながら, 一般的にかんちがいされがちなのですが (わたしもその一人でした ...), **WebAssembly で実装しても必ずしも実行パフォーマンスが改善するわけではありません**. また, **実装コストに見合うだけの改善を得られるとも限りません**. 理由としては, **JavaScript から WebAssembly で実装された関数を呼び出すオーバヘッドがそれなりに大きい**ことや, 現状の WebAssembly は, **数値 (32 bit, 64 bit の整数と浮動小数点数) 演算に対して高速であり, 構造化されたデータ (文字列や配列) をあつかうことをあまり得意としていません**. **線形メモリ** (C/C++ でいうヒープ領域, JavaScript でいう ArrayBuffer のようなメモリ領域) を **JavaScript と WebAssembly の間でポインタのように共有して操作する必要がある**ので実装コストが肥大化しがちです. また, **DOM を直接操作することもできない**ので, 実際の開発では, すべてを WebAssembly で書き換えるのは現実的に不可能であり, **最適化する処理を十分に検討する必要があります**.

### 既存ライブラリの活用

もし, C/C++ や Rust, Go のような高級言語で実装して, [Ecmascripten](https://emscripten.org/) などのツールチェーンを利用して, WebAssembly の実行ファイル (wasm) を生成する場合, これまでの資産の多くが活用できます. しかし, 実行パフォーマンスを最大限に引き出したい場合, 後述する WAT や AssemblyScript ほどの最適化を望めないかもしれません. WebAssembly を使う目的を明確にして, このメリットを考える必要はあります.

### 移植性とセキュリティ

WebAssembly は, Web ブラウザで実行するためのテクノロジーとして始まった経緯があり, どこでも実行できるサンドボックス環境へ発展しました. また, 非常に安全なランタイムを作成しています. 現在では, Web ブラウザだけでなく, サーバー上でも, 組み込み環境でも同様にセキュアなランタイムが提供されています.

#### WASI

WASI (**W**eb**A**ssembly **S**ystem **I**nterface) は, WebAssembly のライタイム仕様で, WebAssembly が OS の機能を呼び出すための仕組み, すなわち, WebAssembly 版システムコールのようなものです. 具体的には, WebAssembly から I/O 処理したり, ファイルシステムを使ったりすることが可能になります.

## WebAssembly Text format (WAT)

**WAT** (**W**eb**A**ssembly **T**ext format) は WebAssembly (仮想マシン) におけるアセンブリ言語に相当します.
実プロダクトの生産性を考慮すると, WAT で実装する機会はほとんどないかもしれませんが, 極限までネイティブコードに近いパフォーマンスを発揮したい場合に, WebAssembly がローレベルでなにをしているのかを知っているとパフォーマンスチューニングの生産性を向上できるかもしれません.

さらに, WAT は他のアセンブリ言語と比較すると命令セットは少なく, **数値演算の命令セットが基本です** (加減乗除, 剰余, シフト演算, 比較演算, 論理演算 ... etc). そして, サポートしているデータ型も, i32, i64, f32, f64 (32 bit と 64 bit の整数と浮動小数点数) の 4 種類のみです.
もちろん, 将来のバージョンアップによって, 命令セットやデータ型も増える可能性はありますが, 差分の習得だけですみます
(ちなみに, 数値演算以外の命令セットでよく使う命令セットは, 線形メモリから読み込む `load`, 線形メモリに書き込む `store` 命令があります).

## WebAssembly の開発環境

WebAssembly の開発環境 (ツール) はいくつかありますが, このブログでは, WAT を実行ファイル (wasm) にコンパイルするのみなので, [wat-wasm](https://github.com/battlelinegames/wat-wasm) をインストールします (WAT がコンパイル可能であればなんでもよいでしょう).

```bash
$ npm install -g wat-wasm
```

## Web Audio API と WebAssembly

Web Audio API の API 設計は, Web Audio API が定義する `AudioNode` を接続して高度なオーディオ処理を実装することですが, その設計にしたがうだけでは, 実装できないオーディオ処理も存在します.

- ノイズ
- ボーカルキャンセラ
- ノイズゲート / ノイズサプレッサ
- チャンネルのごとの FFT / IFFT
- ピッチシフター
- タイムストレッチ

例として, 上記にあげたオーディオ処理は, 直接音データにアクセスして, オーディオ処理を実装する必要があります (そのための `AudioNode` が **`ScriptProcessorNode`** (非推奨), **`AudioWorklet`** です).

そのオーディオ処理を JavaScript で実装してもよいのですが, より高速化をはかるのであれば, WebAssembly を利用することで解決できるかもしれません.

## 実装

ボーカルキャンセラとノイズゲートを WebAssembly で実装してみます.

### ボーカルキャンセラ

ボーカルキャンセラのオーディオ処理は単純な数値演算で, 左右のチャンネルの入力の差を出力のサウンドデータにすることで, ボーカルパートを除去することができます (`depth` のパラメータを追加することで, 除去の度合いを調整することもできます).

WAT には, 2 種類の記述スタイルがあり, 1 つがスタックマシンをそのまま記述する**線形命令リスト**スタイル, そして, もう 1 つが, やや高級言語ライクに, 命令セット (オペレータ) と対象のデータ (オペランド) を木構造のように記述する **S式** (**S-expression**) スタイルです.

高級言語に慣れた方には, S 式の方が馴染みやすいかもしれませんが, S 式で記述するにしても, スタックマシン (線形命令リスト) を考慮して実装する必要があるので, 徐々に線形命令リストに慣れていくことをおすすめします.

データ型, 命令セット, 簡易的な制御構文, そして, 線形メモリの操作などは, それなりのボリュームになるので, リファレンスなどを参照していただければと思います.

#### 線形命令リスト

線形命令リストで実装した, ボーカルキャンセラのオーディオ処理です.
スタックで実装可能な, 逆ポーランド記法 (後置記法) に慣れていると, 理解しやすいかもしれません.

```
(module
  (func (export "vocalcanceler") (param $left f32) (param $right f32) (param $depth f32) (result f32)
    local.get $left
    local.get $right
    local.get $depth
    f32.mul
    f32.sub
  )
)
```

#### S 式 (S-expression)

S 式で実装した, ボーカルキャンセラのオーディオ処理です.
線形命令リストと比較すると, やや高級言語に近い記述になります.

```
(module
  (func (export "vocalcanceler") (param $left f32) (param $right f32) (param $depth f32) (result f32)
    (f32.sub (local.get $left) (f32.mul (local.get $depth) (local.get $right)))
  )
)
```

#### コンパイル

`wat-wasm` をインストールすると, WAT から WebAssembly のバイナリ (実行ファイル) にコンパイルする `wat2wasm` コマンドが使えるようになっているので, このコマンドで WAT を wasm にコンパイルします.

```bash
$ wat2wasm vocalcanceler.wat
```

#### wasm の読み込み

Node.js であれば, ファイルシステムにアクセスできるので, wasm を読み込み, 実行ファイルのインタンスを生成して, WebAssembly の関数を呼び出すことができます.

```JavaScript
const fs    = require('fs');
const bytes = fs.readFileSync(`{__dirname}/vocalcanceler.wasm`);

WebAssembly
  .instantiate(new Uint8Array(bytes))
  .then(({ instance }) => {
    // do something ...
  })
  .catch(console.error);
```

Web ブラウザではファイルシステムにアクセすることはできないので, ネットワーク上にアップロードした wasm を `fetch` を利用して, 実行ファイルを読み込んでその戻り値である, `Promise<ArrayBuffer>` を `instantiateStreaming` に渡します.

```JavaScript
WebAssembly
  .instantiateStreaming(fetch('./vocalcanceler.wasm'))
  .then(({ instance }) => {
    // do something ...
  })
  .catch(console.error);
```

#### WebAssembly の関数呼び出し

Web Audio API の `ScriptProcessorNode` を利用して, その関数を呼び出し, ボーカルキャンセラのためのオーディオ処理を実装します. 

```JavaScript
WebAssembly
  .instantiateStreaming(fetch('./vocalcanceler.wasm'))
  .then(({ instance }) => {
    // `processor` は, `ScriptProcessorNode` のインスタンス
    processor.onaudioprocess = (event) => {
      const inputLs  = event.inputBuffer.getChannelData(0);
      const inputRs  = event.inputBuffer.getChannelData(1);
      const outputLs = event.outputBuffer.getChannelData(0);
      const outputRs = event.outputBuffer.getChannelData(1);

      for (let i = 0; i < bufferSize; i++) {
        outputLs[i] = instance.exports.vocalcanceler(inputLs[i], inputRs[i], depth);
        outputRs[i] = instance.exports.vocalcanceler(inputRs[i], inputLs[i], depth);
      }
    };
  })
  .catch(console.error);
```

### ノイズゲート

ノイズゲートのオーディオ処理も単純な数値演算で, 制御構文が必要なになるので, 少し複雑になります.

#### 線形命令リスト

線形命令リストで実装した, ノイズゲートのオーディオ処理です.

```
(module
  (func (export "noisegate") (param $input f32) (param $level f32) (result f32)
    (local $output f32)

    local.get $input
    local.get $level
    f32.gt

    local.get $input
    f32.const 0
    local.get $level
    f32.sub
    f32.lt

    i32.or

    if
      local.get $input
      local.set $output
    else
      f32.const 0
      local.set $output
    end

    local.get $output
  )
)
```

少し余談ですが, 負数をとる場合, `-1` を乗算するよりも, `0` から対象の値を減算する方が高速です.

#### S 式 (S-expression)

S 式で実装した, ノイズゲートのオーディオ処理です.

```
(module
  (func (export "noisegate") (param $input f32) (param $level f32) (result f32)
    (local $output f32)

    (if
      (i32.or
        (f32.gt (local.get $input) (local.get $level))
        (f32.lt (local.get $input) (f32.mul (f32.const -1) (local.get $level)))
      )
      (then
        (local.set $output (local.get $input))
      )
      (else
        (local.set $output (f32.const 0))
      )
    )

    local.get $output
  )
)
```

#### コンパイル

`wat-wasm` をインストールすると, WAT から WebAssembly のバイナリ (実行ファイル) にコンパイルする `wat2wasm` コマンドが使えるようになっているので, このコマンドで WAT を wasm にコンパイルします.

```bash
$ wat2wasm noisegate.wat
```

#### wasm の読み込み

```JavaScript
WebAssembly
  .instantiateStreaming(fetch('./noisegate.wasm'))
  .then(({ instance }) => {
    // do something ...
  })
  .catch(console.error);
```

#### WebAssembly の関数呼び出し

Web Audio API の `ScriptProcessorNode` を利用して, その関数を呼び出し, ノイズゲートのためのオーディオ処理を実装します. 

```JavaScript
WebAssembly
  .instantiateStreaming(fetch('./noisegate.wasm'))
  .then(({ instance }) => {
    // `processor` は, `ScriptProcessorNode` のインスタンス
    processor.onaudioprocess = (event) => {
      const inputLs  = event.inputBuffer.getChannelData(0);
      const inputRs  = event.inputBuffer.getChannelData(1);
      const outputLs = event.outputBuffer.getChannelData(0);
      const outputRs = event.outputBuffer.getChannelData(1);

      for (let i = 0; i < bufferSize; i++) {
        outputLs[i] = instance.exports.noisegate(inputLs[i], level);
        outputRs[i] = instance.exports.noisegate(inputRs[i], level);
      }
    };
  })
  .catch(console.error);
```

## リファレンス

- [入門WebAssembly](https://www.amazon.co.jp/%E5%85%A5%E9%96%80WebAssembly-Rick-Battagline/dp/4798173592)
- [Understanding WebAssembly text format](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format)
