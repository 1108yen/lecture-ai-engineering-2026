# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：glossary_search_glossary
- STEP 2 の [tool call]：glossary_search_glossary({'query': '3レベル'})
- STEP 3 の [tool call]：fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
- STEP 4 の [tool call]：aws_aws___search_documentation({'search_phrase': 'Amazon S3 versioning overview'})
- 気づき・疑問：エージェント本体（app/mcp_agent.py）のPythonコードを一切書き換えずに、設定ファイル（app/mcp_servers.json）へサーバーの起動コマンドやURLを追記するだけで、ローカル起動・公開サーバー（uvx）・外部リモートサーバー（URL接続）のツールを統一的な「サーバー名_ツール名」の形式でモデルに渡して呼び出せることが確認できた。外部のURLに接続するサーバーでは、ツールに渡す検索クエリが提供者のサーバーへ送信される点も把握した。


## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：基礎
- やったこと：`app/mcp_server.py` の末尾に `server.tool()(tools.get_lecture_summary)` の1行を書き足し、エージェント側（`app/mcp_agent.py` や `app/mcp_servers.json`）を変更せずにツール一覧の確認（`--list-tools`）、質問実行（`--trace "第2回では何を学びますか?"`）、自動テスト（`uv run pytest -q`）を行った。
- 結果：`--list-tools` で `glossary_get_lecture_summary` が追加され、合計ツール数が7個から8個に増えた。質問実行時のトレースでは `[tool call] glossary_get_lecture_summary({'lecture': 2})` が呼ばれ、第2回の講義要約（ファインチューニングとLoRA）が正しく回答された。自動テスト（`uv run pytest -q`）もすべて通過した。
- 気づき・疑問：エージェントは起動するたびに接続先の MCP サーバーへ公開ツール一覧を問い合わせるため、クライアント側の設定やコードを一切変えずにサーバー側へ機能を追加できる仕組みを実感した。

