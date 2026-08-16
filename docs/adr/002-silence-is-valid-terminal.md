# ADR-002: Silence は正当な終了状態である。Judgment の一種ではない

- Status: Accepted
- Date: 2026-08-17
- 根拠: [03-judgment.md §3.5](../03-judgment.md#35-silence)、[I8](../04-invariants.md#i8)、[HEXIS Discussion #2](https://github.com/shuji-bonji/HEXIS/discussions/2)

## Context

現行のエージェント設計の多くは、「応答を返さないことは失敗である」を暗黙の前提にする。この前提は judgment leakage を生産する。

判断が得られない状況で「何も返さない」ことが許されないなら、システムは判断を得られたことにするしかない。

- 権限が確認できない → 「たぶん大丈夫」で通す
- 情報が不足している → 補完して答える
- エスカレーション先が応答しない → タイムアウトして続行

失敗として集計される Gate は SLA を悪化させる。悪化した指標は改善対象になり、最も簡単な改善は Gate を緩めることである。fail-closed は運用上ペナルティを受け、やがて外される。

Silence を `decision: 'Silence'` として Judgment に含める案もある。その場合、`authority` が不在のときに「誰が発行したのか」が答えられなくなる。判断が存在しないことを、判断の一種として記録することになる。

## Decision

`Silence` を、成功でも失敗でもない **第3の正当な終了状態** とする。

- `Terminal` の値である。`Judgment.decision` の値ではない。
- `decision` は `Permit` / `Deny` の二値に限る。
- Silence は `judgment` フィールドを持たない。判断が存在しないから Silence なのである。
- 「たぶん許可」は返してはならない。確信が持てないなら Silence である。

Silence になるのは、権威主体が応答しない、権威主体が特定できない、判断に必要な情報が不足している、Canon の Invariant が相互に矛盾している、のいずれかである。

Silence は「何もしなかった」ではない。「判断が得られなかったので実行しなかった」という事実であり、記録される。[I8](../04-invariants.md#i8) により、`Failed` と同じカウンタに入れてはならない。

## Consequences

### 得るもの

- fail-closed（[I1](../04-invariants.md#i1)）を運用で維持できる。判断不能回数をシステムの失敗とは別の指標に載せられる。
- Silence 率は Structure の設計不足（権威主体の不在、Canon の穴）を示す。Gate を緩める根拠にならない。
- `AuthorityUnavailable` / `AuthorityUndefined` のとき、「誰が Silence を発行したのか」を問わずに済む。

### 失うもの / 負うもの

- 終了状態が3値になり、監視・SLA の定義を書き換える必要がある。既存の成功/失敗二値の運用にそのまま載らない。
- Silence 率が 0 であることは、完璧さの証拠ではない。Gate が機能していない疑いとして検査対象になる（断定ではない）。構造的に Silence が発生しない設計なら、その旨を Structure に書く。
- 初期適用では Silence が多発する。これは仕様どおりであり、失敗として扱ってはならない。
