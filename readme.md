潜在変数モデルと学習法に関して
================================================================================

________________________________________________________________________________
はじめに
--------------------------------------------------------------------------------
### 自己紹介
はじめまして！いしはたです！

### 前置き
この記事は Machine Learning Advent Calendar の 12月8日 の記事として書かれています。  
思いの外長くなった割に内容が無いという悲惨な状況ですがご容赦ください。  

### 内容
今日の内容ですが、生成的確率モデリングに関して述べたいと思います。  
具体的には潜在変数（隠れ変数）を含む確率モデルの学習アルゴリズムについて簡単に解説します。  

### 想定する読者

対象は「__機械学習はじめてみよう！__」とか「__確率モデルで遊んでみたい！__」というビギナー向けのつもりです。  
難易度としては高校時代の同級生が「へー」って言ってくれるレベルを目指します。  
場合によってはあえて正確な表現を避け、簡単な言い回しを用いる予定です。  
なので玄人の方が読むと「__何を今更…__」とか「__それは違うだろ！__」となると思いますがご容赦ください。

________________________________________________________________________________
潜在変数モデル
--------------------------------------------------------------------------------
### 確率モデルと潜在変数
まず「__潜在変数モデル__」というものについて、簡単な実例を交えて簡単に解説します。  
（簡単を繰り替えしてごめんなさい。）  

