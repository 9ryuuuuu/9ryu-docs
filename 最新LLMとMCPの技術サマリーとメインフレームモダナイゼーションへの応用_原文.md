ありがとうございます。最新のLLM（Gemini 2.5、gpto3-mini、gpto1、Claude Sonnet 2.7）について、性能・推論速度・API対応・コンテキスト長の観点で比較し、あわせてMCP（Model Context Protocol）の技術仕様と活用事例を調査します。

その上で、これらのLLMとMCPがメインフレームのモダナイゼーションにどう活用できるか、特に現場で困難とされるポイント（例：COBOL資産変換、業務ロジックのドキュメント化、テストデータ生成など）に対して有効なユースケースを提案します。

調査が完了次第、わかりやすく整理してご報告します。

# 最新LLMとMCPの技術サマリーとメインフレームモダナイゼーションへの応用

## 最新LLMの比較

最新の大規模言語モデル（LLM）として、**Google Gemini 2.5**、**OpenAI GPT o3-mini（以下、o3-mini）**、**OpenAI GPT o1（以下、o1）**、**Anthropic Claude 3.7 Sonnet（以下、Claude 3.7）**の特徴を比較します。それぞれ性能や推論速度、提供形態（API対応）、コンテキスト長に特徴があります。

### 性能（ベンチマーク）

- **Gemini 2.5**（特に2.5 Pro版）は、総合的な推論力で非常に高い性能を示しています。例えば、人間の専門知識を問う難問揃いのベンチマーク「Humanity’s Last Exam」では18.8%というスコアでトップとなり、o3-miniの14%やClaude 3.7の8.9%を大きく上回りました ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=,2))。数学・論理系の試験であるAIMEでも最高性能（2024年度版で92.0%、2025年度版で86.7%）を記録し、後述のo3-miniと僅差のトップです ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=These%20are%20benchmarks%20where%20Gemini%E2%80%99s,reasoning%20architecture%20seems%20to%20shine))。総合的に知識・推論・長文理解に優れ、**LMArena**のLLM評価でも1位を獲得しています ([Gemini 2.5: Our newest Gemini model with thinking](https://blog.google/technology/google-deepmind/gemini-model-thinking-updates-march-2025/#:~:text=Today%20we%E2%80%99re%20introducing%20Gemini%202,LMArena%20by%20a%20significant%20margin))。

- **OpenAI GPT o3-mini**は、OpenAIの「推論重視」モデル系列（oシリーズ）の最新軽量版です。難解な問題に対する多段推論力が高く、Geminiに次ぐ性能を発揮します。特に数学ではAIME 2025で86.5%とGeminiに肉薄し ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=,mini%20%2886.5))、コード生成では**LiveCodeBench**で74.1%とGemini（70.4%）を上回るスコアを出しています ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=On%20benchmarks%20that%20test%20code,performs%20well%20but%20doesn%E2%80%99t%20dominate))。一方、総合知識テストではGeminiには及ばないものの高得点を維持し ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=,2))、OpenAIの従来モデル（GPT-4など）より多くの分野で優位と報告されています ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=It%20generally%20benchmarks%20higher%20than,o1%20but%20not%20across%20everything))。特筆すべきは**競技プログラミング**分野で、Codeforcesの指標となるELOレーティングが高く（高推論設定で2130）、従来のGPT-4ベースモデル(約900)やo1を大きく凌駕しました ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=It%20generally%20benchmarks%20higher%20than,o1%20but%20not%20across%20everything))。

- **OpenAI GPT o1**は、2024年にプレビュー版が登場した**深い推論特化モデル**です。最新モデル群と直接のベンチマーク比較は少ないものの、難問解決能力に定評があります ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=As%20opposed%20to%20OpenAI%27s%20prior,solving%20strategies))。例えば数学試験のAIME 2024では標準o1で78%の合格率、強化版のo1-proモードでは86%に達したと報告されています ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=o1%20pro%20mode))。これは先述のGeminiやo3-miniと同程度の高水準です。一方、汎用知識や日常会話においてはGPT-4相当（GPT-4oと呼ばれるモデル）よりも思考重視の設計ゆえ、必ずしも全分野で上回るわけではありません ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=Confusing%20matters%20further%2C%20the%20benchmarks,o1%20but%20not%20across%20everything))。とはいえ、総合的な論理推論やコード理解では依然強力であり、特に**コーディングタスク**での正確性向上が意図されています ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=o1))。

- **Claude 3.7 Sonnet**（Anthropic社）は、Claude 2から進化した最新モデルです。バージョン番号以上に大幅な改善があり、**推論モード（Thinking Mode）**の導入でモデルが内部の思考過程を経て回答するアプローチを採用しています ([Claude 3.7 Sonnet: How it Works, Use Cases & More | DataCamp](https://www.datacamp.com/blog/claude-3-7-sonnet#:~:text=First%20off%2C%20Claude%203,which%20lets%20us%20see%20the))。その結果、ソフトウェアエンジニアリング分野では前版Claude 3.5に比べ精度が向上し、SWE-Bench（コードに関する難問集）では62.3%→**70.3%**へ大きくスコアが伸びています ([Claude 3.7 Sonnet: How it Works, Use Cases & More | DataCamp](https://www.datacamp.com/blog/claude-3-7-sonnet#:~:text=Claude%203,5%20Sonnet)) ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=SWE))。この70.3%はGemini 2.5 Proの63.8%を上回り、同ベンチマークではClaude 3.7が最優でした ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=,3))。一方で、知識推論系では先述の通りGeminiやo3-miniほど高得点ではなく、Humanity’s Last Examで8.9%に留 ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=,2))。総じて、Claude 3.7は**コード生成・エージェント的タスク**に強みを発揮しつつ、全体としても前世代モデルを着実に凌駕する性能を示しています。

### 推論速度

- **思考プロセスと速度のトレードオフ:** これら最新LLMの多くは高精度のために内部で「思考チェーン」を実行するため、従来モデルより応答時間が長くなりがちです。特にOpenAIのo1シリーズは「回答を急がずじっくり考える」方針であり、複雑な質問では従来モ ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=As%20opposed%20to%20OpenAI%27s%20prior,solving%20strategies))す。実際、o1-proモードでは応答に時間がかかる場合があるため、ChatGPTでは処理進捗を示すプ ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=mode%20had%20an%2086,for%20standard%20o1))た。これはユーザーが待ち時間を許容すべきケースがあるほど、モデルが深く推論していることを意味します。

- **o3-miniの効率化:** o1の次世代であるo3-miniは、こうした推論の深さと応答速度のバランス改善を図っています。モデルサイズを抑えつつ高性能を実現しており、APIでは**推論努力度（reasoning_effort）**を「low・mediu ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=I%20released%20LLM%200,reasoning%20effort%E2%80%94details%20in%20this%20issue))能です。例えば簡単な質問には`low`で素早く回答し、難問には`high`で時間をかけるといった具合に使い分けられます。OpenAIによれば、o3-miniは同じタスクでGPT-4ベースモデル(GPT-4o)よりも半分以下のコストで利用できる一方、精度も多くの ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=o3,tokens%20you%20get%20charged%20for)) ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=Confusing%20matters%20further%2C%20the%20benchmarks,o1%20but%20not%20across%20everything))63】。これは内部の最適化により**低遅延・低コスト**での高推論を可能にしているためです。

- **Gemini 2.5の応答体感:** Gemini 2.5 Proは非常に大規模なモデルですが、Googleの大規模インフラ上で最適化されており応答は比較的スムーズです。実例として、ある開発者がGemini 2.5にスネークゲームのPythonコード生成を依頼した際、30秒足らずで要望通りのコードと実行手 ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=That%E2%80%99s%20pretty%20cool%20for%20just,two%20ways%20to%20run%20it))す。この程度の待ち時間であれば実用上許容でき、Gemini 2.5は**高度な内容でも現実的な速度**で回答できることが示唆されます。内部的には“Flash Thinking”と呼ばれる手法で推論を効率化しており、大規模コンテキスト処理にも関わらず高いスループットを実現しています。

- **Claude 3.7のモード切替:** Claude 3.7では通常のチャットモードに加え「Thinking Mode（思考モ ([Claude 3.7 Sonnet: How it Works, Use Cases & More | DataCamp](https://www.datacamp.com/blog/claude-3-7-sonnet#:~:text=First%20off%2C%20Claude%203,which%20lets%20us%20see%20the))た。思考モードではモデルがステップバイステップで考える過程をユーザーに可視化できますが、その分応答完了までの時間は若干増加します。ただしモードは任意に切替可能で、一般的な要約や会話では通常モードで素早く回答し、深い推論が必要な時だけ思考モードを使う運用も可能です。全体としてClaude 3.7の応答速度はGPT-4クラスのモデルと同程度と考えられ、重いコード解析では数十秒程度、単純なQAなら数秒〜十数秒で返答する印象です（公開ベンチマークはありませんが、体感的な報告に基づく）。Anthropicは前世代まで比較的応答が速いことで評価されていた経緯もあり、Claude 3.7も十分実用的な速度を保っています。

### API対応

- **Gemini 2.5:** 現在、Gemini 2.5 ProはGoogleの提供するAIプラットフォームで限定公開されています。具体的には**Google AI Studio**や専用のGeminiアプリから利用可能で、近くGoogle CloudのVertex AI（商用向けAIサ ([Gemini 2.5: Our newest Gemini model with thinking](https://blog.google/technology/google-deepmind/gemini-model-thinking-updates-march-2025/#:~:text=Gemini%202,limits%20for%20scaled%20production%20use))4†L350-L355】。Vertex AIに組み込まれればAPI経由での呼び出しやスケーラブルな利用が可能になり、企業システムにも組み込みやすくなります。料金体系も整備中とされており、大規模モデルながらエンタープライズで使いやすい ([Gemini 2.5: Our newest Gemini model with thinking](https://blog.google/technology/google-deepmind/gemini-model-thinking-updates-march-2025/#:~:text=leading%20on%20common%20coding%2C%20math,and%20science%20benchmarks))4†L348-L355】。

- **OpenAI o3-mini:** o3-miniはChatGPT（OpenAI）上で**2025年1月 ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=OpenAI%20o3,LLM))した。APIでも利用できますが、現時点では使用者に制限があります。OpenAIの開発者向けには**ティア3（一定以上の利用実績があるユーザ）**向けに提供されており、API ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=reasoning%20effort%E2%80%94details%20in%20this%20issue))す。料金は入力100万トークンあたり$1.10、出力100万トークンあたり$4.40とGPT-4系の半 ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=o3,tokens%20you%20get%20charged%20for))す。ChatGPTのUI上でも、高度推論モデルとしてo1と並び利用可能（※ただしPlusプラン以上での提供）で、推論努力度の指定など高 ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=I%20released%20LLM%200,reasoning%20effort%E2%80%94details%20in%20this%20issue))す。将来的には一般開放も見込まれますが、2025年初時点では一部開発者・高度ユーザ向けの提供です。

