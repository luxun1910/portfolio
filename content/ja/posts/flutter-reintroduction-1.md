+++
title = 'Flutter再入門1'
date = 2025-07-04T18:59:49+08:00
draft = false
+++

## 初めに

Flutterでいくつかアプリを作ってきたが、あまりちゃんと理解せず感覚的に書いている個所が多くある。そのため、『Flutter実践開発 ── iPhone／Android両対応アプリ開発のテクニック』を読んで、知らなかったことなどを備忘録としてメモしていきたい。

## 文法

### 変数宣言

#### finalについて

final修飾子の場合、利用時までに初期化されていれば問題ない。

```dart

  final flag = DateTime.now().hour.isEven;

  final int number; // 宣言時に初期化しない（この場合もfinalで宣言可能）
  if (flag) {
    number = 0;
  } else {
    number = 255;
  }
  print(number);

```

- 変数の初期化をDartコンパイラが正しく判断できない場合はlate修飾子でコンパイラのチェックを回避可能
  - グローバル変数の初期化など

#### final late

一度初期化されたら変更不可。

#### lateについて

- 宣言時に初期化処理を記述すると、変数にアクセスされるまで初期化処理を遅延できる
  - 使用されるか分からない変数や、初期化処理の実行コストが高い場合に用いると効果的

### 組み込み型

#### 文字列

##### 変数の値を挿入する方法

```dart

final name = 'Taro';
final message = 'Hello, $name!';
print(message);

```

##### 式の結果を挿入する方法

```dart

final a = 10;
final b = 20;
final message = 'a + b = ${a + b}';
print(message);

```

##### 文字列リテラルの連結

- 隣接する文字列リテラルは自動的に連結される（改行しても問題ない）が、+演算子を使ってもいい

##### 複数行の文字列の定義

三重のシングルまたはダブルクオーテーションで囲む。

##### 文字列リテラルの前にrを置くことで、エスケープシーケンスを無視できる

```dart

final message = r'Hello\nWorld';
print(message);
// => Hello\nWorld

```

#### コレクション

```dart
// List
final list1 = [0,1,2,3];
final list2 = [0,1,2,3,];
final list = <int>[0,1,2,3];

// 固定長の配列を作成
final fixedList = List.unmodifiable([0,1,2,3]);

// Set
final set1 = {'A', 'B', 'C'};
final set2 = {'A', 'B', 'C',};
final set = <String>{'A', 'B', 'C'};

// Map
final map1 = {'A': 1, 'B': 2, 'C': 3};
final map2 = {'A': 1, 'B': 2, 'C': 3,};
final map = <String, int>{'A': 1, 'B': 2, 'C': 3};

// 空の{}はMapとして推論される
final map = {};
print(setOrMap is Map); // => true

// Record
// 匿名型、タプル
// 位置フィールド（名前を付けないフィールド）
final record1 = (404, 'Not Found');
final (int, String) record2 = (404, 'Not Found');

// 名前付きフィールド
final record3 = (code: 404, message: 'Not Found');
final ({int code, String message}) record4 = (code: 404, message: 'Not Found');

// 型注釈によって、位置フィールドに名前を付ける
// 名前付きフィールドではない
final (int code, String message) record4 = (404, 'Not Found');

// 等値性
final record1 = (code: 404, message: 'Not Found');
final record2 = (message: 'Not Found', code: 404);
print(record1 == record2); // => true

final (int code, String name) record5 = (404, 'Not Found');
final (int x, String y) record6 = (404, 'Not Found');
print(record5 == record6); // => true

final ({int code, String name}) record7 = (code: 404, name: 'Not Found');
final ({int x, String y}) record8 = (x: 404, y: 'Not Found');
print(record7 == record8); // => false

// 名前付きフィールドと位置フィールドが混在した場合、型注釈では位置フィールドが常に先頭に配置される
final record9 = (code: 404, message: 'Not Found', "ページが見つかりません");

final (String description, {int code, String message}) record10 = record9;


```

#### Objectクラス

Dartの全てのクラスはObjectクラスを継承している。

```dart
// これはList<Object>として推論される
final list = [0, 'abc', true];
```

- dynamicは、コンパイル時に型のチェックが行われない
  - 存在しないメソッドを呼び出してもコンパイルエラーにならない
  - null判断も行われない
