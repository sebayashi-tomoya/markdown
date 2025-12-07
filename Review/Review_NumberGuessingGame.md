# GameLauncherとNumberBattleGameのクラスを分けた理由は？
READMEを読む限り、<br/>
GameLauncher 👉 ゲームの流れ管理<br/>
NumberBattleGame 👉 ゲームの実処理<br/>
と理解しています。

あくまで私の感じ方なのでいろんな考え方があると思いますが、ゲーム開始までの流れを担うだけの今のクラスであれば`main`メソッドにそのまま書いちゃえばいいのでは？と感じました。

仮に分けて実装するとしたら、GameLauncherを見るだけでレベル選択からリプレイ確認やゲーム終了までの流れが理解できるような設計になっていれば意義が出てくるかなと思います。

NumberBattleGameの`startGame`メソッドまでをGameLauncherの役割とするとかですかね？

あとは、このようなクラスではアプリケーション内で適切に処理されなかった例外を捕捉してアプリケーションが落ちないように対処することもあります。

👇　仮に実装してみた
```java
public class GameLauncher {

    public void Run() {
        try {
            this.RunCore();
        } catch(Exception e) {
            // 未処理の例外を捕捉してアプリケーションが落ちないようにする
            System.out.println("例外が適切に処理されていません。" + e.Message);
        }
    }

    // ゲーム終了までの流れをこのメソッドだけで理解できるようにする
	public void RunCore() {
		Scanner scanner = new Scanner(System.in);
		Config config = new Config();
		
        // レベル選択
		config.setCpuLevel(selectCpuLevel(scanner));
		
        // プレイヤーの設定
		HumanPlayer player = new HumanPlayer(scanner, "");
		player.inputName();
		config.setPlayerName(player.getName());
		
        // メインループ
		NumberBattleGame game = new NumberBattleGame(config, player, scanner);
        ReplayManager replay = new ReplayManager(scanner);

        // 一連の流れになっていればNumberBattleGame.starGameで実施していた名前の未設定チェックも不要になりそう

        do {
            game.playOneGame();
        } while (replay.askReplay());

        System.out.println("ゲームを終了します。ありがとうございました！");
		
		scanner.close();
	}
}
```

実際にこれで動作するかはあまり自信はないですが、イメージはこんな感じです。

クラスを分割しすぎるとかえって処理が追いにくくなってしまうこともあります。セルフレビューの際にご自身がクラスを分けた意図を思い返して、不要なクラスが生まれてしまっていないか確認してみると良いと思います。

もちろん見直した上で必要なクラスだと判断できれば全く問題ありません！「なんとなく分けておいた方がいいだろう」でクラスが増えていくのがよくないです。

# Configクラスの危険性
ゲームに必要な設定をConfigクラスにまとめたの非常に素晴らしいと思いました。ただ、`playerName`がHumanPlayerクラスのフィールドにも存在しており、情報の重複が気になりました。

仮に今後プレイヤー名を別の箇所で変更するといった追加機能を実装するとなった場合に、お行儀よくHumanPlayerとConfigの`playerName`を同時に更新してくれる実装を継いでくれれば良いのですが、そこが漏れると整合性が取れずに不具合になる可能性が高いです。

すみません。以下のコードの意図が分からず...<br/>
もしかしたらここで考慮されていたのですかね？
```java
// NumberBattleGame.java
public void startGame() {
    ReplayManager replay = new ReplayManager(scanner);
    
    // ここで考慮している？---------------------------
    if(config.getPlayerName().isEmpty()) {
        player.inputName();
        config.setPlayerName(player.getName());
    } else {
        player.setName(config.getPlayerName());
    }
    // --------------------------------------------

    do {
        playOneGame();
    } while (replay.askReplay());

    System.out.println("ゲームを終了します。ありがとうございました！");
}
```

どちらにせよ同じ情報を他の場所に分散させるのは得策ではないです。HumanPlayerに統合する or どうしてもConfigに`playerName`が必要ならHumanPlayerの`setName`にConfigの設定を追加して漏れを防ぐかの対策は必要かなと思います。(本来はHumanPlayerの`playerName`のみで機能実現できる設計を検討すべき)

Configはいろんな設定を格納できる便利クラスであるが故に、本当に追加すべき設定であるか否かは慎重に検討するべきでしょう。

# 基底クラスの危険性
CPUクラス群の基底クラスであるCpuPlayerですが、`updateRangeByCpu`が以下のような使われ方をしています。

```java
if (cpu instanceof CpuSmart smartCpu) {
    smartCpu.updateRangeByCpu(cpuGuess, secretNumber);
}
```

本来であればCpuPlayerを継承しているクラスは全て`updateRangeByCpu`の振る舞いを持っているということが保証されるべきです。つまり、ある子クラスなら使う、ある子クラスなら使わないみたいに利用条件が分岐するメソッドなら親クラスに定義すべきでないというのがセオリーです。

パッと確認したところ、CpuEasyの`updateRangeByCpu`を呼び出している箇所は見つかりませんでした。（あったらごめんなさい）

共通の処理を利用するのに継承は便利ですが、全ての子クラスに共通する振る舞いでないなら継承は利用すべきではありません。

一部の子クラスで利用したい処理は継承ではなくコンポジットを使うと良いです。

👇 修正案
```java
// 範囲を更新するクラス
public class CpuRangeUpdater {

    private CpuPlayer cpu;
    private int guess;
    private int secretNumber;
    
    public CpuRangeUpdater(CpuPlayer cpu, int guess, int secretNumber) {
        this.cpu = cpu;
        this.guess = guess;
        this.secretNumber = secretNumber;
    }

    // 中身一緒なのでByPlayerとByCpuは統合
    public void execute() {
        if (this.guess < this.secretNumber) {
            this.cpu.setMin(
                Math.max(this.cpu.getMin(), this.guess + 1));
        } else {
            this.cpu.setMax(
                Math.min(this.cpu.getMax(), this.guess - 1));
        }
    }

}
```
👇 呼び出し例
```java
if (cpu instanceof CpuSmart smartCpu) {
    var updater = new CpuRangeUpdater(smartCpu,cpuGuess, secretNumber);
    updater.execute();
}
```
安易に継承を使うことによる危険性についてはいろんな文献があるので調べてみるといいと思います。

軽く伝えておくと、<br/>
- 処理があっちこっち行き来するいわゆるスパゲッティコードになりがち（単純に読みにくくなる）
- 子クラス間の結合度が高くなってコード修正がしにくくなる
- 一部で使うコードが親クラスに集結して役割の多すぎるGodクラスが生まれてしまう<br/>

私が愛読していた『達人プログラマー』という本には、「継承が答えになることはほぼない」とまで記載されていました。

もちろん絶対に使うなということではありません。継承を選択する際はご慎重に。ということですね。
