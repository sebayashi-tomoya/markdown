# What is 非同期プログラミング？
非同期プログラミングは、アプリが「固まらずに」動くようにするための仕組みです。サーバー通信やファイル読み込みといった“待ち時間のある処理”を、アプリが他の仕事を進めながら行えるようにするために使用されます。

今回は、Microsoft公式ドキュメントの「朝食作りの例え」に則って、非同期プログラミング(async/await)の本質を読み解いていきましょう！

https://learn.microsoft.com/ja-jp/dotnet/csharp/asynchronous-programming/

# 朝食を作る工程
朝食は以下の工程で作られるとします。
1. コーヒーを淹れる
2. フライパンを温める
3. 卵を焼く
4. パンを焼く
5. パンにバターとジャムを塗る

※　公式ドキュメントに記載があっても本記事で触れていない工程は削除しています

# 教訓1: 同期処理では「冷めた朝食」になる
"同期処理"とは、「一つずつ終わるまで待つ」方式です。<br/>
非同期処理を意識せずに書いた処理は基本的に同期処理として実行されていきます。

ただ、これでは、コーヒーを淹れ終わってから卵を焼き、それが終わってからベーコンを焼き...と一つ一つの作業を順々に処理していくため、最後の工程が終わる頃には朝食は冷めてしまいますね🥲

実際のアプリケーションの場合も、API通信やDBアクセス処理などを常に同期的に処理していると、処理は動いているのにユーザーからは画面が固まったように見えてしまいます。

非同期処理を使えば、コーヒーを淹れながら卵を焼き、トーストなども同時に実行できます。<br/>
まずは、「待つ時間に別の仕事をする」のが非同期処理だということを覚えましょう。

# 教訓2: すぐに待たない
卵を焼きながら、パンを焼く術を手に入れたとしても結局卵が焼き終わるのを待っていては元も子もありません。

なかなかあり得ないケースのように思えますが、実際に非同期メソッドを呼び出す際にすぐawaitしてしまう実装はやってしまいがちです。

例えば、以下の場合では非同期処理を使っているのにも関わらず、卵が焼き終わるまでパンと焼き始めることができません。

```c#
Egg eggs = await FryEggsAsync(2);
Toast toast = await ToastBreadAsync(2);

// 非同期で卵を焼く
private static async Task<Egg> FryEggsAsync(int howMany)
{
    // フライパンを温める
    Console.WriteLine("Warming the egg pan...");
    await Task.Delay(3000);
    // 卵を焼く
    Console.WriteLine($"cracking {howMany} eggs");
    Console.WriteLine("cooking the eggs ...");
    await Task.Delay(3000);
    // 盛り付けて完了
    Console.WriteLine("Put eggs on plate");

    return new Egg();
}

// 非同期でパンを焼く
 private static async Task<Toast> ToastBreadAsync(int slices)
{
    // 指定の枚数にスライスする
    for (int slice = 0; slice < slices; slice++)
    {
        Console.WriteLine("Putting a slice of bread in the toaster");
    }
    // トーストをスタート
    Console.WriteLine("Start toasting...");
    await Task.Delay(3000);
    // トースターから取り出して完了
    Console.WriteLine("Remove toast from toaster");

    return new Toast();
}
```

これを改善するには、まず全部のタスクを開始してから処理の完了を待つようにします。

```c#
// 呼び出し時はawaitしない→ スレッドをブロックしない
Task<Egg> eggsTask = FryEggsAsync(2);
Task<Toast> toastTask = ToastBreadAsync(2);

// すべてのタスクを実行してから完了を待つ
await Task.WhenAll(eggsTask, toastTask);
```

これで同時に調理（実行）され、全体の時間を短縮できます。「awaitは結果が必要になってから使う」 ということが大切です☝️

# 教訓3: 非同期処理は連鎖する
メソッドの中で、他の非同期メソッドを呼び出すときはそのメソッド自体も`async`となります。

例えば、以下の一連の処理のメソッドを作る場合。<br/>
パンを焼く（非同期）<br/>
↓<br/>
バターを塗る（同期）<br/>
↓<br/>
ジャムを塗る（同期）

このような実装になります。
```c#
// 内部で非同期処理が一つでも実行されていればasyncをつける
static async Task<Toast> MakeToastWithButterAndJamAsync(int number)
{
    var toast = await ToastBreadAsync(number); // 非同期
    ApplyButter(toast); // 同期
    ApplyJam(toast); // 同期
    return toast;
}
```

非同期メソッド内でawaitを使う処理があると、そのメソッド自体もasyncにする必要があります。こうして非同期が上の階層へ伝わることで、アプリ全体が「一貫して非同期」で動作できるようになります。

# 教訓4: 例外捕捉はタスクを待つタイミングで
非同期メソッド内でエラーが起きた場合、C#は自動的に例外を捕捉し、`await`した時点でスローします。これにより、通常のtry/catchで自然に処理できます。

例えば、パンを焼いている途中でトーストが壊れたとします。<br/>
トーストを実行する非同期メソッドを以下のように書き換えます。

```c#
// 非同期でパンを焼く
 private static async Task<Toast> ToastBreadAsync(int slices)
{
    // 指定の枚数にスライスする
    for (int slice = 0; slice < slices; slice++)
    {
        Console.WriteLine("Putting a slice of bread in the toaster");
    }
    // トーストをスタート
    Console.WriteLine("Start toasting...");
    await Task.Delay(3000);
    
    // トースターが故障！
    Console.WriteLine("Fire! Toast is ruined!");
    throw new InvalidOperationException("The toaster is on fire");
    
    // トースターから取り出して完了
    Console.WriteLine("Remove toast from toaster");

    return new Toast();
}
```

この場合、`Task`内で発生した例外は即時にはスローされず、タスク完了時に再スローされます。そのため、`await`のタイミングが例外捕捉のタイミングになります。

つまり、以下のように例外処理をしても意味がないということです。
```c#
try
{
    Task<Toast> toastTask = ToastBreadAsync(2);
}
catch  (InvalidOperationException ex)
{
    Console.WriteLine($"トースト失敗: {ex.Message}");
}
await Task.WhenAll(toastTask);
```

正しく例外を捕捉するためには、awaitのタイミングに注意しましょう！

```c#
try
{
    var toastTask = ToastBreadAsync(2);
    // awaitのタイミングをtryに入れる
    await Task.WhenAll(toastTask);
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"トースト失敗: {ex.Message}");
}
```

# まとめ：「人間らしく」コードを書く
非同期プログラミングは、コンピュータに“人間の賢いやり方”を教える技術です。同時並行で進め、効率よく待ち、必要なときにだけ手を止める。これがasync/awaitの本質です。

**キーポイントまとめ**
- 同期処理は時間がかかる（冷めた朝食）
- awaitは“すぐにつけない”のが並行のコツ
- 非同期は連鎖する（asyncが伝染）
- 例外処理もシンプル、try/catchでOK(捕捉はawaitのタイミングで)