- **OpenAI o1:** o1（正式リリース版）は、当初「o1-preview」として登場し、その後ChatGPTの有料プラン ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=The%20initial%20launch%20in%20September,2024%20included%20two%20models))9†L173-L182】。現在ではOpenAI APIにも`gpt-o1`等の名称で実装されており、標準版および軽量版（o1- ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=The%20initial%20launch%20in%20September,2024%20included%20two%20models)) ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=o1))9†L193-L201】。ただし最上位の**o1-proモード**は月額$200のChatGPT Proプ ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=mode%20had%20an%2086,for%20standard%20o1))9†L205-L214】、APIから直接o1-proの性能を引き出すことはできません。企業向けには、OpenAIと契約して専用インスタンスを用意するケースもありますが、一般にはChatGPT経由または通常APIでの利用となります。いずれにせよ、OpenAIのAPIエコシステム上で利用できるため既存のGPT-4等からの切替も比較的容易です。

- **Claude 3.7:** ClaudeシリーズはAnthropic社が提供するLLMで、従来からWeb上の**Claude.ai**インタフェースやSlack連携等で利用されてきました。Claude 3.7 Sonnetも同様に、Anthropicが提供するクラ ([Claude 3.7 Sonnet: How it Works, Use Cases & More | DataCamp](https://www.datacamp.com/blog/claude-3-7-sonnet#:~:text=Claude%203))。開発者はAnthropicの開発者ポータルからAPIキーを取得し、モデルID`claude-3-7-sonnet-YYYYMMDD`（日付はリリース ([Claude 3.7 Sonnet: How it Works, Use Cases & More | DataCamp](https://www.datacamp.com/blog/claude-3-7-sonnet#:~:text=API%20model%20name))。従量課金制でトークン数に応じた料金となっており、従来のClaude 2（100k長文対応版）と同様に**高トークン数入力に ([Claude 3.7 Sonnet: How it Works, Use Cases & More | DataCamp](https://www.datacamp.com/blog/claude-3-7-sonnet#:~:text=Developers%20can%20integrate%20Claude%203,is%20accessible%20via%20Anthropic%E2%80%99s%20developer))。注意点として、Claude 3.7の新機能「Thinking Mode」で内部思考を見るには、Claude.ai上の有料プランでUIを通じて行う必要があり、API経由では通常のテ ([Claude 3.7 Sonnet: How it Works, Use Cases & More | DataCamp](https://www.datacamp.com/blog/claude-3-7-sonnet#:~:text=match%20at%20L388%20web%20interface%2C,locked%20behind%20a%20paid%20tier))。しかし通常用途ではAPIで十分強力な機能を引き出せ、各種統合にも用いられ始めています。

### コンテキスト長

各モデルが扱える入力文脈の長さ（コンテキストウィンドウ）は、LLMの実用性を左右する重要な要素です。以下に比較を示します。

- **Gemini 2.5:** 非常に長いコンテキストを扱えるよう最適化されています。現行の2.5 Proでは**最大100万トークン**という桁外れのコンテキスト長をサポートしており、今後は**200万トークン**まで拡大予定 ([Gemini 2.5: Our newest Gemini model with thinking](https://blog.google/technology/google-deepmind/gemini-model-thinking-updates-march-2025/#:~:text=long%20context%20window,and%20even%20entire%20code%20repositories))6†L393-L401】。100万トークンはテキストにして数千万文字規模に相当し、書籍数百冊分にも匹敵します。この大容量により、Gemini 2.5は巨大なデータセット（長大なレポートやコードベースなど）を一度に読み込み分析できる強みがあります。

- **OpenAI o3-mini / o1:** OpenAIのoシリーズ（推論特化モデル）もコンテキスト長が大きく、**最大200,000トー ([LLMs with largest context windows](https://codingscape.com/blog/llms-with-largest-context-windows#:~:text=,200%2C000%20token%20windows%2C%20optimized%20for))です。これは従来のGPT-4（8k〜32k、一部128k拡張版）より遥かに長く、長大なコードやドキュメントでも切り分けず一括でモデルに与えられることを意味します。実際、o1およびo3-miniはいずれも200kトークン対応であり、軽量版のo1-miniでさえ12 ([LLMs with largest context windows](https://codingscape.com/blog/llms-with-largest-context-windows#:~:text=,200%2C000%20token%20windows%2C%20optimized%20for))ます。この大きなウィンドウにより、モデルが内部で長い思考チェーンを展開しても文脈から外れにくくなり、またユーザ側も大量の関連資料を与えて高度な質問をすることが可能です。

- **Claude 3.7:** Claude 2（Claude 100k）で話題となった長文対応はさらに拡張され、Claude 3.5/3.7 Sonnetでは**最大2 ([LLMs with largest context windows](https://codingscape.com/blog/llms-with-largest-context-windows#:~:text=,200%2C000%20token%20windows%2C%20optimized%20for))**のコンテキストを扱えます。Anthropicは長文入力をいち早く商用化した企業ですが、Claude 3.7はその伝統を踏襲し、他のモデルと同 ([LLMs with largest context windows](https://codingscape.com/blog/llms-with-largest-context-windows#:~:text=,200%2C000%20token%20windows%2C%20optimized%20for))のやり取りを可能にしています。これにより、小説や技術文書を丸ごと分析したり、大量の会話ログを一括処理したりといった用途にも耐えられます。もっとも、コンテキスト長が増えるほど**応答時間やコストも増大**するため、実運用では必要に応じて使い分ける形になります（これについては後述のリスク節で触れます）。

- *(参考)* **他モデルとの比較:** なお、OpenAI GPT-4.5やGPT-4oといった派生モデル、Met ([LLMs with largest context windows](https://codingscape.com/blog/llms-with-largest-context-windows#:~:text=200%2C000%20token%20windows%2C%20making%20them,code%20generation%2C%20and%20multilingual%20processing))128k程度が上限であり、現在主流の長文対応モデルは概ね128kまたは200kが上限です。Geminiの100万トークンは突出しています。また、これを超える特殊モデルとしてMagic.devのLTM- ([LLMs with largest context windows](https://codingscape.com/blog/llms-with-largest-context-windows#:~:text=LLMs%20with%20largest%20context%20windows%3A))ど研究的な存在もありますが、一般利用可能な範囲では上記のコンテキスト長が現状最大級と言えます。

## Model Context Protocol (MCP) の概要

**Model Context Protocol (MCP)**は、LLMを外部データやツール ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=Today%2C%20we%27re%20open,produce%20better%2C%20more%20relevant%20responses))ープン標準プロトコルです。2024年11月にAnthropic社から提唱・オープンソース化され、LLMを企業内のデータソースやアプリケーションと**統一的 ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=Today%2C%20we%27re%20open,produce%20better%2C%20more%20relevant%20responses)) ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=MCP%20addresses%20this%20challenge,to%20the%20data%20they%20need))。

- **背景と目的:** 従来、チャットGPTなどのLLMに業務データを参照させたり、ツールを使わせたりするには、個別にプラグインやスクリプトを開発する必要がありました。しかしデータソースごとに異なる実装を行うのは非効率で保守も困難です。MCPはそれ ([Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction#:~:text=MCP%20is%20an%20open%20protocol,different%20data%20sources%20and%20tools))IのUSB-Cポート**」とも言える統一規格を提供します。この規格に沿えば、新しいデータ源が増えても同じ手順でLLMと繋ぎ込 ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=As%20AI%20assistants%20gain%20mainstream,connected%20systems%20difficult%20to%20scale))既存システムのサイロ（孤立環境）から解放しやすくなります。

- **技術仕様 ([Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction#:~:text=General%20architecture))クライアント-サーバアーキテクチャ**に基づいています。具体的には、LLMを使ったホストアプリケーションが「MC ([Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction#:~:text=At%20its%20core%2C%20MCP%20follows,can%20connect%20to%20multiple%20servers))種データ源ごとに用意された「MCPサーバ」と通信します。MCPサーバは軽量なサービスで、特定の機能やデータ ([Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction#:~:text=,MCP%20servers%20can%20connect%20to))ベース、外部APIなど）を標準化された手順で提供します。LLM側（ホスト）は必要に応じてサーバにリクエストを送り、 ([Model context protocol (MCP) - OpenAI Agents SDK](https://openai.github.io/openai-agents-python/mcp/#:~:text=MCP%20servers%20can%20be%20added,on%20that%20server))に組み込んだり、ツール実行結果として利用したりします。この仕組みにより、LLMはあたかも自分で外部にアクセスしているかのように最新の情報や社内データを参照できます。

- **双方 ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=The%20Model%20Context%20Protocol%20is,that%20connect%20to%20these%20servers))* MCPの特徴は「**双方向**」かつ「**セキュア**」であることです。双方向とは、単にLLMがデータを読むだけでなく、外部システム側への書き込みやアクションも標準APIで行えることを意味します（もちろん許可された範囲で）。セキュリティ面では、各MCPサーバはホスト側と**標準化プロトコル**で通信するため、認証・認可や通信内容の構造化が明確です。また、企業は自社インフラ内にサーバをホスティングし、LLM（クラウドサービスであっても）との間で必要なデータだけをやり取りする構成が可能です。AnthropicはMC ([Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction#:~:text=MCP%20helps%20you%20build%20agents,and%20tools%2C%20and%20MCP%20provides))内でデータを安全に取り扱うためのベストプラクティスを提供する」と述べており、機密データを扱う統合でもセキュリティガイドラインに沿って実装できるよう配 ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=Today%2C%20we%27re%20introducing%20three%20major,Model%20Context%20Protocol%20for%20developers))**提供物:** MCPはオープン標準であり、以下の要素が公開されています。
  - **プロトコル仕様書とSDK** – ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=Today%2C%20we%27re%20introducing%20three%20major,Model%20Context%20Protocol%20for%20developers))と各種言語用のSDKが提供され、独自のMCPサーバ/クライアントを実装可能。
  - **Claude DesktopにおけるローカルMCP対応** – Anthropicが提供するデスクトップ版Claudeアプリで ([Understanding Model Context Protocol (MCP) | by Ashraff Hathibelagal | Predict | Mar, 2025 | Medium](https://medium.com/predict/understanding-model-context-protocol-mcp-771f1cfb3c0a#:~:text=and%20contributions%20from%20the%20community))報を追記するだけでローカルMCPサーバに接続できる機能が組み込まれています【26†L78-L ([Understanding Model Context Protocol (MCP) | by Ashraff Hathibelagal | Predict | Mar, 2025 | Medium](https://medium.com/predict/understanding-model-context-protocol-mcp-771f1cfb3c0a#:~:text=and%20contributions%20from%20the%20community))簡単に使える実装例として「MCP統合のゴールドスタンダード」とされています。
  - **MCPサー ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=,source%20repository%20of%20MCP%20servers)) 代表的なデータソースに対するサーバ実装がオープンソースで公開されています。現在利用できるサーバには、Google Driveファイルシステム、Slackメッセージ履歴、GitHubリポジトリ、Git（汎用）、Postg ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=Claude%203,GitHub%2C%20Git%2C%20Postgres%2C%20and%20Puppeteer))Puppeteer（Web閲覧用ヘッドレスブラウザ）などがあります。これらを組み合わせることで、ファイル取得や社内チャット検索、コードリポジトリからのコード取り込み、Webサイトからの情報収集等がすぐに試せます。

- **採用事例と展望:** MCPは発表直後から複数の企業・プロジェクトで採用が進んでいます。例えば決済企業のBlock社（旧Square社）や通信プラットフォームのApollo社では自社システムにMCPを統合し始めており、Zed（エディタ開発会社）、Replit、Codeium、Sourc ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=Early%20adopters%20like%20Block%20and,functional%20code%20with%20fewer%20attempts))者向けツール企業も自社プラットフォームへのMCP対応を進めています。これにより、これらのIDEやコード検索ツール上でAIエージェン ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=while%20development%20tools%20companies%20including,functional%20code%20with%20fewer%20attempts))報を即座に取得し、より高度なコーディング支援を行えるようになります。また、コミュニティでも**Windsurf**や**Cursor**、**Cline**といった独自のMCPクライアント（LL ([Understanding Model Context Protocol (MCP) | by Ashraff Hathibelagal | Predict | Mar, 2025 | Medium](https://medium.com/predict/understanding-model-context-protocol-mcp-771f1cfb3c0a#:~:text=But%20right%20now%2C%20Claude%20Desktop,coders%20and%20quick%20tasks))、Claude以外のモデルでもMCPを活用する動きが出てきています【2 ([Model context protocol (MCP) - OpenAI Agents SDK](https://openai.github.io/openai-agents-python/mcp/#:~:text=The%20Agents%20SDK%20has%20support,provide%20tools%20to%20your%20Agents))nAIも自社のエージェント向けSDKでMCPをサポートする方針を示しており、今後はベンダーを超えて統一的に使われる可能性があります。総じてMCPは、LLMを企業システムへ**つなぎ込むハブ**として今後重要な役割を果たすと期待されています。

## LLMとMCPのメインフレーム移行・モダナイゼーションへの応用

メインフレームのモダナイゼーション（近代化、他プラットフォームへの移行）は、多くの企業が直面する課題です。典型的な課題として、以下のような点が挙げられます。

- **レガシーコードの巨大さと複雑さ:** 何十年も改修を重ねたCOBOLなどのコードが数百万行に及び、システム全体の把握が困難。動いているがブラックボックス化しているプログラムも多い。
- **ドキュメント不足:** 古いシステムでは設計書や仕様書が散逸・未整備であり、現場の担当者の頭の中にしか業務知識がない状況もある。
- **レガシー人材の不足:** COBOL技術者の高齢化・減少により、システム理解や改修ができる人材が不足している。
- **ビジネスフローの不透明:** 現行業務がコードでどう実現されているか全体像を把握しづらく、新システムへの再実装要件を洗い出すのが難しい。
- **技術的負債と互換性:** 新システムへの移行時に、既存機能との互換性を保ちつつ古い非効率な実装を改善する必要があるが、その両立が難しい。
- **テスト工数の膨大さ:** レガシーシステムの振る舞いを保証するには莫大なテストが必要だが、テストケースの網羅も難しい。

これらの課題に対して、最新LLMとMCPの組み合わせは強力なソリューションを提供し得ます。以下に具体的なユースケースと有効性を提案します。

### LLMによるレガシーコードの理解・ドキュメント化

メインフレームのCOBOLコードのような長大なソースコードを、LLMを用いて解析・要約することが可能です。前述の通りGemini 2.5やo3系モデルは数十万トークン規模のコンテキストを扱えるため、**大きなCOBOLプログラムを丸ごと入力しその処理概要を日本語で説明させる**といったこともできます。実際、AIを使って「アプリケーションがどのように動作し、どんなデ ([The future of mainframe modernization with artificial intelligence (AI)  and generative AI](https://www.kyndryl.com/content/dam/kyndrylprogram/doc/en/2024/future-mainframe-modernization.pdf#:~:text=Some%20of%20the%20ways%20that,leveraging%20Kyndryl%E2%80%99s))析・ドキュメント化する」ことがモダナイゼーションに有用だと指摘されています。LLMは自然言語でコードの意図を解説したり、モジュール間の依存関係を列挙したりすることができ、**失われた設計書の代替**を生成できます。

例えば、あるCOBOLプログラムについてLLMに「このプログラムの目的と主要な処理手順を要約せよ」とプロンプトを与えると、該当コードを読み込み「このプログラムは○○業務のバッチ処理であり、ファイルAを読み込んで集計し、結果をDBに書き出す。主なステップは...(以下略)」といった要約を返すでしょう。さらに詳細な質問（「○○というエラーコードが出る条件は？」等）にも、コードロジックを追って回答できます。これはシニアエンジニアにコードをインタビューするようなもので、**LLMが仮想的なレガシーシステムの専門家**として機能するイメージです。

MCPを活用すれば、コード解析の効率は一段と上がります。LLMエージェントがMCP経由で社内のソースコード管理システム（Gitなど）から必要なプログラムを自動取得し、内容を解析してドキュメントを生成するといったことが可能になります。例えば「在庫更新バッチの仕様書を生成して」というユーザ指示に対し、エージェントがMCP連携したGitサーバから`INVUPDT.cbl`を取得し、LLMで解析し、その結果をWikiフォーマットでまとめてMCP経由でドキュメント管理システムに登録する、といった自動化が考えられます。これにより**コードからドキュメントへの変換作業**が大幅に効率化され、人手不足の中でも知識の形式知化が進みます。

### コード変換（COBOL→モダン言語）とリファクタリング

レガシーコードを現代的な言語・アーキテクチャに変換する作業にもLLMが活用できます。例えばIBMは、自社のLLM（watsonx.ai）を訓練してCOBOLコード ([IBM trains its LLM to read, rewrite COBOL apps | CIO Dive](https://www.ciodive.com/news/IBM-COBOL-AI-watsonx/691395/#:~:text=,the%20year%2C%20the%20company%20said))換える支援を行う「Code Assistant for Z」を発表しました。LLMはCOBOLの構造を解析し、同等のビジネスロジックを持つJavaコードを生成します。さらに、変換後のコードの検証（バリデ ([IBM trains its LLM to read, rewrite COBOL apps | CIO Dive](https://www.ciodive.com/news/IBM-COBOL-AI-watsonx/691395/#:~:text=,the%20year%2C%20the%20company%20said))ングの提案も行い、開発者が効率よくレガシー資産をモダン化できるようにします。

このように**COBOLからJava等への自動変換**はLLMの得意分野の一つです。大規模言語モデルはプログラミング言語間の対応を学習しており、ある程度の規模までは一貫性のある翻訳が可能です。完全自動化は難しいにせよ、**全体の7～8割のコード変換をLLMが下訳**し、残りを人間が修正・最適化するという形なら大幅な工数削減が期待できます。特に単純なCRUD処理や計算ロジックなどは自動変換精度が高く、開発者は細かな構文修正よりも**設計上の改良**に注力できます。

LLMを使った変換の際も、MCPがあると便利です。例えばエージェントがMCP経由でコンパイラやテスト実行環境と連携し、変換したJavaコードを即座にコンパイル・実行してみて、COBOL版と出力が一致するか検証する、といった自動チェックが可能です。Anthro ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=Early%20adopters%20like%20Block%20and,functional%20code%20with%20fewer%20attempts))は外部ツールを組み合わせた**エージェント的なコード改善**に注力しており、MCPで統一された環境があれば「コード取得 → 変換 → テスト → レビュー」を一連のフローとしてAIが回すことも見えてきます。現に「AIがコードを ([IBM trains its LLM to read, rewrite COBOL apps | CIO Dive](https://www.ciodive.com/news/IBM-COBOL-AI-watsonx/691395/#:~:text=,the%20year%2C%20the%20company%20said))、変換結果を検証する」という一連のプロセスはモダナイゼーションで有効であり、これを自動化する基盤としてMCP+LLMは有望です。

### 業務フローの可視化・ビジネスルール抽出

モダナイゼーションでは、現行システムの**業務フロー**や**ビジネスルール**を洗い出して新システムに再定義する作業が不可欠です。LLMは自然言語による要件定義やルール記述の生成も得意なため、コードからビジネスルールを抽出・整理する支援が期待できます。

具体的には、複数のプログラムやモジュールにまたがる**ビジネスロジックの一貫性**をLLMでチェックし、人間にとって理解しやすい形で提示することが可能です。例えば「受注から出荷までの処理フローを説明せよ」とLLMに尋ねると、関連する複数のCOBOLプログラム（受注入力、在庫引当、出荷指示、請求書発行など）の役割をまとめ、どのデータがどう受け渡されて最終的に出荷が完了するかをストーリー仕立てで説明できるでしょう。これは従来であれば担当者の経験に頼っていた部分ですが、LLMはコードとシステム設定からその因果関係を推測してくれます。

また、条件分岐の多いビジネスルール（例：「顧客タイプがXかつ注文金額がY以上の場合は特別割引を適用」など）について、LLMが**条件網羅表**や**決定木**の形で出力することも考えられます。コード中の`IF/ELSE`ロジックを解析し、「入力パターンと対応処理」の対応表に整理することで、人間が読み解きやすいビジネ ([The future of mainframe modernization with artificial intelligence (AI)  and generative AI](https://www.kyndryl.com/content/dam/kyndrylprogram/doc/en/2024/future-mainframe-modernization.pdf#:~:text=Some%20of%20the%20ways%20that,leveraging%20Kyndryl%E2%80%99s))こうした「プログラムが何をしているかの記述」はAIが得意とするところであり、後継システムの設計資料として非常に有用です。

MCPの活用により、LLMエージェントは業務フロー可視化のために様々なデータにアクセスできます。例えば、MCPサーバ経由で**ジョブスケジューラ**からバッチ実行順序を取得したり、**データベース**からテーブル定義（どのテーブルにどんなデータが溜まっているか）を取得したりできます。LLMはそれらの情報を総合して、「日次バッチ処理はまずジョブA（在庫更新）、次にジョブB（売上集計）が走り、それぞれテーブルT_INVとT_SALESを更新する」といった**システム全体のフロー図**を描けるかもしれません。現在は人手で行っているシステム構成の把握・ドキュメント化を、LLMが自動支援する未来像が見えてきます。

### テストケース生成と自動検証

モダナイゼーションの過程で避けて通れないのが**回帰テスト**です。現行システムと新システムの挙動を照合し、業務結果が一致することを保証しなければなりません。LLMはこのテスト工程にも多方面で寄与できます。

まず、LLMは**網羅的なテストケースの生成**を支援します。レガシ ([The future of mainframe modernization with artificial intelligence (AI)  and generative AI](https://www.kyndryl.com/content/dam/kyndrylprogram/doc/en/2024/future-mainframe-modernization.pdf#:~:text=Some%20of%20the%20ways%20that,leveraging%20Kyndryl%E2%80%99s))うな入力パターンが重要か、エッジケースは何か、といった洞察を与えてくれます。例えば「このプログラムで考慮すべき異常系シナリオは？」と問えば、「在庫数量がマイナスになるケース」「ファイルが存在しないケース」等、見落としがちなシナリオを提案してくれるでしょう。これは長年そのシステムを知るエンジニアの知見に近く、AIが**テストエンジニアの補佐**を務める形です。

さらに、LLM自体に**テスト実行と結果確認**をさせることも可能です。例えば新旧システムに同じ入力データを与え、それぞれの出力ログをMCP経由で取得し、LLMに差異を説明させることができます。LLMは「旧システムでは○○という値になっているが、新システムでは△△になっている。おそらく○○の計算式に違いがある」といった分析を行い、人間にフィードバックできます。これにより不一致の原因調査が迅速化し、バグ修正サイクルが短縮します。

加えて、テストデータの生成にもLLMは有用です。例えば本番データに似たダミーデータを大量に作ったり、特定の条件を再現する入力データを作成したりできます。MCPで社内のデータベースに接続し、実データの傾向を掴んだ上でLLMが類似分布の架空データを生成するといったことも可能でしょう。これらにより、テストに必要な下準備も自動化でき、移行検証の確実性が増します。

### AIアシスタントによる開発効率化と知識継承

LLMはモダナイゼーションにおいて**人材不足を補完するAIアシスタント**としても機能します。メインフレーム技術者の減少という課題に対し、AIは新人エンジニアの強力な相棒となり得ます。例えば、開発者が「このCOBOLコードの意味が分からない」といった際、LLMが即座に解説し疑問に答えてくれれば、生産性が向上します。Kyndrylも「ジェネレーティブAIをま ([The future of mainframe modernization with artificial intelligence (AI)  and generative AI](https://www.kyndryl.com/content/dam/kyndrylprogram/doc/en/2024/future-mainframe-modernization.pdf#:~:text=advanced%20AI,them%20to%20focus%20on%20very))て使い、人が結果を検証する形でスキル不足を補う」のが効果的だと述べています。AIが定型的・下調べ的な作業を肩 ([The future of mainframe modernization with artificial intelligence (AI)  and generative AI](https://www.kyndryl.com/content/dam/kyndrylprogram/doc/en/2024/future-mainframe-modernization.pdf#:~:text=AI%20exploitation%2C%20in%20general%2C%20is,level%20issues%20to%20AI%20engines))思決定に集中することで、熟練者の負担を減らしつつ新人の育成にもつながります。

MCPと組み合わせれば、このAIアシスタントはさらに有用になります。開発者との対話において、AIが必要な情報をMCP経由で社内から引き出しながら回答できれば、まるで社内Wiki＋ベテラン社員の知識を合わせ持つ相談役となります。例えば「〇〇というエラーが昨夜のバッチで出たが原因は？」と質問すれば、AIがログ管理システムから該当エラーのスタックトレースを取得（MCP経由）し、さらにソースコード上でエラーコードを検索して原因個所を特定、「モジュールXのデータ変換でNULL値が発生したためです」と回答する、といった高度な応答も理論上可能です。これは将来的な展望ですが、MCP対応が各種システムに広がれば**AIの社内システム横断力**が飛躍的に高まり、メインフレームからクラウドまで一貫したサポートを提供できるでしょう。

## LLM活用のリスクとMCP導入時の注意点

LLMとMCPをモダナイゼーションに活用する際には、いくつか留意すべきリスクや制限事項も存在します。

- **回答の正確性・信頼性:** LLMは流暢な回答を生成しますが、内容が常に正しいとは限りません。いわゆる**幻覚（hallucination）**により、コードの説明や変換結果に誤りが混入する可能性があります。例えばCOBOLコードの微妙な挙動を誤解して説明したり、コンパイルは通るが要件を満たさないJavaコードを生成した ([The future of mainframe modernization with artificial intelligence (AI)  and generative AI](https://www.kyndryl.com/content/dam/kyndrylprogram/doc/en/2024/future-mainframe-modernization.pdf#:~:text=advanced%20AI,them%20to%20focus%20on%20very))、LLMの出力は**必ず人間が検証するプロセス**を組み込む必要があります。AIはあくまで提案者・補助者であり、最終的な責任を持つのは人間です。この点を怠ると、不備のあるドキュメントやバグを含んだプログラムを元にシステムを再構築してしまうリスクがあります。

- **機密データ・プライバシー:** レガシーシステムのコードやデータには機密情報が含まれる場合があります。クラウド上のLLMサービス（OpenAIやAnthropic等）にそのまま投入すると、情報漏洩や規約違反となる可能性があります。対策として、**オンプレミス版LLM**（企業内サーバで動かすモデル）を利用したり、入力データをマスキング・要約してから送信したりする工夫 ([Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction#:~:text=MCP%20helps%20you%20build%20agents,and%20tools%2C%20and%20MCP%20provides))の一つは「データを自社インフラ内に留めたままAIと連携できる」点にあります。例えば社内のMCPサーバがデータベースにアクセスし、その要点だけをLLMに渡すようにすれば、生データを直接外部に出さずに済みます。ただしMCP自体のセキュリティ設定も重要です。各サーバに適切な認可を設計し、LLMが不必要なデータにアクセスしないよう制御する必要があります。また、MCP通信ログを監査することで、AIがどのデータを参照したか履歴を残すことも考慮すべきです。

- **コストとパフォーマンス:** 大容量コンテキストを活用できるLLMですが、その分トークン課金費用も増大します。20万トークンの入力はAPI利用料が高額になり得ますし、処理時間も長くなります。モダナイゼーショ ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=o3,tokens%20you%20get%20charged%20for))大な場合、**すべてをLLM任せにするとコストが膨れ上がる**懸念があります。したがって、現実的には人間が要所を見極め、AIを使う部分とそうでない部分を切り分けることが重要です。例えば「まず要約だけさせて重要箇所を絞り込み、その部分の詳細変換をAIに任せる」「頻出する似た処理は1つだけAIに検討させて他はテンプレート適用で済ませる」といった工夫です。また ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=I%20released%20LLM%200,reasoning%20effort%E2%80%94details%20in%20this%20issue))ランスを調整するために、o3-miniの**推論努力レベル**を使い分けたり、必要に応じて小型モデル（社内にある小規模LLMなど）と大規模モデルを使い分けたりする戦略も有効でしょう。

- **モデルの専門性:** 汎用LLMは膨大な一般知識を持ちますが、メインフレーム特有の知識（例えばCOBOLのレガシー機能、JCLの書式、CICSトランザクション処理など）については十分にカバーしていない可能性があります。実際、メインフレー ([XMainframe: A Large Language Model for Mainframe Modernization](https://arxiv.org/html/2408.04660v2#:~:text=interact%20with%20legacy%20codebases,art%20LLMs%20across%20these))例：XMainframeというCOBOL知識に特化したLLM）の研究も進められているほどです。従って、生成された回答やコードが業務的に妥当かどうか、メインフレームの文脈で意味が通じるかは、ドメイン知識を持つ人間が確認する必要があります。また場合によっては、LLMに追加学習させる（自社のCOBOLコードを学習データとしてファインチューニングする、あるいは社内用プロンプト集を用意する）ことで精度向上を図ることも検討すべきです。

- **MCP導入の手間と課題:** MCPは統合を容易にしますが、導入初期には設定や社内システム側の対応が必要です。各種コネクタ（MCPサーバ）を立ち上げ、社内のどのデータにどうアクセスさせるかを決めるには、IT部門の連携が欠かせません。また新しいプロトコルゆえにバグや仕様変更への対応も追随していく必要があります。 ([Understanding Model Context Protocol (MCP) | by Ashraff Hathibelagal | Predict | Mar, 2025 | Medium](https://medium.com/predict/understanding-model-context-protocol-mcp-771f1cfb3c0a#:~:text=and%20contributions%20from%20the%20community))cのClaude Desktopがスムーズに使えるものの、他の環境で利用する場合はSDKを用いた実装が必要になることもあります。とはいえ、長期的 ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=MCP%20addresses%20this%20challenge,to%20the%20data%20they%20need))が効きますし、**一対多の統合を一元化できるメリット**は手間を上回る価値があります。導入に際しては、小規模なPoC（例えばGitリポジトリ閲覧サーバとClaudeを繋いでコード要約を試す等）から始め、徐々に接続先を増やしていくのが安全策です。運用上は、MCPサーバ自体の負荷管理やバージョン管理、セキュリティパッチ適用にも注意を払い、基盤の一部として位置付ける必要があります。

- **倫理・ガバナンス:** AIに高い自由度を与えると、意図しない操作（例えば誤った判断でデータを削除する等）をするリスクもあります。MCPではAIが外部ツールを実行できるため、その権限をどう制限するかガバナンスが重要です。基本的には読み取り専用のコネクタから試し、書き込み系（チケット発行やコード自動コミット等）は十分検証してから有効化するのが望ましいでしょう。また、AIの提案に最終決定を下すのは人間 ([The future of mainframe modernization with artificial intelligence (AI)  and generative AI](https://www.kyndryl.com/content/dam/kyndrylprogram/doc/en/2024/future-mainframe-modernization.pdf#:~:text=advanced%20AI,them%20to%20focus%20on%20very))共有し、結果への責任の所在を明確にしておくことも大切です【31†L45-L53 ([The future of mainframe modernization with artificial intelligence (AI)  and generative AI](https://www.kyndryl.com/content/dam/kyndrylprogram/doc/en/2024/future-mainframe-modernization.pdf#:~:text=advanced%20AI,them%20to%20focus%20on%20very))ルは「人が確認・修正する前提」で使うことが推奨されており、過信せず適切に役割分担すれば有益なパートナーとなります。

以上、最新LLMとMCPの技術概要、およびそれらを活用したメインフレーム近代化への応用例と注意点を述べました。LLMは急速に進歩しており、従来人手に頼っていたレガシー資産の理解・変換作業を大幅に効率化できる可能性を秘めています。同時に、MCPのような標準化されたインターフェースを用いることで、安全かつスケーラブルにAIを既存システムへ組み込める道筋が見えてきました。適切なガバナンスの下でこれら新技術を活用すれば、レガシー資産のモダナイズを加速し、ビジネスの変革につなげることができるでしょう。

【参考資料】　本回答中で引用した文献・情報源を以下に示します。

- **LLM比較関連:**# 最新LLMとMCPの技術 ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=That%E2%80%99s%20pretty%20cool%20for%20just,two%20ways%20to%20run%20it))ダナイゼーションへの応用

## 最新LLMの比較

最新の大規模言語モデル（LLM）として、**Google Gemini 2.5**、**OpenAI GPT o3-mini（以下、o3-mini）**、**OpenAI GPT  ([Claude 3.7 Sonnet: How it Works, Use Cases & More | DataCamp](https://www.datacamp.com/blog/claude-3-7-sonnet#:~:text=Claude%203,5%20Sonnet))、**Anthropic Claude 3.7 Sonnet（以下、Claude 3.7）**の特徴を比較します。それぞれ性能や推論速度、提供形態（API対応）、コンテキスト長に特徴があります。

### 性能（ベンチマーク）

- **G ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=It%20generally%20benchmarks%20higher%20than,o1%20but%20not%20across%20everything)) ([OpenAI o3-mini, now available in LLM](https://simonwillison.net/2025/Jan/31/o3-mini/#:~:text=I%20released%20LLM%200,reasoning%20effort%E2%80%94details%20in%20this%20issue))）は、総合的な推論力で非常に高い性能を示しています。例えば、人間の専門知識を問う難問揃いのベンチマーク「Humanity’s Last Exam」では18.8%というスコアでトップとなり、o3-miniの14%やClaude 3. ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=o1%20pro%20mode)) ([OpenAI o1 explained: Everything you need to know](https://www.techtarget.com/whatis/feature/OpenAI-o1-explained-Everything-you-need-to-know#:~:text=mode%20had%20an%2086,for%20standard%20o1))337】。数学・論理系の試験であるAIMEでも最高性能（2024年度版で92.0%、2025年度版で86.7%）を記録し、後述のo3-miniと僅差のトップです。総合 ([LLMs with largest context windows](https://codingscape.com/blog/llms-with-largest-context-windows#:~:text=,200%2C000%20token%20windows%2C%20optimized%20for))理解に優れ、**LMArena**のLLM評価でも1位を獲得しています。

- **OpenAI GPT o3-mini**は、OpenAIの「推論重視」モデル系列（oシリーズ）の最新軽量版です。難解な問題に対する ([Gemini 2.5: Our newest Gemini model with thinking](https://blog.google/technology/google-deepmind/gemini-model-thinking-updates-march-2025/#:~:text=Today%20we%E2%80%99re%20introducing%20Gemini%202,LMArena%20by%20a%20significant%20margin))iに次ぐ性能を発揮します。特に数学ではAIME 2025で86.5%とGeminiに肉薄し、コード生成では**L ([Gemini 2.5: Our newest Gemini model with thinking](https://blog.google/technology/google-deepmind/gemini-model-thinking-updates-march-2025/#:~:text=long%20context%20window,and%20even%20entire%20code%20repositories))で74.1%とGemini（70.4%）を上回るスコアを出しています。一方、総合知識テストではGeminiには及ばな ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=,2)) ([Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/gemini-2-5-pro#:~:text=On%20benchmarks%20that%20test%20code,performs%20well%20but%20doesn%E2%80%99t%20dominate))enAIの従来モデル（GPT-4など）より多くの分野で優位と報告されています。特筆すべきは**競技プログラミング**分野で、Codeforcesの指標となるELOレーティングが高く（ ([Claude 3.7 Sonnet: How it Works, Use Cases & More | DataCamp](https://www.datacamp.com/blog/claude-3-7-sonnet#:~:text=Claude%203))従来のGPT-4ベースモデル(約900)やo1を大きく凌駕しました。

- **OpenAI GPT o1**は、2024年にプレビュー版が登場した**深い推論特化モデル**です。最新モデル群と ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=Today%2C%20we%27re%20open,produce%20better%2C%20more%20relevant%20responses)) ([Introducing the Model Context Protocol \ Anthropic](https://www.anthropic.com/news/model-context-protocol#:~:text=Early%20adopters%20like%20Block%20and,functional%20code%20with%20fewer%20attempts))力に定評があります。例えば数学試験のAIME 2024では標準o1で78%の合格率、強化版のo1-proモードでは86%に達したと報告されています【9†L199- ([Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction#:~:text=MCP%20is%20an%20open%20protocol,different%20data%20sources%20and%20tools)) ([Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction#:~:text=MCP%20helps%20you%20build%20agents,and%20tools%2C%20and%20MCP%20provides))niと同程度の高水準です。一方、汎用知識や日常会話においてはGPT-4相当（GPT-4oと呼ばれるモデル）よりも思考重視の設計ゆえ、必ずしも全分野で上回るわけではありません。とはいえ、総合的な論理推論やコード理解では依然強力であり、 ([Understanding Model Context Protocol (MCP) | by Ashraff Hathibelagal | Predict | Mar, 2025 | Medium](https://medium.com/predict/understanding-model-context-protocol-mcp-771f1cfb3c0a#:~:text=and%20contributions%20from%20the%20community)) ([Understanding Model Context Protocol (MCP) | by Ashraff Hathibelagal | Predict | Mar, 2025 | Medium](https://medium.com/predict/understanding-model-context-protocol-mcp-771f1cfb3c0a#:~:text=But%20right%20now%2C%20Claude%20Desktop,coders%20and%20quick%20tasks))図されています。

- **Claude 3.7 Sonnet**（Anthropic社）は、Claude 2から進化した最新モデルです。バージョン番号以上に大幅な改善があり、**推論モード（Thinking Mod ([IBM trains its LLM to read, rewrite COBOL apps | CIO Dive](https://www.ciodive.com/news/IBM-COBOL-AI-watsonx/691395/#:~:text=,the%20year%2C%20the%20company%20said))内部の思考過程を経て回答するアプローチを採用しています。その結果、ソフトウェアエンジニアリング分野では前版Claude 3.5に比べ精度が向上し、SWE-Bench（コードに関する難問集）では62.3%→**70.3%**へ大 ([The future of mainframe modernization with artificial intelligence (AI)  and generative AI](https://www.kyndryl.com/content/dam/kyndrylprogram/doc/en/2024/future-mainframe-modernization.pdf#:~:text=Some%20of%20the%20ways%20that,leveraging%20Kyndryl%E2%80%99s)) ([The future of mainframe modernization with artificial intelligence (AI)  and generative AI](https://www.kyndryl.com/content/dam/kyndrylprogram/doc/en/2024/future-mainframe-modernization.pdf#:~:text=advanced%20AI,them%20to%20focus%20on%20very))17†L303-L311】。この70.3%はGemini 2.5 Proの63.8%を上回り、同ベンチマークではClaude 3.7が最優でした。一方で、知識推論系では先述の通りGeminiやo3-miniほど高得点ではなく、Humanit ([XMainframe: A Large Language Model for Mainframe Modernization](https://arxiv.org/html/2408.04660v2#:~:text=interact%20with%20legacy%20codebases,art%20LLMs%20across%20these))mで8.9%に留まるなど弱めの分野もあります。総じて、Claude 3.7は**コード生成・エージェント的タスク**に強みを発揮しつつ、全体としても前世代モデルを着実に凌駕する性能 ([Model context protocol (MCP) - OpenAI Agents SDK](https://openai.github.io/openai-agents-python/mcp/#:~:text=The%20Model%20context%20protocol%20,From%20the%20MCP%20docs)) ([Model context protocol (MCP) - OpenAI Agents SDK](https://openai.github.io/openai-agents-python/mcp/#:~:text=The%20Agents%20SDK%20has%20support,provide%20tools%20to%20your%20Agents))セスと速度のトレードオフ:** これら最新LLMの多くは高精度のために内部で「思考チェーン」を実行するため、従来モデルより応答時間が長くなりがちです。特にOpenAIのo1シリーズは「回答を急がずじっくり考える」方針であり、複雑な質問では従来モデル以上の計算時間を要します。実際、o1-proモードでは応答に時間がかかる場合があるため、ChatGPTでは処理進捗を示すプログレスバーが実装されました。これはユーザーが待ち時間を許容すべきケースがあるほど、モデルが深く推論していることを意味します。

- **o3-miniの効率化:** o1の次世代であるo3-miniは、こうした推論の深さと応答速度のバランス改善を図っています。モデルサイズを抑えつつ高性能を実現しており、APIでは**推論努力度（reasoning_effort）**を「low・medium・high」で調整可能です。例えば簡単な質問には`low`で素早く回答し、難問には`high`で時間をかけるといった具合に使い分けられます。OpenAIによれば、o3-miniは同じタスクでGPT-4ベースモデル(GPT-4o)よりも半分以下のコストで利用できる一方、精度も多くのベンチマークで上回っています。これは内部の最適化により**低遅延・低コスト**での高推論を可能にしているためです。

- **Gemini 2.5の応答体感:** Gemini 2.5 Proは非常に大規模なモデルですが、Googleの大規模インフラ上で最適化されており応答は比較的スムーズです。実例として、ある開発者がGemini 2.5にスネークゲームのPythonコード生成を依頼した際、30秒足らずで要望通りのコードと実行手順を返したと報告されています。この程度の待ち時間であれば実用上許容でき、Gemini 2.5は**高度な内容でも現実的な速度**で回答できることが示唆されます。内部的には“Flash Thinking”と呼ばれる手法で推論を効率化しており、大規模コンテキスト処理にも関わらず高いスループットを実現しています。

- **Claude 3.7のモード切替:** Claude 3.7では通常のチャットモードに加え「Thinking Mode（思考モード）」が導入されました。思考モードではモデルがステップバイステップで考える過程をユーザーに可視化できますが、その分応答完了までの時間は若干増加します。ただしモードは任意に切替可能で、一般的な要約や会話では通常モードで素早く回答し、深い推論が必要な時だけ思考モードを使う運用も可能です。全体としてClaude 3.7の応答速度はGPT-4クラスのモデルと同程度と考えられ、重いコード解析では数十秒程度、単純なQAなら数秒〜十数秒で返答する印象です（公開ベンチマークはありませんが、体感的な報告に基づく）。Anthropicは前世代まで比較的応答が速いことで評価されていた経緯もあり、Claude 3.7も十分実用的な速度を保っています。

### API対応

- **Gemini 2.5:** 現在、Gemini 2.5 ProはGoogleの提供するAIプラットフォームで限定公開されています。具体的には**Google AI Studio**や専用のGeminiアプリから利用可能で、近くGoogle CloudのVertex AI（商用向けAIサービス）にも統合予定です。Vertex AIに組み込まれればAPI経由での呼び出しやスケーラブルな利用が可能になり、企業システムにも組み込みやすくなります。料金体系も整備中とされており、大規模モデルながらエンタープライズで使いやすい形で提供される見込みです。

- **OpenAI o3-mini:** o3-miniはChatGPT（OpenAI）上で**2025年1月に提供開始**されました。APIでも利用できますが、現時点では使用者に制限があります。OpenAIの開発者向けには**ティア3（一定以上の利用実績があるユーザ）**向けに提供されており、APIキーでリクエストできます。料金は入力100万トークンあたり$1.10、出力100万トークンあたり$4.40とGPT-4系の半額以下に設定されています。ChatGPTのUI上でも、高度推論モデルとしてo1と並び利用可能（※ただしPlusプラン以上での提供）で、推論努力度の指定など高度なオプションも使えます。将来的には一般開放も見込まれますが、2025年初時点では一部開発者・高度ユーザ向けの提供です。

- **OpenAI o1:** o1（正式リリース版）は、当初「o1-preview」として登場し、その後ChatGPTの有料プラン向けに提供されてきました。現在ではOpenAI APIにも`gpt-o1`等の名称で実装されており、標準版および軽量版（o1-mini）が利用可能です。ただし最上位の**o1-proモード**は月額$200のChatGPT Proプラン限定で提供されており、APIから直接o1-proの性能を引き出すことはできません。企業向けには、OpenAIと契約して専用インスタンスを用意するケースもありますが、一般にはChatGPT経由または通常APIでの利用となります。いずれにせよ、OpenAIのAPIエコシステム上で利用できるため既存のGPT-4等からの切替も比較的容易です。

- **Claude 3.7:** ClaudeシリーズはAnthropic社が提供するLLMで、従来からWeb上の**Claude.ai**インタフェースやSlack連携等で利用されてきました。Claude 3.7 Sonnetも同様に、Anthropicが提供するクラウドAPIで利用可能です。開発者はAnthropicの開発者ポータルからAPIキーを取得し、モデルID`claude-3-7-sonnet-20250219`（日付はリリース日）を指定して利用します。従量課金制でトークン数に応じた料金となっており、従来のClaude 2（100k長文対応版）と同様に**高トークン数入力に対応**した価格設定です。注意点として、Claude 3.7の新機能「Thinking Mode」で内部思考を見るには、Claude.ai上の有料プランでUIを通じて行う必要があり、API経由では通常のテキスト応答のみとなります。しかし通常用途ではAPIで十分強力な機能を引き出せ、各種統合にも用いられ始めています。

### コンテキスト長

各モデルが扱える入力文脈の長さ（コンテキストウィンドウ）は、LLMの実用性を左右する重要な要素です。以下に比較を示します。

- **Gemini 2.5:** 非常に長いコンテキストを扱えるよう最適化されています。現行の2.5 Proでは**最大100万トークン**という桁外れのコンテキスト長をサポートしており、今後は**200万トークン**まで拡大予定と公式に言及されています。100万トークンはテキストにして数千万文字規模に相当し、書籍数百冊分にも匹敵します。この大容量により、Gemini 2.5は巨大なデータセット（長大なレポートやコードベースなど）を一度に読み込み分析できる強みがあります。

- **OpenAI o3-mini / o1:** OpenAIのoシリーズ（推論特化モデル）もコンテキスト長が大きく、**最大200,000トークン**まで入力可能です。これは従来のGPT-4（8k〜32k、一部128k拡張版）より遥かに長く、長大なコードやドキュメントでも切り分けず一括でモデルに与えられることを意味します。実際、o1およびo3-miniはいずれも200kトークン対応であり、軽量版のo1-miniでさえ128kをサポートしています。この大きなウィンドウにより、モデルが内部で長い思考チェーンを展開しても文脈から外れにくくなり、またユーザ側も大量の関連資料を与えて高度な質問をすることが可能です。

- **Claude 3.7:** Claude 2（Claude 100k）で話題となった長文対応はさらに拡張され、Claude 3.5/3.7 Sonnetでは**最大200,000トークン**のコンテキストを扱えます。Anthropicは長文入力をいち早く商用化した企業ですが、Claude 3.7はその伝統を踏襲し、他のモデルと同様に20万トークン級のやり取りを可能にしています。これにより、小説や技術文書を丸ごと分析したり、大量の会話ログを一括処理したりといった用途にも耐えられます。もっとも、コンテキスト長が増えるほど**応答時間やコストも増大**するため、実運用では必要に応じて使い分ける形になります（これについては後述のリスク節で触れます）。

- *(参考)* **他モデルとの比較:** なお、OpenAI GPT-4.5やGPT-4oといった派生モデル、MetaのLlama 3などは128k程度が上限であり、現在主流の長文対応モデルは概ね128kまたは200kが上限です。Geminiの100万トークンは突出しています。また、これを超える特殊モデルとしてMagic.devのLTM-2（最大1億トークン）など研究的な存在もありますが、一般利用可能な範囲では上記のコンテキスト長が現状最大級と言えます。

## Model Context Protocol (MCP) の概要

**Model Context Protocol (MCP)**は、LLMを外部データやツールと接続するための新しいオープン標準プロトコルです。2024年11月にAnthropic社から提唱・オープンソース化され、LLMを企業内のデータソースやアプリケーションと**統一的かつセキュアに連携**させることを目指しています。

- **背景と目的:** 従来、チャットGPTなどのLLMに業務データを参照させたり、ツールを使わせたりするには、個別にプラグインやスクリプトを開発する必要がありました。データソースごとに異なる実装を行うのは非効率で保守も困難です。MCPはそれを解決するため、「**AIのUSB-Cポート**」とも言える統一規格を提供します。この規格に沿えば、新しいデータ源が増えても同じ手順でLLMと繋ぎ込めるため、AIシステムを既存システムのサイロ（孤立環境）から解放しやすくなります。

- **技術仕様の概略:** MCPは**クライアント-サーバアーキテクチャ**に基づいています。具体的には、LLMを使ったホストアプリケーションが「MCPクライアント」となり、各種データ源ごとに用意された「MCPサーバ」と通信します。MCPサーバは軽量なサービスで、特定の機能やデータ（ファイルシステム、データベース、外部APIなど）を標準化された手順で提供します。LLM側（ホスト）は必要に応じてサーバにリクエストを送り、返ってきたデータをプロンプトに組み込んだり、ツール実行結果として利用したりします。この仕組みにより、LLMはあたかも自分で外部にアクセスしているかのように最新の情報や社内データを参照できます。

- **双方向かつセキュアな設計:** MCPの特徴は「**双方向**」かつ「**セキュア**」であることです。双方向とは、単にLLMがデータを読むだけでなく、外部システム側への書き込みやアクションも標準APIで行えることを意味します（もちろん許可された範囲で）。セキュリティ面では、各MCPサーバはホスト側と**標準化プロトコル**で通信するため、認証・認可や通信内容の構造化が明確です。また、企業は自社インフラ内にサーバをホスティングし、LLM（クラウドサービスであっても）との間で必要なデータだけをやり取りする構成が可能です。AnthropicはMCPについて「自社インフラ内でデータを安全に取り扱うためのベストプラクティスを提供する」と述べており、機密データを扱う統合でもセキュリティガイドラインに沿って実装できるよう配慮されています。

- **提供物:** MCPはオープン標準であり、以下の要素が公開されています。
  - **プロトコル仕様書とSDK** – 開発者向けに詳細な仕様と各種言語用のSDKが提供され、独自のMCPサーバ/クライアントを実装可能。
  - **Claude DesktopにおけるローカルMCP対応** – Anthropicが提供するデスクトップ版Claudeアプリでは、設定ファイルに接続情報を追記するだけでローカルMCPサーバに接続できる機能が組み込まれています。非エンジニアでも簡単に使える実装例として「MCP統合のゴールドスタンダード」とされています。
  - **MCPサーバのOSS実装集** – 代表的なデータソースに対するサーバ実装がオープンソースで公開されています。現在利用できるサーバには、Google Driveファイルシステム、Slackメッセージ履歴, GitHubリポジトリ, Git（汎用）, PostgreSQLデータベース, Puppeteer（Web閲覧用ヘッドレスブラウザ）などがあります。これらを組み合わせることで、ファイル取得や社内チャット検索、コードリポジトリからのコード取り込み、Webサイトからの情報収集等がすぐに試せます。

- **採用事例と展望:** MCPは発表直後から複数の企業・プロジェクトで採用が進んでいます。例えば決済企業のBlock社（旧Square社）や通信プラットフォームのApollo社では自社システムにMCPを統合し始めており、Zed（エディタ開発会社）、Replit、Codeium、Sourcegraphといった開発者向けツール企業も自社プラットフォームへのMCP対応を進めています。これにより、これらのIDEやコード検索ツール上でAIエージェントがコードベースの関連情報を即座に取得し、より高度なコーディング支援を行えるようになります。また、コミュニティでも**Windsurf**や**Cursor**、**Cline**といった独自のMCPクライアント（LLMアプリ）が登場しており、Claude以外のモデルでもMCPを活用する動きが出てきています。OpenAIも自社のエージェント向けSDKでMCPをサポートしており、今後はベンダーを超えて統一的に使われる可能性があります。総じてMCPは、LLMを企業システムへ**つなぎ込むハブ**として今後重要な役割を果たすと期待されています。

## LLMとMCPのメインフレーム移行・モダナイゼーションへの応用

メインフレームのモダナイゼーション（近代化、他プラットフォームへの移行）は、多くの企業が直面する課題です。典型的な課題として、以下のような点が挙げられます。

- **レガシーコードの巨大さと複雑さ:** 何十年も改修を重ねたCOBOLなどのコードが数百万行に及び、システム全体の把握が困難。動いているがブラックボックス化しているプログラムも多い。
- **ドキュメント不足:** 古いシステムでは設計書や仕様書が散逸・未整備であり、現場の担当者の頭の中にしか業務知識がない状況もある。
- **レガシー人材の不足:** COBOL技術者の高齢化・減少により、システム理解や改修ができる人材が不足している。
- **ビジネスフローの不透明:** 現行業務がコードでどう実現されているか全体像を把握しづらく、新システムへの再実装要件を洗い出すのが難しい。
- **技術的負債と互換性:** 新システムへの移行時に、既存機能との互換性を保ちつつ古い非効率な実装を改善する必要があるが、その両立が難しい。
- **テスト工数の膨大さ:** レガシーシステムの振る舞いを保証するには莫大なテストが必要だが、テストケースの網羅も難しい。

これらの課題に対して、最新LLMとMCPの組み合わせは強力なソリューションを提供し得ます。以下に具体的なユースケースと有効性を提案します。

### LLMによるレガシーコードの理解・ドキュメント化

メインフレームのCOBOLコードのような長大なソースコードを、LLMを用いて解析・要約することが可能です。前述の通りGemini 2.5やo3系モデルは数十万トークン規模のコンテキストを扱えるため、**大きなCOBOLプログラムを丸ごと入力しその処理概要を日本語で説明させる**といったこともできます。実際、AIを使って「アプリケーションがどのように動作し、どんなデータと関係しているかを分析・ドキュメント化する」ことがモダナイゼーションに有用だと指摘されています。LLMは自然言語でコードの意図を解説したり、モジュール間の依存関係を列挙したりすることができ、**失われた設計書の代替**を生成できます。

例えば、あるCOBOLプログラムについてLLMに「このプログラムの目的と主要な処理手順を要約せよ」とプロンプトを与えると、該当コードを読み込み「このプログラムは○○業務のバッチ処理であり、ファイルAを読み込んで集計し、結果をDBに書き出す。主なステップは...(以下略)」といった要約を返すでしょう。さらに詳細な質問（「○○というエラーコードが出る条件は？」等）にも、コードロジックを追って回答できます。これはシニアエンジニアにコードをインタビューするようなもので、**LLMが仮想的なレガシーシステムの専門家**として機能するイメージです。

MCPを活用すれば、コード解析の効率は一段と上がります。LLMエージェントがMCP経由で社内のソースコード管理システム（Gitなど）から必要なプログラムを自動取得し、内容を解析してドキュメントを生成するといったことが可能になります。例えば「在庫更新バッチの仕様書を生成して」というユーザ指示に対し、エージェントがMCP連携したGitサーバから`INVUPDT.cbl`を取得し、LLMで解析し、その結果をWikiフォーマットでまとめてMCP経由でドキュメント管理システムに登録する、といった自動化が考えられます。これにより**コードからドキュメントへの変換作業**が大幅に効率化され、人手不足の中でも知識の形式知化が進みます。

### コード変換（COBOL→モダン言語）とリファクタリング

レガシーコードを現代的な言語・アーキテクチャに変換する作業にもLLMが活用できます。例えばIBMは、自社のLLM（watsonx.ai）を訓練してCOBOLコードを読み取りJavaに書き換える支援を行う「Code Assistant for Z」を発表しました。LLMはCOBOLの構造を解析し、同等のビジネスロジックを持つJavaコードを生成します。さらに、変換後のコードの検証（バリデーション）やリファクタリングの提案も行い、開発者が効率よくレガシー資産をモダン化できるようにします。

このように**COBOLからJava等への自動変換**はLLMの得意分野の一つです。大規模言語モデルはプログラミング言語間の対応を学習しており、ある程度の規模までは一貫性のある翻訳が可能です。完全自動化は難しいにせよ、**全体の7～8割のコード変換をLLMが下訳**し、残りを人間が修正・最適化するという形なら大幅な工数削減が期待できます。特に単純なCRUD処理や計算ロジックなどは自動変換精度が高く、開発者は細かな構文修正よりも**設計上の改良**に注力できます。

LLMを使った変換の際も、MCPがあると便利です。例えばエージェントがMCP経由でコンパイラやテスト実行環境と連携し、変換したJavaコードを即座にコンパイル・実行してみて、COBOL版と出力が一致するか検証する、といった自動チェックが可能です。AnthropicのClaudeなどは外部ツールを組み合わせた**エージェント的なコード改善**に注力しており、MCPで統一された環境があれば「コード取得 → 変換 → テスト → レビュー」を一連のフローとしてAIが回すことも見えてきます。現に「AIがコードを解析・リファクタリングし、変換結果を検証する」という一連のプロセスはモダナイゼーションで有効であり、これを自動化する基盤としてMCP+LLMは有望です。

### 業務フローの可視化・ビジネスルール抽出

モダナイゼーションでは、現行システムの**業務フロー**や**ビジネスルール**を洗い出して新システムに再定義する作業が不可欠です。LLMは自然言語による要件定義やルール記述の生成も得意なため、コードからビジネスルールを抽出・整理する支援が期待できます。

具体的には、複数のプログラムやモジュールにまたがる**ビジネスロジックの一貫性**をLLMでチェックし、人間にとって理解しやすい形で提示することが可能です。例えば「受注から出荷までの処理フローを説明せよ」とLLMに尋ねると、関連する複数のCOBOLプログラム（受注入力、在庫引当、出荷指示、請求書発行など）の役割をまとめ、どのデータがどう受け渡されて最終的に出荷が完了するかをストーリー仕立てで説明できるでしょう。これは従来であれば担当者の経験に頼っていた部分ですが、LLMはコードとシステム設定からその因果関係を推測してくれます。

また、条件分岐の多いビジネスルール（例：「顧客タイプがXかつ注文金額がY以上の場合は特別割引を適用」など）について、LLMが**条件網羅表**や**決定木**の形で出力することも考えられます。コード中の`IF/ELSE`ロジックを解析し、「入力パターンと対応処理」の対応表に整理することで、人間が読み解きやすいビジネスルール集が得られます。こうした「プログラムが何をしているかの記述」はAIが得意とするところであり、後継システムの設計資料として非常に有用です。

MCPの活用により、LLMエージェントは業務フロー可視化のために様々なデータにアクセスできます。例えば、MCPサーバ経由で**ジョブスケジューラ**からバッチ実行順序を取得したり、**データベース**からテーブル定義（どのテーブルにどんなデータが溜まっているか）を取得したりできます。LLMはそれらの情報を総合して、「日次バッチ処理はまずジョブA（在庫更新）、次にジョブB（売上集計）が走り、それぞれテーブルT_INVとT_SALESを更新する」といった**システム全体のフロー図**を描けるかもしれません。現在は人手で行っているシステム構成の把握・ドキュメント化を、LLMが自動支援する未来像が見えてきます。

### テストケース生成と自動検証

モダナイゼーションの過程で避けて通れないのが**回帰テスト**です。現行システムと新システムの挙動を照合し、業務結果が一致することを保証しなければなりません。LLMはこのテスト工程にも多方面で寄与できます。

まず、LLMは**網羅的なテストケースの生成**を支援します。レガシーコードを読んで、どのような入力パターンが重要か、エッジケースは何か、といった洞察を与えてくれます。例えば「このプログラムで考慮すべき異常系シナリオは？」と問えば、「在庫数量がマイナスになるケース」「ファイルが存在しないケース」等、見落としがちなシナリオを提案してくれるでしょう。これは長年そのシステムを知るエンジニアの知見に近く、AIが**テストエンジニアの補佐**を務める形です。

さらに、LLM自体に**テスト実行と結果確認**をさせることも可能です。例えば新旧システムに同じ入力データを与え、それぞれの出力ログをMCP経由で取得し、LLMに差異を説明させることができます。LLMは「旧システムでは○○という値になっているが、新システムでは△△になっている。おそらく○○の計算式に違いがある」といった分析を行い、人間にフィードバックできます。これにより不一致の原因調査が迅速化し、バグ修正サイクルが短縮します。

加えて、テストデータの生成にもLLMは有用です。例えば本番データに似たダミーデータを大量に作ったり、特定の条件を再現する入力データを作成したりできます。MCPで社内のデータベースに接続し、実データの傾向を掴んだ上でLLMが類似分布の架空データを生成するといったことも可能でしょう。これらにより、テストに必要な下準備も自動化でき、移行検証の確実性が増します。

### AIアシスタントによる開発効率化と知識継承

LLMはモダナイゼーションにおいて**人材不足を補完するAIアシスタント**としても機能します。メインフレーム技術者の減少という課題に対し、AIは新人エンジニアの強力な相棒となり得ます。例えば、開発者が「このCOBOLコードの意味が分からない」といった際、LLMが即座に解説し疑問に答えてくれれば、生産性が向上します。Kyndrylも「ジェネレーティブAIをまず人間のアシスタントとして使い、人が結果を検証する形でスキル不足を補う」のが効果的だと述べています。AIが定型的・下調べ的な作業を肩代わりし、人間は検証と意思決定に集中することで、熟練者の負担を減らしつつ新人の育成にもつながります。

MCPと組み合わせれば、このAIアシスタントはさらに有用になります。開発者との対話において、AIが必要な情報をMCP経由で社内から引き出しながら回答できれば、まるで社内Wiki＋ベテラン社員の知識を合わせ持つ相談役となります。例えば「〇〇というエラーが昨夜のバッチで出たが原因は？」と質問すれば、AIがログ管理システムから該当エラーのスタックトレースを取得（MCP経由）し、さらにソースコード上でエラーコードを検索して原因個所を特定、「モジュールXのデータ変換でNULL値が発生したためです」と回答する、といった高度な応答も理論上可能です。これは将来的な展望ですが、MCP対応が各種システムに広がれば**AIの社内システム横断力**が飛躍的に高まり、メインフレームからクラウドまで一貫したサポートを提供できるでしょう。

## LLM活用のリスクとMCP導入時の注意点

LLMとMCPをモダナイゼーションに活用する際には、いくつか留意すべきリスクや制限事項も存在します。

- **回答の正確性・信頼性:** LLMは流暢な回答を生成しますが、内容が常に正しいとは限りません。いわゆる**幻覚（hallucination）**により、コードの説明や変換結果に誤りが混入する可能性があります。例えばCOBOLコードの微妙な挙動を誤解して説明したり、コンパイルは通るが要件を満たさないJavaコードを生成したりするケースです。従って、LLMの出力は**必ず人間が検証するプロセス**を組み込む必要があります。AIはあくまで提案者・補助者であり、最終的な責任を持つのは人間です。この点を怠ると、不備のあるドキュメントやバグを含んだプログラムを元にシステムを再構築してしまうリスクがあります。

- **機密データ・プライバシー:** レガシーシステムのコードやデータには機密情報が含まれる場合があります。クラウド上のLLMサービス（OpenAIやAnthropic等）にそのまま投入すると、情報漏洩や規約違反となる可能性があります。対策として、**オンプレミス版LLM**（企業内サーバで動かすモデル）を利用したり、入力データをマスキング・要約してから送信したりする工夫が必要です。MCPの利点の一つは「データを自社インフラ内に留めたままAIと連携できる」点にあります。例えば社内のMCPサーバがデータベースにアクセスし、その要点だけをLLMに渡すようにすれば、生データを直接外部に出さずに済みます。ただしMCP自体のセキュリティ設定も重要です。各サーバに適切な認可を設計し、LLMが不必要なデータにアクセスしないよう制御する必要があります。また、MCP通信ログを監査することで、AIがどのデータを参照したか履歴を残すことも考慮すべきです。

- **コストとパフォーマンス:** 大容量コンテキストを活用できるLLMですが、その分トークン課金費用も増大します。20万トークンの入力はAPI利用料が高額になり得ますし、処理時間も長くなります。モダナイゼーションのように解析対象が膨大な場合、**すべてをLLM任せにするとコストが膨れ上がる**懸念があります。したがって、現実的には人間が要所を見極め、AIを使う部分とそうでない部分を切り分けることが重要です。例えば「まず要約だけさせて重要箇所を絞り込み、その部分の詳細変換をAIに任せる」「頻出する似た処理は1つだけAIに検討させて他はテンプレート適用で済ませる」といった工夫です。また、推論精度とコストのバランスを調整するために、o3-miniの**推論努力レベル**を使い分けたり、必要に応じて小型モデル（社内にある小規模LLMなど）と大規模モデルを使い分けたりする戦略も有効でしょう。

- **モデルの専門性:** 汎用LLMは膨大な一般知識を持ちますが、メインフレーム特有の知識（例えばCOBOLのレガシー機能、JCLの書式、CICSトランザクション処理など）については十分にカバーしていない可能性があります。実際、メインフレーム専用に調整したモデル（XMainframeなど、COBOL知識に特化したLLM）が提案されており、この領域での特化も進んでいます。従って、生成された回答やコードが業務的に妥当かどうか、メインフレームの文脈で意味が通じるかは、ドメイン知識を持つ人間が確認する必要があります。また場合によっては、LLMに追加学習させる（自社のCOBOLコードを学習データとしてファインチューニングする、あるいは社内用プロンプト集を用意する）ことで精度向上を図ることも検討すべきです。

- **MCP導入の手間と課題:** MCPは統合を容易にしますが、導入初期には設定や社内システム側の対応が必要です。各種コネクタ（MCPサーバ）を立ち上げ、社内のどのデータにどうアクセスさせるかを決めるには、IT部門の連携が欠かせません。また新しいプロトコルゆえにバグや仕様変更への対応も追随していく必要があります。現状ではAnthropicのClaude Desktopがスムーズに使えるものの、他の環境で利用する場合はSDKを用いた実装が必要になることもあります。とはいえ、長期的には一度整備すれば再利用が効きますし、**一対多の統合を一元化できるメリット**は手間を上回る価値があります。導入に際しては、小規模なPoC（例えばGitリポジトリ閲覧サーバとClaudeを繋いでコード要約を試す等）から始め、徐々に接続先を増やしていくのが安全策です。運用上は、MCPサーバ自体の負荷管理やバージョン管理、セキュリティパッチ適用にも注意を払い、基盤の一部として位置付ける必要があります。

- **倫理・ガバナンス:** AIに高い自由度を与えると、意図しない操作（例えば誤った判断でデータを削除する等）をするリスクもあります。MCPではAIが外部ツールを実行できるため、その権限をどう制限するかガバナンスが重要です。基本的には読み取り専用のコネクタから試し、書き込み系（チケット発行やコード自動コミット等）は十分検証してから有効化するのが望ましいでしょう。また、AIの提案に最終決定を下すのは人間であるという原則を組織で共有し、結果への責任の所在を明確にしておくことも大切です。幸い、現在のAIツールは「人が確認・修正する前提」で使うことが推奨されており、過信せず適切に役割分担すれば有益なパートナーとなります。

以上、最新LLMとMCPの技術概要、およびそれらを活用したメインフレーム近代化への応用例と注意点を述べました。LLMは急速に進歩しており、従来人手に頼っていたレガシー資産の理解・変換作業を大幅に効率化できる可能性を秘めています。同時に、MCPのような標準化されたインターフェースを用いることで、安全かつスケーラブルにAIを既存システムへ組み込める道筋が見えてきました。適切なガバナンスの下でこれら新技術を活用すれば、レガシー資産のモダナイズを加速し、ビジネスの変革につなげることができるでしょう。

## 参考資料

- 【1】 *DataCamp* – Mehul Gupta, “Gemini 2.5 Pro: Features, Tests, Access, Benchmarks & More” (Gemini 2.5によるコード生成の速度実験)  
- 【4】 *DataCamp* – Mehul Gupta, “Claude 3.7 Sonnet: Features, Access, Benchmarks & More” (Claude 3.7 Sonnetのベンチマーク結果)  
- 【8】 *Simon Willison’s Weblog* – Simon Willison, “OpenAI o3-mini, now available in LLM” (OpenAI o3-miniの公開と性能・価格)  
- 【9】 *TechTarget* – Alan R. Earls, “OpenAI o1 explained: Everything you need to know” (OpenAI o1モデルの概要とProモード)  
- 【13】 *Codingscape Blog* – Cole, “LLMs with largest context windows” (主要LLMのコンテキストウィンドウ比較)  
- 【14】 *Google Keyword Blog* – Google DeepMind Team, “Gemini 2.5: Our most intelligent AI model” (Gemini 2.5 Proの発表; LMArena1位の言及)  
- 【16】 *Google Keyword Blog* – 同上 (Gemini 2.5 Proのコンテキスト長1Mトークン対応)  
- 【17】 *DataCamp* – Mehul Gupta, 同記事 (Gemini 2.5と他モデルのベンチマーク比較データ)  
- 【22】 *DataCamp* – Vinitha Subhash, “Claude 3.7 Sonnet API: A Guide” (Claude 3.7 SonnetのAPI提供状況)  
- 【24】 *Anthropic News* – Anthropic, “Introducing the Model Context Protocol” (MCP発表: 背景と導入事例)  
- 【25】 *ModelContextProtocol.io* – “Introduction – Model Context Protocol” (MCPの概要と利点: USB-Cの比喩, セキュリティ)  
- 【26】 *Medium (Predict)* – Ashraff Hathibelagal, “Understanding Model Context Protocol (MCP)” (Claude DesktopでのMCP統合, MCPクライアントの例)  
- 【30】 *CIO Dive* – Matt Ashare, “IBM trains its LLM to read, rewrite COBOL apps” (IBMのWatsonx Code AssistantによるCOBOL→Java変換)  
- 【31】 *Kyndryl Whitepaper* – *The future of mainframe modernization with AI* (AIが支援するモダナイゼーション例と人材不足への対処)  
- 【32】 *arXiv* – Anh T.V. Dau et al., “XMainframe: A Large Language Model for Mainframe Modernization” (メインフレーム・COBOL知識に特化したLLMの研究)  
- 【33】 *OpenAI Agents SDK Docs* – OpenAI, “Model context protocol (MCP)” (OpenAIエージェントSDKによるMCPサポート)