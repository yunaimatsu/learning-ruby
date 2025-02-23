## 1. 導入

これまでジュークボックスのコードの一部を実装して楽しんできましたが、少し怠っていた部分があります。配列、ハッシュ、プロックについては触れましたが、Rubyの他の基本的な型――数値、文字列、範囲、正規表現――については十分に扱ってきませんでした。これらの基本的な構成要素について、これから少しページを割いて学んでいきましょう。

## 2. 数値
Rubyは整数と浮動小数点数をサポートしています。整数は任意の長さを持つことができ（システムの空きメモリによって最大値が決まります）、一定の範囲内の整数（通常は-230から230-1または-262から262-1）は内部的に2進数形式で保持され、Fixnumクラスのオブジェクトとして扱われます。この範囲を超える整数は、Bignumクラスのオブジェクトとして格納されます（現在は可変長の短整数のセットとして実装されています）。この変換は透明で、Rubyは自動的に変換を管理します。

num = 8
7.times do
  print num.type, " ", num, "\n"
  num *= num
end

出力：

Fixnum 8
Fixnum 64
Fixnum 4096
Fixnum 16777216
Bignum 281474976710656
Bignum 79228162514264337593543950336
Bignum 6277101735386680763835789423207666416102355444464034512896

整数は、オプションで先頭に符号を付けたり、オプションで基数の指示子（0は8進数、0xは16進数、0bは2進数）を付け、その後に適切な基数の数字を並べることで記述できます。数字の文字列内のアンダースコアは無視されます。

123456                    # Fixnum
123_456                   # Fixnum（アンダースコアは無視される）
-543                      # 負のFixnum
123_456_789_123_345_789   # Bignum
0xaabb                    # 16進数
0377                      # 8進数
-0b101_010                # 2進数（負）

また、ASCII文字やエスケープシーケンスに対応する整数値を取得するためには、その前に疑問符を付けて記述できます。制御文字やメタ文字の組み合わせも、?\C-x、?\M-x、?\M-\C-xを使って生成できます。制御バージョンの値は「value & 0x9f」、メタバージョンの値は「value | 0x80」で生成されます。最後に、?\C-?はASCIIの削除（0177）を生成します。

?a                        # 文字コード
?\n                       # 改行コード (0x0a)
?\C-a                     # 制御文字a = ?A & 0x9f = 0x01
?\M-a                     # メタ文字a（ビット7をセット）
?\M-\C-a                  # メタと制御のa
?\C-?                     # 削除文字

小数点や指数を含む数値リテラルは、Floatオブジェクトに変換され、これはネイティブアーキテクチャの倍精度浮動小数点型に対応します。小数点の後には必ず数字が必要です。例えば、1.e3はFixnumクラスのメソッドe3を呼び出そうとしてしまうためエラーになります。

すべての数値はオブジェクトであり、さまざまなメッセージに応答します（詳細は290ページ、313ページ、315ページ、323ページ、349ページで説明されています）。そのため、例えばC++のように数値の絶対値を求める場合、aNumber.absと書きます。abs(aNumber)とは書きません。

整数はまた、いくつかの便利なイテレータをサポートしています。すでに1つ見ました――7.timesは47ページのコード例で使われています。他にも、2つの整数間で上下に反復処理を行うuptoやdownto、従来のforループに近いstepがあります。

3.times        { print "X " }
1.upto(5)      { |i| print i, " " }
99.downto(95)  { |i| print i, " " }
50.step(80, 5) { |i| print i, " " }

出力：

X X X 1 2 3 4 5 99 98 97 96 95 50 55 60 65 70 75 80

最後に、Perlユーザーへの警告です。数字を含む文字列は、式で使用される際に自動的に数値に変換されません。これは、ファイルから数字を読み取るときに最もよく問題になります。次のコードは（おそらく）意図した動作をしません。

DATA.each do |line|
  vals = line.split    # 行を分割し、トークンをvalsに格納
  print vals[0] + vals[1], " "
end

ファイルの内容が以下である場合：

3 4
5 6
7 8

出力は次のようになります：

34 56 78

