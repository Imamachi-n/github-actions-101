# github-actions-101

`GitHub Actions` コトハジメ。

## Hello World

とにもかくにも、Hello World。

```yml
on:
  push: # Push時をトリガーとする
    branches: [master] # 対象のBranch

name: github-actions-101 # Actions名

jobs:
  first-github-actions-job: # ジョブの ID（jobs 内でユニークであればよい）
    name: "日本語がかけるかどうかチェック" # ジョブ名
    runs-on: ubuntu-latest # ジョブが実行される仮想環境（VM）
    steps: # ジョブ内で実行されるワークフロー
      - run: echo "Hello, World!" # 実行するコマンド
```

## Docker コンテナ上で実行する

使用する Docker イメージの指定（`container.image`）だけでなく、Docker を動作させる VM（`runs-on`）も設定する必要あり。

```yml
on:
  push:
    branches: [master]

name: github-actions-101

jobs:
  docker-github-actions-job:
    name: "Dockerイメージを使用できるかどうかチェック"
    runs-on: ubuntu-latest # ジョブが実行される仮想環境（VM）
    container:
      image: circleci/node:10.15.3 # 使用するDockerイメージ
    steps:
      - name: Display versions
        run: |
          node -v
          npm -v
          yarn --version
```

## VM 上で特定のバージョンの Node.js を動作させる

`actions/setup-node@v1` を使い、特定の Node.js をセットアップすることもできる。また、このとき、マトリクスビルドの書き方を使って、複数の Node.js のバージョンで動作させることも可能。上記の Docker イメージを使ったやり方よりも高速。

```yml
on:
  push:
    branches: [master]

name: github-actions-101

jobs:
  vm-github-actions-job:
    name: "VMでNode.jsを使用できるかどうかチェック"
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [10.15.3]
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: Display versions
        run: |
          node -v
          npm -v
          yarn --version
```

### 参考資料

[Software installed on GitHub-hosted runners](https://help.github.com/en/actions/reference/software-installed-on-github-hosted-runners)  
[GitHub Actions - Ubuntu 18.04.4 LTS](https://github.com/actions/virtual-environments/blob/master/images/linux/Ubuntu1804-README.md)
[GitHub Actions - setup-node](https://github.com/actions/setup-node)

## Cron ジョブを実行する

トリガーを `schedule` にすることで、定時実行（Cron ジョブ）もできる。

cron の場合、UTC であることに注意。日本標準時の現在時刻(JST/UTC+0900)なので、日本時間から「-9 時間」する。

```yml
name: Cron test

on:
  schedule:
    - cron: "55 9 * * *" # UTCであることに注意。日本標準時の現在時刻(JST/UTC+0900)なので、日本時間から「-9時間」
```

### 特定の Branch に対して Cron ジョブを実行したい

`jobs.<job_id>.steps.uses` ジョブでステップの一部として実行されるアクションを選択できる。以下の例のように、パブリックリポジトリで定義されているアクションを実行できる。

ここでは、`git checkout development` を実行し、`development` ブランチにチェックアウトする処理を行っている。

```yml
steps:
  - uses: actions/checkout@v2 # git checkoutを使用
    with:
      ref: development # 特定のbranchにチェックアウトできる
```

サンプルの全体像。  
以下の例では、Cron ジョブを `development` ブランチに対して実行している。

```yml
name: Cron test

on:
  schedule:
    - cron: "55 9 * * *" # UTCであることに注意。日本標準時の現在時刻(JST/UTC+0900)なので、日本時間から「-9時間」

jobs:
  first-cron-job:
    name: "Cronが実行できるかどうかチェック"
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [10.15.3]
    steps:
      - uses: actions/checkout@v2 # git checkoutを使用
        with:
          ref: development # 特定のbranchにチェックアウトできる
      - name: Display versions
        run: |
          node -v
          npm -v
          yarn --version
      - name: Check Branch
        run: |
          node cron.js
```

### リファレンス

[jobs.<job_id>.steps.uses](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepsuses)

## jobs.<job_id>.steps.run を複数行で書きたい

```yml
steps:
  run: |
    node -v
    npm -v
    yarn --version
```

### リファレンス

[jobs.<job_id>.steps.run](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepsrun)

## jobs.<job_id>.steps.run の実行に名前をつけたい

```yml
steps:
  name: Check Branch
    run: |
      node cron.js
```

### リファレンス

[jobs.<job_id>.steps.run](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepsrun)

## 参考資料

[GitHub Actions のワークフロー構文](https://help.github.com/ja/actions/reference/workflow-syntax-for-github-actions)  
[hisasann/github-actions-samples](https://github.com/hisasann/github-actions-samples)
