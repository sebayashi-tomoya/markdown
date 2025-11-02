# C#のList<T>を使いこなせていますか？
普段のコーディングで`List<T>`をこれでもかってくらい酷使していると思いますが、実は実装しているインターフェースが6種類あります。

https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1?view=net-9.0

```c#
public class List<T> :
System.Collections.Generic.ICollection<T>,
System.Collections.Generic.IEnumerable<T>,
System.Collections.Generic.IList<T>,
System.Collections.Generic.IReadOnlyCollection<T>,
System.Collections.Generic.IReadOnlyList<T>,
System.Collections.IList
```

これ、それぞれの役割と使い分けできていますか？
今回は使い分けを学んで`List<T>`のポテンシャルを存分に活かせるCShaperになろうと思います。

# それぞれの特徴・用途
まずは特徴と用途をざっくり理解しましょう。
ほぼ使わない非ジェネリックのインターフェースは除外します。

<table>
  <thead>
    <tr>
      <th>インターフェース名</th>
      <th>説明・特徴</th>
      <th>使い分け・主な用途</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>IEnumerable&lt;T&gt;</td>
      <td>foreachやLINQ用の列挙機能、読み取りのみ。</td>
      <td>コレクション中身を順次処理したい場合、LINQ適用時</td>
    </tr>
    <tr>
      <td>IReadOnlyCollection&lt;T&gt;</td>
      <td>読み取り専用のコレクション機能。</td>
      <td>参照のみで、サイズ等のコレクション統計だけ取得したい場合</td>
    </tr>
    <tr>
      <td>IReadOnlyList&lt;T&gt;</td>
      <td>インデックスアクセス可能な読み取り専用リスト。</td>
      <td>順序を意識した参照のみ・外部からの変更禁止を保証したい場合</td>
    </tr>
    <tr>
      <td>ICollection&lt;T&gt;</td>
      <td>Add/Remove/Countなどコレクション操作可。</td>
      <td>要素数取得やコレクション全体の操作が欲しい場合</td>
    </tr>
    <tr>
      <td>IList&lt;T&gt;</td>
      <td>インデックス操作・挿入・削除可。</td>
      <td>順序付きコレクションで、ランダムアクセスや追加・削除が必要な場合</td>
    </tr>
  </tbody>
</table>

# 使い分け方
では、具体的にどのように使い分けていくのか。
メソッドの引数や戻り値にインターフェースを宣言して使い分けることが多いです。

例えば、引数で`List<string>`を受け取って中身を一つずつコンソールに出力するメソッドを作りたいとき。コンソールに出力するのみで特に中身は書き変わらないので、`IReadOnlyList<string>`で十分です。

```c#
private static void WriteItems(IReadOnlyList<string> items)
{
  foreach (var item in items)
  {
    Console.WriteLine(item);
  }
}

var members = new List<string>{ "MOMO", "SANA", "MINA" };
WriteItems(members);
```

このように宣言しておくことで、メソッドを呼び出す側はリストの中身が変わることによる影響を考えなくて良くなります。

戻り値に宣言する場合は逆ですね。
メソッド側の事情で返すリストの中身を変えて欲しくないということを呼び出し側に伝えることができます。

# 使い分けルール
引数や戻り値の宣言で使い分けられるのはわかったが、どのようなルールで分けるべきなのか。

基本的には抽象度の高い（機能の少ない）インターフェースから利用できないかを検討します。機能に不足がある場合に抽象度の低いインターフェースへの置き換えていきます。

抽象度の順序は以下の通り。上から高い順です。
```
IEnumerable<T>
↓
IReadOnlyCollection<T>
↓
IReadOnlyList<T>
↓
ICollection<T>
↓
IList<T>
```

つまり、まずは提供する機能が少ない`IEnumerable<T>`で十分でないかを確認します。その後実装の都合で別インターフェースの機能が必要だと判明したら抽象度を下げていくという感じです。

引数や戻り値に必要な機能を設計時点で明言および制限をかけておくことで、設計者の意図しない使われ方をされることを防止できるわけです☝️

# ListとCollectionの違い
「問答無用で`IList<T>`を使います！`ICollection<T>`なんていつ使うんですか？」
なんて人もいることでしょう。

ListとCollectionの違いはずばり、"インデックスでの操作"ができるかどうかです。

そもそも`IList<T>`が`ICollection<T>`の拡張なので、`IList<T>`を使っておけば
大体のことは事足ります。

ただ、インターフェースが分けられている以上使い分けるメリットはもちろんあります。

- **カスタムコレクション作成や委譲で役立つ**<br/>
独自のコレクション型を作ったり、内部実装を隠蔽して委譲する際に最低限の契約として利用することが可能です。例えば`Dictionary<K,V>`は`ICollection<KeyValuePair<K,V>>`を実装していて、順序や位置に依存しない汎用的なコレクション操作の契約として使えます。

- **パフォーマンス面の利点も間接的に**<br/>
自明ですが、不必要にインデックスアクセスを要求しない分、実装に対して余計な負担をかけず、より効率の良い構造を使いやすくする面もあります。

使い分けルールに則るのであれば、インデックスアクセスが不要なケースではListよりCollectionが適切ということになります。

# まとめ
- List<T>は多くのインターフェースを実装しており、さまざまな場面で汎用的に使える強力な型である
- インターフェースを引数や戻り値に使い分け、必要な機能だけを提供する契約を明確に作る
- まずは抽象度の高いIEnumerable<T>やIReadOnly系から検討し、機能不足を感じたら抽象度の低いもの使う




