# 質問内容
```java
public Collection<? extends GrantedAuthority> getAuthorities() {}
```
1. この記述はSpring Bootだけの記法か？
2. 「GrantedAuthorityがもつメソッドやそれを継承した子クラスが持つメソッドを利用出来るという」という認識であっているか？

# 1. 使うのはSpring Bootだけか？
Spring Bootだけではなく、Javaのコーディングで一般的に使われる記法らしいです。

じゃあどういう意図で使われているのかというと、メソッドでreturnしたCollectionに新しい値をaddさせないため。

`Collection<? extends GrantedAuthority>`という型で値を返すと、呼び出し元は返ってきた値に新たに値を追加することはできません。

`GrantedAuthority`に`SimpleGrantedAuthority`というサブクラスがあるらしいのですが、その場合以下のような動きになります。

```java
// ただのGrantedAuthorityで宣言した場合
Collection<GrantedAuthority> exact = new ArrayList<>();
exact.add(new SimpleGrantedAuthority("ROLE_USER"));  // addが可能

// ワイルドカード付きで宣言した場合
Collection<? extends GrantedAuthority> wildcard = exact;
wildcard.add(new SimpleGrantedAuthority("ROLE_USER"));  // コンパイルエラー(add不可)
```

`getAuthorities()`はユーザーの権限リスト（例：ROLE_USER, ROLE_ADMIN）を返すため、実装者に勝手にリスト追加されてしまうのは避けたい。「認証済みの権限を覗くだけの窓口」で、改変は絶対禁止ということ。(追加が必要ならフレームワークのルールに従って追加する必要がある。コレクションの直接操作はNG)

**まとめると**<br/>
メソッドでreturnするCollectionに改変を加えて欲しくないときに使う記法と言えるでしょう。

# 2. 該当クラス + 子クラスが持つメソッドを利用できるという認識であっているか？
`GrantedAuthority`がそのサブタイプをすべてを意味するというのはその通り。だけど、子クラスが持つすべてのメソッドを利用できるかというとそれはNoです。

これを理解するためには**インターフェース**が何者かというところから復習する必要があります。

例えば、以下のコード。<br/>
```java
public interface Animal {
    // 鳴き声を定義
    String makeSound();
    // 鳴き声を出力する
    void speak();
}

class Dog implements Animal {
    // インターフェース由来のメソッド（実装必須）
    @Override
    public String makeSound() {
        return "ワンワン";
    }

    @Override
    public void speak() {
        System.out.println(makeSound());
    }
    
    // Dogクラス独自のメソッド（インターフェースにはない）
    public void wagTail() {
        System.out.println("尻尾を振る！");
    }
}

// DogをAnimalとしてインスタンス化した場合
Animal animal = new Dog();
animal.speak();    // Animalに定義されているメソッドは利用可
animal.wagTail();  // コンパイルエラー(Dogにしか存在しないメソッドは使えない)

// DogをDogのままインスタンス化した場合
Dog dog = new Dog();
dog.speak();    // インターフェースのメソッドも利用可
dog.wagTail();  // 独自のメソッドももちろん利用可
```

つまり、メソッドを利用できるのはインターフェースに実装されているメソッドのみということになります。

`GrantedAuthority`はインターフェースらしいのでそこに実装されているメソッドは使えますが、その子クラスにしか実装されていないメソッドは利用不可ということです☝️