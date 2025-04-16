---
title: "なぜReactではDIがあまり語られないのか？:DI/DIPの定義、なぜDIが語られないのか、結論"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "react"
  - "vue"
  - "DI(Dependency Injection)"
  - "DIP(Dependency Inversion Principle)"
published: true
---

# 前置き
サーバーサイドではDI(依存性注入)とかDIP(依存性逆転の原則)について語られることが多いです。

実際Springなどのサーバーサイフレームワークでは自動でDI,DIPを  
`DIコンテナー`と言うものを提供しています。

ただ、フロントエンドではあまりDIとDIPが語れないことが多いです。

そう言うことはフロントエンドでは  
このDIとDIPと言うことが必要ではないって言うことでしょうか。

DI/DIPは保守の観点では大事な概念のため  
恐らく違うではないかと自分は思っています。

その理由で今回はなぜReactではDIがあまり語られないのかと言う疑問についてお話したいと思っています。

# DI(依存性注入)/DIP(依存性逆転の原則)の定義

## DI(依存性注入)/DIP(依存性逆転の原則)の定義について

まずはDI(依存性注入)とDIP(依存性逆転の原則)の定義から始めて本論に行きたいと思います。

辞書的な定義は以下の通りです。

>依存性注入（Dependency Injection, DI）とは、  
コンピュータプログラムのデザインパターンの一つで、  
オブジェクトなどの間に生じる依存関係をオブジェクト内のコードに直接記述せず、  
外部から何らかの形で与えるようにする手法。 [^1]

>依存関係逆転（Dependency Inversion Principle, DIP）とは、  
高水準モジュールと低水準モジュールは、具体ではなく抽象に依存すべきであり  
抽象は詳細に依存してはならず、詳細が抽象に依存すべきである。

々元この二つの概念は**オブジェクト指向プログラミング**で良く現れる概念になります。

ただ、フロントエンド系言語(React、Vue、Angular)も含めて   
現在ほぼ全てのプログラミング言語では**オブジェクト指向プログラミング**が可能です。

結局、どのプログラミング言語でも**オブジェクト指向プログラミング**を採用されているのであれば気にするべきかと思います。

文書だけだと理解できないので、
Reactのコードでもう少し詳しく説明したいと思っています。

### DI(依存性注入)/DIP(依存性逆転)の例

フロントエンド内でDIのポイントは`外部から依存性を注入してもらう`ことになります。

もうちょっと理解しやすくすると  
`外部からオブジェクト(機能/コンポーネント)を引っ張ってくる`ことになるかと思います。

DIPの場合は全てのモジュール/コンポーネント/オブジェクトが`抽象化に依存する`ことにあります。 

抽象化してその内に変数/関数などを具現することで拡張性/保守性を担保できます。

まずは、依存性を注入/依存性逆転が含めている場合のReactコードを見ながら詳しく説明したいと思います。

```react:IconButton.tsx
import React, { ReactNode } from "react";

const IconButton = ({ icon, onClickIcon = () => {} }: IconButtonType) => {
  return (
    <div onClick={onClickIcon}>
      {icon}
    </div>
  );
};

export default IconButton;

type IconButtonType = {
  icon: ReactNode;
  onClickIcon: () => void;
};
```

```react:app.tsx
import IconButton from "./IconButton";

const App = () => {
  const onClickClose = () => {
    // クローズした時のロジック
  };
  const onClickOpen = () => {
    // オープンした時のロジック
  };
  return (
    <div>
      <IconButton
        icon={<CloseImage />}
        onClickIcon={onClickClose}
      />

      <IconButton
        icon={<OpenImage />}
        onClickIcon={onClickOpen}
      />
    </div>
  );
};

export default App;
```
上の例は特定な`アイコン(iconオブジェクト)`と`クリックした場合の挙動(onClickIcon)`を外部から注入してもらっているのでDIを満足、

