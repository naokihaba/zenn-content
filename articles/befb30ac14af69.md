---
title: "React+FireBase初心者の備忘録 < デプロイまで>"
emoji: "📚"
type: "tech"
topics:
  - "firebase"
  - "react"
  - "備忘録"
  - "りあクト"
published: true
published_at: "2021-12-12 23:58"
---


# 概要

りあクト！ Firebaseで始めるサーバーレスReact開発の備忘録です

https://booth.pm/ja/items/1572683


# 基本環境を作成
https://qiita.com/rinpa/items/81766cd6a7b23dea9f3c

```bash:bash
# 様々な **env 系ツールをまとめて管理する
brew install anyenv

# 初期設定
echo 'eval "$(anyenv init -)"' >> ~/.zshrc

# シェルを再起動
exec $SHELL -l

# 今回はnodeのバージョン管理を行うnodenvをインストール
anyenv install nodenv

# インストールできるリストを表示
nodenv install -l

nodenv install 12.7.0
nodenv install 10.16.0

# グローバルの設定・変更
nodenv global 12.7.0

# yarnをグローバルインストール
npm install -g yarn

exec $SHELL -l
```

# Reactプロジェクトの作成
https://create-react-app.dev/docs/adding-typescript/

```bash:bash
npx create-react-app mangarel-demo --template typescript
cd mangarel-demo
```

# TypeScriptの設定を更新

https://qiita.com/ryokkkke/items/390647a7c26933940470#baseurl

:::message
`baseUrl`　*相対的なカレントディレクトリをどこにするか指定*
`incremental`　*以前コンパイルを実行したコードと現在のコードとの差分だけコンパイル*
`exclude`　*コンパイルする対象から外すファイルを記述*

:::

```json:tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "baseUrl": "src",
    "incremental": true
  },
  "include": [
    "src"
  ],
  "exclude": [
    "node_modules",
    "build",
    "scripts",
    "functions"
  ]
}
```

# FireBaseプロジェクトの作成
任意のプロジェクト名で作成する
![](https://storage.googleapis.com/zenn-user-upload/c6d7ecc73aa2-20211212.png)

リソースロケーションを東京に変更
![](https://storage.googleapis.com/zenn-user-upload/ef2fc61ac9c0-20211212.png)

```bash:bash
# インストール
npm install --global firebase-tools

# ログイン
firebase login

# ローカル・プロジェクト新規作成
firebase init

# ビルド
yarn build

# Hosting のリソースのみデプロイ
firebase deploy --only hosting
```

## デプロイ完了

![](https://storage.googleapis.com/zenn-user-upload/1e5a8b046172-20211212.png)

