# Rubyで作るテキストエディタ

とちぎRuby会議07  2017-08-26




前田 修吾
株式会社ネットワーク応用通信研究所

## 今日のテーマ

* テキストエディタでRubyを作った話

## Textbringer

* ぼくのかんがえたさいきょうのテキストエディタ
* https://github.com/shugo/textbringer
* このスライドを表示しているもの

## 名前の由来

![Stormbringer](stormbringer.jpg)

## Elric Saga

* Michael Moorcockが書いたファンタジー小説群
* 主人公 Elric
    * メルニボネの皇帝
    * 白子 (白髪、白い肌、赤い目) にして病弱
    * 混沌の神Ariochに仕える
* Stormbringer
    * 黒の剣
    * 魂を啜ってElricにその力を与える

## 法と混沌、そして天秤

* 法 (Law)
    * 秩序・規律 → 停滞
* 混沌 (Chaos)
    * 多様性・変化 → 破壊
* 宇宙の天秤 (Cosmic Balance)
    * 法と混沌の釣り合い

## Cosmic Balance

![Balance](balance.png)

## 特徴

* ほぼEmacs
* ターミナルで動く
    * ターミナルでしか動かない
* Pure Ruby

## インストール

    $ gem install textbringer

## 起動

    $ textbringer

## 終了

* Ctrl+X、Ctrl+Cを続けて押す
* Emacsと同じ

## デモ

## ヘルプ

* 特定のコマンド
    * F1 f (コマンド名)
* 特定のキー操作
    * F1 k (調べたいキー)
* キー操作一覧
    * F1 bまたはM-x describe_bindings

## Rubyで作ってよかったこと

* そこそこ作りやすかった
* 一通り作った後も弄りやすい

## 具体例

* バッファ
* 再表示
* コマンド定義

## バッファ

* 編集対象のテキストを表すデータ構造
* 主な要素
    * テキストの内容
    * 現在の編集位置(point)
    * コピー操作のしてんなどに使う位置(mark)

## 代表的実装方式

* ギャップバッファ
    * Emacs
* 行の連結リスト
    * JED

## ギャップバッファ

* ギャップ（隙間）の空いた文字列を用意する

      0 1 2 3 4 5 6 7 8           9 10
      ---------------------------------
      |h|e|l|l|o| |w|o| | | | | |r|l|d|
      ---------------------------------

* 文字を挿入する時はpoint位置にギャップを移動
* 同じ位置に文字を挿入する場合は移動が不要

## 行の連結リスト

* 行の双方向リスト
* 新しい行はリストの適切な位置に挿入される

## 採用方式

* Stringを使ったギャップバッファ
* 理由
    * 組み込みクラスの高速なメソッドを活かせる
    * 行をまたいだ検索などの処理が実装しやすい
    * オブジェクトの数が少なくて済む

## 文字コード

* US-ASCII
    * 日本語が使えなくてつらい
* UTF-32LE/BE
    * 文字列リテラルが使えなくてつらい
* UTF-8
    * 採用

## UTF-8の利点

* 表現できる文字が多い
    * 𩸽 / 远东多线鱼 / 이면수어
* 任意の位置から前後に進める
* 拡張コードを書きやすい
    * default source encoding
    * 様々なライブラリがUTF-8をサポート

## UTF-8の欠点

* 1文字の長さが可変
    * 正確には、符号化された時のバイト列の長さが
      コードポイントによって異なる
* 文字(コードポイント)単位のインデックスアクセスがO(n)

## 対策

* UTF-8のバイト列をASCII-8BITで保持
* バッファ上の位置はバイト単位で扱う
* 必要に応じてUTF-8にforce_encoding

## 拡張性

* Pure Ruby
    * Rubyの動的特徴を活かす
* やろうと思えば表示コードも拡張できる

## プラグイン

* Gemとして実装
    * `gem install textbringer-presentation`
* lib/textbringer_plugin.rbがロードされる

## コマンド定義

```ruby
define_command(:fizzbuzz,
               doc: "Do FizzBuzz from 1 to n.") do
 |n = number_prefix_arg|
  (1..n).each do |i|
    insert ["Fizz"][i%3]
    insert ["Buzz"][i%5]
    insert i if beginning_of_line?
    newline
  end
end
fizzbuzz(15) # メソッドとして呼べる
```

## キーマップ定義

```ruby
GLOBAL_MAP.define_key("\C-xf", :fizzbuzz)
```

## モード定義

```ruby
class ProgrammingMode < FundamentalMode
  define_generic_command :indent_line

  PROGRAMMING_MODE_MAP = Keymap.new
  PROGRAMMING_MODE_MAP.define_key("\t", :indent_line_command)

  def initialize(buffer)
    super(buffer)
    buffer.keymap = PROGRAMMING_MODE_MAP
  end
```

## 今後の構想

* バックグラウンド処理
* MUA

## END
