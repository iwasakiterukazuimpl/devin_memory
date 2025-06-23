# Task Management API 完全仕様書

## 1. プロジェクト概要

### 1.1 システム概要
Node.js/Express を使用したタスク管理 REST API システム。MongoDB をデータベースとして使用し、Bearer トークン認証によるセキュアなタスク管理機能を提供します。

### 1.2 主要機能
- タスクの作成、読み取り、更新、削除（CRUD操作）
- Bearer トークンによる認証機能
- MongoDB による永続化
- ログ機能
- RESTful API 設計

## 2. システムアーキテクチャ

### 2.1 レイヤー構成
```
┌─────────────────────────────────────┐
│           Client Layer              │
├─────────────────────────────────────┤
│        Authentication Layer         │
│         (authMiddleware.js)         │
├─────────────────────────────────────┤
│         Controller Layer            │
│        (taskController.js)          │
├─────────────────────────────────────┤
│          Service Layer              │
│         (taskService.js)            │
├─────────────────────────────────────┤
│           Model Layer               │
│          (taskModel.js)             │
├─────────────────────────────────────┤
│         Database Layer              │
│      (MongoDB + Mongoose)           │
└─────────────────────────────────────┘
```

### 2.2 ファイル構造と役割

#### コアファイル
1. **server.js** - アプリケーションエントリーポイント
   - データベース接続の初期化
   - サーバー起動処理
   - ポート設定（環境変数 PORT または 5000）

2. **app.js** - Express アプリケーション設定
   - ミドルウェア設定
   - ルーティング設定
   - JSON パーサー設定

3. **database.js** - データベース接続管理
   - MongoDB 接続設定
   - 接続エラーハンドリング
   - 接続先: `mongodb://localhost:27017/taskdb`

#### ビジネスロジック層
4. **taskModel.js** - データモデル定義
   - Mongoose スキーマ定義
   - バリデーション設定

5. **taskService.js** - ビジネスロジック層
   - CRUD 操作の実装
   - データベースアクセス抽象化

6. **taskController.js** - コントローラー層
   - HTTP リクエスト/レスポンス処理
   - サービス層との連携

7. **taskRoutes.js** - ルーティング定義
   - REST API エンドポイント定義
   - HTTP メソッドとコントローラーのマッピング

#### セキュリティ・ユーティリティ
8. **authMiddleware.js** - 認証ミドルウェア
   - Bearer トークン検証
   - 認証失敗時のエラーハンドリング

9. **logger.js** - ログユーティリティ
   - 統一されたログ出力機能

#### ドキュメント
10. **README.md** - プロジェクト概要
    - API 使用方法
    - 認証方法の説明

## 3. データモデル仕様

### 3.1 Task スキーマ
```javascript
const taskSchema = new mongoose.Schema({
  title: { 
    type: String, 
    required: true 
  },
  completed: { 
    type: Boolean, 
    default: false 
  },
  dueDate: { 
    type: Date 
  }
});
```

### 3.2 フィールド詳細
| フィールド | 型 | 必須 | デフォルト値 | 説明 |
|-----------|----|----|------------|------|
| title | String | ✓ | - | タスクのタイトル |
| completed | Boolean | - | false | 完了状態 |
| dueDate | Date | - | - | 期限日 |

## 4. API 仕様

### 4.1 ベース URL
```
http://localhost:5000/api/tasks
```

### 4.2 認証
全エンドポイントで Bearer トークン認証が必要です。

**ヘッダー例:**
```
Authorization: Bearer valid-token
```

### 4.3 エンドポイント詳細

#### 4.3.1 全タスク取得
```
GET /api/tasks
```

**レスポンス例:**
```json
[
  {
    "_id": "507f1f77bcf86cd799439011",
    "title": "サンプルタスク",
    "completed": false,
    "dueDate": "2024-12-31T00:00:00.000Z"
  }
]
```

#### 4.3.2 タスク作成
```
POST /api/tasks
```

**リクエストボディ:**
```json
{
  "title": "新しいタスク",
  "completed": false,
  "dueDate": "2024-12-31T00:00:00.000Z"
}
```

**レスポンス:**
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "title": "新しいタスク",
  "completed": false,
  "dueDate": "2024-12-31T00:00:00.000Z"
}
```

#### 4.3.3 タスク更新
```
PUT /api/tasks/:id
```

**リクエストボディ:**
```json
{
  "title": "更新されたタスク",
  "completed": true
}
```

**レスポンス:**
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "title": "更新されたタスク",
  "completed": true,
  "dueDate": "2024-12-31T00:00:00.000Z"
}
```

#### 4.3.4 タスク削除
```
DELETE /api/tasks/:id
```

**レスポンス:**
```
HTTP 204 No Content
```

### 4.4 エラーレスポンス

#### 認証エラー
```json
{
  "error": "Unauthorized"
}
```
**ステータスコード:** 401

## 5. 技術仕様

### 5.1 技術スタック
- **Runtime:** Node.js
- **Framework:** Express.js
- **Database:** MongoDB
- **ODM:** Mongoose
- **Authentication:** Bearer Token

### 5.2 依存関係
```json
{
  "express": "^4.x.x",
  "mongoose": "^7.x.x"
}
```

### 5.3 環境変数
| 変数名 | デフォルト値 | 説明 |
|--------|-------------|------|
| PORT | 5000 | サーバーポート番号 |

### 5.4 データベース設定
- **接続先:** `mongodb://localhost:27017/taskdb`
- **接続方式:** 非同期接続
- **エラーハンドリング:** 接続失敗時はプロセス終了

## 6. セキュリティ

### 6.1 認証方式
- Bearer トークン認証
- 全エンドポイントで認証必須
- 有効トークン: `"valid-token"`

### 6.2 セキュリティ考慮事項
- 本番環境では実際のJWT実装を推奨
- HTTPS通信の使用を推奨
- 環境変数での機密情報管理を推奨

## 7. 運用

### 7.1 起動手順
1. MongoDB サーバーの起動
2. 依存関係のインストール: `npm install`
3. アプリケーション起動: `node server.js`

### 7.2 ログ
- カスタムログ関数 `log(message)` を使用
- フォーマット: `[LOG]: {message}`

### 7.3 エラーハンドリング
- データベース接続エラー時はプロセス終了
- 認証失敗時は 401 エラーを返却
- 不正なリクエスト時は適切なHTTPステータスコードを返却

## 8. 開発・拡張性

### 8.1 アーキテクチャの利点
- **レイヤー分離:** 各層の責任が明確
- **拡張性:** 新機能追加が容易
- **テスタビリティ:** 各層を独立してテスト可能
- **保守性:** コードの可読性と保守性が高い

### 8.2 今後の拡張案
- JWT実装による本格的な認証
- バリデーション強化
- ページネーション機能
- 検索・フィルタリング機能
- ユーザー管理機能
- テストスイートの追加

---

**作成日:** 2025年6月23日  
**バージョン:** 1.0  
**対象システム:** Task Management API
