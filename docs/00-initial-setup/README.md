# モジュール0：初期セットアップ

このモジュールでは、サーバーレスアプリケーションをデプロイします。これは、以後のワークショップでセキュリティを向上させていくために利用します。Wild Rydesのパートナー企業がブランドソックスやケープなどのユニコーンカスタマイズを送信して会社を宣伝できるように、REST APIエンドポイントを作成します。下記は、展開するアーキテクチャの概要です。 

![base-architecture](images/00-base-architecture.png)



## 前提条件

### AWS アカウント
このワークショップでは、Cloud9、Cognito、API Gateway、Lambda、RDS、WAF、Secrets Manager、IAM ポリシー、IAMロールを利用します。このAWSリソースを作成、管理するためAWSアカウントとアクセス権が必要です。

このワークショップの手順は、一度に1人の参加者のみが特定のAWSアカウントを使用していることを前提としています。別の参加者とアカウントを共有しようとすると、特定のリソースで名前の競合が発生する場合があります。個別のリージョンを使用することでこの問題を回避できますが、この作業を行うための手順は記載されていません。 

本番用のAWS環境、アカウントを使用しないでください。代わりに、必要なサービスへの**フルアクセス**できる権限をもった**検証アカウント**を使用することをお勧めします。 




### リージョンの選択
このワークショップは全体を通して１つのリージョンを使用します。このワークショップは、北米の2つのリージョンとヨーロッパの1つのリージョンをサポートしています。以下の起動スタックリンクから1つのリージョンを選択し、以後そのリージョンを使用し続けてください。



## モジュール-0A：このワークショップに必要なVPCおよびCloud9環境を作成します


下記を作成します。

* Cloud9環境：IDEとして利用します（統合開発環境）
* RDS Aurora MySQL ：サーバーレスアプリケーションのバックエンドデータベースとして利用します

以下の手順に従って、セットアップリソース（VPC、Cloud9環境など）を作成します。 

