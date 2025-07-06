# sample-devcontainer-hotreload

DevContainerでReactアプリケーションをホットリロード機能付きで開発できるサンプルプロジェクトです。

## 概要

このプロジェクトは、Visual Studio CodeのDevContainer機能を使用して、Reactアプリケーションの開発環境をコンテナ化し、ホットリロード機能を有効にしたサンプルです。

## プロジェクト構造

```bash
sample-devcontainer-hotreload/
├── .devcontainer/          # DevContainer設定ファイル
│   └── devcontainer.json   # DevContainer設定
├── my-app/                 # Reactアプリケーション
│   ├── package.json        # 依存関係とスクリプト
│   ├── public/             # 静的ファイル
│   ├── src/                # ソースコード
│   └── README.md           # Reactアプリの説明
└── README.md               # このファイル
```

## 特徴

- **DevContainer対応**: VS CodeでDevContainerを使用して開発環境を統一
- **ホットリロード**: ファイル変更時に自動でブラウザが更新される
- **React 19**: 最新のReact機能を利用
- **開発環境の統一**: チーム全体で同じ開発環境を共有可能
- **自動セットアップ**: コンテナ作成時に自動でnpm installが実行される
- **パフォーマンス最適化**: node_modulesをNamed volumeで管理

## セットアップ

### 前提条件

- Docker Desktop
- Visual Studio Code
- VS Code拡張機能: "Dev Containers"

### 手順

#### 方法1: 既存のReactプロジェクトを使用する場合

1. このリポジトリをクローン

    ```bash
    git clone https://github.com/KenichiroArai/sample-devcontainer-hotreload.git
    cd sample-devcontainer-hotreload
    ```

2. VS Codeでプロジェクトを開く

3. DevContainerで開く

   - VS Codeでプロジェクトを開いた後、右下に表示される「Reopen in Container」をクリック
   - または、コマンドパレット（Ctrl+Shift+P）で「Dev Containers: Reopen in Container」を実行

4. コンテナ内でアプリケーションを起動

    ```bash
    cd my-app
    npm start
    ```

**注意**: 依存関係のインストールは`postCreateCommand`により自動で実行されるため、手動で`npm install`を実行する必要はありません。

#### 方法2: 新しくReactプロジェクトを作成する場合

1. 空のディレクトリを作成し、VS Codeで開く

2. DevContainer設定ファイルを作成（`.devcontainer/devcontainer.json`）

3. **重要**: 最初はmounts設定をコメントアウトまたは削除する

    ```json
    // "mounts": [
    //     "source=${localWorkspaceFolderBasename}-my-app-node-modules,target=/workspaces/${localWorkspaceFolderBasename}/my-app/node_modules,type=volume"
    // ]
    ```

4. DevContainerで開く

5. コンテナ内でReactプロジェクトを作成

    ```bash
    npx create-react-app my-app
    ```

6. コンテナを終了し、`.devcontainer/devcontainer.json`のmounts設定を有効にする

7. DevContainerを再作成（Rebuild Container）

8. アプリケーションを起動

    ```bash
    cd my-app
    npm start
    ```

## 使用方法

### 開発サーバーの起動

DevContainer内で以下のコマンドを実行：

```bash
cd my-app
npm start
```

アプリケーションは `http://localhost:3000` で起動し、ホットリロード機能が有効になります。

### ホットリロードの確認

1. `my-app/src/App.js` を編集
2. 保存すると自動でブラウザが更新される
3. 変更が即座に反映されることを確認

## 技術スタック

- **フロントエンド**: React 19
- **開発環境**: DevContainer
- **Node.js**: 22 (Bookworm)
- **パッケージマネージャー**: npm
- **ホットリロード**: React Scripts (Create React App)

## package.jsonの変更点

DevContainer環境でのホットリロード機能を確実に動作させるため、`package.json`の`start`スクリプトをデフォルトから変更しています：

```json
"start": "CHOKIDAR_USEPOLLING=true WATCHPACK_POLLING=true react-scripts start"
```

### 追加された環境変数

- **CHOKIDAR_USEPOLLING=true**: Chokidar（ファイル監視ライブラリ）にポーリングモードを強制
- **WATCHPACK_POLLING=true**: Webpackのファイル監視にポーリングモードを強制

### 変更理由

DevContainer環境では、ファイルシステムの監視が通常のinotifyベースの監視では正しく動作しない場合があります。特にWindowsホストからLinuxコンテナへのマウント時に、ファイル変更イベントが適切に伝播されないことがあります。

ポーリングモードを有効にすることで、定期的にファイルの変更をチェックし、確実にホットリロードが動作するようになります。

## DevContainer設定詳細

### 基本設定

- **ベースイメージ**: `mcr.microsoft.com/devcontainers/javascript-node:1-22-bookworm`
- **ユーザー**: `node`（非rootユーザー）
- **ポート転送**: 3000番ポート

### 自動化

- **postCreateCommand**: コンテナ作成時に`cd my-app && npm install`を自動実行
- **Named volume**: node_modulesをボリュームで管理し、パフォーマンスを向上

### VS Code設定

- **ファイル監視除外**: node_modulesディレクトリを監視対象外
- **ファイル除外**: node_modulesディレクトリをエクスプローラーから非表示

### 重要な設定項目

#### Reactプロジェクト名の設定

このプロジェクトでは、Reactアプリケーションのディレクトリ名を`my-app`として設定しています。この名前は以下の設定で使用されています：

- **postCreateCommand**: `cd my-app && npm install`
- **mounts設定**: `target=/workspaces/${localWorkspaceFolderBasename}/my-app/node_modules`

もし異なる名前でReactプロジェクトを作成する場合は、これらの設定を適切に変更する必要があります。

#### mounts設定の注意点

```json
"mounts": [
    "source=${localWorkspaceFolderBasename}-my-app-node-modules,target=/workspaces/${localWorkspaceFolderBasename}/my-app/node_modules,type=volume"
]
```

この設定により、`node_modules`ディレクトリがNamed volumeとして管理され、コンテナ再作成時も依存関係が保持されます。ただし、**Reactプロジェクト作成前にこの設定を有効にすると、プロジェクト作成が妨げられる可能性がある**ため、上記の手順に従って設定してください。

## トラブルシューティング

### DevContainerが起動しない場合

1. Docker Desktopが起動していることを確認
2. VS CodeのDev Containers拡張機能がインストールされていることを確認
3. コンテナのビルドログを確認

### ホットリロードが動作しない場合

1. ポート3000が正しく公開されていることを確認
2. ブラウザのキャッシュをクリア
3. 開発サーバーを再起動

### 依存関係のインストールエラー

1. コンテナを再作成（Dev Containers: Rebuild Container）
2. 手動で`npm install`を実行

### Reactプロジェクトが作成できない場合

1. mounts設定が有効になっていないか確認
2. 必要に応じてmounts設定を一時的にコメントアウト
3. Reactプロジェクト作成後にmounts設定を有効化
4. コンテナを再作成

### node_modulesが正しくマウントされない場合

1. Named volumeが正しく作成されているか確認
2. コンテナを再作成（Dev Containers: Rebuild Container）
3. 必要に応じてDocker volumeを手動で削除して再作成

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。

## 貢献

プルリクエストやイシューの報告を歓迎します。
