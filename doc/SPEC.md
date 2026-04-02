# CATIA ドア色変更ツール（Door Color Changer）仕様書

## 1. プロジェクト概要

| 項目 | 内容 |
|---|---|
| **アプリ名** | Door Color Changer |
| **概要** | CATIA V5 上で指定したドア部品の色をグレーに変更する自動化スクリプト |
| **目的** | GitHub Copilot ハンズオン用サンプルアプリケーション |

---

## 2. 技術スタック

| 技術 | バージョン | 用途 |
|---|---|---|
| Python | 3.10+ | メイン言語 |
| pycatia | 0.6.x 以上 | CATIA V5 COM API ラッパー |
| CATIA V5 | V5-6R20 以降 | 3D CAD ソフトウェア（操作対象） |
| pip | 最新 | パッケージ管理 |

### 動作前提条件

- Windows OS であること
- CATIA V5 がインストール済みかつ起動中であること
- 対象の CATProduct ファイルが CATIA 上で開かれていること
- Python 環境に `pycatia` がインストールされていること

---

## 3. ドメインモデル

### DoorPart（ドア部品）データモデル

| フィールド名 | 型 | 必須 | 説明 |
|---|---|---|---|
| `product` | `Product` | ✅ | CATIA プロダクトツリー上の部品オブジェクト |
| `part_number` | `str` | ✅ | パーツ番号（例：`Door_Front_Left`） |
| `name` | `str` | ✅ | プロダクトツリー上の表示名 |
| `current_color` | `tuple[int,int,int]` | ✅ | 現在の RGB カラー値 |

### TargetColor（変更先の色）

| 値 | RGB | 説明 |
|---|---|---|
| `GRAY` | `(192, 192, 192)` | グレー（デフォルト変更色） |

> **注**: グレーの RGB 値は `(192, 192, 192)` を標準とするが、設定で変更可能とする。

---

## 4. コンソール仕様（CLI）

### 実行イメージ

```
$ python door_color_changer.py --door-name "Door_Front_Left"

============================================
🚗 CATIA ドア色変更ツール (Door Color Changer)
============================================
CATIA に接続中...                  ✅ 接続成功
アクティブドキュメントを取得中...     ✅ Car_Assembly.CATProduct
ドア部品を検索中: Door_Front_Left... ✅ 部品が見つかりました
現在の色: RGB(255, 0, 0)
色をグレー RGB(192, 192, 192) に変更中... ✅ 変更完了
============================================
✅ ドア "Door_Front_Left" の色をグレーに変更しました
============================================
```

### コマンドライン引数一覧

| 引数 | 短縮形 | 必須 | デフォルト | 説明 |
|---|---|---|---|---|
| `--door-name` | `-d` | ✅ | — | 色を変更する対象ドア部品の名前 |
| `--rgb` | `-c` | — | `192,192,192` | 変更先の RGB 値（カンマ区切り） |
| `--recursive` | `-r` | — | `False` | サブプロダクトを再帰的に検索するか |
| `--dry-run` | — | — | `False` | 実際に色変更せずプレビューのみ |

### 実行コマンド例

```bash
# 基本的な使い方（ドア名を指定してグレーに変更）
python door_color_changer.py --door-name "Door_Front_Left"

# カスタム RGB を指定
python door_color_changer.py --door-name "Door_Front_Left" --rgb 128,128,128

# 再帰検索 + ドライラン
python door_color_changer.py --door-name "Door" --recursive --dry-run
```

---

## 5. モジュール構成

### ファイル構成

```
door_color_changer/
├── door_color_changer.py      # メインスクリプト（エントリポイント）
├── catia_connector.py         # CATIA 接続・ドキュメント取得
├── door_finder.py             # ドア部品の検索ロジック
├── color_changer.py           # 色変更ロジック
├── config.py                  # 設定値（デフォルト RGB 等）
├── requirements.txt           # 依存パッケージ
└── README.md                  # 使い方
```

---

## 6. 関数仕様

### catia_connector.py

| 関数名 | 引数 | 戻り値 | 説明 |
|---|---|---|---|
| `connect_catia()` | — | `CATIAApplication` | 起動中の CATIA に接続し、アプリケーションオブジェクトを返す |
| `get_active_document()` | `caa: CATIAApplication` | `Document` | アクティブな CATProduct ドキュメントを取得 |

