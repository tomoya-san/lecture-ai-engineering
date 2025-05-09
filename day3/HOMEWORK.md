# RAG による質問回答の実装と検討

## 質問文

| 質問 ID | 質問文                                       |
| ------- | -------------------------------------------- |
| A1      | トランプ氏のゴールドカード制度とは？         |
| B1      | バイブコーディングとは？                     |
| C1      | オゼンピックにはどのような作用がありますか？ |
| A2      | アベノミクスとは？                           |
| B2      | LLM とは？                                   |
| C2      | カフェインにはどのような作用がありますか？   |

## 質問の意図

RAG は、LLM が知識として持っていない情報を、外部の知識ベースから取得して回答する手法です。そこで、gemma-2-2b-jpn-it の学習データには含まれていないであろう最新の情報を問う質問を 3 つ（A1~C1）用意し、それに対しておそらく回答するための知識を獲得しているであろう内容を問う質問を 3 つ（A2~C2）用意しました。また、A, B, C はそれぞれ異なるトピックを表しています。（A: 政治、　 B: ソフトウェア、 C: 化学・薬学）これにより、比較的対照的な質問に対して、RAG がどのように回答するかを観察することができます。

## RAG の実装方法

RAG の実装は、以下の手順で行います。

### 参照データの準備

参照するデータは以下のウェブサイトから取得します。
| 質問 ID | URL |
| ------- | -------------------------------------------- |
| A1 | https://www3.nhk.or.jp/news/html/20250226/k10014733601000.html |
| B1 | https://www.technologyreview.jp/s/359884/what-is-vibe-coding-exactly/ |
| C1 | https://www.wwdjapan.com/articles/1817522 |
| A2 | https://www.bbc.com/japanese/features-and-analysis-62103859 |
| B2 | https://www.hitachi-solutions-create.co.jp/column/technology/llm.html |
| C2 | https://www.ncnp.go.jp/hospital/guide/sleep-column14.html |

### ドキュメントの作成

それぞれの生のテキストデータは句点で区切り、1 文を 1 ドキュメントとします。合計で、183 文のドキュメントを作成しました。これらのドキュメントは、RAG のリトリーバルモデルに渡されます。

### 回答の生成

候補文書の取得は、infly/inf-retriever-v1-1.5b と言うモデルを使用します。これは、Hugging Face 上に公開されている事前学習済みのリトリーバルモデルです。質問と最も関連度の高い 5 つのドキュメントを取得し、以下の形式でプロンプトを作成します。

> [参考資料]  
> ここに５つの文が挿入されます。
>
> [質問]  
> ここに質問文が挿入されます。

このプロンプトが、gemma-2-2b-jpn-it に渡され、回答が生成されます。

## 評価指標

ここでは LLM as a judge を用いて、RAG を用いないテキスト生成と RAG を利用したテキスト生成の性能を評価します。具体的には、以下の 3 つの観点から評価を行います。

1. **正確性**: 回答が正確であるかどうかを評価します。
2. **質問関連性**: 回答が質問に対して関連性があるかどうかを評価します。
3. **コンテキスト関連性**: 回答が RAG により取得されたコンテキストとなるドキュメントに対して関連性があるかどうかを評価します。

全ての指標は 0~1 の間で表され、1 に近いほど良い評価となります。

## 生成結果

生成結果は /results/{質問 ID}.md に保存されています。

## 評価結果

| 質問 ID | RAG の有無 | 正確性 | 質問関連性 | コンテキスト関連性 |
| ------- | ---------- | ------ | ---------- | ------------------ |
| A1      | ×          | 0.44   | 0.95       |                    |
| A1      | ◯          | 0.28   | 0.57       | 1.0                |
| B1      | ×          | 0.54   | 1.0        |                    |
| B1      | ◯          | 0.42   | 1.0        | 1.0                |
| C1      | ×          | 0.22   | 0.67       |                    |
| C1      | ◯          | 0.81   | 1.0        | 1.0                |
| A2      | ×          | 0.53   | 1.0        |                    |
| A2      | ◯          | 0.70   | 1.0        | 0.86               |
| B2      | ×          | 0.53   | 0.95       |                    |
| B2      | ◯          | 0.63   | 1.0        | 1.0                |
| C2      | ×          | 0.55   | 1.0        |                    |
| C2      | ◯          | 0.81   | 1.0        | 1.0                |

## 考察

### 正確性

RAG を用いた方が正確性が高い場合と低い場合がある。RAG を用いた場合、正確性は向上することが多いが、必ずしもそうではない。特に、A1 と B1 では RAG を用いた場合の正確性が低下している。これは、回答そのものの正確性が低いと言うよりは、評価に使用した GPT-4o が正確な回答を生成できなかったためと考えられる。実際に、GPT-4o に A1, B1, C1 の質問を投げたところ、A1 と B1 はハルシネーションを起こしており、正確な回答を生成できていなかった。C1 は正確な回答を生成できていたため、その評価の信頼性も高い。また、RAG の有無で大きく違ったこととしては、回答の完全性が挙げられる。C1 以外の質問に対する RAG を用いない回答は、回答が不完全であることが多かった。RAG を用いることで、必要な事実が限定され完結かつ完全に回答することができていた。

### 質問関連性

特に質問関連性が低かったものに着目する。A1 は RAG を用いない場合、質問関連性が 0.95 と高かったが、RAG を用いた場合は 0.57 と低下している。これは、正確性と同様に評価に使用した GPT-4o が正確な回答を生成できなかったためと考えられる。また、C1 は RAG を用いなかった場合、質問関連性が 0.67 と低かったが、RAG を用いた場合は 1.0 と高くなっている。RAG を用いる前は、質問が要求している「オゼンピックの作用」
の他に服用時の注意点や副作用などの情報が含まれていたため、質問関連性が低くなっていたと考えられる。RAG を用いることで、質問に対して必要な情報のみを抽出し、回答することができたため、質問関連性が高くなったと考えられる。

### コンテキスト関連性

A2 を除いた全ての質問においてコンテキスト関連性が高くなっている。RAG で取得したドキュメントの中には、「女性の活躍」の公約を果たせなかったと言う内容が含まれていたのにも関わらず、回答ではそれを果たしたという文言が生成されたため、そこで一貫性が崩れてしまい減点されたと考えられる。

## まとめ

RAG を用いることで、質問に対して必要な情報を抽出し、回答することができるため、正確性や関連性が向上することが多い。しかし、RAG を用いた場合でも、必ずしも正確な回答が生成されるわけではないため、注意が必要である。
特に、RAG で取得するドキュメントのソースの質や信頼性が低い場合、RAG を用いても正確な回答が生成されない可能性がある。また、盲点になりがちなのは、タイムリーな情報を問う質問に対する回答を LLM で評価する際に、評価する LLM がタイムリーな情報を持っていない場合に正確に評価することができない点である。したがって、このような場合は人手で評価する必要性が出てくる。