何が起こったのでしょうか？

問題は、入力が数値ではなく文字列として読み込まれたことです。プラス演算子は文字列を連結するので、その結果としてそのような出力が得られます。これを修正するには、String#to_iメソッドを使って文字列を整数に変換する必要があります。

DATA.each do |line|
  vals = line.split
  print vals[0].to_i + vals[1].to_i, " "
end

出力：

7 11 15

## 3. 文字列
Rubyの文字列は、単純に8ビットバイトのシーケンスです。通常は印刷可能な文字を保持しますが、それは必須ではなく、文字列はバイナリデータも保持できます。文字列はStringクラスのオブジェクトです。

文字列は、文字列リテラル（区切り文字で囲まれた文字のシーケンス）を使ってよく作成されます。プログラムのソース内でバイナリデータを表現するのが難しいため、文字列リテラルにはさまざまなエスケープシーケンスを挿入できます。これらは、プログラムがコンパイルされる際に対応するバイナリ値に置き換えられます。文字列の区切り文字の種類によって、行われる置換の度合いが決まります。シングルクォーテーションで囲まれた文字列内では、2つの連続したバックスラッシュが1つのバックスラッシュに置き換えられ、バックスラッシュの後にシングルクォーテーションが続くと、シングルクォーテーションになります。

'escape using "\"'  →  escape using "" 'That's right'  →  That's right

ダブルクォーテーションで囲まれた文字列では、さらに多くのエスケープシーケンスをサポートしています。最も一般的なのはおそらく\n、つまり改行文字です。完全なリストはページ203の表18.2に記載されています。また、#{ expr }の形式で、文字列に任意のRuby式の値を埋め込むことができます。式がグローバル変数、クラス変数、インスタンス変数だけの場合は、波括弧を省略できます。

"Seconds/day: #{246060}"  →  Seconds/day: 86400
"#{'Ho! '*3}Merry Christmas"  →  Ho! Ho! Ho! Merry Christmas
"This is line #$."  →  This is line 3

文字列リテラルを構築する方法は他にも3つあります：%q、%Q、および「ヒアドキュメント」です。

