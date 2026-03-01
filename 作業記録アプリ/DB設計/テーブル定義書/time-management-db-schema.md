# users（ユーザー）
ユーザーを管理するテーブル

## カラム定義

| No. | 論理名      | 物理名      | データ型   | 制約 | デフォルト | 備考 |
|-----|-------------|------------|------------|-----|-----------|------|
| 1 | ユーザーID     | user_id   | integer     | PK, GENERATED ALWAYS AS IDENTITY | - | 主キー（内部採番） |
| 2 | ユーザー名     | user_name | varchar(255) | NOT NULL | - | 表示名 |
| 3 | メールアドレス | mail_address | varchar(255) | UNIQUE, NOT NULL | - | ログイン用メールアドレス |
| 4 | 登録日時      | created_at  | timestamp with time zone | NOT NULL | DEFAULT NOW() | 監査用の登録日時（UTC） |
| 5 | 更新日時      | updated_at  | timestamp with time zone | NOT NULL | DEFAULT NOW() | 監査用の更新日時（UTC）、更新時はトリガー「trg_users_set_updated_at」で自動更新 |

---

## 制約

| No. | 制約名        | 種類        | 対象カラム | 制約定義 | 備考 |
|-----|---------------|--------------|---------|------|------|
| 1 | pk_users       | PRIMARY KEY  | user_id | - | |
| 2 | uq_users_mail_address | UNIQUE | mail_address | - | |

---

## インデックス

| No. | インデックス名 | 種類          | 対象カラム | 備考 |
|-----|---------------|--------------|-----------|------|
| (none) |            |              |           |      |

---

## 外部キー制約

| No. | 外部キー名 | From（参照元カラム） | To（参照先カラム） | OnDelete | OnUpdate |
|-----|---------|------|----|----------|----------|
| (none)  |      |    |          |          |         |

---

<br>

# tasks（タスク）
ユーザーが持つタスクを管理するテーブル

## カラム定義

| No. | 論理名      | 物理名      | データ型   | 制約 | デフォルト | 備考 |
|-----|------------|------------|------------|-----|-----------|------|
| 1 | ユーザーID    | user_id    | integer    | FK, NOT NULL | - | |
| 2 | タスクID      | task_id    | integer    | PK, GENERATED ALWAYS AS IDENTITY | - | 主キー（内部採番） |
| 3 | タスク名      | task_name   | text      | NOT NULL | - | 表示名 |
| 4 | タスクの色    | task_color  | varchar(7) | NOT NULL, CHECK | - | HEXカラー |
| 5 | 登録日時      | created_at  | timestamp with time zone | NOT NULL | DEFAULT NOW() | 監査用の登録日時（UTC） |
| 6 | 更新日時      | updated_at  | timestamp with time zone | NOT NULL | DEFAULT NOW() | 監査用の更新日時（UTC）、更新時はトリガー「trg_tasks_set_updated_at」で自動更新 |

---

## 制約

| No. | 制約名        | 種類        | 対象カラム | 制約定義 | 備考 |
|-----|---------------|--------------|---------|------|------|
| 1 | pk_tasks       | PRIMARY KEY  | task_id  | - | |
| 2 | uq_tasks_user_id_task_name | UNIQUE  | user_id, task_name | UNIQUE (user_id, task_name) | 同一ユーザー内ではタスク名は一意 |
| 3 | check_task_color | CHECK | task_color | CHECK (task_color ~ '^#[0-9A-Fa-f]{6}$') | HEXカラー |

---

## インデックス

| No. | インデックス名 | 種類          | 対象カラム | 備考 |
|-----|---------------|--------------|-----------|------|
| 1 | idx_tasks_user_id | INDEX | user_id | ユーザーのタスク一覧取得用 |

---

## 外部キー制約

| No. | 外部キー名 | From（参照元カラム） | To（参照先カラム） | OnDelete | OnUpdate |
|-----|---------|------|----|----------|----------|
| 1 | fk_tasks_user_id   | tasks.user_id  | users.user_id  | CASCADE | RESTRICT |

---

<br>

# work_time（作業時間）
タスクの作業開始時刻、作業終了時刻を管理するテーブル

## カラム定義

| No. | 論理名      | 物理名      | データ型   | 制約 | デフォルト | 備考 |
|-----|------------|------------|------------|-----|-----------|------|
| 1 | タスクID    | task_id    | integer    | FK, NOT NULL | - | |
| 2 | 作業ID      | work_id    | integer    | PK, GENERATED ALWAYS AS IDENTITY | - | 主キー（内部採番） |
| 3 | 作業開始時刻      | work_start_time  | timestamp with time zone | NOT NULL | DEFAULT NOW() | |
| 4 | 作業終了時刻      | work_end_time  | timestamp with time zone | CHECK | - | |
| 5 | 登録日時      | created_at  | timestamp with time zone | NOT NULL | DEFAULT NOW() | 監査用の登録日時（UTC） |
| 6 | 更新日時      | updated_at  | timestamp with time zone | NOT NULL | DEFAULT NOW() | 監査用の更新日時（UTC）、更新時はトリガー「trg_work_time_set_updated_at」で自動更新 |

---

## 制約

| No. | 制約名        | 種類        | 対象カラム | 制約定義 | 備考 |
|-----|---------------|--------------|---------|------|------|
| 1 | pk_work_time       | PRIMARY KEY  | work_id  | - | |
| 2 | check_work_time_order | CHECK | work_start_time, work_end_time | CHECK (work_end_time IS NULL OR work_end_time >= work_start_time) | 作業終了時刻は作業開始時刻以降である |

---

## インデックス

| No. | インデックス名 | 種類          | 対象カラム | 備考 |
|-----|---------------|--------------|-----------|------|
| 1 | idx_work_time_task_id_start_time | INDEX | task_id, work_start_time | タスクの作業履歴取得、ユーザーの（時間指定での）作業履歴取得用 |

---

## 外部キー制約

| No. | 外部キー名 | From（参照元カラム） | To（参照先カラム） | OnDelete | OnUpdate |
|-----|---------|------|----|----------|----------|
| 1 | fk_work_time_task_id   | work_time.task_id  | tasks.task_id  | CASCADE | RESTRICT |