追加でAppとIconButtonと言うオブジェクトは以下のように  
`({ icon, onClickIcon = () => {} }: IconButtonType)`  
Propsと言う**抽象化**したオブジェクトに依存しているため,DIPを一部満足しています。

ご存知の方もいらっしゃると思いますが、  
フロントエンドではこれを`コンポーネント化`と呼ばれています。

良く使っているまた使えそうな部分をコンポーネント化して
重複のコード作成防止・保守性を高められます。

関数化とほぼ同じ概念と思います。

このコードを最悪のパータンとして依存性を完璧に削除するとどうなるでしょうか。

```react:app.tsx
import CloseImage from "./CloseImage";
import OpenImage from "./OpenImage";

const App = () => {
  const onClickClose = () => {
    // クローズした時のロジック
  };

  const onClickOpen = () => {
    // オープンした時のロジック
  };

  return (
    <div>
      <div onClick={onClickClose}>
        <CloseImage />
      </div>
      <div onClick={onClickOpen}>
        <OpenImage />
      </div>
    </div>
  );
};

export default App;
```
関数化をしない時に現れる重複コードが発生してしまい、
保守性が低くなるのはもちろん再利用性も低くなります。

そのため、フロントエンドでは良く使えるUI(ボタン、カード、アイコン、機能など)を`コンポーネント化`することが多いです。

# なぜDI/DIPが語られないのか

ここからが本論になります。

結論から言うとフロントエンド`DI/DIPが必要ではない`よりは  
サーバーサイドでDI/DIPを実現して現れる効果を  
`他の対応によりほぼ同じ効果が現れる`からかと考えられます。

ただ、上のコードの例だと外部からアイコンイメージをコンポーネント
DIは満足していますが、  
DIPを半分しか満足してないことになります。 

なぜかと言うとDIPって言うものは`高水準モジュール`と`低水準モジュール`が、  
**具体ではなく抽象に依存すべき**のため、  
interfaceで抽象化して抽象化したものを相続してモジュールを実装する必要があります。

そのため、正確にDIPを満足するためには以下のコードになるべきです。

## なぜDI/DIPが語られないのか：完璧なDIPの例

```react:ButtonAction.ts
export interface ButtonAction {
  execute(): void;
}
```
まずは上のように抽象化を行い

```react:CloseAction.ts
import { ButtonAction } from "./ButtonAction";

export class CloseAction implements ButtonAction {
  execute(): void {
    // クローズした時のロジック
    console.log("Close action executed");
  }
}
```
```react:OpenAction.ts
import { ButtonAction } from "./ButtonAction";

export class OpenAction implements ButtonAction {
  execute(): void {
    // オープンした時のロジック
    console.log("Open action executed");
  }
}
```
詳細な実装は抽象化した`ButtonAction`から相続して**低水準モジュール**を実装します。

```react:IconButton.tsx
import React, { ReactNode } from "react";
import { ButtonAction } from "./ButtonAction";

type IconButtonType = {
  icon: ReactNode;
  action: ButtonAction;
};

const IconButton = ({ icon, action }: IconButtonType) => {
  return (
    <div onClick={() => action.execute()}>
      {icon}
    </div>
  );
};

export default IconButton;
```
その後、IconButtonと言う**低水準モジュール**を実装、
Propsのタイプとして抽象化した`ButtonAction`をタイプで設定します。

```react:App.tsx
import IconButton from "./IconButton";
import { CloseAction } from "./CloseAction";
import { OpenAction } from "./OpenAction";

const App = () => {
  const closeAction = new CloseAction();
  const openAction = new OpenAction();

  return (
    <div>
      <IconButton
        icon={<CloseImage />}
        action={closeAction}
      />
      <IconButton
        icon={<OpenImage />}
        action={openAction}
      />
    </div>
  );
};

export default App;
```
Appと言う**高水準モジュール**も抽象化した`ButtonAction`から相続して実装した
`closeAction`と`openAction`をPropsとして`IconButton`コンポーネントに受け渡します。