%qと%Qは、それぞれシングルクォーテーションとダブルクォーテーションで囲まれた文字列を作成します。
%q/general single-quoted string/  →  general single-quoted string
%Q!general double-quoted string!  →  general double-quoted string
%Q{Seconds/day: #{246060}}  →  Seconds/day: 86400

qやQの後に続く文字が区切り文字です。区切り文字が開き括弧、波括弧、丸括弧、または小なり記号の場合、対応する閉じる記号が見つかるまで文字列が読み込まれます。それ以外の場合は、次に同じ区切り文字が現れるまで文字列が読み込まれます。

最後に、ヒアドキュメントを使って文字列を構築できます。

aString = <<END_OF_STRING
    The body of the string
    is the input lines up to
    one ending with the same
    text that followed the '<<'
END_OF_STRING

ヒアドキュメントは、ソース内の行を指定した終了文字列まで読み込みますが、終了文字列自体は含まれません。通常、この終了文字列は最初の列から始める必要があります。しかし、<<の後にハイフンを付けると、終了文字列をインデントすることができます。

print <<-STRING1, <<-STRING2
   Concat
   STRING1
      enate
      STRING2

出力は次の通りです：

Concat
        enate

## 4. 文字列の操作
文字列はおそらく最も多くの組み込みメソッドを持つRubyのクラスで、標準メソッドは75以上あります。すべてのメソッドをここで紹介するわけではなく、ライブラリリファレンスに完全なリストがあります。代わりに、日常的にプログラミング中に登場しやすい一般的な文字列の使い方（イディオム）について見ていきます。

さて、私たちのジュークボックスに戻りましょう。ジュークボックスはインターネットに接続できるように設計されていますが、人気のある曲のいくつかをローカルハードドライブにも保存しています。これで、もしリスがネット接続をかじってしまっても、まだ顧客を楽しませることができます。

歴史的な理由（他に理由があるでしょうか？）で、曲のリストはフラットファイルとして行ごとに保存されています。各行には、曲を含むファイル名、曲の再生時間、アーティスト、タイトルが縦棒で区切られたフィールドに格納されています。典型的なファイルは次のようになっています：

/jazz/j00132.mp3  | 3:45 | Fats     Waller     | Ain't Misbehavin'
/jazz/j00319.mp3  | 2:58 | Louis    Armstrong  | Wonderful World
/bgrass/bg0732.mp3| 4:09 | Strength in Numbers | Texas Red
         :                  :           :                   :

このデータを見ると、これらのフィールドを抽出して整形するために、Stringクラスのメソッドを使う必要があることがわかります。最低限、次の作業が必要です：

行をフィールドに分割すること、

再生時間をmm:ssから秒数に変換すること、

アーティスト名の余分な空白を取り除くこと。


最初のタスクは、各行をフィールドに分割することで、String#splitメソッドを使うと簡単にできます。この場合、分割する正規表現は、縦棒の周りにオプションでスペースがあるパターンである/\s*|\s*/です。ファイルから読み込んだ行には改行が含まれているので、String#chompを使って改行を取り除いてからsplitを適用します。

songs = SongList.new

songFile.each do |line|
  file, length, name, title = line.chomp.split(/\s*\|\s*/)
  songs.append Song.new(title, name, length)
end
puts songs[1]

出力：

Song: Wonderful World--Louis Armstrong (2:58)

残念ながら、元のファイルを作成した人はアーティスト名を列に分けて入力したため、いくつかの名前に余分な空白があります。これらの空白は、私たちのハイテクなスーパーツイスト型フラットパネルDay-Gloディスプレイで見栄えが悪くなるので、作業を進める前にこれらの空白を取り除く必要があります。これにはいくつか方法がありますが、最もシンプルなのはString#squeezeを使って連続した文字の繰り返しを取り除く方法です。ここでは、文字列をその場で変更するsqueeze!メソッドを使います。

songs = SongList.new

songFile.each do |line|
  file, length, name, title = line.chomp.split(/\s*\|\s*/)
  name.squeeze!(" ")
  songs.append Song.new(title, name, length)
end
puts songs[1]

出力：

Song: Wonderful World--Louis Armstrong (2:58)

次に、時間形式の問題があります：ファイルでは「2:58」と表示されており、これを秒数で「178」に変換したいと考えています。再度splitを使って、コロンで時間を分割する方法もありますが、ここでは関連するメソッドString#scanを使用します。scanはsplitに似ていますが、文字列をパターンに基づいてチャンクに分けるために使用します。今回は、分と秒の両方に一致する1つ以上の数字を取得したいので、パターンは/\d+/です。

songs = SongList.new
songFile.each do |line|
  file, length, name, title = line.chomp.split(/\s*\|\s*/)
  name.squeeze!(" ")
  mins, secs = length.scan(/\d+/)
  songs.append Song.new(title, name, mins.to_i*60 + secs.to_i)
end
puts songs[1]

出力：

Song: Wonderful World--Louis Armstrong (178)

さらに、ジュークボックスにはキーワード検索機能があります。曲のタイトルやアーティスト名の単語を入力すると、すべての一致するトラックをリスト表示します。「fats」と入力すると、例えばFats Domino、Fats Navarro、Fats Wallerの曲が表示されるかもしれません。この機能は、インデックス作成クラスを作成することで実装します。オブジェクトといくつかの文字列を与えると、そのオブジェクトを、文字列の中で出現するすべての2文字以上の単語の下でインデックスします。これにより、Stringクラスの他のメソッドをいくつか利用することができます。

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

String#scanメソッドは、文字列から正規表現に一致する要素を抽出します。ここでは、パターン\w[-\w']+が、単語の中に現れる可能性のある任意の文字の後に、ハイフン、別の単語の文字、またはアポストロフィが1回以上続く部分をマッチします。検索を大文字小文字を区別しないようにするため、抽出した単語と検索時に使う単語をすべて小文字に変換します。最初のdowncase!メソッドの名前の末尾に付いている感嘆符は、このメソッドが受け取った文字列をその場で変更することを示しています。

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

