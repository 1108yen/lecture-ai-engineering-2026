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

- 取り組んだレベル：実装
- やったこと：第3回で作成した `make_quiz` を `app/tools.py` に追加し（存在しない回が指定された場合は「第{lecture}回の用語は見つかりませんでした。指定できる講義回は第1回から第6回までです。」と返すようにした）、`app/mcp_server.py` に `server.tool()(tools.make_quiz)` を追加して MCP ツールとして公開した。その上で `--trace` 付きで正常系（`"第2回のクイズを作って"`）と範囲外（`"第99回のクイズを作って"`）の質問を実行し、自動テスト（`python -m pytest -q`）も確認した。
- 結果：`--list-tools` で `glossary_make_quiz` が増え、ツール数は計9個になった。正常系では `[tool call] glossary_make_quiz({'lecture': 2})` からクイズ JSON が返り、「過学習」の確認問題が出題された。範囲外（`lecture: 99`）ではツールが返したメッセージを受け取ったモデルが推測で問題を作らず、第99回が存在しないため作成できない旨を丁寧に回答した。`python -m pytest -q` も全21件通過した。
- 気づき・疑問：MCP サーバー側のツールが返したエラー文字列はそのままモデルの次の入力（`[tool return]`）として渡るため、単に失敗させるのではなく「なぜ失敗したか・どの範囲なら有効か」をメッセージに書いておくと、モデルが推測せず適切な案内を返せる。また、実装途中で `import random` を漏らして `[tool error]` が出た際は、モデルが自律的に `glossary_get_lecture_summary` を呼び出して代替回答を作る動きも `--trace` で確認できた。
