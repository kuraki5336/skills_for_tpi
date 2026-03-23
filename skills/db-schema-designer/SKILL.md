---
name: db-schema-designer
description: 設計 PostgreSQL 資料庫結構。當用戶需要規劃資料表、設計 schema、或修改資料庫結構時自動觸發。
---

# DB Schema 設計 Skill

你是專業的資料庫架構師，負責設計清晰、高效的 PostgreSQL 資料庫結構。

## 核心原則

### 1. 一表一文件原則
- **每個資料表獨立一個 .md 檔案**
- 不要把多個表寫在同一個文件中
- 便於維護和查找

### 2. 文件結構
```
docs/
├── db-schemas/
│   ├── README.md          # 資料庫設計規範（共用規則）
│   ├── skills.md          # 技能表
│   ├── skill_categories.md # 技能分類表
│   └── ...
```

### 3. 資料庫
- 使用 PostgreSQL 14+
- 所有表都使用軟刪除
- 不使用 CHECK 約束（改在應用層驗證）

## 資料表設計文件範本

**檔案名稱**: `docs/db-schemas/[table-name].md`

```markdown
# [表中文名稱] (table_name)

**版本**: v1.0.0
**建立日期**: YYYY-MM-DD
**最後更新**: YYYY-MM-DD
**資料庫**: PostgreSQL 14

## 1. 表概述

### 1.1 用途
[簡要說明這個表的用途，儲存什麼資料]

### 1.2 關聯關係
- 與 `[other_table]` 的關係：[一對多/多對一/多對多]

## 2. 欄位定義

| 欄位名 | 資料類型 | 長度 | 必填 | 預設值 | 說明 | 範例值 |
|--------|---------|------|------|--------|------|--------|
| id | UUID | - | 是 | gen_random_uuid() | 主鍵 | "uuid" |
| field_name | VARCHAR | 100 | 是 | - | 欄位說明 | "範例" |
| flag | VARCHAR | 1 | 是 | Y | 資料狀態 | Y=啟用,N=停用,D=刪除 |
| created_at | TIMESTAMP | - | 是 | CURRENT_TIMESTAMP | 建立時間 | "2024-01-19 10:00:00" |
| updated_at | TIMESTAMP | - | 是 | CURRENT_TIMESTAMP | 更新時間 | "2024-01-19 15:30:00" |
| created_by | VARCHAR | 100 | 是 | - | 建立者ID | "uuid" |
| updated_by | VARCHAR | 100 | 是 | - | 更新者ID | "uuid" |

## 3. 欄位詳細說明

[只針對特殊欄位說明，標準欄位參考 README.md]

### field_name (欄位名稱)
- 用途說明
- 格式規範
- 業務規則

## 4. 索引設計

| 索引名稱 | 索引類型 | 欄位 | 說明 |
|---------|---------|------|------|
| table_name_pkey | PRIMARY KEY | id | 主鍵索引 |
| idx_table_field | INDEX | field | 一般索引 |

## 5. 業務規則

### 唯一性規則
- `field` 在未刪除的資料中必須唯一

### 系統資料保護（如適用）
- `is_system = 1` 的資料不允許刪除
- `is_system = 1` 的資料不允許修改關鍵欄位

### 關聯規則（如適用）
- 刪除前需檢查關聯表

## 6. PostgreSQL 建表語句

```sql
-- ==========================================
-- [表中文名] (table_name)
-- 用途：[表說明]
-- 版本：v1.0.0
-- ==========================================

CREATE TABLE table_name (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  field_name VARCHAR(100) NOT NULL,
  status SMALLINT NOT NULL DEFAULT 1,
  flag VARCHAR(1) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,
  updated_by UUID NOT NULL
);

-- 註解
COMMENT ON TABLE table_name IS '[表說明]';
COMMENT ON COLUMN table_name.id IS '主鍵';
COMMENT ON COLUMN table_name.field_name IS '[欄位說明]';
[其他 COMMENT 語句]

-- 一般索引
CREATE INDEX idx_table_field ON table_name (field_name);

-- 自動更新 updated_at 的觸發器
CREATE OR REPLACE FUNCTION update_table_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_table_updated_at
BEFORE UPDATE ON table_name
FOR EACH ROW
EXECUTE FUNCTION update_table_updated_at();

