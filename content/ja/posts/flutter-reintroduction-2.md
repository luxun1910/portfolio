+++
title = 'Flutter再入門1'
date = 2025-07-07T18:59:49+08:00
draft = false
+++

## 前回の記事

- [Flutter再入門1](/ja/posts/flutter-reintroduction-1/)

## 文法

### 関数

#### 引数の宣言方法

```dart
// 省略可能引数
// int? bとすれば省略時はnullが渡る
// 引数リストの末尾に置く必要がある
void doSomething(int a, [int b = 200]){
    print('a: $a, b: $b');
}

doSomething(100);
doSomething(100, 300);

// 名前付き引数
// デフォルトでは省略可能
// 必須にする場合はrequired int aとする
// デフォルト値も与えられる
void doSomething({int? a, int? b}){
    print('a: $a, b: $b');
}

doSomething();
doSomething(a: 100);
doSomething(b: 200);
doSomething(a: 100, b: 200);
doSomething(b: 100, a: 200);

// 名前付き引数のリストも、引数リストの末尾に置く必要があるが、呼び出し時は位置引数を後方に置いても可
```

#### 第一級関数と匿名関数

Flutterでは、関数を変数に代入したり、引数に受け取ったりできる（第一級関数をサポート）。
匿名関数もサポートしている。

```dart
int doubleValue(int x){
  return x * 2;
}

// 関数オブジェクトの型
// 戻り値の型 Function(引数リストの型)
final int Function(int) f = doubleValue;
final result = f(8);
print("num:$result");
// => num:16

// 匿名関数
// （引数リスト）{関数本体}
final int Function(int) f2 = (x){return x * 2;};
final result = f2(8);
print("num:$result");
// => num:16

// Dartの匿名関数はクロージャの性質を持つ
// クロージャとは、外部スコープから変数を「キャプチャ」する関数
// 内部関数（匿名関数）が外部関数の変数へのアクセスを保持
Function multiple(int i) {
   return (x) => x * i;
 }

 final f1 = multiple(3);
 final f2 = multiple(7);

 print(f1(2));
 // => 6
 print(f2(6));
 // => 42
```

### クラス

#### コンストラクタ基礎

```dart
// x,yの初期値をコンストラクタで設定
class Point{
  Point(int xPos, int yPos):x=xPos,y=yPos;

  int x;
  int y;
}

// 上記のシュガーシンタックス
class Point{
  Point(this.x,this.y);

  int x;
  int y;
}

// コンストラクタ本体も記述可能
// ただし、本体が実行される前に非null許容型のクラス変数は初期化済みである必要がある
// 下記はコンパイルエラー
class Point {
  Point(int xPosition, int yPosition) {
    // => Non-nullable instance field 'x' must be initialized.
    // => Non-nullable instance field 'y' must be initialized.
    x = xPosition;
    y = yPosition;
  }

  int x;
  int y;
}

// 初期化リストでパラメーターのアサーションを記述することも可
class Point{
  Point(this.x, this.y):assert(x => 0), assert(y => 0);

  final int x;
  final int y;
}
```

#### ゲッターとセッター

全てのインスタンス変数は暗黙的にゲッターを持つ。
final修飾子のないインスタンス変数は暗黙的にセッターを持つ。

```dart
class User {
  User(this.id, this._password);

  //セッターなし
  final int id;
  //プライベートフィールド
  String _password;

  // カスタムゲッター
  // パスワードを伏せ字にして返す
  String get password => '*******';

  // カスタムセッター
  // パスワードをハッシュ化して保存する
set password(String newPassword) {
    _password = hash(newPassword);
  }
}

```

#### 色々なコンストラクタ

##### 引数なしのデフォルトコンストラクタ

コンストラクタを記述しなければ自動的に生成される。

##### constantコンストラクタ

constantコンストラクタは、インスタンス変数が全てfinalであることを保証する。

```dart
class Point {
  const Point(this.x, this.y);

  final int x;
  final int y;
}

// const変数に代入した場合、またはconstantコンストラクタの前にconstを付与した場合、常に同じインスタンスが使われるようになり、Flutterのパフォーマンス向上に役立つ
const point = Point(1, 2);
final point2 = const Point(1, 2);
```

