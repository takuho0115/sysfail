# 6. 成功条件（goals）の種類

成功条件は 1〜8 件書きます。各項目の `check` には、次の種類のうちちょうど 1 つを書きます。プレイヤーには `text` と任意の `section` が見え、`check` の中身は見えません。

各項目には schema_version: 1 のまま次の任意の欄を指定できます。省略した場合は従来どおり表示・評価されます。

| 欄 | 意味 |
|---|---|
| `id` | レベル内で一意の条件名（1〜64 文字）。`after` を書く条件には必須。参照されない条件は省略可 |
| `section` | グループの表示名（1〜100 文字、空・空白だけは不可）。グループは初出順、グループ内は宣言順 |
| `after: [web, db]` | 指定した全 id が達成されるまで判定を実行せず「保留」と表示。前方参照可、未知の id・自己参照・循環は不可 |
| `sticky: true` | 一度達成したら現在のセッション中は達成を保持。環境のリセット・次レベルで破棄 |

`section`・`after`・`sticky: true` を使うと達成数（例 `3/7`）が表示されます。verify でも全項目の達成が必要で、開始時確認・模範解答・リセット確認・各回避策は別の評価セッションです。sticky はディスクに保存せず、中断再開機能はありません。英語の `section` は `i18n/en.yaml` の `goals.<番号>.section` に書けます（`text` と同じ番号）。

翻訳はレベルディレクトリの `i18n/<lang>.yaml` に表示用の差分を書き、`level.yaml` は既定言語（ja）の文字列を保持します。配列の番号は 1 始まり、省略した項目は日本語のままです。従来の `level.<lang>.yaml` も読み込めますが、両方ある場合は `i18n/<lang>.yaml` を優先します。

| 種類 | 何を見るか | 主な欄 |
|---|---|---|
| `http` | URL の応答 | `url`（必須）、`status`、`contains`、`matches`、`response_headers`、`max_ms` |
| `tcp` | ポートが開いているか | `host`・`port`（必須）、`open`（既定 true）、`from` |
| `dns` | 名前解決の結果 | `name`（必須）、`type`、`server`、`rcode`、`answers_include` |
| `unit` | systemd のユニットの状態 | `host`・`name`（必須）、`active`、`enabled` |
| `process` | プロセスの有無・数 | `host`、`name` か `pattern`、`running`、`min_count`、`max_count` |
| `file` | ファイルの有無・中身・権限 | `host`・`path`（必須）、`exists`、`contains`、`mode`、`owner` |
| `command` | コマンドの結果 | `host`・`run`（必須）、`exit_code`、`stdout_contains`、`stdout_number` |
| `sql` | SELECT の結果 | `host`・`engine`（mariadb か postgresql）・`database`・`query`（必須）、`equals`・`gt` など |
| `metric` | 模擬部品が出す数値 | `host`・`name`・`op`・`value`。**動作保証外**（下の注） |
| `survives` | 再起動後の永続性 | `host`・`restart`（container または service）・`then`（必須）。service の場合は `service` にユニット名 |
| `all_of`・`any_of`・`not` | 組み合わせ | 中に他の種類を並べる（入れ子は 3 段まで、子は 1〜8 件） |

`survives` は他の条件がすべて達成されてから再起動し、`then` を評価します。同じ達成状態では一度だけ実行し、他の条件が未達になってから再び達成すると再実行します。それまでは「前提条件待ち」と表示します。`then` 内に `survives` は置けません。コンテナの再起動は書き込み層を保持し、sf-init の起動完了を待ちます。条件が `sticky` の場合もこの再実行規則を使います。`survives` を含む条件は最後に置き、他の条件からその条件への `after` 依存を作らないでください。verify では達成段階で再起動し、開始時と `--with-reset` 後は `then` の現在の状態だけを観測します。schema_version は 1 のままです。

`http` の `url`・`command` の `run` などには、テンプレートが使えます。`{{ip:web01}}` はホスト web01 の IP アドレス、`{{fqdn:web01}}` は `web01.<domain>`（`domain` が必要）、`{{domain}}` はドメインそのものに置き換わります。

`http` で `contains` だけを書いて `status` を省くと、404 や 500 の本文でも条件を満たしてしまいます（警告 LV-W08）。ふつうは `status` も書きます。

`kind: guard` を付けた条件は「満たし続けなければならない状態」です。一度でも外れるとクリアになりません。「データを消して直す」のような乱暴な解決を禁じたいときに使います。guard だけでは成功条件になりません。

> `metric` は、イメージ内の模擬部品が記録する数値を読みます。模擬部品の設定方法は公開しておらず、バージョンによって変わります。自作ステージでは使わないことをおすすめします。

## 成功条件を書くこつ

- 「直った状態」をプレイヤーの目線で書きます。「nginx の設定を直す」ではなく「http://web01/ が 200 を返す」。
- 回避策では満たせないようにします。サンプルの `contains: 社内ポータル` は、index.html を別のページに置き換えただけでは条件を満たさないようにするための欄です。
- 時間をかけて確かめたいときは `hold_seconds`（0〜180 秒）を使い、どれかの `text` に「60秒」「2分」のような時間を書きます（書かないと LV-E31）。
