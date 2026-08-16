# ADR-001: Judgment は層ではなく、Gate で発生するイベントである

- Status: Accepted
- Date: 2026-08-17
- 根拠: [Discussion #27](https://github.com/shuji-bonji/Situational-Awareness-and-Decision-Making/discussions/27) の未決着論点、[HEXIS Discussion #2](https://github.com/shuji-bonji/HEXIS/discussions/2)、[03-judgment.md](../03-judgment.md)

## Context

Discussion #27 では、Judgment を Canon → Structure の下流に置く単一の層として描く案があった。層として固定すると、次の3つが記述できなくなる。

1. **判断は古びる。** Gate 1 で「read-only だから安全」は、その文脈では正しい。同じ結果を Gate 2（egress）に持ち出すと漏洩になる。1箇所に固定したモデルでは、この2つを別の判断として扱えない。
2. **判断には文脈がある。** 「このファイルを削除してよい」は命題として完結していない。文脈が落ちた状態（`isPermitted: true`）は、真理値を失う。
3. **判断は消費される。** 「1回削除してよい」を2回の削除に使ってはならない。状態として持つ限り、単回性は表現できない。

既存のプロセス枠組み（OODA の Decide、See-Judge-Act の Judge）は、決定が行われることを記述するが、その決定がどの境界で、どの文脈で、何回有効かを問わない。

## Decision

Judgment を構造上の場所（層）として置かない。Structure が配置する **Gate** で、権威主体が発行する **イベント** とする。

イベントである以上、次を必須とする。

- 発行した Gate（持ち越し禁止。[I3](../04-invariants.md#i3)）
- 発行時文脈のハッシュと有効期限（鮮度。[I2](../04-invariants.md#i2)）
- 単回消費（[I4](../04-invariants.md#i4)）
- Canon の Invariant への参照（[I9](../04-invariants.md#i9)）

Gate 自身は判断を下さない。Proposal の受理、文脈の計算、既存 Judgment の検証、権威主体への要求、判断が得られないときの Silence への遷移を行う。

図に Judgment を単一の箱として描かない。これは簡略化ではなく、設計判断である。

## Consequences

### 得るもの

- judgment leakage を失敗として記述できる。上流 Gate の Permit を下流が継承する構成は、[I3](../04-invariants.md#i3) 違反として検査できる。
- 同一 Gate への再到達と、別 Gate への持ち越しを、I2 と I3 で分けて検査できる。
- replay と、部分リトライによる多重実行を、消費フラグで封じられる。

### 失うもの / 負うもの

- 図が単純な6層スタックにならない。
- 「Judgment 層の担当者」を組織図に書けない。Gate ごとに権威主体を定義する負担が生じる。この負担は HEXIS が作ったものではなく、元からあった曖昧さの可視化である。
- 実行時イベントそのものは ADR の対象にならない。ADR が記録するのは「Judgment をどう表現・検証するか」という設計である。
