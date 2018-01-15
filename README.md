## 言葉遣い

|言葉|説明|
|:--|:--|
|エラー|コンパイル不可能なこと。|
|合法, OK|コンパイル可能なこと。エラーの反対語。valid, legal|


## コンパイルと実行
```shell-session
$ vi hello.go
$ go run hello.go
```
 - `go run` を用いてコンパイルして実行できます。
 - コンパイル結果は実行後自動的に削除されるので何もゴミは残りません。

<details><ul><li>実用プログラムを開発する場合には、環境変数 <code>GOPATH</code> の設定が必要です。この記事のサンプルの範囲では不要です。
</ul>
</details>

## プログラム
```go
package main
import "fmt"
func main() {
    fmt.Println("hello world")
}
```
 - プログラム（ソースファイル）は必ず `package` 句で始まらなければなりません。省略はできません。
 - 実行プログラムの場合 `package main` でなければなりません。
 - その次は必ず `import` 宣言です（不要なら省略できます）。
 - 実行プログラムのエントリーポイントは `func main()`（main 関数）です。

<details><ul><li><code>package</code> 句、<code>import</code> 宣言はトップレベルにしか書けません。関数の中など、他の場所には書けません。
<li><code>import</code> 宣言の後に書けるのは <code>const</code>, <code>type</code>, <code>var</code> 宣言と <code>func</code> 宣言（関数宣言またはメソッド宣言）です。
<li>このうち <code>func</code> 宣言はトップレベルにしか書けません。関数やメソッドはネストできないということです。
<li>関数宣言内に関数を書きたい場合は関数式を使います。ただし簡単に自身を呼び出すことができません。
<li><code>main</code> 関数は引数なしで返り値なしでなければなりません。
</ul></details>

## Go 初心者が戸惑うエラー (1) 改行
```go
func main
{
//        ↑ エラー
```
```go
var addr = mail.Address{
    Name:    "Your Name",
    Address: "you@example.com"
}
//                            ↑ エラー
```
 - 識別子やリテラルが行末に来た場合、`;` が自動的に挿入された上で構文解析されます。
 - よって `{` は行頭に来てはならず、前の字句と同じ行になければなりません。
 - また `,` 区切りのリストを改行を交えて書いた場合、最後の要素の行末に `,` を書かなければエラーになります。

<details><ul><li>行末に以下に示すトークンが来た場合、その後ろに <code>;</code> が挿入されます: 識別子、数値・文字・文字列リテラル、<code>break</code>, <code>continue</code>, <code>fallthrough</code>, <code>return</code>, <code>++</code>, <code>--</code>, <code>)</code>, <code>]</code>, <code>}</code>
</ul></details>

## Go 初心者が戸惑うエラー (2) 未使用の識別子
```go
import "fmt" // エラー
func main() {
}
```
```go
func main() {
    var msg = "hello" // エラー
}
```
 - 宣言しただけで使われない識別子が存在するとエラーになります。
 - 仕様はこれをエラーとすることを許容していますが、実際にエラーとするかどうかは処理系の実装依存です。

