バッチ処理の練習

ディレクトリ構成の元ネタ
----
はい、AWSで汎用バッチ処理システムをモノレポで管理することには積極的なメリットがあります。

主なメリットは、**一貫性の確保、依存関係の管理、そしてCI/CDパイプラインの効率化**です。

-----

### メリット

  * **一貫性:** すべての設定ファイル（AWS CDK/CloudFormation/Terraformのコード）とアプリケーションコードが単一のリポジトリに存在するため、環境やサービス間で設定の基準を統一しやすくなります。例えば、すべてのLambda関数で共通のIAMロールや環境変数を使用する場合、一つの場所で管理・更新できます。

  * **依存関係の可視化:** バッチ処理のワークフローが複雑になるほど、サービス間の依存関係が増えます。モノレポは、Step Functionsの定義、Batchジョブのコンテナイメージ、Lambda関数などがすべて同じリポジトリにあるため、変更が他のコンポーネントに与える影響を簡単に追跡できます。

  * **CI/CDの効率化:** 関連するすべての変更（コード、設定ファイル、Dockerfileなど）を単一のプルリクエストでレビューし、単一のパイプラインでデプロイできます。これにより、デプロイのタイミングやバージョン管理の不整合を防ぐことができます。例えば、Batchジョブのコンテナコードを変更し、それを使用するStep Functionsの定義も同時に更新するといったことが容易になります。

-----

### ディレクトリ構成のサンプル

この構成は、一般的なAWSアプリケーションのモノレポ構造をベースにしています。`apps`ディレクトリにアプリケーションの実行コードを、`infra`ディレクトリにインフラストラクチャの定義を分離することで、コードと設定の役割を明確にしています。

```
.
├── apps/
│   ├── batch-jobs/
│   │   ├── job-a/
│   │   │   ├── src/
│   │   │   │   └── main.py
│   │   │   ├── Dockerfile
│   │   │   └── requirements.txt
│   │   └── job-b/
│   │       ├── src/
│   │       │   └── worker.py
│   │       ├── Dockerfile
│   │       └── requirements.txt
│   └── lambda-functions/
│       ├── pre-processing/
│       │   ├── src/
│       │   │   └── index.py
│       │   └── requirements.txt
│       └── post-processing/
│           ├── src/
│           │   └── handler.py
│           └── requirements.txt
│
├── infra/
│   ├── cdk/
│   │   ├── bin/
│   │   │   └── my-batch-system.ts
│   │   └── lib/
│   │       ├── batch-stack.ts
│   │       ├── lambda-stack.ts
│   │       └── step-functions-stack.ts
│   └── templates/
│       ├── batch-job-definition.json
│       ├── step-functions-workflow.asl.json
│       └── ecr-repositories.json
│
├── .gitignore
├── package.json (CDKの場合)
└── README.md
```

#### 各ディレクトリの説明

  * **`apps/`**: バッチジョブやLambda関数などの実行コードを格納します。
      * **`batch-jobs/`**: 各バッチジョブのコンテナイメージをビルドするためのコードやDockerfileを格納します。
      * **`lambda-functions/`**: 各Lambda関数のソースコードと依存関係を格納します。
  * **`infra/`**: インフラストラクチャの設定ファイルを格納します。
      * **`cdk/`**: AWS CDKを使用してインフラをコード化する場合のディレクトリです。各スタック（Batch、Lambda、Step Functionsなど）を個別のファイルで定義し、依存関係を管理します。
      * **`templates/`**: CloudFormationやStep Functionsのテンプレートなど、宣言的な設定ファイルを格納します。