- 基本的にはobjectを使う

### カスケード記法

オブジェクトのメソッドやプロパティへドット2つでアクセスすると、そのオブジェクトそのものが戻り値になる。

```dart

final sb = StringBuffer()
  ..write('Hello')
  ..write(' ')
  ..write('World');

print(sb.toString());
// => Hello World

```

C#だと似たようなものでメソッドチェーンがあるが、メソッドチェーンはメソッドの戻り値をオブジェクト自身（this）にする必要があるのに対し、Dartのカスケード記法はその必要がない（writeメソッドの戻り値はvoid）。

### コレクションのオペレータ

```dart
// Spread演算子
final list1 = [0,1,2];
final list2 = [-1, ...list1, 3];
print(list2);
// => [-1, 0, 1, 2, 3]

// 制御構文演算子
// flagがtrueの場合は3が追加される
final list3 = [0,1,2,if(flag) 3];

// forも記述できる
final list4 = [0, for(var i in list1) i * 2];
print(list4);
// -> [0, 0, 2, 4]

```

### 制御構文

#### if-case-when文

パターンマッチングをした上で変数へ分解する。

```dart

// responseのmessageかstatusCodeいずれかがnullの場合はelseに入る
final (String?, int?) response = ('OK', 200);
if (response case (String message, int statusCode)){
    print('message: $message, statusCode: $statusCode');
}else{
    print('response is null');
}

// caseの後whenを付けると条件式を記述できる
final (String?, int?) response = ('OK', 200);
if (response case (String message, int statusCode) when statusCode == 200){
    print('message: $message, statusCode: $statusCode');
}else{
    print('response is null');
}

```

#### switch文

- 途中でswitch文を抜けて後の処理を続けて実行したときだけbreakを使う（毎回のケースの終わりにbreakを書く必要がない）
  - returnやthrowも使える
- continueとラベルを使うと、ケースの順に関係なく直接フォールスルーできる

```dart
switch (color) {
  case 'red':
    doSomethingIfRed();
    continue other;
  case 'blue':
    doSomethingIfBlue();
  other:
  case 'black':
    throw 'Unexpected color'; // redとblackのときに実行される
}
```

- whenを使うと、ケースの条件式を記述できる

```dart

final int? statusCode = null;
 switch (statusCode) {
   case (int statusCode) when 100 <= statusCode && statusCode < 200:
     print('informational');
   case (int statusCode) when 200 <= statusCode && statusCode < 300:
     print('successful');
   case (int statusCode) when 300 <= statusCode && statusCode < 400:
     print('redirection');
   case (int statusCode) when 400 <= statusCode && statusCode < 500:
     print('client error');
   case (int statusCode) when 500 <= statusCode && statusCode < 600:
     print('server error');
   case (null):
     print('no response received.');
   default:
     print('unknown status code');
 }

```

- switchを式として使える

```dart
final int statusCode = 0;
final message = switch (statusCode) {
  >= 100 && < 200 => 'informational',
  >= 200 && < 300 => 'successful',
  >= 300 && < 400 => 'redirection',
  >= 400 && < 500 => 'client error',
  >= 500 && < 600 => 'server error',
  _ => 'unknown status code', // デフォルトケース
};
```

- コレクションの一致判定ではリテラルにconst修飾子を付与する必要がある

```dart

switch (variable){
    case const [0,1,2]:
        doSomething();
    case const {3,4,5}:
        doSomethingElse();
    default:
        doSomethingDefault();
}

```

#### ループ

- Iterableクラスに対してはfor-inやforEachメソッドが使える

### パターン

- 2つの機能がある
  - オブジェクトのマッチング
  - オブジェクトの分解宣言