<details><ul><li>これらのエラーを避けるには (1) ブランク識別子 <code>_</code> をうまく使う方法[<sup>→1</sup>](#1)や、(2) goimports ツールを使う方法があります。
</ul></details>

## コメント
```go
// comment
/* comment
   comment */
```
 - 1 行コメント `//` と複数行コメント `/* */` です。

<details><ul><li>複数行コメントはネストしません。<code>/&#42;</code> 出現後、最初の <code>&#42;/</code> でコメントは終了します。
<li>改行を含まない <code>/* comment */</code> はスペース相当、それ以外のコメントは改行相当です。
</ul></details>

## 文字列リテラル
```go
"hello\nworld"
`hello
world`
```
 - 文字列リテラル `"literal"` はエスケープを含む形式です。リテラル中での改行はエラーになります。
 - 文字列リテラル `` `literal` `` はエスケープを含まない形式です。`` ` `` 以外の任意の文字を含むことができ、改行も含めそのまま文字列となります。

<details><ul><li><code>&#96;&#96;</code> 内の CR (ASCII 0D) は無視されます。Windows 改行でソースコードを書いても Unix 改行で書いた場合と同じ文字列になります。
<li><code>"</code> を <code>""</code> 内に入れるには <code>"... \" ..."</code> とします。
<li><code>&#96;</code> は <code>&#96;&#96;</code> 内に入れることはできません。<code>&#96;... &#96;+"&#96;"+&#96; ...&#96;</code> とします。
</ul></details>

## 文字リテラル
```go
'H'
'\n'
```
 - `''` 内の文字の Unicode コードポイント整数です。
 - エスケープを含むことができます。
 - Go では文字のことを**ルーン**と呼びます。

## エスケープ
```go
'\n'
"hello world\n"
```
 - `\` を用いて印字不可能な文字などを表します。
 - `""` 内でも `''` 内でもほぼ同じものが使えます。
 - ``` `` ``` 内では使えません（`\` は特別な意味を持ちません）。
 - `\n` . . . U+000A LF, &nbsp;&nbsp; `\r` . . . U+000D CR, &nbsp;&nbsp; `\t` . . . U+0009 HT
 - `\\` . . . `\`, &nbsp;&nbsp; `\'` . . . `'`, &nbsp;&nbsp; `\"` . . . `"`
 - `"\'"`, `'\"'` はエラーです。正しくは `"'"`, `'"'` です。

<details><ul><li><code>\a</code> . . . U+0007 BEL, &nbsp;&nbsp; <code>\b</code> . . . U+0008 BS, &nbsp;&nbsp; <code>\f</code> . . . U+000C FF, &nbsp;&nbsp; <code>\v</code> . . . U+000B VT
</ul></details>

## 数値リテラル
```go
12345
0xDeadBeaf
0644
```
 - 10 進、16 進、8 進の整数リテラルです。

## 論理値定数

```go
true
false
```
 - `true` と `false` は論理値を表す宣言済み定数です。

<details><ul><li>リテラル（予約語）ではありません。暗黙に事前宣言された識別子なので、上書きしようと思えばできます[<sup>→2</sup>](#2)。
</ul></details>

## 型
```go
string
int
byte
bool
```
 - 変数や関数の返り値は型を持ちます。型は取りうる値の範囲を指定します。
 - 型は識別子（名前）で表します。
 - 代表的なものが事前に定義されています。
 - `string` . . . 文字列（任意のバイト列）, &nbsp;&nbsp; `int` . . . 32 ビットまたは 64 ビットの符号付き整数, &nbsp;&nbsp; `byte` . . . 8 ビット符号無し整数, &nbsp;&nbsp; `bool` . . . 論理値 (`true` or `false`)

<details><ul><li>型名もリテラル（予約語）ではなく暗黙に事前宣言された識別子です。
<li>サイズ別の符号付き・符号無し整数、浮動小数点数、複素数が用意されています[<sup>→3</sup>](#3)。
<li>識別子（型名）による以外に型リテラルを用いて複合型が記述できます。<code>[3]int</code>, <code>struct{ x, y int }</code>, <code>*int</code>, <code>func(int) string</code>, <code>interface{ M(string) bool }</code>, <code>[]int</code>, <code>map[string]int</code>
</ul></details>

## 変数 (1) 宣言
```go
var s string
var m, n int
var (
    i, j int
    t, u string
)
```
 - `var` で変数を宣言します。
 - Go では型で修飾するときは後置します。
 - `var ()` という構文も用意されています。

## 変数 (2) 初期化
```go
var s string = "hello"
var t, u string = s, "world"
var ok bool = true
```
 - `=` を用いて、変数や定数やリテラルで変数を初期化できます。
 - `=` の左右に複数の変数と値を置くことができます。

## 変数 (3) 初期化（型の省略）
```go
var s = "hello"       // var s string = "hello"
var t, u = s, "world" // var t, u string = s, "world"
var n, p = 1, "one"   // var n int = 1; var p string = "one"
var ok = true         // var ok bool = true
```
 - 型が省略された変数を初期化すると、変数は右辺の型として宣言されます。
 
<details><ul><li>型を持たない定数による初期化の場合、右辺の型は定数の種類ごとの既定の型となります（→「[定数の型の既定値]( #定数の型の既定値 )」）。
<li>右辺が <code>nil</code> の場合はエラーになります。
</ul></details>

## 変数 (4) 初期化（短縮形式）
```go
s := "hello"       // var s = "hello"
t, u := s, "world" // var t, u = s, "world"
n, p := 1, "one"   // var n, p = 1, "one"
ok := true         // var ok = true
```
 - `:=` を用いると型が省略された初期化をさらに短い形式で書くことができます。
 - この形式は関数の内側にしか書けません。トップレベルに書くことはできません。

<details><ul><li><code>var x, y, z = a, b, c</code> は同じブロックに既に <code>var x</code>, <code>var y</code>, <code>var z</code> のうち１つでも存在しているとエラーになりますが、<code>x, y, z := a, b, c</code> では同じブロックに既に <code>var x</code>, <code>var y</code>, <code>var z</code> の全部が存在している場合のみエラーになります。全部が存在しているのではない場合、存在しているものについては代入が行われ、存在しない変数だけが宣言され初期化されます。
</ul></details>

## 変数 (5) エラーになる書き方
```go
var n int, s string // エラー
var i, j int, t, u string // エラー
var (
    n int
    s string
) = 1, "hello" // エラー
```

## ゼロ値
```go
var n int
var s string
var b bool
fmt.Println(n) // → 0
fmt.Println(s) // →
fmt.Println(n) // → false
```
 - 変数宣言時に初期化を省略すると、不定な値になるのではなく、型によって定まる既定値で初期化されます。これを**ゼロ値**と呼びます。
 - 整数型は `0`、文字列型は `""`、論理値型は `false` がゼロ値です。

<details><ul><li>その他の型のゼロ値は[<sup>→4</sup>](#4)を参照してください。
<li>初期化時に右辺の項目数が足りない場合はゼロ値で初期化はされず、エラーになります。<code>var s, t, u = "hello", "world" // エラー</code>
</ul></details>

## 代入
```go
var m, n int
var b byte
n = 2
m = n
b = 3
n = b // エラー
```
 - 代入は `=` を用います。
 - 型が厳密に一致しないとエラーです。
 - 最後の `n = b` のように明らかに安全なケースでも、暗黙の変換などは用意されておらずエラーになります。
 - 後述のインターフェース型の変数には異なる型の代入が許される場合があります。

<details><ul><li>代入の左辺に置けるのは**アドレス可能** ( *addressable* ) なオペランド、もしくはマップのインデックス式です.
<li>アドレス可能なオペランドは変数、ポインター間接参照、スライスのインデックス式, 構造体のフィールドセレクター式, 配列のインデックス式のどれかです.
</ul></details>

## 定数の型の既定値
```go
var n = 1    // var n int = 1
s := "hello" // var s string = "hello"
b := true    // var b bool = true
```
 - 定数は型を持ちませんが、変数宣言の短縮形など型を持つ値が要求されるケースでは、定数の種類ごとに定められた既定の型に変換されます。

|定数|既定型|
|:--|:--|
|`123`|`int`|
|`"hello"`|`string`|
|`'H'`|`rune`|
|`true`, `false`|`bool`|

## 定数の宣言
```go
const one = 1
const one_typed int = 1
const (
    two = 2
    three = 3
)
var b byte = one // OK
var bb byte = one_typed // エラー
```
 - `const` で定数に名前を付けることができます。
 - `const` を用いると型のない定数も型のある定数も宣言できます。
 - `const ()` という構文も用意されています。

## 演算子
```go
var a, b, c, d int
(a + b - c) * d
var n int
var b byte
n + b // エラー
```
 - C や Java と似ています。
 - 演算子のオペランドは型が厳密に一致している必要があります。
 - 一方のオペランドが型を持たない定数の場合はもう一方のオペランドの型に変換されます。

<details><ul><li>シフト演算子 <code><<</code>, <code>>></code> についてはオペランドの暗黙の型変換が起こることがあります。
<li>定数間の演算については別に定めがあります。
</ul></details>

## インクリメントとデクリメント
```go
var m, n int
n++
m = n++ // エラー
++n // エラー
```
 - C や Java と同様に `++`, `--` でインクリメントとデクリメント演算ができます。
 - ただし後置のみで、式ではなく文です。

## 関数 (1)
```go
func succ(n int) int {
    return n + 1
}

func p(n int) {
    fmt.Println(n)
}
```
 - `func` で関数を定義します。
 - 関数の返り値型は後置します。返り値を返さない場合は省略します。
 - `return` で関数の実行は終了し、オペランドが返り値になります。

## 関数 (2) 複数の返り値
```go
func divide(n int) (int, int) {
    q := n / 2
    r := n % 2
    return q + r, q
}

func main() {
    former, latter := divide(7)
    fmt.Println(former, latter) // → 4 3
}
```
 - 複数の値を返す関数を定義できます。
 - 返り値の型の組は `()` で囲む必要があります。
 - 複数の返り値は複数の変数への代入または初期化で受け取ることができます。

<details><ul><li>１つの値を返す関数宣言で返り値の型を <code>()</code> で囲むことは合法です。<code>func succ(n int) (int) {</code>
<li>値を返さない関数宣言で返り値の型を <code>()</code> とするのは合法です。<code>func p(n int) () {</code>
</ul></details>

## 関数 (3) 関数内の関数
```go
func main() {
    func succ(n int) int { // エラー
        return n + 1
    }
    fmt.Println(succ(3))
}
```
 - 関数内で関数を宣言することはできません。

## 関数 (4) 関数リテラル
```go
func main() {
    succ := func(n int) int {
        return n + 1
    }
    fmt.Println(succ(3)) // → 4
}
```
 - 関数内で関数を宣言したい場合に関数リテラルを使うことができます。
 - ただし自分自身を簡単に呼ぶ方法はありません。`succ` 内から `succ` を簡単には呼び出せないということです。

 ## 関数 (5) 関数型
 ```go
var succ func(int) int = func(n int) int {
    return n + 1
}
 ```
 - 関数型は関数リテラルから本体部分を取り去った形式で記述します。

## 関数 (6) できないこと
```go
func add(a, b int) int {
    return a + b
}
func add(a, b string) string { // エラー
    return a + b
}
func succ(n int, d int = 1) int  { // エラー
    return n + d
}
```
 - 関数のオーバーロードはできません。
 - 関数の引数の既定値は指定できません。

<!---
可変長引数については配列・スライス後に説明した方がいい
-->

## 構造体 (1)
```go
var p, q struct {
    x int
    y int
}
p.x = 3
p.y = 4
```
 - 識別子による定義済みの型を組み合わせた複合型を `struct` で表記することができます。これを**構造体**と呼びます。

## 構造体 (2) 参照ではなく値
```go
// ↑ の続き
q = p
p.x = 5
fmt.Println(p) // → 5 4
fmt.Println(q) // → 3 4
```
 - 構造体型の変数が保持しているのは参照ではなく値です。
 - 代入するとコピーされ、一方を変更しても他方はそのままです。

<details><ul><li>構造体への参照を取り扱いたい場合はポインターを用います。
</ul></details>

## 構造体 (3) 名前付け
```go
type point struct {
    x int
    y int
}
var p point
```
 - `type` を用いて構造体に識別子による型名を付けることができます。

## 構造体 (4) リテラル
```go
// ↑ の続き
p = point{x: 3, y: 4}
q := point{
    x: 3,
    y: 4,
}
r := point{
    x: 3,
    y: 4  // エラー
}
```
 - 構造体型の値をリテラルで記述できます。
 - `;` の自動補填によるエラーに注意が必要です（→「[Go 初心者が戸惑うエラー (1) 改行]( #go-初心者が戸惑うエラー-1-改行 )」）。

## ポインター (1) アドレスとデリファレンス
```go
n := 5
p := &n
q := &n
*p = 6
fmt.Println(n, *p, *q) // → 6 6 6
```
 - `&` 単項演算子によって値を保持する記憶領域のアドレスを取得できます。これを**ポインター**と呼びます。
 - ポインターで値を参照することで値の共有が可能です。
 - 共有した値の取得や変更は `*` 単項演算子を用いて行います。

## ポインター (2) ポインター型
```go
n := 5
var p *int = &n
```
<!--
```go
func succ(p *int) {
    *p++
}
n := 5
succ(&n)
fmt.Println(n) // → 6
```
-->
 - 型 `T` の値へのポインターを表す型は `*T` と記述します。

## ポインター (3) リテラルへのポインター
```go
type point struct { x, y int }
p := &point{x: 3, y: 4}
q := &5 // エラー
```
 - 構造体を表すリテラルに `&` 演算子を適用することで、構造体型の値へのポインターを生成できます。
 - 整数リテラルなどに対してはできません。

## ポインター (4) `nil`
```go
var p *int = nil
n := *p // 実行時エラー
```
 - ポインター型の値として、どのアドレスとも異なる特別な無効値 `nil` が用意されています。
 - `nil` は事前に定義された識別子で、型のない `nil` 値を持ちます。型がないのでどのポインター型の変数にも代入できます。
 - `nil` を値に持つポインターに対して `*` 単項演算子を適用すると実行時エラーが発生します。
 - ポインター型の[ゼロ値]( #ゼロ値 )は `nil` です。ポインター型の初期化していない変数は `nil` を値として持ちます。

<details><ul><li><code>nil</code> はポインター型以外の参照的な型の無効値としても用いられます。
<li>具体的にはスライス型、マップ型、関数型、インターフェース型の変数に <code>nil</code> を代入できます。
<li><code>nil</code> はこれらの型の[ゼロ値]( #ゼロ値 )でもあります。
</ul></details>

## ポインター (5) ローカル変数へのポインター
```go
func pn() *int {
   n := 0
   return &n
}
p := pn()
fmt.Println(*p) // → 0
```
 - 関数はローカル変数へのポインターを返しても安全です（C 言語とは違います）。

## 型宣言 (1) 別名
```go
type I = int
```
 - `type` `=` で型の別名を宣言できます。
 - 別名は元の型と完全に同じ型です。

## 型宣言 (2) 新しい型名
```go
type point struct {
    x int
    y int
}
```
 - `type` で既存の型に識別子による名前を付けることができます。
 - 名前をつけられた既存の型を *underlying type* と言います。以下では**起源型**と言うことにします。

## 型宣言 (3) 新しい型
```go
type I int
type J int
var i I
var j J
i = j // エラー
i = 3 // OK
i = i + i // OK
```
 - `type` によって宣言された名前の型は他のどの型とも異なる新しい型になります。
 - 起源型が同じ複数の新規型の間で代入や演算はエラーになります。
 - 型のない定数の代入はそれが可能なら合法です。
 - 起源型で定義されていた演算は引き継ぎます。

## ブロックとスコープ
```go
func main() {
    {
        var pre int = a // エラー
        var a int
        var post int = a // OK
    }
    var outside int = a // エラー
}
```
```go
package main

var pre int = a // OK
var a int
var post int = a // OK

func main() { /* ... */ }
```
- Go は一般的なレキシカルスコープ規則を持つ言語です。
- 関数内で宣言された識別子のスコープは最内のブロックに渡ります。
- ブロックは大まかに言えば中括弧 <code>{}</code> で区切られたソースコード上の区画です。
- 関数外（トップレベル）で宣言された識別子のスコープは同一パッケージ（後述）に属する複数のファイルに渡ります。

<details><ul><li>関数内で宣言された変数名、定数名、型名のスコープは宣言位置以前には渡りません。
<li>それ以外は最内ブロック全体に渡ります。例えばトップレベルで宣言された変数は、宣言位置の前後から参照可能です。
<li>中括弧によるブロック以外に、ファイル全体を占めるブロック、同じパッケージに属する複数のファイル全体を占めるブロック、全ファイルを占めるブロックがあります。後ろに書いたブロックが前に書いたブロックを包含します。
</ul></details>

## 制御構造 (1)
```go
if n == 3 { /* ... */ }
for i := 0; i < 10; i++ { /* ... */ }
switch n { /* ... */ }
```
 - 制御構造には `if`, `for`, `switch` の 3 種類があります。
 - `if`, `for`, `switch` に続く表現を囲む `()` は不要です。
 - その後に続く文を囲む `{}` は必須です。

## 制御構造 (2) 前置文
```go
if result, err := f(); err != nil {
    return err
} else {
    fmt.Println(result)
}
```
 - 先頭に置く式の前に単純な文を `;` で区切って置くことができます。

## 制御構造 (3) 制御構造を囲むブロック
```go
if result, err := f(); err != nil {
    return err
} else {
    fmt.Println(result)
}
fmt.Println(result, err) // エラー
```
 - 制御構造は暗黙のブロックで囲まれます。前置文もそのブロックの内部です。
 - 前置文で宣言された変数のスコープは `if`, `for`, `switch` の終端の `}` で終わります。

## `if` 文
```go
if expr1 {
    // ...
} else if expr2 {
    // ...
} else {
    // ...
}
```
 - 条件式は `bool` である必要があります。
 - `elif`, `elsif`, `elseif` ではなく `else if` です。
 
<details><ul><li>条件式は厳密には起源型が <code>bool</code> であれば OK です。
</ul></details>

## `for` 文 (1) 無限ループ
```go
for {
    // ...
    if terminal_condition {
        break
    }
}
```
- 無限ループからの脱出は `break` で行います。

## `for` 文 (2) while
```go
for a < b {
    a = a + 16
}
```

## `for` 文 (3) 3 項 for
```go
for i := 0; i < 10; i++ {
    // ...
}
```
- `i := 0` は `var i int = 0` の意ですが、`for var i int = 0; i < 10; i++ {` とは書けません。

<details><ul><li>書けるのは式、インクリメント・デクリメント文、代入、変数宣言の短縮形式です。
</ul></details>

## `for` 文 (4) `range`
```go
for i, n := range [...]string{"zero", "one", "two", "three"} {
    // ...
}
```
- 別項で後ほど説明します。

## `switch` 文 (1)
```go
switch n {
case 0:
    /* case 句 */
case 1:
    /* case 句 */
default:
    /* case 句 */
}
```
- `case` 後の式を上から順に評価し、`n` と一致したら `:` 後の case 句を実行し、`switch` 文の実行を終わる。
- どの `case` 後の式も `n` と一致しなかったら `default:` 後の case 句を実行する。
- case 句は `{}` に囲まれてはいないがブロックである。case 句の内部で宣言された変数のスコープは case 句外には渡らない。

<details><ul><li>`default:` はどの位置にあってもいい。
</ul><details>





<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

###### 1
```go
import _ "fmt"
func main() {
    var msg = "hello"
    var _ = msg
}
```
###### 2
```go
var false = true
if false {
   fmt.Println("false is true")
}
// yields "false is true"
```
###### 3
|||
|:--           |:--|
| `uint8`      | the set of all unsigned  8-bit integers (0 to 255) |
| `uint16`     | the set of all unsigned 16-bit integers (0 to 65535) |
| `uint32`     | the set of all unsigned 32-bit integers (0 to 4294967295) |
| `uint64`     | the set of all unsigned 64-bit integers (0 to 18446744073709551615) |
|            | |
| `int8`       | the set of all signed  8-bit integers (-128 to 127) |
| `int16`      | the set of all signed 16-bit integers (-32768 to 32767) |
| `int32`      | the set of all signed 32-bit integers (-2147483648 to 2147483647) |
| `int64`      | the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807) |
|            | |
| `float32`    | the set of all IEEE-754 32-bit floating-point numbers |
| `float64`    | the set of all IEEE-754 64-bit floating-point numbers |
|            | |
| `complex64`  | the set of all complex numbers with float32 real and imaginary parts |
| `complex128` | the set of all complex numbers with float64 real and imaginary parts |
|            | |
| `byte`       | alias for uint8 |
| `rune`       | alias for int32 |
|            | |
| `uint`       | either 32 or 64 bits |
| `int`        | same size as uint |
| `uintptr`    | an unsigned integer large enough to store the uninterpreted bits of a pointer value |

##### 4
|型|ゼロ値|
|:--|:--|
|整数型|`0`|
|浮動小数点数型|`0.0`|
|文字列型|`""`|
|論理値型|`false`|
|その他|`nil`|