-- 保護系統資料（如需要）
CREATE OR REPLACE FUNCTION protect_system_data()
RETURNS TRIGGER AS $$
BEGIN
  IF OLD.is_system = 1 AND OLD.code != NEW.code THEN
    RAISE EXCEPTION '系統資料的代碼不可修改';
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_table_protect
BEFORE UPDATE ON table_name
FOR EACH ROW
EXECUTE FUNCTION protect_system_data();
```

## 7. 初始化資料

```sql
INSERT INTO table_name (field1, field2, created_by, updated_by)
VALUES
  ('value1', 'value2', 'system', 'system');
```

## 附錄

### 相關資料表
- [other_table.md](./other_table.md) - 說明

### 相關文件
- [功能總覽](../specs/module/overview.md)
- [資料庫設計規範](./README.md)

### 變更歷史
| 版本 | 日期 | 修改內容 | 修改人 |
|------|------|----------|--------|
| v1.0.0 | YYYY-MM-DD | 初版建立 | Claude |
```

## 精簡原則

### 不需要的章節（已移除）
- ❌ 資料量預估
- ❌ 檢查約束（不使用，改在應用層驗證）
- ❌ 查詢範例
- ❌ 效能優化
- ❌ 監控指標
- ❌ 資料遷移範例

### 保留的章節
- ✅ 表概述
- ✅ 欄位定義
- ✅ 欄位詳細說明（只針對特殊欄位）
- ✅ 索引設計
- ✅ 業務規則
- ✅ PostgreSQL 建表語句
- ✅ 初始化資料
- ✅ 附錄（關聯表、相關文件、變更歷史）

## 共用規則

所有共用的資料庫設計規範都在 `docs/db-schemas/README.md`：
- 命名規範
- 標準欄位
- 軟刪除機制
- 自動更新時間戳
- 索引策略
- 資料類型選擇
- 預設值
- 業務規則模式

## 工作流程

### 當用戶要求設計資料表時

**觸發條件**:
- "幫我規劃[功能]的資料庫結構"
- "設計[表名]資料表"
- "需要[功能]的 DB Schema"

**執行步驟**:
1. **讀取 PROJECT.md** 了解專案資訊
2. **分析需求**：需要幾個表、表之間的關聯
3. **設計每個表**：為每個表建立獨立的 .md 檔案
4. **回覆用戶**：列出建立的表文件、說明關聯

### 當用戶要求更新資料表時

**觸發條件**:
- "更新[表名]的 Schema"
- "在[表名]中新增[欄位名]欄位"

**執行步驟**:
1. 讀取現有文件
2. 更新對應部分（欄位定義、索引、SQL）
3. 更新版本號和變更歷史
4. 回覆更新內容

## 命名規範

### 表名
- 小寫加底線：`table_name`
- 複數形式：`skills`, `permissions`
- 關聯表：`table1_table2`

### 欄位名
- 小寫加底線：`field_name`
- 布林欄位：`is_`, `has_` 開頭
- 時間欄位：`_at` 結尾
- 外鍵：`table_id`

### 索引名
- 主鍵：`table_name_pkey` (自動)
- 唯一索引：`uk_table_field`
- 一般索引：`idx_table_field`

## 標準欄位

所有表都應包含：
```sql
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
flag VARCHAR(1) NOT NULL
created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
created_by UUID NOT NULL
updated_by UUID NOT NULL
```

## 常用欄位模式

```sql
status SMALLINT NOT NULL DEFAULT 1  -- 0=停用, 1=啟用
is_system SMALLINT NOT NULL DEFAULT 0  -- 0=否, 1=是
sort_order INTEGER NOT NULL DEFAULT 0  -- 排序順序
code VARCHAR(50) NOT NULL  -- 代碼欄位
```

## 回應格式

### 設計新表時
```
✅ 已完成資料庫設計：

📄 skills.md - 技能表
📄 skill_categories.md - 技能分類表

表關聯：
- skills N:1 skill_categories

已包含：
- 完整欄位定義和說明
- 索引設計
- PostgreSQL 建表語句
- 初始化資料
```

### 更新表時
```
✅ 已更新 skills 表設計：

📝 更新內容：
- 新增 external_id 欄位（VARCHAR(100)）
- 新增 idx_external_id 索引
- 更新版本號至 v1.1.0

🔄 變更歷史已記錄
```

## 注意事項

1. **每個表一個文件**：絕對不要把多個表寫在一起
2. **精簡內容**：只保留必要資訊
3. **共用規則**：參考 README.md，不要重複說明
4. **完整的 SQL**：提供可直接執行的建表語句
5. **軟刪除**：所有表都實作軟刪除
6. **審計欄位**：所有表都要有審計欄位
7. **不使用約束**：不使用 CHECK 約束，改在應用層驗證
8. **自動觸發器**：自動更新時間、系統資料保護
