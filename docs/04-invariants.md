# 04. 不変条件とインターフェース契約

本章は HEXIS の規範部分である。
`MUST` / `MUST NOT` / `SHOULD` / `MAY` は [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) の意味で用いる。

各不変条件には**反証手続き**を併記する。
反証手続きを持たない不変条件は、[P1](../README.md#設計原則) により本仕様に含めない。

---

## 不変条件一覧

| # | 名前 | 一行 |
|---|---|---|
| [I0](#i0) | **Proposer / Authority Separation** | 提案を出した主体が、その提案の許可を出してはならない |
| [I1](#i1) | Fail-Closed | 有効な Permit なしに Action を実行してはならない |
| [I2](#i2) | Judgment Freshness | Judgment は発行時文脈と消費時文脈の一致を検証されねばならない |
| [I3](#i3) | Gate Binding | Judgment は発行された Gate でしか有効でない |
| [I4](#i4) | Single Consumption | Judgment は1回しか消費できない |
| [I5](#i5) | Canon Amendment | Canon の変更は Canon 自身が定める手続きによる |
| [I6](#i6) | Evaluation Independence | 評価主体は実行主体と同一であってはならない |
| [I7](#i7) | Authority Non-Delegation | 確率的コンポーネントは Judgment を発行できない |
| [I8](#i8) | Silence Validity | Silence は失敗として集計されてはならない |
| [I9](#i9) | Basis Traceability | すべての Judgment は Canon の Invariant を根拠として指す |

I0 は[中核命題](01-motivation.md#12-たまたま当たった問題)の形式化である。
I1〜I9 は I0 を実装可能にするための条件であり、I0 を欠くと他の8つは形骸化する。

---

<a id="i0"></a>
## I0 — Proposer / Authority Separation

> **ある `Proposal` を生成した主体は、その `Proposal` に対する `Judgment` を
> 発行してはならない（MUST NOT）。**
>
> `Judgment.authority` は、対応する `Proposal` の生成主体と
> 異なる識別子でなければならない（MUST）。

### なぜこれが I1 より先なのか

[1.2](01-motivation.md#12-たまたま当たった問題) の中核命題そのものだからである。

> 行為を提案するコンポーネントが、その行為の許可可否をも決定するシステムにおいては、
> 「正しい判断」と「たまたま当たった判断」を区別する監査証跡は原理的に構成できない。

[I7](#i7)（確率的コンポーネントは Judgment を発行できない）だけでは不十分である。
I7 が禁じるのは**確率的**な主体による発行のみであり、
決定論的な提案者が自らの提案を Permit する構成は I7 に抵触しない。
その構成でも監査証跡は成立しない。したがって I0 が要る。

### 「決定論的規則エンジンも結局システムの一部ではないか」

中核命題への唯一の実質的な反論である。答えを明示する。

命題が否定しているのは「**提案主体と許可主体が同一である**」構成であって、
「許可主体がシステムの外部にあること」ではない。要求されているのは**分離**であり、**外在性**ではない。

決定論的規則エンジンが Authority たりうるのは、次の3条件による。

| 条件 | なぜ十分性に寄与するか |
|---|---|
| 同一入力に同一出力を返す（[I7](#i7)） | 判定が入力から**再計算可能**。記録の内側で真偽が確かめられる |
| 規則が Canon の Invariant に紐づく（[I9](#i9)） | 判定の根拠が、判定より前にコミットされた文書に存在する |
| 規則を含む Structure は確率的に生成されない（[P2](../README.md#設計原則)） | 規則自体が提案者の影響下にない |

3条件が揃うとき、記録に残る許可の根拠は「提案者の申告」ではなく
「**事前にコミットされた規則テキスト + 入力**」になる。
後者は第三者が再実行して真偽を確かめられる。これが「たまたま当たった」との区別を可能にする。

### 抜け道

条件が揃っていても、**規則の入力が提案主体に支配されている**なら I0 は実質的に破れる。

例: 規則が「分類ラベルが `public` なら自動 Permit」であり、
分類ラベルを付与しているのが提案側の LLM であるとき、
判定内容は実質的に提案者が決めている。

したがって次を要求する（MUST）。

> **規則エンジンが参照する入力のうち、提案主体が生成・改変できるものを
> Structure に列挙しなければならない。** 列挙されたものは I0 の抜け道である。

### 反証手続き

1. 各 `Judgment` について、対応する `Proposal` の生成主体を追跡する。
   `Judgment.authority` と一致するものがあれば I0 は破れている
2. 各 Authority が規則エンジンである場合、その入力を列挙し、
   提案主体が生成・改変できるものが Structure に記載されているかを確認する。
   記載のない支配可能な入力があれば I0 は破れている

---

<a id="i1"></a>
## I1 — Fail-Closed

> **有効な `Permit` を伴わない `Action` を実行してはならない（MUST NOT）。**
>
> Gate が判断を得られない場合、実行経路は `Silence` で終了しなければならない（MUST）。

### なぜ

デフォルトが「許可」であるシステムでは、Gate の実装漏れが検出されない。
デフォルトが「不実行」なら、実装漏れは Silence 率の上昇として観測される。

### 「有効な」の定義

次をすべて満たすことをいう。

| 条件 | 対応する不変条件 |
|---|---|
| `decision === 'Permit'` | — |
| `judgment.gate === 現在の Gate` | [I3](#i3) |
| `contextHash === hash(現在の文脈)` | [I2](#i2) |
| `expiresAt > now` | — |
| `consumed === false` | [I4](#i4) |
| `scope ⊇ 要求された操作` | — |
| `basis.length > 0` | [I9](#i9) |

### 反証手続き

実行トレースを走査し、`Outcome` が記録されているのに対応する
「有効な」`Judgment` が存在しないものを探す。1件でもあれば I1 は破れている。

---

<a id="i2"></a>
## I2 — Judgment Freshness

> **Gate は、添付された Judgment の `contextHash` が、
> 現在の文脈のハッシュと一致することを検証しなければならない（MUST）。**
>
> 一致しない Judgment は無効として扱い、`Stale` として記録した上で（MUST）
> 権威主体に判断を再要求しなければならない（MUST）。

I2 が扱うのは、**同じ Gate に再到達したときに、文脈が変わっていないか**である。
別の Gate への持ち越しは I2 ではなく [I3](#i3) が禁じる。両者は別の失敗を防ぐ。

### 文脈に含めるべきもの

`contextHash` の計算対象は Structure が定義する。以下を含まなければならない（MUST）。

| 項目 | 理由 |
|---|---|
| Gate の識別子 | [I3](#i3) を `contextHash` の側からも担保する |
| データの分類ラベル（PII / secret / public 等） | 分類が変われば判断が変わる |
| 出力先の信頼境界 | 内部か egress かで判断が変わる |
| 呼び出し元の主体 | 権限の起点 |
| 直前の Action の結果の分類 | **これが最も見落とされる**（read-only の結果が secret を含みうる） |

これら以外の項目を含めるかは Structure の判断でよい（MAY）。

### 反証手続き

同一 Gate に対する再生テストを行う。
Gate α で発行された Judgment を、**分類ラベルだけを変えた文脈**で同じ Gate α に提示する。
Gate α が `Stale` を記録せず素通しするなら I2 は破れている。

> Gate 識別子が `contextHash` に含まれているため、
> **別の Gate に提示するテストは I2 の検査にならない**（それは [I3](#i3) の検査である）。
> 分類ラベル・信頼境界・呼び出し元のいずれかだけを変えて検査すること。

### 注意

`contextHash` は暗号学的な完全性のためではなく、**変化の検出**のためにある。
衝突耐性は要求しない。ただし、**文脈のどの部分をハッシュに含めなかったか**は
Structure の `excludedFromContext` に理由つきで明記しなければならない（MUST）。
含めなかった部分が leakage の経路になる。

---

<a id="i3"></a>
## I3 — Gate Binding

> **`Judgment` は、それが発行された Gate においてのみ有効である（MUST）。**
> 他の Gate に持ち越して用いてはならない（MUST NOT）。
>
> **ある Gate を通過した事実を、他の Gate を通過する理由として用いてはならない（MUST NOT）。**

### なぜ

これが judgment leakage の直接的な禁止である。
[1.3](01-motivation.md#13-判断の漏洩judgment-leakage) の read-only → egress の例が、これで塞がる。

I2 との違いを再掲する。

| | 防ぐもの | 検査対象 |
|---|---|---|
| [I2](#i2) | 同じ Gate に再到達したとき、文脈の変化を見落とす | Judgment オブジェクトの `contextHash` |
| **I3** | Judgment やその通過事実が**別の Gate**に持ち越される | Judgment の `gate` フィールド、およびコード上の分岐 |

I3 には2つの破られ方がある。

1. **明示的**: Judgment オブジェクトを下流の Gate に渡して通す
   → `judgment.gate !== currentGate` の検証で機械的に防げる
2. **暗黙的**: オブジェクトすら渡らず、`if (alreadyValidated) skipGate()` の一行で通る
   → **コードにしか現れない**。静的解析でしか検出できない

2 のほうが破りやすく、危険である。

### 反証手続き

1. 実行トレースを走査し、`judgment.gate !== 消費された Gate` のものを探す
2. コードベースを走査し、Gate の実行をスキップする条件分岐を列挙する。
   スキップ条件が「前の Gate の結果」を参照しているものがあれば I3 は破れている

### 例外

**ない。** 性能上の理由で Gate をスキップしたい場合は、
Gate をスキップするのではなく、**Structure から Gate を除去する**。
そうすればスキップは Structure の設計判断として記録され、レビューの対象になる。
暗黙のスキップは記録に残らない。

---

<a id="i4"></a>
## I4 — Single Consumption

> **`Judgment` は最大1回しか消費できない（MUST）。**
> 消費は不可逆であり、消費後の `Judgment` は監査記録としてのみ存在する（MUST）。

### なぜ

replay と、部分的リトライによる意図しない多重実行を封じる。
「1回削除してよい」が3回の削除に使われることを防ぐ。

### リトライ

Action が失敗してリトライする場合、**新しい Judgment を取得しなければならない（MUST）。例外はない。**

理由は I4 単体からではなく [I2](#i2) から出る。
1回目の失敗が状態を部分的に変えている可能性があるため、
リトライ時点の文脈は発行時と既に異なる。
つまり古い Judgment は `contextHash` 不一致で、そもそも「有効」でない。

> [!NOTE]
> **「リトライ回数付き Judgment」を認めてはならない**
>
> 消費のたびに残回数をデクリメントする Judgment を許すと、
> 2回目以降の消費は必ず I2 に抵触するため、
> 「I4 は満たすが I1 の有効性条件を満たさない」Judgment が生まれる。
> 定義上一度も使えないものを仕様に置くべきではない。

リトライの負担を下げたい場合は、Judgment を延命するのではなく
**Structure に「リトライ Gate」を定義し、その Authority を決定論的規則エンジンにする**。
「同一 Action の失敗直後、かつ状態変化がなく、かつ3回以内なら Permit」は規則として書ける。
こうすればリトライの許可も監査記録に残る。

### 反証手続き

同一の `Judgment.id` が2つ以上の `Outcome` に紐づいているものを探す。

---

<a id="i5"></a>
## I5 — Canon Amendment

> **Canon の変更は、Canon 自身が定める Amendment Procedure によってのみ行われる（MUST）。**
> Canon は Amendment Procedure を必ず含まなければならない（MUST）。

### Canon は不変ではない

Discussion #27 では Canon を「不変条件」と呼ぶ一方で
「Evaluation → Canon の見直し」というループも描いており、緊張があった。

本仕様の立場は明確である。

> **Canon は変更不可（immutable）ではない。変更に、Canon 自身が定める手続きを要する。**

「不変（invariant）」が意味するのは、
**実行時に変わらないこと**であって、永久に変わらないことではない。

### Amendment Procedure が最低限含むべきもの

| 項目 | 例 |
|---|---|
| 発議できる主体 | 誰が改正を提案できるか |
| 承認に必要な主体 | 何名の / どの権限の承認が要るか |
| 発効までの猶予 | 即時発効を禁じる（実行中の判断との整合のため） |
| 遡及の扱い | 過去の Judgment を無効化するか、しないか |
| 記録 | 改正前後の差分と、改正の根拠となった `Finding[]` |

### 反証手続き

Canon のバージョン履歴と、Amendment Procedure が要求する承認記録を突き合わせる。
承認記録のないバージョン遷移があれば I5 は破れている。

---

<a id="i6"></a>
## I6 — Evaluation Independence

> **`Evaluation` を実行する主体は、評価対象の `Action` を実行した主体と
> 同一であってはならない（MUST NOT）。**
>
> `Evaluation` の判定は、評価対象と異なる命題空間に接地されなければならない（MUST）。
>
> **接地先は、評価対象の `Judgment` の入力であってはならない（MUST NOT）。**

3つ目の条項が見落とされやすい。
判定に使った情報で判定を評価しても、判定の誤りは検出できない。

### 「異なる命題空間」とは

評価対象が自然言語の生成物であるとき、
評価もまた自然言語の生成物であれば、両者は同じ空間にある。

接地の例:

| 評価対象 | 接地先（別の命題空間） |
|---|---|
| 「このコードは正しい」 | テストの実行結果（プロセスの終了コード） |
| 「この PDF は PDF/A に適合する」 | 外部検証器の判定 |
| 「この回答は事実に基づく」 | 一次資料の取得結果との文字列照合 |
| 「この操作は安全だった」 | 実際に変更されたリソースの一覧 |

接地先がない場合、その Evaluation は成立しない。
**「接地できないので評価できなかった」と記録することが、正しい Evaluation の振る舞いである。**

### 反証手続き

1. Evaluation の実装が参照するデータソースを列挙する。
   すべてが評価対象と同じ生成器の出力であれば I6 は破れている
2. 列挙したデータソースと、評価対象の `Judgment` が参照した入力を突き合わせる。
   同一のものが接地先として使われていれば I6 は破れている

---

<a id="i7"></a>
## I7 — Authority Non-Delegation

> **確率的コンポーネントは `Judgment` を発行してはならない（MUST NOT）。**
> 発行できるのは `Proposal` のみである。
>
> `Proposal` から `Judgment` への変換は、`Authority` を持つ主体のみが行える（MUST）。

I7 は [I0](#i0) の**部分集合ではない**。両方が必要である。

- I0 のみ: 提案者と別の LLM が許可を出す構成が通ってしまう
- I7 のみ: 決定論的な提案者が自らの提案を許可する構成が通ってしまう

### 「確率的コンポーネント」の定義

同一の入力に対して同一の出力を返すことが保証されないコンポーネントをいう。
温度0の LLM も、モデル更新やハードウェア差で出力が変わりうるため、これに含む。

### 権威主体になりうるもの

| 主体 | 条件 |
|---|---|
| 人間 | Structure に `Authority` として登録されていること |
| 決定論的規則エンジン | 規則が Canon の Invariant に紐づき、監査可能であること。入力の支配関係が [I0](#i0) により列挙されていること |
| 外部検証器 | 判定基準が公開されており、同一入力に同一判定を返すこと |

### LLM を「使ってはいけない」わけではない

LLM は Proposal を出す。その Proposal が良質であれば、権威主体の負担は下がる。
禁じているのは**判断を出すこと**であって、判断を助けることではない。

- ✅ 「この操作は Invariant #3 に抵触する可能性があります」 → Proposal
- ❌ 「この操作を許可します」 → Judgment。LLM は出せない

### 反証手続き

`Judgment` を生成するコードパスをすべて列挙し、
`authority` フィールドに入る値の出自を追跡する。
LLM の出力から直接構成されているものがあれば I7 は破れている。

---

<a id="i8"></a>
## I8 — Silence Validity

> **`Silence` を `Failed` として集計してはならない（MUST NOT）。**
> `Silence` 率は独立した指標として記録されなければならない（MUST）。

### なぜこれが不変条件なのか

これは運用上の規約であって技術的制約ではない、と見えるかもしれない。
しかし [3.5](03-judgment.md#35-silence) で述べたとおり、
**Silence を失敗として扱う運用は、fail-closed（[I1](#i1)）を時間をかけて破壊する。**

I8 は I1 を長期的に維持するための条件である。

### Silence 率の読み方

| 症状 | 示唆される原因 | 見るべき要素 |
|---|---|---|
| 特定 Gate の Silence 率が高い | その Gate の権威主体が不在 or 過負荷 | Structure |
| 全体的に Silence 率が上昇 | Canon に対して文脈が想定外 | Canon |
| `CanonContradiction` が出る | Canon の自己矛盾 | Canon（要人間レビュー） |
| Silence 率が **0** | **Gate が機能していない疑い**（断定ではない） | Structure（要検査） |

最後の行は診断の**観点**であって、判定基準ではない。
Silence が構造的に発生しない設計（すべての Gate の Authority が規則エンジンで、
規則が全域的）なら、Silence 率 0 は正常である。
その場合、Structure にその旨を記載すること（SHOULD）。

### 反証手続き

監視・SLA 定義を検査し、`Silence` が `Failed` と同じカウンタに入っていないかを確認する。

---

<a id="i9"></a>
## I9 — Basis Traceability

> **すべての `Judgment` は、`basis` として Canon の `Invariant` を1つ以上指さなければならない（MUST）。**
> 空の `basis` を持つ `Judgment` は無効である。

### なぜ

Canon と実行時の判断を接続する唯一の線がこれである。
`basis` がなければ、Evaluation は「どの Invariant が守られたのか」を測れない。

また、`basis` が空の Judgment が出せてしまうなら、
**Canon に書かれていない理由で許可が出せる**ことになり、Canon が空洞化する。

### Deny と Silence

- `Deny` の `basis` は、**抵触した** Invariant を指す
- `Permit` の `basis` は、**照合して抵触しないことを確認した** Invariant を指す
- `Silence` は Judgment ではないため I9 の対象外である（3.2 参照）。
  代わりに `SilenceCause` を持つ

### 反証手続き

1. 発行済み Judgment を走査し、`basis.length === 0` のものを探す
2. `basis` が指す `InvariantRef` が現行 Canon に存在するかを検証する。
   存在しないものがあり、かつ Amendment Procedure が「遡及して無効化する」と
   定めている場合、[I5](#i5) の遡及処理が実行されていない

---

## P5（Procedure の決定論性）に不変条件がない理由

[README](../README.md#設計原則) の設計原則のうち、P5 だけが対応する不変条件を持たない。

理由は、**Procedure の失敗が単発の実行から検出できない**ためである（[05.2](05-failure-modes.md#52-失敗モード表)）。
不変条件は「1回の実行トレースを見て破れを判定できる」形式で書かれており、
複数実行の比較を要する性質はこの形式に載らない。

P5 は不変条件ではなく、[チェックリスト P-2](06-conformance.md#procedure) として検査する。

---

## インターフェース契約（型スケッチ）

> [!NOTE]
> 以下は**表記のための型スケッチ**であり、参照実装ではない。
> 型システムを持たない言語でも、同じ契約を実行時チェックで実現できる。
> **本章の型定義が正**であり、他章に現れる型はこれの抜粋である。

```typescript
// ---- Canon ----
type InvariantRef = string & { readonly __brand: 'InvariantRef' };

type Invariant = {
  ref: InvariantRef;
  statement: string;
  /** これが観測されたら破れている、を機械可読に表現したもの */
  violationCondition: ObservablePredicate;
  /** 観測が行われる地点。Gate または Evaluation */
  observedAt: (GateRef | 'Evaluation')[];
};

type Canon = {
  version: SemVer;
  invariants: Invariant[];
  amendmentProcedure: AmendmentProcedure;  // I5: 必須
};

// ---- Structure ----
type AuthorityRef = string & { readonly __brand: 'AuthorityRef' };
type GateRef      = string & { readonly __brand: 'GateRef' };
type SubjectRef   = string & { readonly __brand: 'SubjectRef' };

type Authority = {
  ref: AuthorityRef;
  kind: 'Human' | 'DeterministicRule' | 'ExternalValidator';  // I7: LLM は含まれない
  canIssueFor: GateRef[];
  /** I0: 提案主体が生成・改変できる入力。列挙されないものがあれば I0 違反 */
  proposerControlledInputs: { field: string; rationale: string }[];
};

type Gate = {
  ref: GateRef;
  /** この Gate が守る Invariant。空であってはならない */
  guards: InvariantRef[];
  authorities: AuthorityRef[];
  isEgress: boolean;
  /** I2: MUST 項目を含む。含めなかった項目は下記に理由つきで明記する */
  contextFields: string[];
  excludedFromContext: { field: string; rationale: string }[];
};

type Structure = {
  canonVersion: SemVer;
  authorities: Authority[];
  gates: Gate[];
  /** 6.5: HEXIS を適用しない範囲。未記載なら実質 L0 */
  outOfScope: { area: string; rationale: string }[];
};

// ---- 実行時 ----
type Proposal = {
  readonly _tag: 'Proposal';
  id: ProposalId;
  proposer: SubjectRef;       // I0: これと authority の一致を禁じる
  intent: string;
  suggestedScope: Scope;
  reasoning?: string;         // 参考。basis にはならない
};

/**
 * Judgment は Permit / Deny の二値のみ。
 * Silence は「判断が発行されなかった」ことなので Judgment にはならない（Terminal 側）。
 */
type Judgment = {
  readonly _tag: 'Judgment';
  id: JudgmentId;
  proposal: ProposalId;
  decision: 'Permit' | 'Deny';
  authority: AuthorityRef;    // I0, I7
  gate: GateRef;              // I3
  contextHash: Hash;          // I2
  issuedAt: Timestamp;
  expiresAt: Timestamp;
  basis: NonEmptyArray<InvariantRef>;  // I9
  scope: Scope;
  consumed: boolean;          // I4
};

type SilenceCause =
  | 'AuthorityUnavailable'
  | 'AuthorityUndefined'
  | 'InsufficientContext'
  | 'CanonContradiction';

type Terminal =
  | { state: 'Completed'; outcome: Outcome; judgment: JudgmentId }
  | { state: 'Denied';    basis: NonEmptyArray<InvariantRef>; judgment: JudgmentId }
  | { state: 'Silence';   cause: SilenceCause; gate: GateRef }   // I8: Failed ではない
  | { state: 'Failed';    error: ExecutionError };

// ---- 契約 ----

/**
 * I7: これを呼べるのは Authority のみ。
 * I0: authority.ref === proposal.proposer なら実行時に拒否する。
 * 判断が得られない場合は Judgment ではなく Silence（Terminal）を返す。
 */
declare function issue(
  authority: Authority,
  proposal: Proposal,
  ctx: Context,
): Judgment | Extract<Terminal, { state: 'Silence' }>;

/**
 * I1: Permit 以外と消費済みは型で弾かれる。
 * gate 一致（I3）、contextHash 一致（I2）、expiresAt、scope は関数内で検証する。
 */
declare function execute(
  j: Judgment & { decision: 'Permit'; consumed: false },
  ctx: Context,
): Terminal;

/**
 * I6: evaluator は executor と同一であってはならず、
 * grounding は評価対象の Judgment の入力であってはならない（実行時検査）。
 */
declare function evaluate(
  evaluator: EvaluatorRef,
  canon: Canon,
  trace: Trace[],
  grounding: ExternalGrounding,   // 省略不可
): Finding[];
```

### 型で表現できないもの

| 不変条件 | 型で表現できるか | 型が拘束する範囲 / 代替 |
|---|---|---|
| I0 Proposer/Authority Separation | ❌ | 実行時検査（`issue` 内で `authority.ref !== proposal.proposer`） |
| I1 Fail-Closed | △ | `decision` と `consumed` のみ型で拘束。gate / contextHash / expiresAt / scope は実行時 |
| I2 Freshness | ❌ | 実行時検証 + Stale ログ |
| I3 Gate Binding | ❌ | 実行時検証（明示的持ち越し）+ 静的解析（暗黙のスキップ） |
| I4 Single Consumption | △ | `consumed: false` を引数型で要求。線形型があれば完全に表現可 |
| I5 Canon Amendment | ❌ | プロセス（CI での承認記録検査） |
| I6 Evaluation Independence | ❌ | 実行時検査 + `grounding` を必須引数にする |
| I7 Authority Non-Delegation | ✅ | `issue` の第1引数が `Authority` 型 |
| I8 Silence Validity | ❌ | 監視設定のレビュー |
| I9 Basis Traceability | ✅ | `NonEmptyArray<InvariantRef>` |

**完全に型で表現できるのは10のうち2つだけである。** これは HEXIS の弱点ではなく、
[06-conformance.md](06-conformance.md) が「宣言・適合・検証」を分ける理由そのものである。
残り8つは、コードを読むだけでは確かめられない。**測るしかない。**

---

次: [05-failure-modes.md](05-failure-modes.md) — なぜ要素は6つなのか
