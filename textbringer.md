# Rubyで作るテキストエディタ

とちぎRuby会議07  2017-08-26




前田 修吾
株式会社ネットワーク応用通信研究所

## 今日のテーマ

* Rubyで作ったテキストエディタの話

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

* Emacs風
* 端末上で動く
    * 端末上でしか動かない
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

## Rubyで作ってみて

* 簡単に作れた
* 作った後も弄りやすい
* Ruby意外と速い

## 具体例

* バッファ
* 再表示
* 拡張

## バッファ

* 編集対象のテキストを表すデータ構造
* 主な要素
    * テキストの内容
    * 現在の編集位置(point)
    * コピー操作のしてんなどに使う位置(mark)

## Bufferオブジェクト

```ruby
buffer = Buffer.current
buffer.insert("Hello world\n")
buffer.re_search_backward(/world/)
buffer.replace_match("Tochigi")
```

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
    * 不正なバイト列がなければ
* 拡張コードを書きやすい
    * default source encoding
    * 様々なライブラリがUTF-8をサポート

## UTF-8の欠点

* n文字目のアクセスに先頭からスキャンが必要

## 対策

* UTF-8のバイト列をASCII-8BITで保持
* バッファ上の位置はバイト単位で扱う
* 必要に応じてUTF-8にforce_encoding

## Stringはバッファ向き?

* ランダムアクセスをあまり考慮していない
    * 先頭からシーケンシャルに処理する用途が多い
* やや無理をして使っている
* それでも自前で頑張るより速いのでは…

## 再表示

* 画面全体を再表示する必要はない
* しかし、実装が面倒
* cursesの機能を使う

## curses

* 端末制御ライブラリ
* Rubyでは同名のgemで利用可能
    * 自作のgemで以前は標準添付だった
* Curses::Window#eraseでウィンドウの内容を消去
* Curses.doupdateで必要な箇所だけ再表示される

## 拡張

* Rubyの動的特徴を活かす

## Rubyコードの実行

* eval_expression
* eval_buffer
* eval_region

## eval_expression

* ミニバッファで入力したコードを実行
* 評価結果をエコーエリアに表示

## eval_buffer

* バッファの内容をRubyコードとして実行

## eval_region

* 選択された領域の内容をRubyコードとして実行

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

## プラグイン

* lib/textbringer_plugin.rbがロードされる
* gemとしてインストール
    * `gem install textbringer-presentation`
* あるいは~/.textbringer/plugins以下に置く

## プラグインの例

* GhostTextプラグイン
    * ブラウザ上のテキストエリアを編集

## GhostTextプロトコル

* WebSocketで以下のようなJSONを送受信

      {
        "text": "hello world",
        "selections": [{"start": 0, "end": 0}],
        "title": "test",
        "url": "example.com",
        "syntax": ""
      }

## デモ

## サーバ起動コマンド

```ruby
define_command(:ghost_text_start,
               doc: "Start GhostText server") do
  host = CONFIG[:ghost_text_host]
  port = CONFIG[:ghost_text_port]
  message("Start GhostText server: http://#{host}:#{port}")
  background do
    thin = Rack::Handler.get("thin")
    app = Rack::ContentLength.new(Textbringer::GhostText::Server.new)
    thin.run(app, Host: host, Port: port) do |server|
      server.silent = true
    end
  end
end
```

## テキストの同期

```ruby
syncing_from_remote_text = false

ws.on :message do |event|
  data = JSON.parse(event.data)
  next_tick do
    syncing_from_remote_text = true
    begin
      buffer.replace(data["text"])
      if pos = data["selections"]&.dig(0, "start")
        byte_pos = data["text"][0, pos].bytesize
        buffer.goto_char(byte_pos)
      end
    ensure
      syncing_from_remote_text = false
    end
    if (title = data['title']) && !title.empty?
      buffer.name = "*GhostText:#{title}*"
    end
    switch_to_buffer(buffer)
  end
end
```

## 詳細

* GhostTextプラグインの作成 - NaCl非公式ブログ
    * http://nacl-ltd.github.io/2017/07/08/textbringer-ghost_text.html

## Rubyの利点

* 起動中のテキストエディタを拡張できる
    * eval
    * オープンクラス

## Rubyの欠点

* 壊しやすい
    * ちょっとしたtypoで動かなくなってVimを使ったり
* テストを書くようにした

## 今後の予定

* MUAを開発中
* RubyKaigiで発表予定
        
## END
