# Contract Programming

契約プログラミング

江添亮

# Design By Contract ™

+ 1986年
+ Bertrand Meyerが提唱
+ プログラミング言語Eiffelの設計思想
+ 2003年、商標取得

# コントラクトとは？

+ C++20に入る
+ 事前条件(precondition)
+ 事後条件(postcondition)
+ assert

# 事前条件

~~~cpp
[[ expects : expr ]]
~~~


# 例

~~~cpp
// bはゼロであってはならない
int divide( int a, int b )
[[ expects : b != 0 ]]
{
    return a / b ;
}
~~~

# 事後条件

~~~cpp
[[ ensures : expr ]]
~~~

# 例

~~~cpp
// ポインターの参照する値を返す
int dereference( int * p )
[[ ensures : p != nullptr ]]
{
    return *p ;
}
~~~

# 戻り値の参照

`ensures`の後に識別子を書くと戻り値として参照できる

~~~cpp
[[ ensures identifier : expr ]]
~~~

# 例
~~~cpp
double sin( double x )
[[ ensures r :
    ( (r>=-1.0) && (r<=1.0) )
        ||
    std::is_nan(r)
]]
{
    // 正弦波を計算
}
~~~

# assert

~~~cpp
[[ assert : expr ]]
~~~

# 例

~~~cpp
void f( int x )
{
    int y = x + 1 ;
    [[ assert : y == x + 1 ]]
}
~~~

# 複数のコントラクト

書いた順番にチェックされる

~~~cpp
int f( int x )
[[ expects : x > 100 ]]
[[ expects : x > 1000 ]]
[[ expects : x < 10000 ]]
[[ ensures r : r == x ]]
{
    return x ;
}
~~~

# 副作用

コントラクトが観測可能な副作用をもたらす場合の挙動は未定義

~~~cpp
int x ;

void f( int n )
// 未定義
[[ expects : n > ++x ]]
;
~~~

# 事後条件で引数値を使う場合の注意

事後条件で引数値をodr-useしている関数の本体で直接的、間接的にその値を変えると挙動は未定義

~~~cpp
int f( int x )
// 未定義
[[ ensures r : r == x ]]
{
    return ++x ;
}
~~~

# 引数値が変更されない場合はOK

~~~cpp
int f( int * p )
[[ ensures r : r == *p ]]
{
    return ++*p ;
}
~~~

# コントラクトレベル

コントラクトにはレベルがある

+ default
+ audit
+ axiom

# レベル指定

レベル指定がない場合はdefault

~~~cpp
[[ expects default : expr ]]
[[ ensures audit : expr ]]
[[ assert axiom : expr ]]
~~~

# レベルの意味

コントラクトチェックのコストを指定する

+ default : 関数本体に比べて十分に軽い
+ audit : 関数本体に比べてかなり重い
+ axiom : 実行時に評価できないほど重い

axiomはC++のコードとして意味をもつコメント扱い

# ビルドレベル

C++の実装はビルドレベルを持っている

+ off
+ default
+ audit

# ビルドレベルの意味

+ `default`はdefaultコントラクトをチェック
+ `audit`はdefaultとauditoコントラクトをチェク
+ `off`については規定されていない

ビルドレベルを指定する方法は実装定義

# 契約違反

以下のようなクラスで情報が通知される

~~~cpp
namespace std {
  class contract_violation {
  public:
    uint_least32_t line_number() const noexcept;
    string_view file_name() const noexcept;
    string_view function_name() const noexcept;
    string_view comment() const noexcept;
    string_view assertion_level() const noexcept;
  };
}
~~~

# 契約違反ハンドラー

+ contract_violationへのlvalueリファレンスを引数にとりvoidを返す関数
+ ハンドラーの指定方法は実装定義
