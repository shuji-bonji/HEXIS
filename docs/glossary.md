# 用語集

日本語訳は本仕様内での訳語を固定するためのものである。
一般的な訳語と異なる場合があるが、本仕様内では以下に統一する。

## 6要素

| 英             | 日   | 定義                                                            | 参照                            |
| -------------- | ---- | --------------------------------------------------------------- | ------------------------------- |
| **Canon**      | 規範 | 破れたことを検出できる形で書かれた不変条件の集合                | [02](02-elements.md#canon)      |
| **Structure**  | 構造 | 権限の配分と Gate の配置                                        | [02](02-elements.md#structure)  |
| **Judgment**   | 判断 | 特定 Gate で権威主体が発行する、有効期限付き単回消費の許可/拒否 | [03](03-judgment.md)            |
| **Action**     | 行動 | 有効な Judgment を消費して行う、外界の状態を変える操作          | [02](02-elements.md#action)     |
| **Procedure**  | 手順 | Action を再現可能に実装するための決定論的な手順の列             | [02](02-elements.md#procedure)  |
| **Evaluation** | 評価 | 実行主体と異なる主体が、別の命題空間から Canon の破れを測る活動 | [02](02-elements.md#evaluation) |

## 中核概念

| 英              | 日           | 定義                                                                                       |
| --------------- | ------------ | ------------------------------------------------------------------------------------------ |
| **Gate**        | ゲート       | Structure が配置する、Judgment が発生する地点。Gate 自身は判断を下さず、判断を**要求**する |
| **Proposal**    | 提案         | 確率的コンポーネントが出せる唯一のもの。許可ではない                                       |
| **Authority**   | 権威主体     | Judgment を発行する権限を持つ主体。人間・決定論的規則エンジン・外部検証器のいずれか        |
| **Silence**     | 沈黙         | 判断が得られなかったために実行しなかったという終了状態。成功でも失敗でもない               |
| **Invariant**   | 不変条件     | Canon を構成する個々の規範。`violationCondition` を持つ                                    |
| **contextHash** | 文脈ハッシュ | Judgment 発行時の文脈を表す値。消費時の文脈と照合される                                    |
| **basis**       | 根拠         | Judgment が指す `InvariantRef` の集合。Canon と実行時判断を接続する唯一の線                |
| **egress 境界** | 送出境界     | 情報がシステムの信頼境界の外に出る地点。出力・外部API・ログ・通知・ファイル書き出しを含む  |

## 失敗モード

| 英                          | 日               | どの要素の失敗か                               |
| --------------------------- | ---------------- | ---------------------------------------------- |
| **Canon Contradiction**     | 規範の自己矛盾   | Canon                                          |
| **Authority Misallocation** | 権限の誤配分     | Structure                                      |
| **Gate Omission**           | Gate の欠落      | Structure（2つ目の固有失敗）                   |
| **Judgment Staleness**      | 判断の鮮度切れ   | Judgment                                       |
| **Judgment Leakage**        | 判断の漏洩       | Judgment（Staleness が egress で顕現したもの） |
| **Execution Divergence**    | 実行と意図の乖離 | Action                                         |
| **Non-reproducibility**     | 再現不能         | Procedure                                      |
| **Detection Failure**       | 検出失敗         | Evaluation                                     |
| **Semantic Laundering**     | 意味の洗浄       | Evaluation（自己評価による循環的検証）         |

## 適合に関する語

これらは厳密に使い分ける。混同すると仕様が意味をなさない。

| 英              | 日   | 意味                             | 証明力           |
| --------------- | ---- | -------------------------------- | ---------------- |
| **declaration** | 宣言 | 自己申告                         | なし             |
| **conformance** | 適合 | 実際に満たしている状態           | 証明不能         |
| **validation**  | 検証 | 特定の検査で反証されなかった事実 | 検査の範囲内のみ |

→ [06-conformance.md](06-conformance.md#61-三つを区別する)

## 混同されやすい対

| A                                                 | B   | 違い                                                                                                              |
| ------------------------------------------------- | --- | ----------------------------------------------------------------------------------------------------------------- |
| **Proposal** / **Judgment**                       |     | Judgment は `authority` を持つ。Proposal は持たない                                                               |
| **Gate** / **Procedure**                          |     | Gate は判断を要求する。Procedure は判断を含まない。手順の途中で確認が要るなら、そこは Gate                        |
| **Silence** / **Failed**                          |     | Silence は判断が得られなかった。Failed は実行が失敗した                                                           |
| **Silence** / **Denied**                          |     | Denied は判断が下されて拒否された（Judgment が存在する）。Silence は判断が下されなかった（Judgment が存在しない） |
| **I2 Freshness** / **I3 Gate Binding**            |     | I2 は**同じ Gate**での文脈変化。I3 は**別の Gate**への持ち越し                                                    |
| **I0** / **I7**                                   |     | I0 は提案主体と許可主体の分離。I7 は確率的主体による発行の禁止。片方だけでは穴が残る                              |
| **basis** / **reasoning**                         |     | basis は権威主体が選ぶ InvariantRef。reasoning は LLM の自然言語。後者は根拠にならない                            |
| **Canon** / **Structure**                         |     | 破ったことを実行時に観測できるなら Canon。観測前に配置を決めるなら Structure                                      |
| **Action** / **Procedure**                        |     | Action は1回の実行で評価される。Procedure は実行の集合で評価される                                                |
| **invariant（不変）** / **immutable（変更不可）** |     | Canon は invariant であって immutable ではない。実行時に変わらないが、改正手続きで変わる                          |

## 本仕様が採らなかった語

| 語                    | 採らなかった理由                                                                  |
| --------------------- | --------------------------------------------------------------------------------- |
| **CSJAPE**            | 発音できない。口伝されない略語は普及しない                                        |
| **層 / Layer**        | Judgment は層ではないため、6つを一律に「層」と呼ばない。「要素」を用いる          |
| **human-in-the-loop** | 「ループの中にいる」ことは権限を持つことを意味しない。`Authority` を用いる        |
| **guardrail**         | 何を守るのかが語に含まれない。`Gate` + `guards: InvariantRef[]` を用いる          |
| **LLM-as-judge**      | judge の語が Judgment と衝突する。LLM は Proposal を出す主体であり judge ではない |
| **compliance**        | 法令遵守と紛らわしく、かつ「適合」の証明を含意する。`validation` を用いる         |
