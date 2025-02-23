## 導入

これまでジュークボックスのコードの一部を実装して楽しんできましたが、少し怠っていた部分があります。
配列、ハッシュ、プロックについては触れましたが、Rubyの他の基本的な型―数値、文字列、範囲、正規表現―については十分に扱ってきませんでした。
それらの基本的な構成要素について、これから少し、このページで学んでいきましょう。

> 1. [数値](#num)
> 2. [文字列](#str)
> 3. [範囲](#ran)
> 4. [正規表現](#reg)

<h2 id="num">1. 数値</h2>

Rubyは**整数**と**浮動小数点数**をサポートしています。

> ### 整数

整数は任意の長さを持つことができます（システムの空きメモリによって最大値が決まります）。

> #### Fixnum

一定の範囲内の整数（通常は-230から230-1または-262から262-1）は内部的に2進数形式で保持され、`Fixnum`クラスのオブジェクトとして扱われます。

> #### Bignum

この範囲を超える整数は、`Bignum`クラスのオブジェクトとして格納されます（現在は可変長の短整数のセットとして実装されています）。

この変換は透明で、Rubyは自動的に変換を管理します。

```rb
num = 8
7.times do
  print num.type, " ", num, "\n"
  num *= num
end
```
出力：
```rb
Fixnum 8
Fixnum 64
Fixnum 4096
Fixnum 16777216
Bignum 281474976710656
Bignum 79228162514264337593543950336
Bignum 6277101735386680763835789423207666416102355444464034512896
```

> #### 整数のチップス

- 整数は、オプションで以下のものを先頭に付けることができます。その後に適切な基数の数字を並べることで記述できます。
    - 符号
    - 基数の指示子（`0`は8進数、`0x`は16進数、`0b`は2進数）

- 数字の文字列内のアンダースコアは無視されます。

```rb
123456                    # Fixnum
123_456                   # Fixnum（アンダースコアは無視される）
-543                      # 負のFixnum
123_456_789_123_345_789   # Bignum
0xaabb                    # 16進数
0377                      # 8進数
-0b101_010                # 2進数（負）
```

ASCII文字やエスケープシーケンスに対応する整数値を取得するためには、その前に`?`を付けます。

制御文字やメタ文字の組み合わせも、`?\C-x`、`?\M-x`、`?\M-\C-x`を使って生成できます。

- 制御バージョンの値は`value & 0x9f`
- メタバージョンの値は`value | 0x80`

で生成されます。

`?\C-?`はASCIIの削除（0177）を生成します。

```rb
?a                        # 文字コード
?\n                       # 改行コード (0x0a)
?\C-a                     # 制御文字a = ?A & 0x9f = 0x01
?\M-a                     # メタ文字a（ビット7をセット）
?\M-\C-a                  # メタと制御のa
?\C-?                     # 削除文字
```

> ### 浮動小数点数

小数点や指数を含む数値リテラルは、`Float`オブジェクトに変換され、これはネイティブアーキテクチャの倍精度浮動小数点型に対応します。

小数点の後には必ず数字が必要です。例えば、`1.e3`は`Fixnum`クラスのメソッド`e3`を呼び出そうとしてしまうためエラーになります。

> ### まとめ

すべての数値はオブジェクトであり、さまざまなメッセージに応答します（詳細は290ページ、313ページ、315ページ、323ページ、349ページで説明されています）。そのため、例えばC++のように数値の絶対値を求める場合、`aNumber.abs`と書きます。`abs(aNumber)`とは書きません。

---

整数はまた、いくつかの便利なイテレータをサポートしています。すでに1つ見ました――7.timesは47ページのコード例で使われています。他にも、2つの整数間で上下に反復処理を行うuptoやdownto、従来のforループに近いstepがあります。
```
3.times        { print "X " }
1.upto(5)      { |i| print i, " " }
99.downto(95)  { |i| print i, " " }
50.step(80, 5) { |i| print i, " " }
```
出力：
```
X X X 1 2 3 4 5 99 98 97 96 95 50 55 60 65 70 75 80
```
最後に、Perlユーザーへの警告です。数字を含む文字列は、式で使用される際に自動的に数値に変換されません。これは、ファイルから数字を読み取るときに最もよく問題になります。次のコードは（おそらく）意図した動作をしません。
```rb
DATA.each do |line|
  vals = line.split    # 行を分割し、トークンをvalsに格納
  print vals[0] + vals[1], " "
end
```
ファイルの内容が以下である場合：
```
3 4
5 6
7 8
```
出力は次のようになります：
```
34 56 78
```
何が起こったのでしょうか？

問題は、入力が数値ではなく文字列として読み込まれたことです。プラス演算子は文字列を連結するので、その結果としてそのような出力が得られます。これを修正するには、String#to_iメソッドを使って文字列を整数に変換する必要があります。
```rb
DATA.each do |line|
  vals = line.split
  print vals[0].to_i + vals[1].to_i, " "
end
```
出力：
```rb
7 11 15
```

<h2 id="str">2.文字列</h2>
Rubyの文字列は、単純に8ビットバイトのシーケンスです。通常は印刷可能な文字を保持しますが、それは必須ではなく、文字列はバイナリデータも保持できます。文字列はStringクラスのオブジェクトです。

文字列は、文字列リテラル（区切り文字で囲まれた文字のシーケンス）を使ってよく作成されます。プログラムのソース内でバイナリデータを表現するのが難しいため、文字列リテラルにはさまざまなエスケープシーケンスを挿入できます。これらは、プログラムがコンパイルされる際に対応するバイナリ値に置き換えられます。文字列の区切り文字の種類によって、行われる置換の度合いが決まります。シングルクォーテーションで囲まれた文字列内では、2つの連続したバックスラッシュが1つのバックスラッシュに置き換えられ、バックスラッシュの後にシングルクォーテーションが続くと、シングルクォーテーションになります。
```rb
'escape using "\"'  →  escape using "" 'That's right'  →  That's right
```
ダブルクォーテーションで囲まれた文字列では、さらに多くのエスケープシーケンスをサポートしています。最も一般的なのはおそらく\n、つまり改行文字です。完全なリストはページ203の表18.2に記載されています。また、#{ expr }の形式で、文字列に任意のRuby式の値を埋め込むことができます。式がグローバル変数、クラス変数、インスタンス変数だけの場合は、波括弧を省略できます。
```rb
"Seconds/day: #{246060}"  →  Seconds/day: 86400
"#{'Ho! '*3}Merry Christmas"  →  Ho! Ho! Ho! Merry Christmas
"This is line #$."  →  This is line 3
```
文字列リテラルを構築する方法は他にも3つあります：%q、%Q、および「ヒアドキュメント」です。

`%q`と`%Q`は、それぞれシングルクォーテーションとダブルクォーテーションで囲まれた文字列を作成します。
```rb
%q/general single-quoted string/  →  general single-quoted string
%Q!general double-quoted string!  →  general double-quoted string
%Q{Seconds/day: #{246060}}  →  Seconds/day: 86400
```
qやQの後に続く文字が区切り文字です。区切り文字が開き括弧、波括弧、丸括弧、または小なり記号の場合、対応する閉じる記号が見つかるまで文字列が読み込まれます。それ以外の場合は、次に同じ区切り文字が現れるまで文字列が読み込まれます。

最後に、ヒアドキュメントを使って文字列を構築できます。
```rb
aString = <<END_OF_STRING
    The body of the string
    is the input lines up to
    one ending with the same
    text that followed the '<<'
END_OF_STRING
```
ヒアドキュメントは、ソース内の行を指定した終了文字列まで読み込みますが、終了文字列自体は含まれません。通常、この終了文字列は最初の列から始める必要があります。しかし、<<の後にハイフンを付けると、終了文字列をインデントすることができます。
```rb
print <<-STRING1, <<-STRING2
   Concat
   STRING1
      enate
      STRING2

出力は次の通りです：

Concat
        enate
```
### 文字列の操作
文字列はおそらく最も多くの組み込みメソッドを持つRubyのクラスで、標準メソッドは75以上あります。すべてのメソッドをここで紹介するわけではなく、ライブラリリファレンスに完全なリストがあります。代わりに、日常的にプログラミング中に登場しやすい一般的な文字列の使い方（イディオム）について見ていきます。

さて、私たちのジュークボックスに戻りましょう。ジュークボックスはインターネットに接続できるように設計されていますが、人気のある曲のいくつかをローカルハードドライブにも保存しています。これで、もしリスがネット接続をかじってしまっても、まだ顧客を楽しませることができます。

歴史的な理由（他に理由があるでしょうか？）で、曲のリストはフラットファイルとして行ごとに保存されています。各行には、曲を含むファイル名、曲の再生時間、アーティスト、タイトルが縦棒で区切られたフィールドに格納されています。典型的なファイルは次のようになっています：
```rb
/jazz/j00132.mp3  | 3:45 | Fats     Waller     | Ain't Misbehavin'
/jazz/j00319.mp3  | 2:58 | Louis    Armstrong  | Wonderful World
/bgrass/bg0732.mp3| 4:09 | Strength in Numbers | Texas Red
         :                  :           :                   :
```
このデータを見ると、これらのフィールドを抽出して整形するために、Stringクラスのメソッドを使う必要があることがわかります。最低限、次の作業が必要です：

- 行をフィールドに分割すること、
- 再生時間をmm:ssから秒数に変換すること、
- アーティスト名の余分な空白を取り除くこと。


最初のタスクは、各行をフィールドに分割することで、String#splitメソッドを使うと簡単にできます。この場合、分割する正規表現は、縦棒の周りにオプションでスペースがあるパターンである/\s*|\s*/です。ファイルから読み込んだ行には改行が含まれているので、String#chompを使って改行を取り除いてからsplitを適用します。
```rb
songs = SongList.new

songFile.each do |line|
  file, length, name, title = line.chomp.split(/\s*\|\s*/)
  songs.append Song.new(title, name, length)
end
puts songs[1]
```
出力：
```rb
Song: Wonderful World--Louis Armstrong (2:58)
```
残念ながら、元のファイルを作成した人はアーティスト名を列に分けて入力したため、いくつかの名前に余分な空白があります。これらの空白は、私たちのハイテクなスーパーツイスト型フラットパネルDay-Gloディスプレイで見栄えが悪くなるので、作業を進める前にこれらの空白を取り除く必要があります。これにはいくつか方法がありますが、最もシンプルなのはString#squeezeを使って連続した文字の繰り返しを取り除く方法です。ここでは、文字列をその場で変更するsqueeze!メソッドを使います。
```rb
songs = SongList.new
```
songFile.each do |line|
  file, length, name, title = line.chomp.split(/\s*\|\s*/)
  name.squeeze!(" ")
  songs.append Song.new(title, name, length)
end
puts songs[1]
```
出力：
```rb
Song: Wonderful World--Louis Armstrong (2:58)
```
次に、時間形式の問題があります：ファイルでは「2:58」と表示されており、これを秒数で「178」に変換したいと考えています。再度splitを使って、コロンで時間を分割する方法もありますが、ここでは関連するメソッドString#scanを使用します。scanはsplitに似ていますが、文字列をパターンに基づいてチャンクに分けるために使用します。今回は、分と秒の両方に一致する1つ以上の数字を取得したいので、パターンは/\d+/です。
```rb
songs = SongList.new
songFile.each do |line|
  file, length, name, title = line.chomp.split(/\s*\|\s*/)
  name.squeeze!(" ")
  mins, secs = length.scan(/\d+/)
  songs.append Song.new(title, name, mins.to_i*60 + secs.to_i)
end
puts songs[1]
```
出力：
```rb
Song: Wonderful World--Louis Armstrong (178)
```
さらに、ジュークボックスにはキーワード検索機能があります。曲のタイトルやアーティスト名の単語を入力すると、すべての一致するトラックをリスト表示します。「fats」と入力すると、例えばFats Domino、Fats Navarro、Fats Wallerの曲が表示されるかもしれません。この機能は、インデックス作成クラスを作成することで実装します。オブジェクトといくつかの文字列を与えると、そのオブジェクトを、文字列の中で出現するすべての2文字以上の単語の下でインデックスします。これにより、Stringクラスの他のメソッドをいくつか利用することができます。
```rb
class WordIndex
  def initialize
    @index = Hash.new(nil)
  end
  def index(anObject, *phrases)
    phrases.each do |aPhrase|
      aPhrase.scan /\w[-\w']+/ do |aWord|   # 各単語を抽出
        aWord.downcase!
        @index[aWord] = [] if @index[aWord].nil?
        @index[aWord].push(anObject)
      end
    end
  end
  def lookup(aWord)
    @index[aWord.downcase]
  end
end
```
`String#scan`メソッドは、文字列から正規表現に一致する要素を抽出します。ここでは、パターン\w[-\w']+が、単語の中に現れる可能性のある任意の文字の後に、ハイフン、別の単語の文字、またはアポストロフィが1回以上続く部分をマッチします。検索を大文字小文字を区別しないようにするため、抽出した単語と検索時に使う単語をすべて小文字に変換します。最初のdowncase!メソッドの名前の末尾に付いている感嘆符は、このメソッドが受け取った文字列をその場で変更することを示しています。

次に、SongListクラスを拡張して、曲を追加する際にインデックスを作成し、単語を使って曲を検索できるメソッドを追加します。

class SongList
  def initialize
    @songs = Array.new
    @index = WordIndex.new
  end
  def append(aSong)
    @songs.push(aSong)
    @index.index(aSong, aSong.name, aSong.artist)
    self
  end
  def lookup(aWord)
    @index.lookup(aWord)
  end
end

最後に、すべてをテストします。

songs = SongList.new
songFile.each do |line|
  file, length, name, title = line.chomp.split(/\s*\|\s*/)
  name.squeeze!(" ")
  mins, secs = length.scan(/\d+/)
  songs.append Song.new(title, name, mins.to_i*60 + secs.to_i)
end
puts songs.lookup("Fats")
puts songs.lookup("ain't")
puts songs.lookup("RED")
puts songs.lookup("WoRlD")

出力：

Song: Ain't Misbehavin'--Fats Waller (225)
Song: Ain't Misbehavin'--Fats Waller (225)
Song: Texas Red--Strength in Numbers (249)
Song: Wonderful World--Louis Armstrong (178)

RubyのStringクラスにあるすべてのメソッドを紹介するには次の50ページを費やせますが、ここでは次に進んで、より簡単なデータ型である「範囲」について見ていきます。


a = "The moon is made of cheese"
showRE(a, /\w+/)     # => <<The>> moon is made of cheese
showRE(a, /\s.*\s/)   # => The<< moon is made of >>cheese
showRE(a, /\s.*?\s/)  # => The<< moon >>is made of cheese
showRE(a, /[aeiou]{2,99}/)  # => The m<<oo>>n is made of cheese
showRE(a, /mo?o/)     # => The <<moo>>n is made of cheese

<h2 id="ran">3. 範囲</h2>

範囲（Ranges）は至る所に現れます：1月から12月、0から9、レアからウェルダン、50行目から67行目まで、などです。もしRubyが現実をモデル化する手助けをするのであれば、範囲をサポートするのは自然なことです。実際、Rubyはさらに一歩進んで、範囲を使って3つの異なる機能を実装しています：シーケンス、条件、そしてインターバルです。

### 範囲（Ranges）をシーケンスとして使う 

範囲の最初であり、おそらく最も自然な使い方は、シーケンスを表現することです。シーケンスには開始点、終了点があり、シーケンスの中で次々に値を生成する方法があります。Rubyでは、このシーケンスは「..」と「...」という範囲演算子を使って作成されます。二重ドット形式は包括的な範囲を作成し、三重ドット形式は指定した最大値を除外する範囲を作成します。

1..10
'a'..'z'
0...anArray.length

Rubyでは、以前のPerlのバージョンとは異なり、範囲は内部的にリストとして表現されません。たとえば、シーケンス 1..100000 は、2つのFixnumオブジェクトへの参照を含むRangeオブジェクトとして保持されます。必要であれば、範囲をリストに変換することができます。

(1..10).to_a    # => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
('bar'..'bat').to_a  # => ["bar", "bas", "bat"]

範囲は、それを反復処理したり、その内容をさまざまな方法でテストしたりするためのメソッドを実装しています。

digits = 0..9
digits.include?(5)  # => true
digits.min          # => 0
digits.max          # => 9
digits.reject { |i| i < 5 }  # => [5, 6, 7, 8, 9]
digits.each do |digit|
  dial(digit)
end

これまでに数字や文字列の範囲を示しましたが、オブジェクト指向言語であるRubyでは、ユーザー定義のオブジェクトを基にした範囲を作成することもできます。オブジェクトが必要とする制約は、次のオブジェクトを返すsuccメソッドに対応し、<=>（比較演算子）を使ってオブジェクトが比較できることです。<=>は、2つの値を比較し、最初の値が2番目の値より小さい場合は-1、等しい場合は0、大きい場合は+1を返します。

以下は、#記号の行を表す簡単なクラスです。これを使って、ジュークボックスの音量調整をテストする際に、テキストベースのスタブとして利用することができます。

class VU
  include Comparable

  attr :volume

  def initialize(volume)  # 0..9
    @volume = volume
  end

  def inspect
    '#' * @volume
  end

  # 範囲のサポート

  def <=>(other)
    self.volume <=> other.volume
  end

  def succ
    raise(IndexError, "Volume too big") if @volume >= 9
    VU.new(@volume.succ)
  end
end

このクラスを使って、VUオブジェクトの範囲を作成し、テストすることができます。

medium = VU.new(4)..VU.new(7)
medium.to_a    # => [####, #####, ######, #######]
medium.include?(VU.new(3))  # => false

### 範囲（Ranges）を条件として使う

範囲はシーケンスを表現するだけでなく、条件式としても使用できます。たとえば、以下のコードは標準入力から行のセットを印刷します。各セットの最初の行には「start」という単語が含まれ、最後の行には「end」という単語が含まれます。

while gets
  print if /start/../end/
end

内部では、範囲が各テストの状態を追跡しています。この動作については、ページ82以降で説明されるループの例で示します。

### 範囲（Ranges）を区間として使う

範囲のもう一つの用途は、区間テストとして使うことです。これは、ある値が範囲が表す区間内に含まれているかどうかを調べるために行います。これには、ケース等価演算子 === を使用します。

(1..10)    === 5     # => true
(1..10)    === 15    # => false
(1..10)    === 3.14159  # => true
('a'..'j') === 'c'   # => true
('a'..'j') === 'z'   # => false

ページ81のケース式の例では、このテストが実際に動作して、年によってジャズのスタイルを決定する様子が示されています。

<h2 id="reg">4. 正規表現</h2>

ページ50で、ファイルから曲のリストを作成する際に、入力ファイル内のフィールド区切りを一致させるために正規表現を使用しました。line.split(/\s*\|\s*/) がオプションの空白で囲まれた縦棒（|）に一致することを説明しました。ここでは、この主張がなぜ正しいのか、正規表現について詳しく見ていきます。

正規表現は文字列に対してパターンを一致させるために使用されます。Rubyはパターン一致や置換を便利かつ簡潔にするための組み込みサポートを提供しています。このセクションでは、正規表現の主要な機能をすべて扱います。詳細については、ページ205をご覧ください。

正規表現は Regexp 型のオブジェクトです。これらは、コンストラクタを明示的に呼び出すか、リテラル形式 /pattern/ や %r\pattern\ を使って作成できます。

a = Regexp.new('^\s*[a-z]')    # => /^\s*[a-z]/
b = /^\s*[a-z]/                # => /^\s*[a-z]/
c = %r{^\s*[a-z]}              # => /^\s*[a-z]/

正規表現オブジェクトを作成したら、Regexp#match(aString) やマッチ演算子 =~（肯定的マッチ）と !~（否定的マッチ）を使用して、文字列に対してマッチさせることができます。マッチ演算子は String と Regexp の両方のオブジェクトに対して定義されています。もしマッチ演算子の両方のオペランドが文字列の場合、右辺は正規表現に変換されます。

a = "Fats Waller"
a =~ /a/      # => 1
a =~ /z/      # => nil
a =~ "ll"     # => 7

マッチ演算子は、一致が発生した文字位置を返します。また、いくつかのRubyの変数を設定する副作用もあります。$& にはパターンに一致した部分、$`` には一致の前の部分、$'には一致の後の部分が格納されます。これを利用して、特定のパターンがどこに一致するかを示すメソッドshowRE` を作成できます。

def showRE(a,re)
  if a =~ re
    "#{$`}<<#{$&}>>#{$'}"
  else
    "no match"
  end
end

show

### パターン

すべての正規表現にはパターンが含まれており、このパターンは正規表現を文字列に対して一致させるために使用されます。パターン内では、.、|、(、)、[、{、+、\、^、$、*、? 以外のすべての文字は、自身と一致します。

showRE('kangaroo', /angar/)   # => k<<angar>>oo
showRE('!@%&-_=+', /%&/)      # => !@<<%&>>-_=+

これらの特殊文字を文字通り一致させたい場合、バックスラッシュで前置きします。これが、歌詞行を分割するために使用したパターン /\s*\|\s*/ の一部の説明です。\| は「縦棒に一致」を意味します。バックスラッシュなしで | は後で説明する選択（alternation）を意味します。

showRE('yes | no', /\|/)      # => yes <<|>> no
showRE('yes (no)', /no/)  # => yes <<(no)>>
showRE('are you sure?', /e\?/) # => are you sur<<e?>>

アルファベット文字と数字の後にバックスラッシュを付けると、特別な一致構造を導入することができます。これについては後で詳しく説明します。さらに、正規表現には #{...} を使った式の置換が含まれる場合があります。

### アンカー

デフォルトでは、正規表現はパターンに一致する最初の部分を文字列内で探します。たとえば、文字列「Mississippi」に対して /iss/ をマッチさせると、位置1から始まる部分文字列「iss」が見つかります。しかし、パターンを文字列の先頭や末尾でのみ一致させたい場合はどうでしょうか？

パターン ^ と $ はそれぞれ行の始まりと終わりに一致します。これらはパターン一致をアンカーするために使われます。例えば、/^option/ は「option」という単語が行の先頭に現れる場合にのみ一致します。また、\A は文字列の始まり、\z と \Z は文字列の終わりに一致します。（実際には、\Z は文字列が改行 \n で終わっていない場合に文字列の終わりに一致しますが、改行 \n で終わっている場合は改行直前に一致します。）

showRE("this is\nthe time", /^the/)   # => this is\n<<the>> time
showRE("this is\nthe time", /is$/)    # => this <<is>>\nthe time
showRE("this is\nthe time", /\Athis/) # => <<this>> is\nthe time
showRE("this is\nthe time", /\Athe/)  # => no match

同様に、パターン \b と \B はそれぞれ単語境界と非単語境界に一致します。単語文字とは、文字、数字、およびアンダースコアです。

showRE("this is\nthe time", /\bis/)  # => this <<is>>\nthe time
showRE("this is\nthe time", /\Bis/)  # => th<<is>> is\nthe time

### 文字クラス

文字クラスは、角括弧 [ ] の中にある文字の集合です。[characters] は角括弧内の任意の1文字に一致します。例えば、[aeiou] は母音に一致し、[,.:;!?] は句読点に一致します。角括弧内では、特殊な正規表現文字である .、|、(、)、[ 、{、+、^、$、*、? の意味は無効になります。ただし、文字列置換は通常通り行われるため、例えば \b はバックスペース文字を表し、\n は改行を表します（詳細は表18.2参照）。また、表5.1に示されている省略形を使うことができます。例えば、\s は単なる空白文字だけでなく、すべての空白文字に一致します。

showRE('It costs $12.', /[aeiou]/)  # => It c<<o>>sts $12.
showRE('It costs $12.', /[\s]/)     # => It<< >>costs $12.

角括弧内では、c1-c2 の形式で c1 と c2 の間にあるすべての文字に一致します（両端を含む）。

文字クラス内でリテラル文字 ] や - を含めたい場合、それらはクラスの最初に置く必要があります。

a = 'Gamma [Design Patterns-page 123]'
showRE(a, /[]]/)    # => Gamma [Design Patterns-page 123>>
showRE(a, /[B-F]/)  # => Gamma [<<D>>esign Patterns-page 123]
showRE(a, /[-]/)    # => Gamma [Design Patterns<<->>page 123]
showRE(a, /[0-9]/)  # => Gamma [Design Patterns-page <<1>>23]

開き括弧直後に ^ を置くことで、文字クラスを否定できます。例えば、[^a-z] は小文字アルファベット以外の任意の文字に一致します。

Rubyではよく使われる文字クラスに省略形が用意されています。これらは角括弧内でもパターン内でも使用できます。省略形の一覧は表5.1に示されています。

showRE('It costs $12.', /\s/)  # => It<< >>costs $12.
showRE('It costs $12.', /\d/)  # => It costs $<<1>>2.

文字クラスの省略形

最後に、角括弧外のドット（.）は、改行を除く任意の文字に一致します（複数行モードでは改行にも一致します）。

a = 'It costs $12.'
showRE(a, /c.s/)  # => It <<cos>>ts $12.
showRE(a, /./)    # => <<I>>t costs $12.
showRE(a, /\./)   # => It costs $12<<.>>

### 繰り返し

歌のリストを分割するパターン /\s*\|\s*/ を指定したとき、垂直バーを任意の量の空白で囲んで一致させたいと言いました。ここで、\s シーケンスが空白文字を1文字に一致させることを知っているので、アスタリスク（*）が「任意の量」を意味する可能性が高いと思われます。実際、アスタリスクは、パターン内の直前の正規表現の出現を複数回一致させるための修飾子の一つです。

もし r がパターン内の直前の正規表現を表すなら、以下のように解釈できます：

r * は r の出現が0回以上の一致

r + は r の出現が1回以上の一致

r ? は r の出現が0回または1回の一致

r {m,n} は r の出現が最小 m 回、最大 n 回の一致

r {m,} は r の出現が少なくとも m 回の一致


これらの繰り返し構文は優先度が高く、直前の正規表現にのみ適用されます。例えば、/ab+/ は「a の後に1回以上の b 」に一致し、「ab の連続」には一致しません。また、* 構文には注意が必要です。パターン /a*/ は任意の文字列に一致します。すべての文字列にはゼロ回以上の「a」が含まれているからです。

これらのパターンは「貪欲（greedy）」と呼ばれます。なぜなら、デフォルトでは一致する部分をできるだけ多く取得しようとするからです。この動作を変更して最小限の一致にするには、疑問符（?）を追加することで可能です。

a = "The moon is made of cheese"
showRE(a, /\w+/)     # => <<The>> moon is made of cheese
showRE(a, /\s.*\s/)   # => The<< moon is made of >>cheese
showRE(a, /\s.*?\s/)  # => The<< moon >>is made of cheese
showRE(a, /[aeiou]{2,99}/)  # => The m<<oo>>n is made of cheese
showRE(a, /mo?o/)     # => The <<moo>>n is made of cheese

### 選択肢

縦棒（|）は特別な意味を持つことがわかっています。なぜなら、私たちの行分割パターンではそれをエスケープするためにバックスラッシュを使わなければならなかったからです。これは、エスケープされていない縦棒（|）が、それが前にある正規表現か、それとも後ろにある正規表現のどちらかに一致するためです。

a = "red ball blue sky"
showRE(a, /d|e/)    # => r<<e>>d ball blue sky
showRE(a, /al|lu/)   # => red b<<al>>l blue sky
showRE(a, /red ball|angry sky/)  # => <<red ball>> blue sky

ここには注意すべき罠があります。というのも、| は非常に低い優先度を持っています。上記の最後の例では、「red ball」または「angry sky」に一致しますが、「red ball sky」や「red angry sky」には一致しません。もし「red ball sky」や「red angry sky」に一致させたければ、グループ化を使ってデフォルトの優先度を上書きする必要があります。

### グルーピング

正規表現内で項目をグループ化するために、括弧を使用できます。グループ内のすべては、1つの正規表現として扱われます。

showRE('banana', /an*/)     # => b<<an>>ana
showRE('banana', /(an)*/)   # => <<>>banana
showRE('banana', /(an)+/)   # => b<<anan>>a

a = 'red ball blue sky'
showRE(a, /blue|red/)   # => <<red>> ball blue sky
showRE(a, /(blue|red) \w+/)  # => <<red ball>> blue sky
showRE(a, /(red|blue) \w+/)  # => <<red ball>> blue sky
showRE(a, /red|blue \w+/)    # => <<red>> ball blue sky
showRE(a, /red (ball|angry) sky/)  # => no match

a = 'the red angry sky'
showRE(a, /red (ball|angry) sky/)  # => the <<red angry sky>>

括弧は、パターンマッチングの結果を収集するためにも使用されます。Rubyは開き括弧を数え、それぞれに対応する閉じ括弧との間での部分一致の結果を保存します。この部分一致は、パターン内でもRubyプログラム内でも使用できます。パターン内では、\1 は最初のグループの一致、\2 は2番目のグループの一致、というように参照します。パターン外では、特殊変数 $1, $2 などが同じ目的で使用されます。

"12:50am" =~ /(\d\d):(\d\d)(..)/    # => 0
"Hour is #$1, minute #$2"           # => "Hour is 12, minute 50"
"12:50am" =~ /((\d\d):(\d\d))(..)/  # => 0
"Time is #$1"                       # => "Time is 12:50"
"Hour is #$2, minute #$3"           # => "Hour is 12, minute 50"
"AM/PM is #$4"                      # => "AM/PM is am"

現在の一致の一部を後で再利用できる能力を使うと、さまざまな形の繰り返しを探すことができます。

同じ文字の繰り返しを一致させる

showRE('He said "Hello"', /(\w)\1/)   # => He said "He<<ll>>o"

同じ部分文字列の繰り返しを一致させる

showRE('Mississippi', /(\w+)\1/)   # => M<<ississ>>ippi

また、バックリファレンスを使用して区切り文字に一致させることもできます。

showRE('He said "Hello"', /(["']).*?\1/)  # => He said <<"Hello">>
showRE("He said 'Hello'", /(["']).*?\1/)  # => He said <<'Hello'>>

### パターンベースの置換

時には、文字列の中でパターンを見つけることが十分な場合があります。例えば、友達が「a、b、c、d、eという順番で文字を含む単語を見つけて」と挑戦してきた場合、/a.*b.*c.*d.*e/というパターンで単語リストを検索し、「absconded」や「ambuscade」を見つけることができます。これは何かしらの意味があるでしょう。

しかし、パターンマッチに基づいて物事を変更する必要がある時もあります。例えば、私たちの歌のリストファイルを見てみましょう。リストを作成した人はすべてのアーティスト名を小文字で入力しましたが、ジュクボックスの画面に表示するには、ミックスケース（単語の最初の文字が大文字）の方が見栄えが良いです。どうやって各単語の最初の文字を大文字に変更するか？

String#subとString#gsubのメソッドは、文字列内で最初の引数と一致する部分を探し、第二引数で置換します。String#subは1回だけ置換を行い、String#gsubは一致するすべての部分を置換します。どちらも置換を行った新しい文字列を返します。変更版であるString#sub!とString#gsub!は、元の文字列を直接変更します。

a = "the quick brown fox"
a.sub(/[aeiou]/,  '*')    # => "th* quick brown fox"
a.gsub(/[aeiou]/, '*')    # => "th* q**ck br*wn f*x"
a.sub(/\s\S+/,  '')        # => "the brown fox"
a.gsub(/\s\S+/, '')        # => "the"

これらの関数の第二引数には、文字列またはブロックを指定できます。ブロックを使用した場合、ブロックの値が文字列に置換されます。

a = "the quick brown fox"
a.sub(/^./) { $&.upcase }    # => "The quick brown fox"
a.gsub(/[aeiou]/) { $&.upcase }    # => "thE qUIck brOwn fOx"

これでアーティスト名をミックスケースに変換する方法がわかります。単語の最初の文字に一致するパターンは\b\wです。これは、単語境界の後に単語文字を探すものです。これをgsubと組み合わせることで、アーティスト名を変更できます。

def mixedCase(aName)
  aName.gsub(/\b\w/) { $&.upcase }
end

mixedCase("fats waller")    # => "Fats Waller"
mixedCase("louis armstrong")    # => "Louis Armstrong"
mixedCase("strength in numbers")    # => "Strength In Numbers"

### 置換におけるバックスラッシュシーケンス

以前、パターン内で\1、\2などのシーケンスが利用できることを説明しました。これらは、これまでにマッチしたn番目のグループを表します。同じシーケンスは、subやgsubの第二引数にも使用できます。

"fred:smith".sub(/(\w+):(\w+)/, '\2, \1')  # => "smith, fred"
"nercpyitno".gsub(/(.)(.)/, '\2\1')  # => "encryption"

置換文字列において使える追加のバックスラッシュシーケンスもあります：\&（最後にマッチした文字列）、\+（最後にマッチしたグループ）、`（マッチ前の文字列）、\'（マッチ後の文字列）、および\\（リテラルのバックスラッシュ）です。リテラルのバックスラッシュを置換に含めたい場合、少し混乱が生じることがあります。明示的に次のように書くことができます。

str.gsub(/\\/, '\\\\')  # バックスラッシュを2つに置き換え

ここで、このコードはstr内の各バックスラッシュを2つに置き換えようとしていることがわかります。プログラマーは置換テキスト内でバックスラッシュを2つにすることで、構文解析時に\\に変換されることを知っており、置換が行われた後、正規表現エンジンはもう一度文字列を走査し、\\を\に変換するため、最終的に各バックスラッシュがもう1つのバックスラッシュに置き換わります。解決策としては、次のように書かなければなりません。

str = 'a\b\c'  # => "a\b\c"
str.gsub(/\\/, '\\\\\\\\')  # => "a\\b\\c"

しかし、&を使ってマッチした文字列を置換することを考慮すると、次のように書くこともできます。

str = 'a\b\c'  # => "a\b\c"
str.gsub(/\\/, '\&\&')  # => "a\\b\\c"

gsubのブロック形式を使う場合、置換用の文字列は構文解析時に一度だけ解析され、その結果が意図した通りに処理されます。

str = 'a\b\c'  # => "a\b\c"
str.gsub(/\\/) { '\\\\' }  # => "a\\b\\c"

最後に、正規表現とコードブロックを組み合わせた素晴らしい表現力を示す例として、Wakou Aoyamaによって書かれたCGIライブラリモジュールの以下のコード断片を紹介します。このコードは、HTMLエスケープシーケンスを含む文字列を通常のASCII文字列に変換します。日本語のユーザー向けに書かれているため、正規表現にn修飾子が使われており、広範囲な文字処理がオフになります。さらに、Rubyのcase式を使っている点も示しています。

def unescapeHTML(string)
  str = string.dup
  str.gsub!(/&(.*?);/n) {
    match = $1.dup
    case match
    when /\Aamp\z/ni           then '&'
    when /\Aquot\z/ni          then '"'
    when /\Agt\z/ni            then '>'
    when /\Alt\z/ni            then '<'
    when /\A#(\d+)\z/n         then Integer($1).chr
    when /\A#x([0-9a-f]+)\z/ni then $1.hex.chr
    end
  }
  str
end

puts unescapeHTML("1&lt;2 &amp;&amp; 4&gt;3")
puts unescapeHTML("&quot;A&quot; = &#65; = &#x41;")

出力結果:

1<2 && 4>3
"A" = A = A

## 18. オブジェクト指向の正規表現

これらの変数が非常に便利であることは認めざるを得ませんが、あまりオブジェクト指向的ではなく、確かに cryptic（難解）です。そして、Rubyではすべてがオブジェクトだと言ったはずでは？ では、何が問題なのでしょうか？

実際には、問題はありません。Rubyを設計したMatzは、完全にオブジェクト指向の正規表現処理システムを作り上げました。それから、Perlのプログラマーに馴染み深いように、すべての$-変数をその上にラッピングしたのです。オブジェクトやクラスは、表面下に隠れています。では、そのオブジェクトやクラスを掘り起こしてみましょう。

すでに1つのクラスに出会っています。それは、正規表現リテラルがRegexpクラスのインスタンスを作成することです（Regexpクラスの詳細は、361ページから始まります）。

re = /cat/
re.type  # => Regexp

Regexp#matchメソッドは、正規表現を文字列に対してマッチさせます。失敗した場合、このメソッドはnilを返します。成功した場合、MatchDataクラスのインスタンスを返します（MatchDataクラスの詳細は、336ページから始まります）。そして、そのMatchDataオブジェクトを使うことで、マッチに関するすべての情報を得ることができます。これまでの$-変数から得られる便利な情報が、この小さなオブジェクトにまとめられています。

re = /(\d+):(\d+)/     # hh:mm形式の時間をマッチ
md = re.match("Time: 12:34am")
md.type  # => MatchData
md[0]         # == $&  => "12:34"
md[1]         # == $1  => "12"
md[2]         # == $2  => "34"
md.pre_match  # == $`  => "Time: "
md.post_match # == $'  => "am"

マッチデータは自分自身のオブジェクトに保存されるため、$-変数を使ってはできないような、複数のパターンマッチの結果を同時に保持することができます。次の例では、同じRegexpオブジェクトを2つの文字列に対してマッチさせています。各マッチは固有のMatchDataオブジェクトを返し、そのサブパターンのフィールドを調べることで、異なるマッチ結果であることを確認できます。

re = /(\d+):(\d+)/     # hh:mm形式の時間をマッチ
md1 = re.match("Time: 12:34am")
md2 = re.match("Time: 10:30pm")
md1[1, 2]  # => ["12", "34"]
md2[1, 2]  # => ["10", "30"]

では、$-変数はどう使われるのでしょうか？ 実は、正規表現パターンマッチ後、Rubyはその結果（nilまたはMatchDataオブジェクト）への参照をスレッドローカル変数（$~）に保存します。その後、他のすべての正規表現関連の変数はこのオブジェクトから派生します。以下のコードは実際に使うことはあまりないかもしれませんが、すべてのMatchData関連の$-変数が実際に$~の値に従っていることを示しています。

re = /(\d+):(\d+)/
md1 = re.match("Time: 12:34am")
md2 = re.match("Time: 10:30pm")
[ $1, $2 ]   # 最後の成功したマッチ => ["10", "30"]
$~ = md1
[ $1, $2 ]   # 前回の成功したマッチ => ["12", "34"]

これまで説明してきたように、正直に言うと、AndyとDaveは通常、MatchDataオブジェクトよりも$-変数を使います。日常的な使用においては、$-変数の方が便利だからです。時には、現実的なアプローチが最も役立つこともあります。



