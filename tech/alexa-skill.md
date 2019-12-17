# Alexaに大石泉すきと喋らせる(またの名をAlexa、大石泉すき姉貴化計画)

## 何をするの？

Alexa Skill Kitを用いて、Alexaに話しかけたら返答するSkillを作ります

## Alexaって何

Amazonが発売している、echoというスマートスピーカーに搭載されているAIです。  
Skillというものを使うことで、任意に喋らせることが出来ます

## Skill / Alexa Skill Kitとは

ざっくり以下のような理解だと思います

* Skill
  * Alexaに備わっている、話しかけることで動かすアプリのようなものの総称
* Alexa Skill Kit
  * Skillを作成する為のAPIや、追加するためのサービス

ユーザは以下の種類のSkillを作ることができます。

* カスタムスキル
* スマートホームスキル
* ビデオスキル
* フラッシュブリーフィングスキル
* 音楽スキル

今回は**カスタムスキル**を作成します。  

参考：[スキルの種類について](https://developer.amazon.com/ja-JP/docs/alexa/ask-overviews/understanding-the-different-types-of-skills.html)

## 作成するスキル

**「アレクサ、デレマスのアイドルで誰が好き？」と聞くと「大石泉すき」と答える**  
スキルを作ります。  
  
アイドルマスターシンデレラガールズP界隈では、大石泉Pや「大石泉すき」と口走る人間を「大石泉すき(兄|姉)貴」と呼ぶ風習があります。  
つまり、このスキルが登録されたAlexaは、大石泉すき姉貴になるわけですね。

## カスタムスキルの作り方

参考：[スキル開発の要件](https://developer.amazon.com/ja-JP/docs/alexa/ask-overviews/requirements-to-build-a-skill.html)

### Amazon開発者アカウントを作成する
  
[こちら](https://developer.amazon.com/ja-JP/alexa/alexa-skills-kit)のサイトにアクセスし、**「スキル開発を始める」**ボタンを押します。  
ログイン画面が表示されますが、いつもお買い物に使っているアカウントも使えます。  
ログインすると、「開発者情報」を登録するフォームに飛びますので、規約に同意したら記入しましょう。  
会社の欄は自宅で埋めました。  

### バックエンドリソースをなにで作るか決める

実は、SkillだけではAlexaは何もできません。  
Skillはリクエストを分解し、**バックエンドリソース**と呼ばれるプログラムに値を引き渡します。  
引き渡された値をもとにバックエンドリソースがAlexaに返す値を渡し、それをAlexaに渡します。  
  
バックエンドリソースの持ち方は下記の二通りが選べます。  

* ユーザが定義する
* Alexaがホスティングする(AWS Lambdaを使う)
  * Node.js
  * Python

今回はAlexaがホスティングするPythonを使います。  
AWS Lambdaという技術を使う→AWSを使う→お金が掛かるのでは？と思われるかも知れませんが、  
AWS Lambdaは月100万件分のリクエストは無料枠として提供されています。個人で遊ぶ分にはまあ問題ない数字でしょう  
参考：[AWS Lambda 料金](https://aws.amazon.com/jp/lambda/pricing/)

### Skillを作成する

#### 準備

* [Alexaスキル開発者コンソール](https://developer.amazon.com/alexa/console/ask)にアクセス
* **「スキルの作成」**を押下し、各種情報を埋める
  * スキル名：好きにどうぞ
  * デフォルトの言語：日本語
  * スキルに追加するモデル：カスタム
  * スキルのバックエンドリソースをホスティングする方法：Alexa-Hosted(Python)
* **スキルに追加するテンプレートを選択**する
  * 今回は、Hello Worldスキルとします

#### 構築

構築の前におさらいですが、今回作成するスキルは、
> 「アレクサ、**デレマスのアイドル(1)**で**誰が好き？(2)**」と聞くと「**大石泉すき(3)**」と答える
アプリです。これらを踏まえて、後述の処理を書いていきましょう

##### ビルド

準備が完了するとalexa developer console ページに遷移すると思います。  
alexa developer consoleの上部のタブが「ビルド」になっていることを確認してください。  
このページでは、「ユーザに話しかけられたら反応する言葉」を定義します

* 画面左の「呼び出し名」をクリック
  * スキルの呼び出し名を設定します。（上記のAlexaへの命令文のうち、(1)に当たる部分です）
  * スペース区切りで2単語以上の名前でないとrejectされるため、表記上は「デレマスの アイドル」と入力します
* 画面左の「インテント」欄の下に「HelloWorldIntent」があるので、クリック
  * **サンプル発話**の下に四角いボックスがあるので、そこに「だれがすき」と入力
    * 上記のAlexaへの命令文のうち、(2)に当たる部分です
  * 「だれがすき」以外にも、「大石泉すき」という言葉が導きだされそうな質問文をできるだけ沢山記入してください
  * Intent名ですが、HelloWorldでもなんでもないので、適当な名前に変えても良いと思います。
    * 今回のスキルでは**IzumiSukiIntent**と名前を変更しています。
    * Pythonのコードの編集は忘れないこと。後述します。
* 上記の設定を触ったら、画面上部の「モデルを保存」を押下してください。画面遷移をすると、編集中の未保存の変更はすべて削除されます。
* 上記の設定は、内部ではJSON形式で書き出されています。だいたいこんな感じ。

```JSON
{
    "interactionModel": {
        "languageModel": {
            "invocationName": "デレマスの アイドル",
            "intents": [
                {
                    "name": "IzumiSukiIntent",
                    "slots": [],
                    "samples": [
                        "だれがすき",
                        "すきなアイドル",
                        "すきなアイドルは",
                        "すきなアイドルを教えて",
                        "静岡県出身十五歳のクールアイドルを教えて",
                        "静岡県出身十五歳のクールアイドルといえば",
                        "一番クールなアイドルを教えて",
                        "一番可愛いアイドルを教えて",
                        "一番クールなアイドルは",
                        "おいしいおみずと言えば",
                        "一番可愛いアイドルは"
                    ]
                }
            ],
            "types": []
        }
    }
}
```

##### コードエディタ

**コードエディタ**タブをクリックしてください。AWS Lambdaを用いてホストされているpythonのコードが存在します。  
上述の「大石泉すき」とAlexaが返答する(3)部分を記載していきます。  
このページでは「話しかけられた結果、何をするのか」を定義します

* 左カラムから``lambda_function.py``ファイルをダブルクリックし、編集

```python:lambda_function.py

# Intentに紐づくクラスの定義
class IzumiSukiIntentHandler(AbstractRequestHandler):
    """Handler for 大石泉すき Intent."""
    def can_handle(self, handler_input):
        # type: (HandlerInput) -> bool
        return ask_utils.is_intent_name("IzumiSukiIntent")(handler_input) # 一つ目の引数を、インテントの項で変更したIntent名に変更する

    def handle(self, handler_input):
        # type: (HandlerInput) -> Response
        speak_output = "ooishi izumi suki" # Alexaが実際にしゃべる言葉。"大石泉すき"より"ooishi izumi suki"の方が抑揚が自然だったため、こちらを採用。

        return (
            handler_input.response_builder
                .speak(speak_output)
                # .ask("add a reprompt if you want to keep the session open for the user to respond")
                .response
        )

# （略）

sb = SkillBuilder()

sb.add_request_handler(LaunchRequestHandler())
sb.add_request_handler(IzumiSukiIntentHandler()) # IntentHandlerをSkillBuilderに紐づける
sb.add_request_handler(SessionEndedRequestHandler())
sb.add_request_handler(IntentReflectorHandler()) # make sure IntentReflectorHandler is last so it doesn't override your custom intent handlers

sb.add_exception_handler(CatchAllExceptionHandler())

lambda_handler = sb.lambda_handler()
```

コードが書き終わったあとは、画面右上の**保存**、**デプロイ**ボタンを押下

### 公開(TBC)

ここまで終わった段階で、開発者アカウント(や、開発者アカウントと同じお買い物アカウント)に紐づいているecho端末では、  
既にこのスキルを使えるようになっています。個人利用ならここで終わりです。やったね！  
  
公開する場合、Amazonでの審査が必要となります。今後書けたら書こうと思います。
