---
layout: default
title: Containers
description: Learn the basic UINavigationController and UITabBarController container controllers.
categories:
- chapters
---

# Containers

さて、私たちは大きな白い画面を手にしました。それをドレスアップしましょう！

みなさんが iOS アプリをいくつか使ったことがあるのでしたら、いくらか似かよった見た目であることに間違いなく気がつくことでしょう。主に、多くのアプリは下に黒いタブバー、上には青いタイトルバーがあります。これらは iOS での標準的な「コンテナ」で、それぞれ `UITabBarController` と `UINavigationController` と知られています。これから、それらで遊ぶつもりです。

これらのコンテナは `UIViewController` のサブクラスで、ほかの `UIViewController` を管理します。ワイルドだろ？コンテナは通常のコントローラのように `view` を持っていて、サブビューとして「子」のコントローラのビューが追加されます。ありがたいことに、これらのコンテナコントローラは、ビューでは無くコントローラを処理するために便利な API が用意されているので、私たちは *どのように* 動くか心配する必要がありません。

## UINavigationController

最も一般的な `UINavigationController` で始めてみましょう。スタックとして子のコントローラを管理し、見た目には水平方向にプッシュ/ポップが行われます。Mail.app では、Accounts -> An Account -> Inbox -> Message というフローのために `UINavigationController` が使われています。ナビゲーションコントローラは自動的に戻るためのボタンを処理してくれるため本当にすばらしいです。みなさんがしなければいけないのは、コントローラをプッシュ/ポップすることだけです。

![Navigation controller in Mail.app](images/nav_bar.png)

`AppDelegate` で、`rootViewController` に割り当てていたコントローラを `UINavigationController` を使って変更しましょう。

```ruby
  ...
  controller = TapController.alloc.initWithNibName(nil, bundle: nil)
  @window.rootViewController = UINavigationController.alloc.initWithRootViewController(controller)
  ...
```

`initWithRootViewController` は指定されたコントローラを受け取り、そのコントローラでスタックを開始します。

アプリを実行する前に、`TapController` でもう少し変更を行います。

```ruby
  ...
  self.view.addSubview @label

  self.title = "Tap"
  ...
```

`rake` を実行して、すこしきれいになったアプリを確認してみましょう。

![Navigation controller](images/1.png)

いいねぇ〜。それでは、何か作ってみましょう。ナビゲーションコントローラのスタックに、いくつかの `TapController` インスタンスをプッシュするナビゲーションバーのボタンを追加してみます。

画面上部のナビゲーションバーに実際にボタンを配置することができます。たとえば、Mail.app では「編集」ボタンがナビゲーションバーに配置されています。これらのボタンは `UIBarButtonItem` のインスタンスで、いくつかオプションを設定できます(テキストを使用する？画像は？システムアイコンは？などなど)。

`TapController` の `viewDidLoad` で、ボタンを追加してみましょう。

```ruby
def viewDidLoad
  ...
  self.title = "Tap"

  rightButton = UIBarButtonItem.alloc.initWithTitle("Push", style: UIBarButtonItemStyleBordered, target:self, action:'push')
  self.navigationItem.rightBarButtonItem = rightButton
end
```

これが何をしているのか詳しく見てみましょう。ボタンのタイトルとスタイルを指定して `UIBarButtonItem` を作成します。スタイルプロパティは、ボタンの見た目を決めます(枠線が無いボタン、枠線があるボタン、操作を「完了」するときなどに用いる青いボタン。違いを確認するために遊んでみてください)。次に、ボタンを コントローラの `navigationItem` の `rightBarButtonItem` へ設定します。すべての `UIViewController` には `navigationItem` があり、画面上部の青いバーで表示されている情報へアクセスするための手段となります。
(注意: `UINavigationItem` は `UIView` ではありません！そのため、サブビューを追加することはできません)。

`target` と `action` の役割は何でしょう？えぇと、これは Ruby の世界へオリジナルの Objective-C SDK が漏れ出てきたものです(´д｀) ごく最近まで、Objecive-C で無名関数をコールバックとして渡すことができませんでした。その代替手段として、オブジェクトとオブジェクトから呼び出すメソッドの名前を API に渡しています。私たちはこの処理を Ruby でブロックとラムダで行いますが、悲しいことに古い iOS の API は古くささを示してます。

それはさておき、`target` は `action` のメソッドを呼び出してもらいたいオブジェクトを指定します。より良いアイディアを得るために `TapController` へ `push` メソッドを実装してみましょう！

```ruby
...
  def push
    new_controller = TapController.alloc.initWithNibName(nil, bundle: nil)
    self.navigationController.pushViewController(new_controller, animated: true)
  end
...
```

どうでしょう？ユーザがバーのボタンをタップしたときに、`target` で指定したオブジェクト(このケースでは私たちのコントローラ) の `push` が呼ばれます。

`navigationItem` に付け加えると、利用可能な場合には `UIViewController` は `navigationController` というプロパティも持っています。ナビゲーションコントローラでは、`pushViewController` を呼び出すと渡されたコントローラをスタック上にプッシュします。デフォルトでは、ナビゲーションコントローラは戻るボタンも表示します。この戻るボタンは現在のコントローラをポップする処理を行います(通常は、ナビゲーションコントローラ上で `popViewControllerAnimated:` を私たちが呼び出します)。ちゃんとわかりましたか？