1. いずれかのリージョンを選択して、[ **Launch Stack** ] リンクをクリックしてください。AuroraやCloud9などのサービスを作成します、

	💡 **リンクをクリックするときは、⌘（mac）またはCtrl（Windows）を押したままにして、新しいタブで開くようにします** 💡

	リージョン| リージョンコード | リンク 
	------|------|-------
	EU (アイルランド) | <span style="font-family:'Courier';">eu-west-1</span> | [![Launch setup resource in eu-west-1](images/cfn-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new?stackName=Secure-Serverless&templateURL=https://s3.amazonaws.com/wildrydes-us-east-1/Security/init-template.yml)
	US West (オレゴン) | <span style="font-family:'Courier';">us-west-2</span> | [![Launch setup resource in us-west-2](images/cfn-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=Secure-Serverless&templateURL=https://s3.amazonaws.com/wildrydes-us-east-1/Security/init-template.yml)
	US East (バージニア北部) | <span style="font-family:'Courier';">us-east-1</span> | [![Launch setup resource in us-east-1](images/cfn-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=Secure-Serverless&templateURL=https://s3.amazonaws.com/wildrydes-us-east-1/Security/init-template.yml)

1. **次へ**をクリックします 

1. [ **ステップ2：スタックの詳細を指定** ]ページで： 
	
	* スタックの名前  ***`Secure-Serverless`***
	* データベースのパスワード ***`Corp123!`***
	を入力して [ **次へ** ]をクリックします 
	
		> 注：別のパスワードも指定できます。ただし、パスワードは8文字以上である必要があります。また、後で`src/app/dbUtils.js`ファイルで指定したパスワードを使用するようにmodule-0DのLambda関数のコードを変更する必要があります。 
	
1. [ **ステップ3：スタックオプションの 設定**]ページで、そのまま[ **次へ** ]をクリックします

1. 構成を確認し、[ **スタックの作成** ]をクリックします

1. CloudFormationスタックの作成が完了するのを待っている間に、ラップトップに**PostMan**がインストールされているか確認してください。そうでない場合は、[https](https://www.getpostman.com/):[//www.getpostman.com](https://www.getpostman.com/)からダウンロードしてインストールします。後で使用します。 

1. スタックの作成には数分かかります。画面左上の[ **スタック** ]を選択して、スタック一覧ページに移動し、スタックが*CREATE_COMPLETEの*ステータスが表示されるまで待ちます。更新アイコンを定期的にクリックして、進捗状況を確認してください。 
	
	>  注：CloudFormationはネストされたスタックをデプロイして、Cloud9リソースを起動します。「aws-cloud9-Secure-Serverless-」というプレフィックスが付いたテンプレートは無視しても問題ありません。 
1. CloudFormationの作成が完了したら、[ **Secure-Serverless** ]スタックの[ **出力** ]タブに移動し、**AuroraEndpoint**をテキストエディターにコピーします。次のステップでAuroraデータベースに接続するために必要になります。（**このワークショップの作業中、このタブを開いたままにすることもお勧めです**）

	![cloudformation output](images/0a-cloudformation-output-with-aurora-endpoint.png)

今実行したCloudFormationスタックは、以下のリソースも作成しています。 

* 4つのサブネットを持つ**VPC**　（プライベートサブネット２つ、パブリックサブネット２つで構成）
* **Cloud9** ワークショップで作業を行う環境
* **MySQL Aurora RDSデータベース**（プライマリDBインスタンスが2つのプライベートサブネットのいずれかに存在）

![initial resources diagram](images/0C-diagram-with-aurora.png)

 さらに、以下のリソースも作成しています。

* **S3バケット**: Lambda関数コードのパッケージ化とアップロードのために後で使用
* **セキュリティグループ**: Lambda関数によって使用

## モジュール-0B：データベースを準備する

いくつかのテーブルを作成し、Auroraデータベースに初期値を挿入する必要があります。セキュリティのベストプラクティスに従って、プライベートサブネットでAuroraデータベースを開始したため、データベースはインターネットから直接接続できません。 

Cloud9インスタンスとAuroraデータベースは同じVPCにあるため、Cloud9インスタンスからデータベースを管理できます（データベースのセキュリティグループは、接続を許可するように構成されています）。 

まず、**Cloud9**環境に移動します。 

1. [Cloud9コンソール](https://console.aws.amazon.com/cloud9/home)移動します（AWSコンソール上部のナビゲーションバーで[ **サービス** ]をクリックして、`cloud9`と検索して移動することもできます） 

1.  ***Your environments***をクリックします（左側のサイドバーを展開する必要がある場合があります） 
	<img src="images/0B-cloud9-environments.png" width="80%" />

	
	
1. *Secure-Serverless-Cloud9*環境の***Open IDE*** をクリックします
	
	![Cloud9 Open IDE](images/0C-open-ide.png)

	cloud9を開けない場合は、以下を使用していることを確認してください。 
	
	* **Chrome**または**Firefox**のブラウザ
	* サードパーティのCookieが有効になっていることを確認　[**シューティングガイド**](https://docs.aws.amazon.com/cloud9/latest/user-guide/troubleshooting.html#troubleshooting-env-loading)

1. 次のように、統合開発環境（IDE）環境が表示されます。AWS Cloud9は、ブラウザのみでコードを記述、実行、デバッグできるクラウドベースのIDEです。ローカルコンピューターで行うのと同じように、ターミナルウィンドウでシェルコマンドを実行できます。 
	![](images/0B-cloud9-start.png)

	多くの作業で使用するため、このワークショップを通してAWS Cloud9 IDEをタブで開いたままにしてください。 

1. このワークショップのコンテンツを取得します。Cloud9ターミナルウィンドウ（画面下部）で、次のコマンドを実行して、このリポジトリのクローンを作成します。 

	`git clone https://github.com/aws-samples/aws-serverless-security-workshop.git`





次にデータベースを準備します。 

1.  リポジトリのフォルダーに移動します 。
	```
	cd aws-serverless-security-workshop/
	```

1. 次のコマンドを使用してクラスターに接続します。エンドポイントを前にコピーしたものに置き換えます。（この情報はまだ残しておいてください。後で必要になります） 

   `mysql -h <YOUR-AURORA-SERVERLESS-ENDPOINT> -u admin -p`

     パスワードを求められるので*`Corp123!`*（前に指定したもの）を入力します

1. mysqlコマンドプロンプト（`mysql> `）内で、次のコマンドを入力します。 

     `source src/init/db/queries.sql`

     次のような出力が表示されるはずです。	 
	
	``` bash
	mysql> source src/init/db/queries.sql
	Query OK, 1 row affected (0.01 sec)
	
	Database changed
	Query OK, 0 rows affected (0.02 sec)
	
	Query OK, 0 rows affected (0.02 sec)
	
	Query OK, 0 rows affected (0.02 sec)
	
	Query OK, 0 rows affected (0.02 sec)
	
	Query OK, 0 rows affected (0.02 sec)
	
	Query OK, 0 rows affected (0.03 sec)
	Query OK, 1 row affected, 1 warning (0.00 sec)

	Query OK, 2 rows affected (0.01 sec)
	Records: 2  Duplicates: 0  Warnings: 0

	Query OK, 8 rows affected (0.01 sec)
	Records: 8  Duplicates: 0  Warnings: 0

	Query OK, 7 rows affected (0.00 sec)
	Records: 7  Duplicates: 0  Warnings: 0

	Query OK, 4 rows affected (0.00 sec)
	Records: 4  Duplicates: 0  Warnings: 0
	
	mysql> 
	```

1. 次のSQLクエリを実行して、作成されたデータベースのテーブルを調べることができます。 

	```sql 
	SHOW tables;
	```

	このようなものが表示されるはずです 。

	```sql 
	mysql> SHOW tables;
	+---------------------------------+
	| Tables_in_unicorn_customization |
	+---------------------------------+
	| Capes                           |
	| Companies                       |
	| Custom_Unicorns                 |
	| Glasses                         |
	| Horns                           |
	| Socks                           |
	+---------------------------------+
	6 rows in set (0.00 sec)
	```

	次のSQLクエリを実行して、テーブルの内容を調べます。

	``` 
	SELECT * FROM Capes;
	```

	このようなものが表示されるはずです 。

	```
	mysql> SELECT * FROM Capes;
	+----+--------------------+-------+
	| ID | NAME               | PRICE |
	+----+--------------------+-------+
	|  1 | White              |  0.00 |
	|  2 | Rainbow            |  2.00 |
	|  3 | Branded on White   |  3.00 |
	|  4 | Branded on Rainbow |  4.00 |
	+----+--------------------+-------+
	4 rows in set (0.00 sec)
	```

1. 確認が終わったら、`exit`コマンドを使用してmysql接続を切断します。

    

## モジュール0C：サーバーレスアプリケーションのコードを確認

Lambda関数のコードは`src/app`にあります。最初に行う必要があるのは、このフォルダーに移動し、次のコマンドを使用してノードの依存関係をインストールすることです。 

`cd src/app && npm install`
	
この`src/app`フォルダーにはいくつかのファイルがあります:	

- **unicornParts.js**: ユニコーンカスタマイズオプションを一覧表示するLambda関数のメインファイル
- **customizeUnicorn.js**:  ユニコーンカスタマイズ構成の作成/記述/削除を処理するLambda関数のメインファイル
- **dbUtils.js**:  このファイルには、アプリケーションのすべてのデータベース/クエリロジックが含まれています。また、すべての接続情報がプレーンテキストで含まれています（疑わしい！） 

さらに、フォルダーには下記のファイルもあります。いまの時点でこれらを厳密に確認する必要はありません。 

- **httpUtils.js**: アプリケーションからのhttp応答ロジックが含まれています 
- **managePartners.js**: 新しいパートナー企業を登録するためのロジックを処理するLambda関数のメインファイル。これについてはモジュール1で詳しく説明します 
- **package.json**: Nodejsプロジェクトマニフェストのファイル（コードの依存関係のリストを含む） 

コードに加えて、Lambda関数とREST APIの構成は`template.yaml`、**AWS SAM**（Serverless Application Model）テンプレートとして記述されています。 

[AWS SAM](https://github.com/awslabs/serverless-application-model)を利用すると、シンプルでクリーンな構文でサーバーレスアプリケーションを定義できます。`template.yaml`では3つのLambda関数が定義されており、Swaggerテンプレートで定義された一連のREST APIにマッピングされていることがわかります。 

<table>
  <tr>
    <th>Lambda 関数</th>
    <th>Main handler code</th>
    <th>API リソース</th>
    <th>HTTP メソッド</th>
    <th>説明</th>
  </tr>
  <tr>
    <td rowspan="4">UnicornPartsFunction</td>
    <td rowspan="4">unicornParts.js</td>
    <td>/horns</td>
    <td>GET</td>
    <td>List customization options for horns</td>
  </tr>
  <tr>
    <td>/glasses</td>
    <td>GET</td>
    <td>List customization options for glasses</td>
  </tr>
  <tr>
    <td>/socks</td>
    <td>GET</td>
    <td>List customization options for socks</td>
  </tr>
  <tr>
    <td>/capes</td>
    <td>GET</td>
    <td>List customization options for capes</td>
  </tr>
  <tr>
    <td rowspan="4">CustomizeUnicornFunction</td>
    <td rowspan="4">customizeUnicorn.js</td>
    <td>/customizations</td>
    <td>POST</td>
    <td>Create unicorn customization</td>
  </tr>
  <tr>
    <td>/customizations</td>
    <td>GET</td>
    <td>List unicorn customization</td>
  </tr>
  <tr>
    <td>/customizations/{id}</td>
    <td>GET</td>
    <td>Describe a unicorn customization</td>
  </tr>
  <tr>
    <td>/customizations/{id}</td>
    <td>DELETE</td>
    <td>Delete a unicorn customization</td>
  </tr>
  <tr>
    <td>ManagePartnerFunction</td>
    <td>managePartners.js</td>
    <td>/partners</td>
    <td>POST</td>
    <td>Register a new partner company</td>
  </tr>
</table>

## Module-0D：SAM Localを使用してサーバーレスアプリケーションをローカルで実行



コードを確認した後、**src/app/dbUtils.js**の*host*情報をAuroraエンドポイントに置き換えます。そして、ファイルを保存します（Macの場合は⌘+ s、Windowsの場合はCtrl + s、またはファイル->保存） 

<img src="images/0D-db-endpoint-in-code.png" width="70%" />

その後、SAM Localを使用してローカルでAPIをテストします。 

1. **右のパネル**にある**AWS Resource**をクリック

	<img src="images/0D-aws-resource-bar.png" width="80%" />

1. **Local Functions (1)**という名前のフォルダーツリーが表示されます

1. `src`フォルダーの下にある**UnicornPartsFunction**を選択します 

1. 上部パネルのドロップダウンをクリックし、**Run APIGateway Local**を選択します 

	<img src="images/0D-run-apigateway-local.png" width="40%" />

1. 次に、再生アイコンをクリックします。APIをローカルでテストするための新しいパネルが表示されます

1. 表示されたパネルの**Path**パラメータに、`/socks`と表示されているはずです。（表示のない場合は、ユニコーンの部品（例えば`/socks`、`/glasses`、`/capes`、`/horns`など）を選択してしてください。）そして**Run**をクリックします

	> APIを初めてローカルでテストする場合、Dockerがプルダウンされている状態でセットアップされているため、初期化するのに最大1〜2分かかります。 

	レスポンスとして `200 OK`を取得できるはずです
	
	
	スクリーンショットの例:
	
	![Local Queries](images/0E-sam-local-result.png)
	

これは、アプリケーションがCloud9環境内（ローカル）で正常に実行できたことを示しています。これで、サーバーレスアプリケーションをデプロイできます！

## Module-0E：サーバーレスアプリケーションをクラウドにデプロイしてテストする

1. CloudFormationスタックが作成したS3バケットの名前を確認します。

	* CloudFormationコンソールを開いたままの場合は、そのタブに移動します。それ以外の場合は、ブラウザの別のタブを開き、CloudFormationコンソール[https://console.aws.amazon.com/cloudformation/home](https://console.aws.amazon.com/cloudformation/home)に移動し、`Secure-Serverless`スタックを選択します。
	* **出力** タブの**DeploymentS3Bucket**の値をメモしてください

	![CloudFormation output](images/0D-cloudformation-output-w-bucket-highlight.png)
	
2. ターミナルで、bash変数を設定します (REGION変数とBUCKET変数を設定)
	
	```
	REGION=<選択したリージョンコードを入力>
	BUCKET=<DeploymentS3Bucketの値を入力>
	```
	
	※ リージョンコード：EU(アイルランド)  eu-west-1, US West (オレゴン) us-west-2 ,  US East (バージニア北部)  us-east-1 
	
3. `src`フォルダー内にいることを確認します

	```
	cd ~/environment/aws-serverless-security-workshop/src
	```

4. 以下を実行してLambdaコードをパッケージ化し、S3にアップロードし、コードをホストするS3パスを参照するようにCloudFormationテンプレートを更新します
	```
	aws cloudformation package --template-file template.yaml --s3-bucket $BUCKET --output-template packaged.yaml
	```

5. 次のコマンドを使用してサーバーレスAPIをデプロイします。このテンプレートは、CloudFormationスタック（`Secure-Serverless`）からサブネットIDなどの出力を参照していることに注意してください
	```
	aws cloudformation deploy --template-file packaged.yaml --stack-name CustomizeUnicorns --region $REGION --capabilities CAPABILITY_IAM --parameter-overrides InitResourceStack=Secure-Serverless
	```

6. スタックが正常にデプロイされるまで待ちます
	```
	Waiting for changeset to be created..
	Waiting for stack create/update to complete
	Successfully created/updated stack - CustomizeUnicorns
	```

7. CloudFormationスタックの出力から、デプロイしたサーバーレスAPIのベースエンドポイントを確認できます
   

コマンドラインからは下記を入力すると、出力で確認できます。

   ```
   aws cloudformation describe-stacks --region $REGION --stack-name CustomizeUnicorns --query "Stacks[0].Outputs[0].OutputValue"
   ```


![get endpoint secreenshot](images/0E-get-endpoint-output.png)



   または、 [CloudFormation コンソール](https://console.aws.amazon.com/cloudformation/home)に移動して、`CustomizeUnicorns`スタックの[ **出力** ]タブでも確認することができます 



8. ブラウザ（または`curl`）で次のAPI をテストします。先ほど確認したAPIベースエンドポイントにAPIパス（例`/socks`）を追加することを忘れないでください 

   <table>
     <tr>
       <th>API</th>
       <th>HTTP Verb</th> 
       <th>path</th> 
     </tr>
     <tr>
       <td>List customization options and prices for horns</td>
       <td>GET</td> 
       <td>/horns</td>
     </tr>
     <tr>
       <td> List customization options and prices for glasses </td>
       <td>GET </td> 
       <td>/glasses</td>
     </tr>
     <tr>
       <td> List customization options and prices for capes </td>
       <td>GET</td> 
       <td>/capes </td>
     </tr>
     <tr>
       <td>List customization options and prices for socks </td>
       <td> GET </td> 
       <td>/socks </td>
     </tr>
   </table>

   出力例：

   ![test api in browser](images/0E-test-browser.png)



## モジュール-0F：PostmanをセットアップしてAPIをテストする

最後に、[**Postman**](https://www.getpostman.com/)を使用してAPIリクエストをテストします。

1. ラップトップにPostmanをインストールしていない場合は、[https://www.getpostman.com/](https://www.getpostman.com/)からダウンロードしてインストールしてください 

1. 各APIをテストできるPostmanコレクションを用意したので、時間節約のために利用します 

	* postmanの[ **Import** ]ボタンをクリックします 
	* 次に、 **Import from Link** から以下のリンクを指定します:

		`https://raw.githubusercontent.com/aws-samples/aws-serverless-security-workshop/master/src/test-events/Customize_Unicorns.postman_collection.json`
	* **Import**をクリックします
	
		<img src="images/0F-import-postman.png" width="50%" />
	
1. 画面左側に、`Customize_Unicorns` というコレクションが表示されます。
	<img src="images/0F-postman-after-import.png" width="60%" />

1. postmanの環境を作成し、`base_url`変数を設定します。

	1. 画面右上の &#9881; アイコンから (“Manage Environments”) をクリックします
	<img src="images/0F-postman-manage-env.png" width="90%" />
		
	2. **Add** ボタンをクリックして新しい環境を作成します。
	3. 環境名`dev`を入力します
	
	4. `base_url`変数を追加し、 先程作成したAPIベースエンドポイントを値として入力します。
	
	​		スクリーンショットの例
	
	​	   <img src="images/0F-postman-environment.png" width="70%" />
	
	> Postmanの環境と変数の詳細は[managing environments](https://www.getpostman.com/docs/v6/postman/environments_and_globals/manage_environments) で確認できます
	
1. **Add**をクリックして`dev`環境を作成したのち 、**X**をクリックしてManage Environments を終了します。

1. 画面右上のドロップダウンメニューから`dev` を選択します。

	<img src="images/0F-select-dev-env.png" width="90%" />

1. Postmanを使ってAPIをテストする用意ができました。画面左のサイドバーから`Customize_Unicorns` コレクションをクリックし、`List customization options` フォルダを開きます。 フォルダの中のAPIを1つ選択し、**Send** ボタンをクリックしてリクエストを送信し、結果を確認します。

	![Postman Get request](images/0F-postman-test-get.png)


## 次のステップ
デプロイしたサーバーレスアプリケーションの保護を開始します。ワークショップの [トップページ](../../README.md) に戻り、作業するモジュールを選択してください！

