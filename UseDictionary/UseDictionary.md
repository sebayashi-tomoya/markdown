
# 値を追加したい！
**使用するメソッド**
```c#
public void Add (TKey key, TValue value);
```

**使用例**
```c#
var dic = new Dictionary<int, string>();

dic.Add(1, "Good Morning!");
dic.Add(2, "Good Evening!");
dic.Add(3, "Good Night!");

foreach (var keyValuePair in dic)
{
    Console.WriteLine($"Key: {keyValuePair.Key}");
    Console.WriteLine($"Value: {keyValuePair.Value}");
}

// 出力結果
// Key: 1
// Value: Good Morning!
// Key: 2
// Value: Good Evening!
// Key: 3
// Value: Good Night!
```

# 値を全部消したい！
**使用するメソッド**
```c#
public void Clear ();
```

**使用例**
```c#
var dic = new Dictionary<int, string>
{
    { 1, "Good Morning!" },
    { 2, "Good Evening!" },
    { 3, "Good Night!" }
};

dic.Clear();
Console.WriteLine($"Dictionary's count is {dic.Count}");

// 出力結果
// Dictionary's count is 0
```

# 指定したキーが含まれているか確認したい！
**使用するメソッド**
```c#
public bool ContainsKey (TKey key);
```

**使用例**
```c#
var dic = new Dictionary<int, string>
{
    { 1, "Good Morning!" },
    { 2, "Good Evening!" },
    { 3, "Good Night!" }
};

for (var i = 1; i <= 4; i++)
{
    if (dic.ContainsKey(i))
    {
        Console.WriteLine(dic[i]);
    }
    else
    {
        Console.WriteLine("No Message");
    }
}

// 出力結果
// Good Morning!
// Good Evening!
// Good Night!
// No Message
```

# 指定の値が含まれているか確認したい！
**使用するメソッド**
```c#
public bool ContainsValue (TValue value);
```

**使用例**
```c#
var morningGreeting = "Good Morning!";
var hello = "Hello!";

var dic = new Dictionary<int, string>
{
    { 1, morningGreeting },
    { 2, "Good Evening!" },
    { 3, "Good Night!" }
};

if (dic.ContainsValue(morningGreeting))
{
    Console.WriteLine(morningGreeting);
}

if (dic.ContainsValue(hello))
{
    Console.WriteLine(hello);
}
else
{
    Console.WriteLine("No Message");
}


// 出力結果
// Good Morning!
// No Message
```

# 　上限数が決まっているから容量を確保しておきたい！
**使用するメソッド**
```c#
public int EnsureCapacity (int capacity);
```

**使用例**
```c#
public static Dictionary<int, string> CreateDictionary(IEnumerable<string> bigData)
{
    var dic = new Dictionary<int, string>();

    // 件数が分かっているなら、先に Dictionary の容量を見積もっておく
    // 仮に大きめのデータを受け取ったとして、そのデータ数で設定
    int dataCount = bigData is ICollection<string> d ? d.Count : 0;
    if (dataCount > 0)
    {
        dic.EnsureCapacity(dataCount);
    }

    int index = 0;
    foreach (var data in bigData)
    {
        dic[index++] = data;
    }

    return dic;
}
```

※ 仮に容量を超えてデータが追加されても超過分がDictionaryに追加されないことはないが、容量拡張処理のオーバーヘッドが発生するため注意です☝️

# 指定したキーのを持つ値を削除したい！
**使用するメソッド**
```c#
// 値を消すだけ
public bool Remove (TKey key);

// 消した値を使いたいとき
public bool Remove (TKey key, out TValue value);
```

**使用例**
```c#
var dic = new Dictionary<int, string>
{
    { 1, "Good Morning!" },
    { 2, "Good Evening!" },
    { 3, "Good Night!" }
};

// 消すだけ
for (var i = 1; i <= 4; i++)
{
    if (dic.Remove(i))
    {
        Console.WriteLine($"Removing is completed. key: {i}");
    }
    else
    {
        // 存在しないキーを指定した場合はfalseが返ってくる
        Console.WriteLine("Key does not exist.");
    }
}

// 消した値を使いたいとき
for (var i = 1; i <= 4; i++)
{
    if (dic.Remove(i, out var removed))
    {
        Console.WriteLine($"Removing is completed. value: {removed}");
    }
    else
    {
        // 存在しないキーを指定した場合はfalseが返ってくる
        Console.WriteLine("Key does not exist.");
    }
}

// 出力結果
// Removing is completed. key: 1
// Removing is completed. key: 2
// Removing is completed. key: 3
// Key does not exist.

// Removing is completed. value: Good Morning!
// Removing is completed. value: Good Evening!
// Removing is completed. value: Good Night!
// Key does not exist.
```

# 値の追加、取得を安全に実行したい！
**使用するメソッド**
```c#
// 値の追加
public bool TryAdd (TKey key, TValue value);

// 値の取得
public bool TryGetValue (TKey key, out TValue value);
```

**使用例**
```c#
var dic = new Dictionary<int, string>
{
    { 1, "Good Morning!" },
    { 2, "Good Evening!" },
    { 3, "Good Night!" }
};

// dic.Add(1, "Hello"); → すでに存在するキーでAddを呼び出すと例外になる
// 例外を使わずに値を追加可能かチェックできる
var addKey = 1;
var addValue = "Hello";
if (dic.TryAdd(addKey, addValue))
{
    // trueの場合は既にDictionaryに値が追加されている
}
else
{
    Console.WriteLine("Duplicate key");
}

// dic[4] →　存在しないキーを指定すると例外になる
// 例外を使わずに値を取得可能かチェックできる
var getKey = 1;
if (dic.TryGetValue(1, out var greeting))
{
    Console.WriteLine(greeting);
}
else
{
    Console.WriteLine("Key does not exist");
}

// 出力結果
// Duplicate key
// Key does not exist
```