確率モデルとはざっくり言えば、__同時分布__ です。  
__生成的__ 確率モデルとはざっくり言えば、
__データを生成できる（サンプリングできる）__ 確率モデルです。  
潜在変数モデルとはざっくり言えば、生成的確率モデルのうち、  
観測可能である __観測変数 x__ と観測不能である __潜在変数 z__ を持つものです。  
ここでモデルの同時分布を具体的に決めるために、モデルは __パラメータ θ__ を持つとします。  
そしてその同時分布を p(x, z | θ）と書くことにします。

生成的確率モデルの素敵なところは、データの生成過程を想像できるので、解釈しやすいところです。  
潜在変数モデルの素敵なところは、複雑なデータが大量にあっても、  
それらを潜在変数という不思議なものでなんとなく説明してくれるところです。  

潜在変数モデルの使い道としてもっともポピュラーなものがクラスタリングです。  
データがいっぱいあるけど、実はそれらは潜在的にグループに分かれているという解釈です。  
例えば、テレビの視聴履歴を集めたとします。  
すると潜在グループとして、__オタク__ or __リア充__ などが考えられます。  
データはその潜在グループに依存して生成されると考えます。  
例えばオタク野郎はアニメばっかり観てて、リア充様はドラマとかスポーツとか観てるんじゃねえの？(激怒)  

この潜在変数を推定するといろいろ便利なんです。  
例えば Amazon の購入履歴を考えます。  
クラスタリングの結果「こいつらは潜在的に萌豚えだな」と分かれば、
アニメBD売りたい放題とかなるわけです。  

では具体的にはどうやってその潜在変数を推定するのでしょうか。

### 学習と推論
潜在変数モデル p(x, z | θ) では x は既知で、 z と θ は未知とします。  
よって x から z, θ を推定する必要があります。

ここで推定ってなんやねん！となることがあります。  
ここでは単純に z と θ の具体的な値を知りたいという意味で「推定」と言っています。  
しかし個人的な感覚だと z を推定することを推論、  
θ を推定する事を学習と呼ぶ気もします。  
なぜ同じ未知の変数なのに扱いが違うのか。それはそれらが別のタイプの変数だからです。  
潜在変数 z は確率変数であるのに対して、θはパラメータであり、単なる変数なのです。  
前置きはまぁ置いといて推定してみましょう。

#### 準備
一気に両方推定するのは難しそうなので１つずつ推定しましょう。  

まず x, z が既知のとき、θを推定することを考えます。  
これはモデル中の全確率変数が観測可能であるときに、  そのパラメータを推定することに対応します。  
このようなデータを __完全データ__ と呼びます。  
よってもはや x,z と区別することなく、p(x | θ) と書いても良いのです。  
完全データからのパラメータの推定は実は中学生くらいでもできます。  
例えばコインを用意し、１００回振ります。そしたら表 70 回、裏 30 回でした。  
するとこのコインのパラメータは 表が出る確率 70 % , 裏が出る確率 30 % としたくなります。  
表が55回で裏が45回なら表55%,裏45%としたくなります。  
これは __最尤推定__ と呼ばれるれっきとした推定法なのです。  
具体的には p(x | θ) を最大化するように θ を決めるのです。  
要するに今起きたことは、『最も尤もらしい出来事』だったと考えるのです。  
最尤推定はまぁ素敵な性質がいっぱいあるのですがここでは割愛します。  

次に x, θ が既知のとき、z を推定することを考えます。  
これは同時分布 p(x, z | θ) が既知であるときに z を推定することに対応します。  
先のコインの例で言えば、コインの表裏の出る確率がわかっている時に、  
今から振るコインの面を推定することに対応します。  
例えば表がでる確率が 99 % で裏の確率が 1 % だったとします。  
次にどっちが出ると思う？と聞かれたらまぁ「表」と答えるでしょう。  
では表55%, 裏45%ではどうでしょう？まぁ一つ選べと言われれば「表」でしょう。  
つまり、p(x, z | θ) を最大化するように z を決めるのです。  
さっきの最尤推定と同じアイディアですね。

しかし、多くの人は 表55%,裏45% の状況で「表！」と叫ぶことを躊躇すると思います。  
それに対し、θを推定するときに表55%,裏45%とすることはあまり抵抗がなかったと思います。  
この違いは z が確率変数であることを知っているからなのです。  
これがさっき z と θ の推定を推論と学習と言いわけたい気持ちの答えであり、  
同時に今日のメインテーマでもあります。  

とりあえずこの問題は棚上げし、z,θ を同時に推定する方法を考えましょう。  
今、x,z から θ を、そして x,θ からzを推定する方法を知っています。  
では x から z,θ を推定するにはどうすればいいか。  
答えは簡単。交互に推定するのです！  


#### MM algorithm (Viterbi Training, K-means etc)

まず適当に θ を決めます。  
すると x,θ がわかるので z を p(x,z | θ) を最大化するように推定します。  
すると x,z がわかるので θ を p(x,z | θ) を最大化するように推定します。  
これを繰り返すといつか z,θ は変化しなくなり、その z,θ が推定結果となります。  

「え？これでいいの？」って感じですがいいのです。  
これは Viterbi Training と呼ばれたりする方法ですが、ここでは MM algorithm と呼びます。  
なぜなら z を p(x,z | θ) を最大化(Maximize)するように決め、  
次に θ を p(x,z | θ) を最大化(Maximize)するように決めるからです。  
k-means というクラスタリング手法はこの一例です。  

初心者の人でも「これならできそうだ！」と思ったでしょう。  
できます。暇な時にやってください。  

この手法では z を推定するとき思い切りを持って p(x, z | θ) を最大化するように決めました。  
しかし、表55%,裏45%のときに「表」と答えるのを躊躇する方は納得出来ないでしょう。  
そこで z をもっとソフトに推定する方法があります。  
それがかの有名な __EM algorithm__ なのです！  

#### EM algorithm

表55%,裏45%のときになんて答えればしっくりくるのか。  
きっとその回答の一つが「表の期待値0.55(どやぁ」です。  
麻雀でもパチンコでも競馬でも皆さん期待値で考えますよね？  
「期待値が1を超えないギャンブルをする奴はクズ」とかいう人もいますが彼らは夢を買っているのです（多分。  

つまり、z を推定するときに p(x, z | θ) を最大化(Maximize)するのでなく、  
期待値(Expectation)を取ればいいんじゃないの？  
それでいいんです。  

まず適当に θ を決めます。  
すると p(z |x, θ) がわかるので z の期待値を出します。  
すると x と z の期待値がわかるので、それを用いて θ を更新します。  
これを繰り返すと p(x | θ) がどんどん増加していきます。  
これが EM algorithm なのです。  

「なんでp(x | θ)が増えるんや！」という声が聞こえますが、Markdown で数式書きたくないので割愛します。  
コインの例で言えば、  
表回数 55, 裏回数 45のとき、表55%,裏45%と計算したのと同様、  
表期待値 0.55, 裏期待値 0.45 のとき、表55%,裏45%と計算できますよね？  

これで「表」と叫ぶことが不安な方々も学習できるようになりました。  
しかし次の問題も生まれます。  
「z の期待値考えるなら、θも期待値考えたくない？」  
MM, EM algorithm がああるなら ME, EE algorithm もあるよね？  
貪欲ですね〜。  
でもθは確率変数じゃないんです。  
ではどうするか。  
θも確率変数としちゃおうよ！  
これがかの有名な「ベイズ」の始まりである…。

#### ME algorithm と EE algorithm

__ベイズ的な__ 確率モデルとはモデルパラメータ θ に __事前分布 p(θ)__ を導入することで、  
θ も確率変数として扱うモデルのことです。  
これにより潜在変数モデルは p(x, z,  θ) = p(x, z | θ)p(θ) はとなります。  
(ベイズは奥が深いのでここでは詳細は述べません。)  
この拡張により θ も p(x, z | θ) を最大化(Maximize)するのではなく、  
p(θ | x) で期待値 (Expectation) を取れるようになります。  
これによって MM, EM algorithm と同様に ME, EE algorithm が構成できますね。  

しかし実際は、ベイズ的なモデルで p(z | x), p(θ | x) を計算することは難しく、  
それらの近似分布 q(z), q(θ) を変分法で求める __変分ベイズ法__ が使われます。  
これはとても説明がめんどくせいので興味のある人は自分で調べてください。  

### まとめ

登場した４つの学習法をまとめると、  

1. MM algorithm : z を最大化, θ も最大化
2. EM algorithm : z を期待値, θ は最大化
3. ME algorithm : z を最大化, θ は期待値
4. EE algorithm : z は期待値, θ も期待値

となります。  
ではどれがいいの？  
難しい質問です。  
答えは「場合による」です。  

一般的に、以下の特徴が挙げられます。  

* M* algorithm は収束が早い。
* M* algorithm は初期値依存性が強い。
* E*, *E algorithm は汎化性能が高い。
* E* algorithm は遅い。
* *E algorithm は近似誤差が含まれる。

上の特徴を踏まえ、データやモデルから適切に学習法を選ぶ必要があります。  
これらを適切に使い分けられるのが今流行りの __データサイエンティスト__ なんじゃないかなぁ（しらね。  

________________________________________________________________________________
実験
================================================================================
ここでは実際に紹介した４手法を利用してクラスタリングを行ってみます。

### データ
今回は本をクラスタリングしてみます。  
データは私が趣味で作っている[本推薦 AI bot][book_rec_ai] のデータを利用します。  
まず bot がフォローしているユーザの本棚中の出現頻度 [Top 200][titles] の本を持ってきます。  
次にユーザのうち、それらの本のうち大体半分くらいはもっているユーザ 65 人を取ってきます。  
最後にユーザと本の関係を行列で表現します。  
縦軸を本 i、横軸をユーザ j、行列の値 xij を  

- 0:本を持ってない
- 1:本を持ってる
- 2:本を持ってる、かつ、高評価

の３値で表現します。  
[この行列][relation]の各列をデータと思って本をクラスタリングしてみます。  
クラスタ数は 10 として各学習法を 10 回ずつ初期値を変えて実行し、  
もっとも尤度の高い結果をクラスタリング結果としました。  

### モデル
今回はクラスタリングに naive Bayes model を用います。  
本 i に関するデータは xi = {xi1,...,xi65} によって表現されます。  
この本 i のクラスを zi とすれば、naive bayes model は  
p(xi | zi, θ) = Πj p(xij | zi, θ) と分解することを許します。  
つまりクラスが与えられたとき、各属性は条件付き独立と仮定するモデルです。  
naive Baeys model に対する MM, EM, ME, EE の[実装][nbc]も一応載っけておきます。  
この実装は「遅い、長い、汚い」の三拍子そろっていますが、唯一の見どころは、  
z, θ の更新ルーチンに渡す関数を変えるだけで MM, EM, ME, EE が実現できるところでしょうか。  
ようするにこれら４つの手法は紙一重ということが伝えたかったのです（フヒヒ、サーセンw  

### 結果

各学習法のクラスタリング結果です。

1. [MM][mm]
2. [EM][em]
3. [ME][me]
4. [EE][ee]

ぶっちゃけおおきな違いはないですね。  
よくよく見ると、漫画の扱いで個性が出ています。  
MM が聖☆おにいさんを３月のライオンやよつばと！とくっつけているのに対して、  
EE は聖☆おにいさんとテルマエロマエをくっつけてますね。  
漫画好きの私としては EE の分け方に賛成です。  

まぁこのように微妙に結果変わるので適切にアルゴリズム選んでねって話です。

________________________________________________________________________________
最後に
================================================================================
長々と書きましたが、後半だれてきてるのが目に見えますね。すいません。  

この記事では潜在変数モデルの学習法がまぁいろいろあって、それらがどういう関係にあるかを述べました。  
みなさんいろんなモデルを作るのが好きなようですが、MMは実装しやすく、早いのでお勧めです。  

もし最後まで読んでくださった方がいれば、大変お疲れ様でした＆ありがとうございました。  
またどこかでお会いしましょう。  

[book_rec_ai]: https://twitter.com/book_rec_ai "book_rec_ai"
[relation]: https://github.com/masakazu-ishihata/advent2013/blob/master/relation.dat "relation.dat"
[titles]: https://github.com/masakazu-ishihata/advent2013/blob/master/titles.dat "titles.dat"
[nbc]: https://github.com/masakazu-ishihata/advent2013/blob/master/nbc.rb "nbc.rb"
[mm]: https://github.com/masakazu-ishihata/advent2013/blob/master/mm.txt "mm.txt"
[em]: https://github.com/masakazu-ishihata/advent2013/blob/master/em.txt "em.txt"
[me]: https://github.com/masakazu-ishihata/advent2013/blob/master/me.txt "me.txt"
[ee]: https://github.com/masakazu-ishihata/advent2013/blob/master/ee.txt "ee.txt"

