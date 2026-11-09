# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：素のLLMは学習済みの知識をもとに回答するが、エージェントはツールで用語集を検索し、講座固有の情報を根拠に回答できる。
- `--trace` で見えたツール呼び出し（ツール名と引数）：`search_glossary({'query': 'LoRA'})`、`search_glossary({'query': 'QLoRA'})`。
- 1つの質問での LLM 呼び出し回数（`requests`）：2回。最初の応答で2つの検索ツールを呼び出し、検索結果を受け取った後の応答で最終回答した。
- 追加したツールと、docstring に書いた説明：`make_quiz(lecture)` を追加。指定した回の用語を1つ選び、「用語の意味は？」という問題と定義をJSON文字列で返す。「第2回の確認問題を出して」のような復習問題の依頼時に使うと説明した。

## 宿題

- 取り組んだレベル：実装
- やったこと：`save_learning_log` を追加し、学習記録を日付ごとのMarkdownファイルに追記するようにした。ツールを `requires_approval=True` で登録し、CLIで承認・拒否を聞いて、`DeferredToolResults` と会話履歴を渡してエージェントを再実行するようにした。
- 結果：
(aie2026-exercises) @tsubastas ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --trace "今日の学習記録として、承認付きツールの仕組みを学んだことを保存してください。"
[model] gemini-3.5-flash-lite

承認が必要な操作: save_learning_log({'content': '承認付きツールの仕組みを学んだ'})
実行して学習記録を保存しますか？ [y/N]: y
学習記録を保存しました。

（関連回：第11回）

--- trace ---
[tool call]   save_learning_log({'content': '承認付きツールの仕組みを学んだ'})
[tool return] save_learning_log -> 学習記録を learning_log/2026-11-09.md に保存しました。…
[usage] RunUsage(cost=Decimal('0.0002248'), details={'text_prompt_tokens': 616}, input_text_tokens=616, input_tokens=616, output_tokens=16, requests=1, tool_calls=1)

承認プロンプトで `y` を入力すると `learning_log/2026-11-09.md` に「承認付きツールの仕組みを学んだ」と保存された。trace では `save_learning_log` が1回呼ばれ、LLM呼び出しは `requests=1`、ツール呼び出しは `tool_calls=1` だった。別途、承認時のみ保存され拒否時には保存されないことも一時ディレクトリで確認し、全15テストが成功した。
- 気づき・疑問：保存記録に根拠がないのに回答で「関連回：第11回」と表示された。全体指示が毎回関連回を示すよう求めていたため、用語集や講義要点で確認できた回だけを示し、確認できなければ省略するよう指示を修正できると思う。