### door_finder.py

| 関数名 | 引数 | 戻り値 | 説明 |
|---|---|---|---|
| `find_door_by_name(product, door_name, recursive)` | `Product, str, bool` | `Product \| None` | プロダクトツリーから指定名のドア部品を検索して返す |
| `list_all_products(product, recursive)` | `Product, bool` | `list[Product]` | プロダクト配下のすべての子部品を一覧取得 |

### color_changer.py

| 関数名 | 引数 | 戻り値 | 説明 |
|---|---|---|---|
| `get_current_color(product)` | `Product` | `tuple[int,int,int]` | 指定部品の現在の表示色を RGB タプルで取得 |
| `change_color_to_gray(product, rgb)` | `Product, tuple[int,int,int]` | `None` | 指定部品の表示色を指定 RGB に変更 |

### door_color_changer.py（メイン）

| 関数名 | 引数 | 戻り値 | 説明 |
|---|---|---|---|
| `main()` | — | `None` | CLI 引数をパースし、CATIA 接続→ドア検索→色変更の一連処理を実行 |

---

## 7. 色変更の処理フロー

```
1. CATIA に接続（pycatia.catia()）
       ↓
2. アクティブドキュメント（CATProduct）を取得
       ↓
3. プロダクトツリーからドア部品を名前で検索
       ↓
4. 対象部品の Selection オブジェクトを取得
       ↓
5. VisPropertySet で色を変更
       ↓
6. ドキュメントを更新（update）
```

### 色変更のコアロジック（参考実装）

```python
from pycatia import catia
from pycatia.enumeration.enumeration_types import cat_vis_property_status

def change_color_to_gray(product, rgb=(192, 192, 192)):
    """
    指定プロダクトの表示色を変更する。

    Args:
        product: 対象の CATIA Product オブジェクト
        rgb: 変更先の色 (R, G, B) タプル。デフォルトはグレー。
    """
    caa = catia()
    document = caa.active_document
    selection = document.selection
    selection.clear()
    selection.add(product)

    vis_properties = selection.vis_properties
    vis_properties.set_real_color(rgb[0], rgb[1], rgb[2], 1)

    selection.clear()
```

---

## 8. エラーハンドリング

| エラー条件 | メッセージ | 動作 |
|---|---|---|
| CATIA が起動していない | `❌ エラー: CATIA が起動していません。CATIA V5 を起動してから再実行してください。` | 終了コード 1 で終了 |
| アクティブドキュメントがない | `❌ エラー: CATIA にドキュメントが開かれていません。CATProduct を開いてください。` | 終了コード 1 で終了 |
| ドキュメントが CATProduct でない | `❌ エラー: アクティブドキュメントが CATProduct ではありません。` | 終了コード 1 で終了 |
| 指定ドアが見つからない | `❌ エラー: ドア "{door_name}" が見つかりません。部品名を確認してください。` | 終了コード 1 で終了 |
| RGB 値が不正 | `❌ エラー: RGB 値が不正です。0〜255 の整数3つをカンマ区切りで指定してください。` | 終了コード 1 で終了 |

---

## 9. 設定値（config.py）

```python
# デフォルトのグレー色 RGB
DEFAULT_GRAY_RGB = (192, 192, 192)

# 検索対象のドキュメントタイプ
SUPPORTED_DOC_TYPE = "CATProduct"

# ログ出力レベル
LOG_LEVEL = "INFO"
```

---

## 10. 依存パッケージ（requirements.txt）

```text
pycatia>=0.6.0
```

### インストール手順

```bash
pip install -r requirements.txt
```

---

## 11. 実行例（ステップバイステップ）

1. **CATIA V5 を起動**し、対象の車両アセンブリ（`.CATProduct`）を開く
2. **ドア部品の名前を確認**する（プロダクトツリー上の名前）
3. **コマンドを実行**する：

```bash
python door_color_changer.py --door-name "Door_Front_Left"
```

4. コンソールに実行結果が表示され、CATIA 上のドアの色がグレーに変わる
