リポジトリ:<br/>
https://github.com/Hachiyan-88/tic-tac-toe

コミット:<br/>
5cf311a5cab559fba2dad28a6b49eb29cd7a33b7

# 総評
全体的に整っていて素晴らしいと感じました!<br/>
クラスの分け方も意図が伝わりますし、一つ一つの処理も大きすぎず読みやすくまとまっているかなと思います。ロジックの正当性という観点に関してはそこまで心配していないので保守性の高いコードを書くためには？という観点でレビューしました。ご確認ください。

gitのコミットメッセージやブランチ管理もできていそうなのもGoodです👍

# Board.java

### 同じ情報を示す値の定数化
同じ情報を示すものはなるべく共通化した方がいいです。<br/>
空を示す`' '`やマス目の上限値の`3`などはコード上の複数箇所で利用されているので、いざ仕様変更があったときに修正箇所が多くて大変になってしまいます。

# CPU.java

### メソッド名の統一感
`RandomMove`のみ大文字始まりの命名となってますが意図的でしょうか？<br/>
意図的ではないなら他に倣って小文字始まりとすべきです。<br/>

この辺もIDEによっては自動で検出またはフォーマットしてくれるものがあると思うので調べてみるといいと思います。


### ローカル変数のスコープはなるべく狭くする
`RandomMove`の中にあるローカル変数3つはwhileの中でしかから利用していないと思います。可能ならスコープは狭くしましょう。<br/>

意図しない箇所からの変数書き換えを防ぐためです。<br/>
巨大なオブジェクトならメモリ節約のために外に出すことはありますが、基本的に中にいた方が安全です。

```java
public void RandomMove() {

    while(true) {
        
        // whileの中で宣言
        int row = random.nextInt(3);
        int col = random.nextInt(3);
        
        if(board.getCell(row, col) == ' ') {
            board.setCell(row, col, '〇');
            System.out.println("CPUが（" + row +","+ col + "）に置きました");
            break;
        }
    }
}
```

# EasyCPU.java

### 　対象セルが空かどうか判断するのは誰？
```java
if(board.getCell(row, col) == ' ') {
    board.setCell(row, col, '〇');
    System.out.println("CPUが（" + row +","+ col + "）に置きました");
    break;
}
```

`board.getCell(row, col) == ' '`でからのセルか判定していますが、オブジェクト指向的にはこれは`Board`の役割かなと感じました。

空のセルかどうかという判定は他のクラスからも使われそうなので`Board`に内包した方が使い勝手が良さそうです。

```java
// Board側
public boolean isEmpty(int row, int col) {
    // cellが''ならtrueを返すロジック
}

// EasyCPU側
if(board.isEmpty(row, col)) {
    board.setCell(row, col, '〇');
    System.out.println("CPUが（" + row +","+ col + "）に置きました");
    break;
}
```

# Game.java

### 不要な変数宣言
`do`の中で`CPU cpu;`と宣言されていますが、どこで初期化されているかわからない変数はあまり好ましくないです。

この宣言を見た後、コードを読むときに「`cpu`はどこで初期化されるのかな？」と余計な邪念が発生するからですね。

```java
Board board = new Board();
Player player = new Player(scanner, board);

int cpuLevel = 0;
while(cpuLevel != 1 && cpuLevel != 2) {
    cpuLevel = InputUtil.readInt(scanner, "CPUの強さを選択してください (1.弱い 2.強い): ");
    if(cpuLevel != 1 && cpuLevel != 2) {
        System.out.println("無効な選択肢です。再度入力してください");
    }
}

// ここで直接宣言でいけないかな？（いけなかったらすみません）
CPU cpu = (cpuLevel == 1) ? new EasyCPU(board) : new HardCPU(board);
System.out.println(cpuLevel == 1 ? "CPUの強さ：弱い" : "CPUの強さ：強い");
```

## checkGameEndの簡略化
`board`と各シンボル（○ or ×）とメッセージを引数に渡して`checkGameEnd`を呼び出していますが、シンボルとメッセージまたは表示名（"あなた" or "CPU"）をオブジェクトの持ち物として持たせておけばこのメソッドも簡略化できます。

```java
public class Player {
    private string symbol = "○";
    private string name = "あなた";

    public string getSymbol() {
        return this.symbol;
    }

    public string getName() {
        return this.name;
    }
}

// CPUクラスも同様にシンボルと名前を持たせる

// checkGameEndメソッド
public static boolean checkGameEnd(
    Board board, Player player, Cpu cpu)
{
    var winSymbol = board.checkWinner();
    if(winSymbol == player.getSymbol()) {
        printWinMessage(player.Name);;
        return true;
    } else if (winSymbol == cpu.getSymbol()) {
        printWinMessage(cpu.Name);;
        return true;
    } 
    else {
        return false;
    }
}

private static void printWinMessage(string name) {
    System.out.println(name + "の勝利です");
}
```

こうすると呼び出し元が少し楽できます。<br/>
この辺の分け方については諸説ありそうですが、シンボルと名前はオブジェクトに持たせておくとこで他にも楽できそうなところが出てきそうですね。オブジェクト指向的にもシンボルと名前の情報は誰が持っておくべきなのかということを考えてみると実装が変わってくるかもしれません。

# GameController.java

### isBoardFullの置き場所
`GameController`自体はメインから複雑なロジックを分離させる意図として意味のあるものだとは思いますが、`isBoardFull`をこのクラスに置くべきかは検討の余地ありです。

他のメソッドが複数のインスタンスや引数を受け取ってなんらからの処理をしているのに比べて、`isBoardFull`は`Board`のインスタンスしか使っていないからですね。

ボードが埋まっているかどうかは誰が判断すべき？と考えてみると良いかもしれません。<br/>
誰かを経由してボードの中身を確認して判断するよりも、直接聞けた方が便利ですよね？

また、`Board`インスタンスしか触っていないロジックが`Board`の外に書かれているとその存在に気づかずに誰かが同じロジックを別のクラスに複数実装してしまう懸念もあります。