**高水準モジュール**と**低水準モジュール**が`ButtonAction`と言うInterfaceから実装されているので
上のコードなら完璧にDIPを満足していると考えられます。

## なぜDI/DIPが語られないのか：理由

```react:ButtonAction.ts
export interface ButtonAction {
  execute(): void;
}
```
上のinterfaceコードを作成し、相続化してモジュールを実装しました。

ただ、私の経験内ではフロントエンドの実務では
なかなか上の構成で開発したことがないです。

自分が考えている理由は以下の通りです。

1. コンポーネント化、状態管理、Custom HooksによってDIとDIPの一部が満足できる
2. Reactはクラス基盤よりHooks、関数基盤コンポーネントをお勧めしている
3. セキュリティーの理由のため、重要・複札なロジックはバックエンドで担当している
4. UIロジック中心であり、仕様変更に快速な対応が必要
5. TypeScriptのinterfaceは、コンパイル時にタイプチェックするだけで、ランタイムの時にはタイプが存在しない
6. ５番の理由で、DIコンテナー不在

もちろん、あくまでも自身の経験に基づいた結論ですので、  
必ずしも正確とは言えませんが、 

上に記載した理由のため、現状一般的なフロントエンド環境では  
完璧なDIPは不可で`緩やかなDIP`のみ、可能になっているではないかと考えております。

ただ、この理由の中で、５、６の場合、プログラミング言語/フレームワークの限界のため、  
少し細くお話したいと思います。

### TypeScriptのinterfaceはランタイムでは存在しない/DIコンテナーの不在

JavaScriptは動的にタイプを制御しているため、  
タイプが存在しないことはご存知かと思います。

```java:
public class Main {
    public static void main(String[] args) {
        int number = 10;
        String name = "John";
        boolean isActive = true;

        System.out.println(number);
        System.out.println(name);
        System.out.println(isActive);
    }
}
```
ご覧の通り、静的タイプである**JAVA**は変数を定義する時に  
`明示的にタイプを設定`する必要があります。

現在はコンパイラの向上でコーディングする時に事前にエラーを出力しますが、
コンパイラーがコンパイルする段階でエラーが発生してしまいます。

もちろんリターンのタイプ、引数、パラメータも同様です。
タイプをしっかり設定しないとコンパイルができないです。

```javascript:
let number = 10;
const name = "John";
var isActive = true;

console.log(number);
console.log(name);
console.log(isActive);
```
動的タイプである`JavaScript`はどうでしょうか。

ご覧の通り、JavaScriptではタイプを設定せず、
実際アプリが稼働するランタイムの時、JavaScriptエンジンが自動で設定します。

ただし、タイプが存在しないため、  
ランタイムでのエラーが多く発生してしまい  
それがシステムでも致命的なエラーが発生するリスクも高くなります。

そのため、JavaScriptはTypeScriptと言うライブラリを利用して(interface、タイプ定義)  
コンパイルした時にタイプチェックを行います。

ただ、TypeScriptはコンパイルする時のみ、タイプをチェックするだけで  
プログラムが動いているランタイムでは定義したタイプが存在しません。

これによって一般的なJavaScriptでは`DIコンテナー`も存在しておりません。

JavaScriptのライブラリ・フレームワークでも状況は同様です。

:::message  
AngularではDIコンテナーを提供していますが、  
こちらのDIコンテナーは`reflect-metadata`で作られています。

`reflect-metadata`はコンパイルするタイミングにmeta data作成して  
タイプをランタイムで参照できるようにするライブラリのため、  
正確にはタイプがランタイムでタイプが存在することではありません。
:::  
なので、完璧にDIPを実装するためには`手動注入`するしか方法がありません。

この`手動注入`する時の問題を含めて  
他の言語ではどんな形でDIコンテナーを利用しているか確認して見ましょう。

DIコンテイナーが存在するSpringでは以下のコードになります。

## JAVAの例：DIコンテナー利用

```java:ButtonAction.java
export interface ButtonAction {
  execute(): void;
}
```

