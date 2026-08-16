# Architecture Decision Records

HEXIS 自身の設計決定を記録する。適用先システムの Structure / Canon の判断を書く場所ではない。

形式は Context / Decision / Consequences。Status は `Accepted` / `Superseded` / `Deprecated`。

| ADR | 決定 | Status |
| --- | --- | --- |
| [001](001-judgment-is-event-not-layer.md) | Judgment は層ではなく、Gate で発生するイベントである | Accepted |
| [002](002-silence-is-valid-terminal.md) | Silence は正当な終了状態である。Judgment の一種ではない | Accepted |
| [003](003-procedure-is-deterministic-steps.md) | Procedure は決定論的な実装手順である。実行前ゲートではない | Accepted |

新規の設計決定は、仕様本文を改訂する前に ADR を追加する。
