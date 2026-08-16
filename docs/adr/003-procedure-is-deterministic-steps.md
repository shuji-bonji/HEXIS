# ADR-003: Procedure は決定論的な実装手順である。実行前ゲートではない

- Status: Accepted
- Date: 2026-08-17
- 根拠: [Discussion #27](https://github.com/shuji-bonji/Situational-Awareness-and-Decision-Making/discussions/27)、[02-elements.md §Procedure](../02-elements.md#procedure)、[05-failure-modes.md §5.3](../05-failure-modes.md#53-畳めるか検証する)

## Context

Discussion #27 では Procedure に2つの用法が混在していた。

1. **実行前ゲート** — 検証 → 承認 → エスカレーション
2. **Action の実装手順** — 例: PDF を開く → Hash 生成 → 署名 → タイムスタンプ

両用法を同じ名前で置くと、失敗モードの分割基準に反する。

- 1 の失敗は権限の誤配分（Authority Misallocation）または Gate の欠落である。検出に必要な情報は権限配分と Gate 配置の全体像であり、Structure の管轄である。
- 2 の失敗は再現不能（Non-reproducibility）である。検出には複数回の実行の比較が要る。単発の実行観測からは原理的に判定できない。

「実行前ゲート」としての Procedure は、Action と同じ単位（1回の実行）で評価されるため Action と混ざる。「実装手順」としての Procedure は実行の集合に対して評価されるため、Action と混ざらない。

手順の途中で人間の確認が要る構成を Procedure の内部に残すと、Procedure が Judgment を発行することになり、提案と許可の分離が Procedure の内側で崩れる。

## Decision

Procedure を **Action を再現可能に実装するための、決定論的な手順の列** に統一する。

実行前の検証・承認・エスカレーションは **Gate** として Structure に属させる。

判定法: 「この手順の途中で人間に確認を取る必要があるか？」 Yes なら、そこは Procedure ではなく Gate である。Procedure を2つに割り、間に Gate を置く。

付随する禁止:

- Procedure の内部で Judgment を発行してはならない。
- 非決定論的な分岐を含んではならない（P5）。ReAct ループは Procedure ではない。
- Procedure が Canon を直接参照してはならない。参照が要るなら、そこは Gate である。
- ロールバックの**記述**は Procedure が持てる。ロールバックの**実行**は外界を変える Action であり、別の Judgment が要る。

P5 に対応する不変条件は置かない。不変条件は1回の実行トレースで破れを判定できる形式であり、複数実行の比較を要する性質はこの形式に載らない。検査は [チェックリスト P-2](../06-conformance.md#procedure) で行う。

## Consequences

### 得るもの

- Action と Procedure を畳めなくなる。1回の実行で見る失敗（Execution Divergence）と、実行の集合で見る失敗（Non-reproducibility）が分離する。
- 承認フローを Procedure と呼ぶ誤りを、分割基準で却下できる。
- pdf-trust の Skill 手順のように、列の中に Gate があり駆動が LLM であるものを Procedure と誤認しなくなる。それは Gate の連なりであり Structure に属する。

### 失うもの / 負うもの

- 「Procedure」が日常語としての手続き（承認手続きを含む）と食い違う。本仕様内では glossary の定義に従う。
- リトライやロールバックを Procedure に「書いておけば自動実行される」設計は取れない。実行のたびに Judgment が要る。負担を下げるなら Structure にリトライ Gate / ロールバック Gate を置き、Authority を決定論的規則エンジンにする。