```java:CloseAction.java
@Component("closeAction")
public class CloseAction implements ButtonAction {
  public void execute() {
    System.out.println("Close action executed");
  }
}
```

```java:OpenAction.java
@Component("openAction")
public class OpenAction implements ButtonAction {
  public void execute() {
    System.out.println("Open action executed");
  }
}
```

```java:IconButton.java
public class IconButton {
  private final String icon;
  private final ButtonAction action;

  public IconButton(String icon, ButtonAction action) {
      this.icon = icon;
      this.action = action;
  }

  public void render() {
    System.out.println("Rendering: " + icon);
    action.execute();
  }
}
```

```java:App.java
@Component
public class App implements CommandLineRunner {

  @Autowired // 自動注入
  @Qualifier("closeAction")
  private ButtonAction closeAction;

  @Autowired // 自動注入
  @Qualifier("openAction")
  private ButtonAction openAction;

  @Override
  public void run(String... args) {
    IconButton closeBtn = new IconButton("❌ Close", closeAction);
    IconButton openBtn = new IconButton("✅ Open", openAction);

    closeBtn.render();
    openBtn.render();
  }
}
```

ご覧通り、実際作成した`CloseAction`と`OpenAction`を**New**キーワードで注入するではなく、  
Annotation(@)を利用してinterfaceを登録します。

つまり、何を何を注入するかはフレームワークにお任せすることになります。

## JAVAの例：DIコンテナー利用なし

```java:App.java
public class App {
  public static void main(String[] args) {
    ButtonAction closeAction = new CloseAction(); // 手動注入
    ButtonAction openAction = new OpenAction();   // 手動注入

    IconButton closeBtn = new IconButton("❌ Close", closeAction);
    IconButton openBtn = new IconButton("✅ Open", openAction);

    closeBtn.render();
    openBtn.render();
  }
}
```
DIコンテナーを利用しない場合は、  
```
ButtonAction closeAction = new CloseAction(); // 手動注入
ButtonAction openAction = new OpenAction();   // 手動注入
```
コードが実装されるMain関数で手動で注入する必要があります。

オブジェクトが二つしかないので、
そこまでDIPを満足する必要ではないかと感じれるかもしれませんが

以下のコードを確認して見ましょう。

```java:App.java
public class App {
    public static void main(String[] args) {
        // 依存性を手動で注入
        UserRepository repo = new DbUserRepository();
        UserAccountService domain = new UserAccountService(repo);
        CloseUserAccountService useCase = new CloseUserAccountService(domain);
        ButtonAction closeAction = new CloseAction(useCase);
        IconButton button = new IconButton("❌ Close", closeAction);

        button.click();
    }
}
```

もちろん、実際サービスされているサーバーはこのコードより  
複札なコートになっているはずなので明らかにフロントエンドよりはバックエンドは
しっかりDIPを考えてプログラミングする方が良いかと思います。


# 終わり

なぜReactではDIがあまり語られないでしょうか？

いろいろ理由があるとは思いますが、  
`コンポーネント化、状態管理、Custom HooksによってDIとDIPの一部が満足できる`と  
`セキュリティーの理由のため、重要・複札なロジックはバックエンドで担当している`が
一番大きな理由ではないかと考えております。

後はJavaScript言語の動的型付け限界で  
一般的にはDIコンテイナーが存在しない理由もあります。  
そのため、DIPを実現するためには、手動で依存性を注入するしかないです。

これは恐らくフロントエンドでバックエンドほどテストコード作成が難しい理由になるかと思います。

もちろんDIコンテナーなどを支援する`Angular`だと可能ですが、  
現場で一般的に使われているものはReact又Vueのため、  
完璧なDIPは中々難しいかと思います。

将来にフロントエンド側でも重いロジック必要な時代になったら  
フロントエンドでもDIPが良く語られるではないでしょうか。

[^1]:https://e-words.jp/w/%E4%BE%9D%E5%AD%98%E6%80%A7%E6%B3%A8%E5%85%A5.html