```dart
// マッチング
final name = 'Taro';
switch(name){
    case 'Taro':
        doSomething();
}

// 分解宣言
final record = ('cake', 300);
final (name, price) = record;
print('${name}は${price}円です');

// マッチした際に変数にバインドするパターン
// Setは利用不可
final [a, b, ..., c] = [0, 1, 2, 3, 4, 5];
print('a=$a, b=$b, c=$c');
// => a=0, b=1, c=5

// Mapの場合はキーが一致するとバインドできる
final {200: successful, 404: notFound} = {
    200: 'OK',
    404: 'Not Found',
    500: 'Internal Server Error',
};
print('200 -> $successful, 404 => $notFound');
// => 200 -> OK, 404 -> Not Found

// Recordの場合はすべての構造が一致する必要がある
final record = (name: 'cake', price: 300);

// マッチする
final (name: n, price: p) = record;

// 名前付きフィールドの場合は名前まで一致する必要がある
// これはフィールド名がないのでマッチしない
final (String name, int price) = record;

// フィールド名を変数名で推論させる記法もある
// マッチする
final (:name :price) = record;

// クラスもゲッターからマッチ・分解宣言できる
class SomeClass {
    const SomeClass(this.x);
    final int x;
}

final someInstance = SomeClass(100);
final SomeClass(x: number) = someInstance;
// これも同じ（フィールド名を変数名で推論）
final SomeClass(:x) = someInstance;
print(number);
// => 100

// for-inでの分解宣言
final list = [0,1,2,3,4,5];
for (final (index, value) in list.indexed){
    print('index: $index, value: $value');
}
// => index: 0, value: 0
// => index: 1, value: 1

// MapはIterableのサブクラスではない
// entriesプロパティを使うことで、キーとバリューのペアを変数にバインドしてループを回せる（MapEntryをObjectとして分解宣言）
final map = {'A': 1, 'B': 2, 'C': 3};
for (var MapEntry(key: key, value: value) in map.entries){
    print('key: $key, value: $value');
}
// => key: A, value: 1
// => key: B, value: 2
// => key: C, value: 3

```

#### パターンを補助する構文

```dart
// キャスト
final List<Object> list = [0, 'abc'];
final [number as int, str as String] = list;

// nullチェック
switch (code){
    case final i? when i >= 0:
        print('i is positive');
    case final i?:
        print('i is null');
    default:
        print('i is negative');
}

// nullアサーション
// null時は実行時エラーになる
switch (code){
    case final i! when i >= 0:
        print('i is positive');
    default:
        print('i is negative');
}

// ワイルドカード
final record = ('cake', 300);
final (name, _) = record;

switch (variable){
    case SomeClass _:
        doSomething();
    case String _:
        doSomethingElse();
}

```

### 例外処理

#### 例外の型

- DartにはError型とException型がある
  - Error型
    - プログラムの失敗、プログラム上の問題によりスローされるもの
      - 例
        - 間違った関数の使い方
        - 無効な引数が渡された
      - 呼び出し元で捕捉する必要なし
  - Exception型
    - 捕捉されることを目的としている
    - エラーに関する情報を持たせるべき
- 任意のオブジェクトを例外としてスローできるが、推奨されていない

#### 例外の捕捉

```dart
try{
    doSomething();
} on MyException{
    print('MyException');
}

try {
  doSomething();
} catch(e, st) {
  print('catch $e');
  print('stackTrace $st');
}

try {
  doSomething();
} on MyException catch(e) {
  print('catch $e');
  rethrow; // 呼び出し元へ例外を再スロー
} finally {
    doCleanUp();
}
```

#### assert

assertの第1引数がfalseの場合、プログラム実行が中断された上で、指定している場合は第2引数のメッセージが出力される。

```dart
assert(variable != null, 'variable is null');
```

debugビルドの時だけassert文が処理される性質を利用して、デバッグ時のみ実行したい処理を以下のように記述可能。

```dart
assert(() {
  print('debug mode');
  return true;
}());
```

#### Flutterの例外処理

```dart
void main() {
  FlutterError.onError = (errorDetails) {
    //Flutterのフレームワーク自身がトリガーするコールバック（レンダリング処理やウィジェットのbuildメソッドなど）で発生した例はここにルーティングされる
    //デフォルトではログをコンソールに出力する
  };

  PlatformDispatcher.instance.onError = (error, stack) {
    // それ以外のFlutter内で発生した例外（ボタンのタップイベントハンドラなど）はここでハンドリングする
    print(error);
    return true; // 例外を処理した場合はtrueを返す
  };

  runApp(const MyApp());
}
```

### null関連演算子

`str1 ?? = 'new string';`と書くと、変数がnullの場合だけ代入される。
