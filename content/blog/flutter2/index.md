---
title: Flutter
date: "2021-01-26"
description: "Flutter公式の翻訳"
tags: ["Flutter", "dart"]
---

Flutter の勉強をしています。
公式チュートリアル の翻訳してみました。

Docs> Development > Data &backend > state management> Simple app state management

### シンプルなアプリの状態管理。

#### Contents

- 例示
- 状態をリフトアップする。
- 状態にアクセスする
- ChangeNotifier
- ChangeNotifierProvider
- Consumer
- Provider.of
- まとめ

宣言型 UI プログラミングについて、そして一時的な状態とアプリの状態の違いについて知ったところで、簡単なアプリの状態管理について学ぶ準備ができました。

このページでは、Provider パッケージを使うことにします。Flutter を使い始めたばかりで、他のアプローチ（Redux、Rx、フックなど）を選ぶ強い理由がないのであれば、このアプローチから始めた方が良いでしょう。プロバイダパッケージは理解しやすく、コードをあまり使いません。また、他のどのアプローチにも適用可能な概念を使用しています。

つまり、他のリアクティブフレームワークの状態管理に強いバックグラウンドを持っている場合は、オプションページに記載されているパッケージやチュートリアルを見つけることができます。

### Our example

![model-shopper-screencast-e0ada0e83cd8e7fdcad84167b8f7ffd7eb5ef85b0cb8957f03c6f05bd16b1cea](https://user-images.githubusercontent.com/50585862/105824398-a1293c00-6001-11eb-816d-2033ccb10c1f.gif)

説明のために、次のような簡単なアプリを考えてみましょう。

このアプリは、カタログとカート（それぞれ MyCatalog、MyCart ウィジェットで表現）の 2 つの画面に分かれています。ショッピングアプリでも良いのですが、シンプルな SNS アプリでも同じ構造が想像できます（カタログを「wall」に、カートを「favorites」に置き換えて）

カタログ画面には、MyAppBar と多数の MyListItems のスクロール表示があります。

これが、このアプリの Widget ツリーを視覚的に表したものです。

![simple-widget-tree-8fe7a45d88d5007510b3f2f27caa845a0453663d4e5c60b5c8d0dc036ad8a966](https://user-images.githubusercontent.com/50585862/105824258-74752480-6001-11eb-9832-269e639587d5.png)
つまり、Widget の少なくとも 5 つのサブクラスがあるということです。それらの多くは、他の場所に "属している "状態へのアクセスを必要とします。例えば、各 MyListItem は自分自身をカートに追加できる必要があります。また、現在表示されているアイテムがすでにカートに入っているかどうかを確認したいかもしれません。
カートの現在の状態をどこに置くべきか？

### 状態をリフトアップする。

Flutter では、それを使用するウィジェットの祖先に状態を維持することは理にかなっています。

なぜでしょう？Flutter のような宣言型フレームワークでは、UI を変更したい場合は再構築しなければなりません。MyCart.updateWith(somethingNew)という簡単な方法はありません。つまり、ウィジェットにメソッドを呼び出して外部から強制的に変更するのは難しいのです。また、仮にこれを実現できたとしても、フレームワークに助けさせるのではなく、フレームワークと戦うことになります。

```dart
void myTapHandler() {
  var cartWidget = somehowGetMyCartWidget();
  cartWidget.updateWith(item);
}
```

上記のコードが動作するようになったとしても、MyCart ウィジェットでは以下のような処理をしなければなりません。

```dart
// BAD: DO NOT DO THIS
Widget build(BuildContext context) {
  return SomeWidget(
    // The initial state of the cart.
  );
}

void updateWith(Item item) {
  // Somehow you need to change the UI from here.
}
```

UI の現状を考慮して、新しいデータを適用する必要があるでしょう。こうやってバグを回避するのは難しいですね。

Flutter では、コンテンツが変わるたびに新しいウィジェットを作成します。MyCart.updateWith(somethingNew) (メソッドコール) の代わりに MyCart(contents) (コンストラクタ) を使います。新しいウィジェットは親のビルドメソッドでしか構築できないので、コンテンツを変更したい場合は、MyCart の親以上で構築する必要があります。

```dart
// GOOD
void myTapHandler(BuildContext context) {
  var cartModel = somehowGetMyCartModel(context);
  cartModel.add(item);
}
```

これで MyCart には、どのバージョンの UI でもビルドできるコードパスが 1 つだけになりました。

```dart
// GOOD
Widget build(BuildContext context) {
  var cartModel = somehowGetMyCartModel(context);
  return SomeWidget(
    // Just construct the UI once, using the current state of the cart.
    // ···
  );
}
```

この例では、コンテンツは MyApp の中に存在する必要があります。コンテンツが変更されるたびに、MyCart は上から再構築されます (詳細は後ほど)。このため、MyCart はライフサイクルを気にする必要がなく、与えられたコンテンツに対して何を表示するかを宣言するだけです。それが変更されると、古い MyCart ウィジェットは消え、新しいウィジェットに完全に置き換えられます。

![simple-widget-tree-with-cart-7665e5a1a8bfdc2c04f2cb6dcb59dabbf0291dc44b8b7f08f6a2e798e6080c9c](https://user-images.githubusercontent.com/50585862/105824251-72ab6100-6001-11eb-9f43-b955823b1edb.png)

これは、ウィジェットが不変であるということを意味しています。変わることはありません。

カートの状態をどこに置くかわかったので、それにアクセスする方法を見てみましょう。

### state へのアクセス

ユーザーがカタログ内のアイテムをクリックすると、カートに追加されます。しかし、カートは MyListItem の上にあるので、どうすればいいのでしょうか？

簡単なオプションは、MyListItem がクリックされたときに呼び出すことができるコールバックを提供することです。Dart の関数はファースト・クラス・オブジェクトなので、好きなように渡すことができます。そのため、MyCatalog の中で以下のように定義できます。

```dart
@override
Widget build(BuildContext context) {
  return SomeWidget(
    // Construct the widget, passing it a reference to the method above.
    MyListItem(myTapCallback),
  );
}

void myTapCallback(Item item) {
  print('user tapped on $item');
}
```

これは問題なく動作しますが、さまざまな場所から修正する必要があるアプリの状態では、多くのコールバックを渡す必要があり、すぐに古くなってしまいます。

幸いなことに、Flutter にはウィジェットが子孫にデータやサービスを提供する仕組みがあります（言い換えれば、子孫だけでなく、その下のウィジェットも含めて）。
Everything is a Widget™ の Flutter で期待されているように、これらのメカニズムは特別な種類のウィジェット-InheritedWidget、InheritedNotifier、InheritedModel などです。これらのメカニズムは、私たちがやろうとしていることに対して少し深い話なので、ここでは取り上げません。
その代わりに、前述の特別なウィジェットで動作するがシンプルなパッケージを使うことにします。これは Provider と呼ばれるものです。

Provider を使う前に、その依存関係を pubspec.yaml に追加することを忘れないでください。

```dart
name: my_name
description: Blah blah blah.

# ...

dependencies:
  flutter:
    sdk: flutter

  provider: ^4.0.0

dev_dependencies:
  # ...
```

Provider を使えば、コールバックや InheritedWidgets を気にする必要はありません。しかし、3 つの概念を理解する必要があります。

- ChangeNotifier
- ChangeNotifierProvider
- Consumer

### ChangeNotifier

ChangeNotifier は Flutter SDK に含まれるシンプルなクラスで、リスナーに変更を通知します。言い換えれば、何かが ChangeNotifier であれば、その変更をサブスクライブすることができます。(この用語に慣れている人には、Observable の一形態です)。

Provider では、ChangeNotifier はアプリケーションの状態をカプセル化するための 1 つの方法です。非常にシンプルなアプリでは、1 つの ChangeNotifier でなんとかなります。複雑なアプリでは、複数のモデルを持つことになるので、複数の ChangeNotifier が必要になります。(プロバイダと一緒に ChangeNotifier を使用する必要はありませんが、作業しやすいクラスです)

ショッピングアプリの例では、ChangeNotifier でカートの状態を管理したいと思います。このように、それを拡張した新しいクラスを作成します。

```dart
class CartModel extends ChangeNotifier {
  /// Internal, private state of the cart.
  final List<Item> _items = [];

  /// An unmodifiable view of the items in the cart.
  UnmodifiableListView<Item> get items => UnmodifiableListView(_items);

  /// The current total price of all items (assuming all items cost $42).
  int get totalPrice => _items.length * 42;

  /// Adds [item] to cart. This and [removeAll] are the only ways to modify the
  /// cart from the outside.
  void add(Item item) {
    _items.add(item);
    // This call tells the widgets that are listening to this model to rebuild.
    notifyListeners();
  }

  /// Removes all items from the cart.
  void removeAll() {
    _items.clear();
    // This call tells the widgets that are listening to this model to rebuild.
    notifyListeners();
  }
}
```

ChangeNotifier に固有のコードは、notifyListeners()の呼び出しだけです。アプリの UI を変更するような方法でモデルが変更された場合はいつでもこのメソッドを呼び出します。CartModel の他のすべてのコードはモデル自体とそのビジネスロジックです。

ChangeNotifier は flutter:foundation の一部であり、Flutter の上位クラスには依存しません。簡単にテストすることができます (ウィジェットのテストを使う必要はありません)。例えば、以下は CartModel の簡単なユニットテストです。

```dart
test('adding item increases total cost', () {
  final cart = CartModel();
  final startingPrice = cart.totalPrice;
  cart.addListener(() {
    expect(cart.totalPrice, greaterThan(startingPrice));
  });
  cart.add(Item('Dash'));
});
```

ChangeNotifierProvider は、その子孫に ChangeNotifier のインスタンスを提供するウィジェットです。これはプロバイダパッケージから来ています。

ChangeNotifierProvider をどこに置くかは既に分かっています: それにアクセスする必要があるウィジェットの上の方です。CartModel の場合、MyCart と MyCatalog の上のどこかを意味します。

ChangeNotifierProvider を必要以上に高く配置しないようにしましょう (スコープを汚したくないからです)。しかし、私たちの場合、MyCart と MyCatalog の両方の上にあるウィジェットは MyApp だけです。

```dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CartModel(),
      child: MyApp(),
    ),
  );
}
```

ここでは CartModel の新しいインスタンスを生成するビルダーを定義していることに注意してください。ChangeNotifierProvider は絶対に必要でない限り CartModel を再構築しない賢いビルダーです。また、インスタンスが不要になった場合は自動的に CartModel の dispose() を呼び出します。

```dart
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (context) => CartModel()),
        Provider(create: (context) => SomeOtherClass()),
      ],
      child: MyApp(),
    ),
  );
}
```

### Consumer

上部の ChangeNotifierProvider 宣言によって CartModel がアプリ内のウィジェットに提供されるようになったので、それを使い始めることができます。

```dart
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return Text("Total price: ${cart.totalPrice}");
  },
);
```

アクセスしたいモデルの種類を指定しなければなりません。この場合、CartModel が欲しいので、Consumer<CartModel>と書きます。ジェネリック(<CartModel>)を指定しないと、Provider のパッケージは役に立ちません。 Provider は型をベースにしているので、型がないと何が欲しいのかわかりません。

Consumer ウィジェットの必須引数はビルダーのみです。ビルダーは、ChangeNotifier が変更されるたびに呼び出される関数です。(言い換えれば、モデル内で notifyListeners() を呼び出すと、対応するすべての Consumer ウィジェットのすべてのビルダーメソッドが呼び出されます)。

ビルダーは 3 つの引数で呼び出されます。最初の引数はコンテキストで、これもすべてのビルドメソッドで取得します。

ビルダー関数の第 2 引数は ChangeNotifier のインスタンスです。そもそもこれが求めていたものです。モデル内のデータを使って、任意のポイントで UI がどのように見えるべきかを定義することができます。

3 番目の引数は子で、これは最適化のために存在します。Consumer の下に、モデルが変わっても変化しない大きなウィジェットサブツリーがある場合、一度構築してビルダーで取得することができます。

```dart
return Consumer<CartModel>(
  builder: (context, cart, child) => Stack(
        children: [
          // Use SomeExpensiveWidget here, without rebuilding every time.
          child,
          Text("Total price: ${cart.totalPrice}"),
        ],
      ),
  // Build the expensive widget here.
  child: SomeExpensiveWidget(),
);
```

Consumer ウィジェットは、できるだけツリーの奥深くに配置するのが最善の方法です。どこかで詳細が変更されたからといって、UI の大部分を再構築するのは避けたいものです。

```dart
// DON'T DO THIS
return Consumer<CartModel>(
  builder: (context, cart, child) {
    return HumongousWidget(
      // ...
      child: AnotherMonstrousWidget(
        // ...
        child: Text('Total price: ${cart.totalPrice}'),
      ),
    );
  },
);
```

代わりに

```dart
// DO THIS
return HumongousWidget(
  // ...
  child: AnotherMonstrousWidget(
    // ...
    child: Consumer<CartModel>(
      builder: (context, cart, child) {
        return Text('Total price: ${cart.totalPrice}');
      },
    ),
  ),
);
```

### Provider.of

UI を変更するためにモデル内のデータが本当に必要ない場合もありますが、それでもアクセスする必要があります。例えば、ClearCart ボタンは、ユーザーがカートからすべてを削除できるようにしたいとします。カートの内容を表示する必要はなく、clear()メソッドを呼び出すだけです。

このために Consumer<CartModel> を使うこともできますが、それは無駄なことです。再構築する必要のないウィジェットを再構築するようフレームワークに求めることになります。
このユースケースでは、Listen パラメータを false に設定して Provider.of を使用することができます。

```dart
Provider.of<CartModel>(context, listen: false).removeAll();
```

ビルドメソッドで上記の行を使用しても、notifyListeners が呼ばれたときにこのウィジェットが再構築されることはありません。

### まとめ

この記事で取り上げた例をチェックしてみてください。もっとシンプルなものが欲しい場合は、シンプルな Counter アプリをプロバイダで構築した場合にどのように見えるかを見てみましょう。

これらの記事に沿って進むことで、状態ベースのアプリケーションを作成する能力が大幅に向上しました。これらのスキルを習得するために、自分でプロバイダを使ってアプリケーションを作ってみてください。
