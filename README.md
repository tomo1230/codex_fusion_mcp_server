# Fusion MCP Server for OpenAI Codex CLI

**バージョン: 0.7.80 (Beta) ファイル連携バージョン**

このプロジェクトは、**OpenAI Codex CLI**がAutodesk Fusion を直接操作するためのModel Context Protocol (MCP)サーバーとの設定ファイルです。このツールをOpenAI Codex CLIに設定することで、チャットのプロンプトを通じて3Dモデルの作成、編集、情報取得が可能になります。

MCPサーバとしては、こちらのFusion MCP Server for Claude Desktop（<https://github.com/tomo1230/claude_fusion_mcp_server>）と連携ができます。

MCPサーバーは、Fusion 内で動作する[対応するPythonアドイン fusion_mcp_server](<https://github.com/tomo1230/fusion_mcp_server>)と連携して機能します。

- **作者:** Kanbara Tomonori
- **X (旧Twitter):** [@tomo1230](https://x.com/tomo1230)
- **ライセンス:** 本ソースコードはプロプライエタリかつ機密情報です。無断での複製、修正、配布、使用は固く禁じられています。

---

## 概要とアーキテクチャ

このツールは、Gemini CLIとの対話を通じて、直感的かつ自然言語ベースでFusion のモデリング作業を行うためのブリッジとして機能します。

**処理フロー:**
1.  ユーザーがOpenAI Codex CLIで `@Fusion` のようなツール名を指定してプロンプトを送信します。（例: `@Fusion 50mmの立方体を作って`）
2.  OpenAI Codex CLIは、このNode.jsサーバーを子プロセスとして起動し、`CallToolRequest` を送信します。
3.  Node.jsサーバーはリクエストをJSONコマンドに変換し、`~/Documents/fusion_command.txt` に書き込みます。
4.  Fusion 内で起動しているPythonアドインがこのファイルを検知し、Fusion のAPIを実行します。
5.  Pythonアドインは実行結果を `~/Documents/fusion_response.txt` に書き込みます。
6.  Node.jsサーバーがレスポンスを読み取り、OpenAI Codex CLIに結果を返します。
7.  OpenAI Codex CLIがその結果を解釈し、ユーザーに応答します。



---

## セットアップガイド for OpenAI Codex CLI

### Step 1: 前提条件の確認
-   **OpenAI Codex CLI**: アプリケーションがインストールされていること。（<https://github.com/openai/codex>）
-   **Node.js**: v18以降がインストールされていること。(<https://nodejs.org/ja/download>)
-   **Autodesk Fusion **: 最新版がインストールされていること。
-   **Fusion  Pythonアドイン**: **これが最も重要です。**[対応するPythonアドイン fusion_mcp_server](<https://github.com/tomo1230/fusion_mcp_server>)がFusion にインストールされ、ツールバーの**「連携開始」**ボタンが押されている状態にしてください。

### Step 2: OpenAI Codex CLIへのツール追加
1.  OpenAI Codex CLIの.codex設定フォルダを開き、config.tomlを編集します。
3.  先ほどクローンした**リポジトリの.codexフォルダにあるconfig.toml**を参考に内容を追加します。OpenAI Codex CLIが自動的に `fusion_mcp_server.js` を認識します。

### Step 3: Gemini CLIでの使用
セットアップが完了すれば、OpenAI Codex CLIチャットでFusion を操作できます。

<img width="548" height="447" alt="スクリーンショット 2025-09-22 232634" src="https://github.com/user-attachments/assets/1f30d86b-2655-455e-8bce-95b752b0dae4" />

**/mcp コマンドで起動できているか確認して下さい。**

**使用例:**
-   `@Fusion360 幅50、奥行き30、高さ20の箱を作って`
-   `@Fusion360 "MyCube" という名前の立方体を作成して、その寸法を教えて`
-   `@Fusion360 最後に作ったボディに半径2mmのフィレットを追加して`

---

## APIリファレンス / 利用可能なツール

Gemini CLIは以下のツールを呼び出すことでFusion を操作します。

### 形状作成ツール
-   **`create_cube`**: 立方体を作成
-   **`create_cylinder`**: 円柱を作成
-   **`create_box`**: 直方体を作成
-   **`create_sphere`**: 球を作成
-   **`create_hemisphere`**: 半球を作成
-   **`create_cone`**: 円錐を作成
-   **`create_polygon_prism`**: 多角柱を作成
-   **`create_torus`**: トーラスを作成
-   **`create_half_torus`**: 半分のトーラスを作成
-   **`create_pipe`**: 2点間にパイプを作成
-   **`create_polygon_sweep`**: ねじれたリング形状を作成

### 編集・変形ツール
-   **`add_fillet`**: フィレット（角丸め）を追加
-   **`add_chamfer`**: 面取りを追加
-   **`combine_by_name`**: ブーリアン演算（結合、切り取り、交差）を実行
-   **`move_by_name`**: ボディを移動
-   **`rotate_by_name`**: ボディを回転

### パターン・コピー
-   **`copy_body_symmetric`**: 対称コピー（ミラー）
-   **`create_circular_pattern`**: 円形状に複製
-   **`create_rectangular_pattern`**: 矩形状に複製

### 情報取得ツール
-   **`get_bounding_box`**: バウンディングボックスを取得
-   **`get_body_center`**: ボディの中心を取得
-   **`get_body_dimensions`**: ボディの寸法（体積、表面積など）を取得
-   **`get_faces_info`**: 全ての面の情報を取得
-   **`get_edges_info`**: 全てのエッジの情報を取得
-   **`get_mass_properties`**: 質量特性を計算
-   **`get_body_relationships`**: 2ボディ間の関係（距離、干渉など）を取得
-   **`measure_distance`**: 2ボディ間の最短距離を測定

### ユーティリティ
-   **`execute_macro`**: 複数のコマンドを連続実行
-   **`select_body` / `select_all_bodies`**: ボディを選択
-   **`hide_body` / `show_body`**: ボディの表示/非表示
-   **`debug_coordinate_info`**: 座標系のデバッグ情報を取得

---

## 使用例

**YouTube モデるんですAI チャンネル**

「しゃべるだけで、世界がカタチになる。」
ことばが、モノになる時代。
『ModerundesuAI』は、AIと会話するだけで3Dモデリングができる、
未来のモノづくり体験をシェアするYouTubeチャンネルです。
Fusion 360やBlenderなどのCADソフトとAI（ChatGPTやClaude）を連携させて、
プロンプト（命令文）でリアルな“カタチ”を自動生成。
初心者からモデリング好きまで、誰でも「つくる楽しさ」に触れられるコンテンツを発信します！

**https://www.youtube.com/@ModerundesuAI**

**「サイコロを設計して」Claude AI＆Autodesk Fusion API 連携🤖AIモデリングチャレンジ！💪**
[![](https://github.com/user-attachments/assets/c5be6840-3321-4431-8342-8ce050bc5314)](https://youtu.be/S_-xYwK5HUc?si=JWE3yv5mxRLGJaXd)

**「400mlのコップを設計して」Claude AI＆Autodesk Fusion API 連携🤖AIモデリングチャレンジ！💪**
[![](https://github.com/user-attachments/assets/820652c7-1199-4ed2-9589-4fc2b1df5a98)](https://youtu.be/abfEWtMKRV4?si=gTVDwvkIkyt81jnb)

**「使えるコマンドのテストをして」Claude AI MCP ＆ Autodesk Fusion API 連携🤖AIモデリングチャレンジ！💪**
[![](https://github.com/user-attachments/assets/aded31be-f6b3-45bb-9461-f1cd3c40ca85)](https://youtu.be/Qn-Skeh3o2c?si=7xKrM_bA7IbXT47-)

---

## 🟢 できること
- **基本形状作成** - 立方体、円柱、球など10種類の基本形状の組み合わせ
- **編集操作** - フィレット、面取り、移動、回転
- **パターン作成** - 円形・矩形配列、対称コピー
- **ブール演算** - 結合、切除、交差
- **情報取得** - 寸法、体積、質量特性の測定

## 🔴 できないこと
- **スケッチ** - 2D図形の自由描画
- **複雑形状** - 自由曲面、有機的な形状
- **アセンブリ** - 複数部品の組み立て
- **解析・製造** - CAM、FEA、レンダリング

---

## ライセンス条項

本ソフトウェアおよびそのソースコードは、著作権者が所有権を有する専有資産であり、著作権法および関連する国際条約によって保護されています。

著作権者の書面による事前の明示的な許可がない限り、本ソースコードの全部または一部を、複製、改変、翻案、結合、サブライセンス、頒布、リバースエンジニアリング、逆コンパイル、または逆アセンブルする行為は、その方法や形態を問わず一切禁じられています。本書で明示的に許諾されていない全ての権利は、著作権者に留保されます。
