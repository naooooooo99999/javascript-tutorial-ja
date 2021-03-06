importance: 5

---

# 拡張可能な計算機を作る

"拡張可能な" 計算機オブジェクトを作るコンストラクタ関数 `Calculator` を作りなさい。

タスクは2つのパートから構成されます。

1. 最初に フォーマット "数値 演算子 数値" (スペース区切り) で、`"1 + 2"` のような文字列を取り、結果を返すメソッド `calculate(str)` メソッドを実装します。それはプラス `+` と マイナス `-` を理解できるようにしてください。

    使い方の例:

    ```js
    let calc = new Calculator;

    alert( calc.calculate("3 + 7") ); // 10
    ```
2. 次に、計算機に新しい操作を教えるメソッド `addOperator(name, func)` を追加します。操作 `name` と、それを実装する2つの引数を持つ関数 `func(a,b)` を取ります。

    例えば、乗算 `*`, 除算 `/` やべき乗 `**`:

    ```js
    let powerCalc = new Calculator;
    powerCalc.addMethod("*", (a, b) => a * b);
    powerCalc.addMethod("/", (a, b) => a / b);
    powerCalc.addMethod("**", (a, b) => a ** b);

    let result = powerCalc.calculate("2 ** 3");
    alert( result ); // 8
    ```

- このタスクではかっこや複雑な表現は不要です。
- 数字と演算子は、正確に1つのスペースで区切られます。
- 追加したい場合にエラー処理があるかもしれません。