##### 名前付きコンストラクタ

コンストラクタに名前を付けることができる。

```dart
class Point {
  Point(this.x, this.y);
  const Point.origin():x=0,y=0;
  //または
  // const Point.origin():this(0,0);

  final int x;
  final int y;
}

const origin = Point.origin();

```

##### factoryコンストラクタ

必ずしも新しいインスタンスを生成しない（キャッシュを利用したい）場合や、初期化リストに記述できないロジックがある場合はfactoryコンストラクタを使用する。
コンストラクタにfactoryキーワードを付与し、コンストラクタ本体でインスタンスを返すreturn文を記述する。

```dart
class UserData {
  static final Map<int, UserData> _cache = {};

  // factoryコンストラクタ
  factory UserData.fromCache(int userId){
    // キャッシュを探す
    final cache = _cache[userId];

    // キャッシュがあれば返す
    if (cache != null) {
      return cache;
    }

    // キャッシュがなければ新しいインスタンスを生成
    final newInstance = UserData();
    _cache[userId] = newInstance;
    return newInstance;
  }
}
```

### クラス継承

- メソッドをオーバーライドできる条件
  - 戻り値の型・引数の型が親クラスのメソッドの戻り値の型・引数の型と同じ、またはそのサブタイプ
  - 位置パラメーターの数が同じ
  - ジェネリックメソッドを非ジェネリックメソッドでオーバーライドできない、その逆も然り

```dart
// 親クラスのコンストラクタを呼び出し
class Animal {
  Animal(this.name);
  final String name;
}

class Dog extends Animal {
  Dog(String name) : super(name);
}

// 以下のコードは同じ意味（シュガーシンタックス）
class Animal {
  Animal(this.name);
  final String name;
}

class Dog extends Animal {
  Dog(super.name);
}
```

#### 暗黙のインターフェース

- Dartでは、全てのクラスは暗黙的にインターフェースが定義されている
  - そのクラスの全ての関数とインスタンスメンバを持ったインターフェース
- インターフェースを実装する際は、全てのメンバーをオーバーライドする必要がある

#### 拡張メソッド

```dart
extension SwapList<T> on List<T> {
   // 引数のインデックスの要素を入れ替える拡張メソッド
   void swap(int index1, int index2) {
     final tmp = this[index1];
     this[index1] = this[index2];
     this[index2] = tmp;
   }
 }

 final list = [1, 2, 3];
 list.swap(0, 2); // インデックス0と2の要素を入れ替える
 print(list);
 // => [3, 2, 1]
```

静的な拡張メソッドは宣言できないが、拡張メソッド内で静的メソッドを定義すれば、拡張メソッド内から呼び出し可能。

拡張名（上記で言えばSwapList<T>）のない拡張メソッドも定義できる。
その場合は同一ファイル内でのみ参照可能。

#### mixin

多重継承に似た仕様としてmixinがある。

```dart
mixin Horse {
   void run() {
     print('run');
   }
 }

 mixin Bird {
   void fly() {
     print('fly');
   }
 }

class Pegasus with Bird, Horse {}

 final pegasus = Pegasus();
 pegasus.run(); // PegasusはHorseのメソッドを持つ
 // => run
 pegasus.fly(); // PegasusはBirdのメソッドも持つ
 // => fly
```

mixinとclassの違いは以下の通り。

- インスタンス化できない
- extendsで他のクラスから継承できない
- コンストラクタを宣言できない

ミックスインを宣言する際、onキーワードで使用するクラスを制限することも可能。
その場合、onで指定したクラスのメソッドを呼び出せる。
mixin classで宣言するとonは使えない。

```dart
class Animal {
  String greet() => 'hi';
}

mixin Horse on Animal {
   void run() {
     greet(); // Animalのメソッドを呼び出せる
     print('run');
   }
 }

mixin Bird on Animal {
  void fly() {
    greet(); // Animalのメソッドを呼び出せる
    print('fly');
   }
 }

class Pegasus extends Animal with Bird, Horse {}
```