アプリを実行する前にもう少しだけ手を加えてみましょう。次のように、ナビゲーションスタックでのコントローラの位置をコントローラのタイトルに反映してみましょう。

```ruby
  def viewDidLoad
    ...
    self.title = "Tap (#{self.navigationController.viewControllers.count})"
    ...
  end
```

さぁ、`rake` を実行して動的に変化するコントローラを観察してみましょう！

![pushable controllers](images/2.png)

さて、`UITabController` も取り扱うよと言いましたので、さっそくそれに取りかかってみましょう。

## UITabBarController

![Navigation controller in Mail.app](images/tab_bar.png)

タブコントローラは `UINavigationController` やほかのコンテナコントローラとよく似て、コンテナのメッキ部分(黒いバーのところ)に提示される `viewControllers` のリストを持っています。しかし異なる点もあり、`UITabBarController` は window の `rootViewController` として動作することのみをサポートしています。(すなわち、ナビゲーションコントローラでタブバーコントローラをプッシュしてはいけません)。

`AppDelegate` で、小さな変更を加えてみましょう。

```ruby
  ...
  controller = TapController.alloc.initWithNibName(nil, bundle: nil)
  nav_controller = UINavigationController.alloc.initWithRootViewController(controller)

  tab_controller = UITabBarController.alloc.initWithNibName(nil, bundle: nil)
  tab_controller.viewControllers = [nav_controller]
  @window.rootViewController = tab_controller
  ...
```

これまでのコード例と異なり、新しいものは何もありません！`UITabBarController` を普通の `UIViewController` のときのように作成し、タブコントローラの `viewControllers` にナビゲーションコントローラを配列に入れたものを設定しています。

`rake` を実行し、チェックしてみましょう！

![tab bar controller](images/3.png)

ちょっと見た目にガッカリしたんじゃないでしょうか。タブバーにアイコンを追加し綺麗にしてみましょう。

`navigationItem` と同様に、すべての `UIViewController` には `tabBarItem` というプロパティがあり、`UITabBarItem` のインスタンスを受け付けます。アイコン、タイトル、あるいはコントローラのタブ部分の見た目に関するオプションをカスタマイズする際にこのオブジェクトを使うことができます。

TapController で `initWithNibName:bundle:` をオーバーライドし、次のようなオブジェクトを作成します。

```ruby
  ...
  def initWithNibName(name, bundle: bundle)
    super
    self.tabBarItem = UITabBarItem.alloc.initWithTabBarSystemItem(UITabBarSystemItemFavorites, tag: 1)
    self
  end
  ...
```

これは `UITabBarItem` のための初期化メソッドです。独自の画像やタイトルを利用したい場合には、`initWithTitle:image:tag:` も使うことができます。独自の画像を利用する場合は、30x30 の黒い透過なアイコンにする必要があります。

`initWithTabBarSystemItem:` はタブのアイコンをデモンストレーションするのを少し楽にしてくれますが、システムで用意されている画像に対応するタイトルが強制的に使用されます(このケースでは、"Favorites")。

なぜ `initWithNibName:bundle:` のなかで記述しているのでしょう？コントローラのビューが用意されたかどうかにかかわらず、コントローラが用意されたらできるだけ早く `tabBarItem` を作成したいからです。もし `viewDidLoad` に記述した場合、アプリが起動したときにすぐに作成できないかもしれません(ユーザによってコントローラに初めてアクセスがあったときに、タブバーコントローラは子のコントローラをロードします)。

One more thing! ほかのタブも作成してみましょう。このタブで行う処理はありませんので、`AppDelegate` で、異なる背景色で空の `UIViewController` を作りましょう。

```ruby
  ...
  other_controller = UIViewController.alloc.initWithNibName(nil, bundle: nil)
  other_controller.title = "Other"
  other_controller.view.backgroundColor = UIColor.purpleColor

  tab_controller = UITabBarController.alloc.initWithNibName(nil, bundle: nil)
  tab_controller.viewControllers = [nav_controller, other_controller]
  @window.rootViewController = tab_controller
  ...
```

`rake` を実行してできあがり！コンテナコントローラの全容です！これらは全体の一部にすぎませんが、iOS アプリの 80% を構成する積み木はわずかなクラスであることが簡単に分かるでしょう。

![multiple tabs](images/4.png)

## Subway Up

私たちがカバーした範囲は、次のことまで広がりました。

- iOS SDK は子のビューとなるコントローラを含むために `UINavigationController` と `UITabBarController` を使用します。
- `UINavigationController` はスタックをコントロールするために 'pushViewController:animated:' と 'popViewControllerAnimated:' を使います。
- 画面上部の青いバーのボタンやそのほかのオプションを変更するのには `controller.navigationItem` を使用します。
- `UITabBarController` は子のコントローラをコントロールするために `viewControllers=` を使用します。注意として、`UITabBarController` は window の `rootViewController` としてしか使えません。
- タブのアイコンやタイトルを変更するのには `controller.tabBarItem` を使用します。

["The Lord of the Rings" のモルドールへ向けて歩こう。次の章ではテーブルにについて学びます！]
(/5-tables)