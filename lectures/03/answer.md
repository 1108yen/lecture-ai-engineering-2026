# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：第1回の素のLLMは講座独自の用語集を持たないため「分からない」と答えるか一般論の推測を出力したが、第3回のエージェントは宿題の3レベル（基礎・実装・発展）と「どれか1つで完了」というルール、各概要、および「（関連する回：第1回 ）」を正確に出力した。
- `--trace` で見えたツール呼び出し（ツール名と引数）：`[tool call] search_glossary({'query': 'LoRA'}) と [tool call] search_glossary({'query': 'QLORA'})
- 1つの質問での LLM 呼び出し回数（`requests`）：2回
- 追加したツールと、docstring に書いた説明：`make_quiz`（指定した回の用語から1つを選び、確認問題と答えを返す。「第2回の確認問題を出して」のように、復習用の問題を求められたときに使う。lecture には対象の講義回（1, 2 など）を数値で指定する。）

## 宿題

- 取り組んだレベル：実装
- やったこと：
  1. `app/tools.py` に `save_learning_log` 関数を実装し、指定した内容をファイル保存する処理を作成した。
  2. `app/agent.py` で `save_learning_log` に `requires_approval=True` を設定して登録した。
  3. `agent = Agent(..., output_type=[str, DeferredToolRequests])` を設定し、承認待ち状態を受け取れるようにした。
  4. `main` 関数で `DeferredToolRequests` を検知し、ターミナルで `(y/n)` の入力を求める処理を追加した。
  5. 承認（`y`）時に `DeferredToolResults` を渡してエージェントの実行を再開する仕組みを実装した。

- 結果：
  `uv run python -m app.agent "トピック名をLoRA、内容を「事前学習済みの重みを固定して低ランク行列を学習する手法」として学習ログに保存して" --trace` を実行し、承認待ちで一時停止することを確認。`y` を入力して承認することで、保存処理が完了し `learning_log/LoRA.txt` が正しく生成された。
  ```text
  [確認] 保存ツールが呼び出されようとしています。
  ファイルへの保存を許可しますか？ (y/n): y
  指定された学習内容を学習ログに保存しました。

  関連する回：第2回

  --- trace ---
  [tool call]   search_glossary({'query': 'LoRA'})
  [tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
  [tool call]   save_learning_log({'content': '事前学習済みの重みを固定して低ランク行列を学習する手法', 'topic': 'LoRA'})
  [tool return] save_learning_log -> ログを learning_log/LoRA.txt に保存しました。…
  [usage] RunUsage(cost=Decimal('0.0003202'), details={'text_prompt_tokens': 909}, input_text_tokens=909, input_tokens=909, output_tokens=19, requests=1, tool_calls=1)

  $ cat learning_log/LoRA.txt
  事前学習済みの重みを固定して低ランク行列を学習する手法

- 気づき・疑問：
  - エージェントのツール呼び出し行動は、プログラムのロジックだけでなく、関数の docstring（自然言語の説明文）に強く依存していることが確認できた。指示を工夫することでツール呼び出し回数（`tool_calls`）を削減でき、LLMとの往復コストやレイテンシの最適化につながると分かった。
  - プロンプト（docstring）だけで確認させようとすると、モデルがチャット上で確認メッセージを返すだけでツール自体を呼ばなくなってしまう。安全機構として確実に一時停止させるには、フレームワーク側の `requires_approval=True` を使ってモデルには直接ツールを呼ばせ、システム層でインターセプトするのが適切であることが理解できた。
  - 中断されたツール呼び出しの再開には、単純な再実行ではなく各 `tool_call_id` の承認状態を持った `DeferredToolResults` を明示的に渡す必要があることを学んだ。
