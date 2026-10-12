# 自作ステージの作り方

sysfail では、同梱の 100 レベルとは別に、自分で作った障害のステージで遊べます。このガイドは、ステージを 1 つ作って検証し、遊ぶまでの手順と、`level.yaml` に書ける項目の一覧をまとめたものです。

はじめての方は 1〜4 章を順に読んでください。サンプルをコピーして動かすだけで、作り方の流れはつかめます。5 章以降は、項目や判定の種類を調べたいときに引くリファレンスです。

> この機能は将来の版に向けた準備段階です。書式や制限は変わることがあります。

## 1. まずサンプルを動かす

配布物の `custom-levels-kit.tar.gz` を展開すると、`custom-levels/examples/custom-level/` にサンプルのステージ `L101-portal-forbidden` が入っています。社内ポータルが 403 を返すようになった、という小さな障害です。

```
custom-level/
└── L101-portal-forbidden/
    ├── level.yaml                 ステージの定義
    └── hosts/
        └── web01/                 ホスト web01 用のファイル
            ├── setup.sh           正常な状態を作る
            ├── inject.sh          障害を起こす
            └── files/             コンテナにそのまま置くファイル
                └── usr/share/nginx/html/index.html
```

検証し、一覧に出し、遊ぶ。この 3 つのコマンドで一通りの流れを試せます。

```sh
tar -xzf custom-levels-kit.tar.gz
cd custom-levels/examples
sysfail level check custom-level                 # 書式の検証（Docker 不要）
sysfail --levels-dir custom-level list           # 一覧に「自作」として出る
sysfail --levels-dir custom-level 101            # 遊ぶ
```

`--levels-dir` を付けている間は、同梱のレベルは一覧に出ません。外せば元に戻ります。

Windows のランチャー（`sysfail-windows-amd64.exe`）では、`--levels-dir C:\Users\me\stages` のように Windows のパスも渡せます。ランチャーが WSL の中の `/mnt/c/Users/me/stages` に読み替えて渡します（WSL の自動マウント先を `/mnt` 以外に変えている場合は、Linux 側のパスを直接書いてください）。相対パスは使えません。

## 2. ステージの骨組み

### ディレクトリ名と番号

ステージ 1 つにつき、ディレクトリを 1 つ作ります。名前は `L<番号>-<短い名前>` です。

- 番号は **101〜999** の 3 桁です。1〜100 は同梱レベル用なので使えません。
- 短い名前は英小文字・数字・ハイフン（41 文字まで）。例: `L101-portal-forbidden`、`L230-dns-typo`。
- `level.yaml` の `number` は、ディレクトリ名の番号と同じにします。

複数のステージを 1 つのディレクトリに並べて置けます。そのディレクトリを `--levels-dir` に渡します。同じ番号が 2 つあると、どちらも読み込まれません。

### ステージが動く順番

ゲームを始めると、次の順に進みます。

1. `level.yaml` の `hosts` ごとにコンテナ（el9 イメージ）を作る
2. `hosts/<ホスト>/files/` の中身を、コンテナの同じパスに置く
3. `hosts/<ホスト>/setup.sh` を root で実行する（正常な状態を作る）
4. `hosts/<ホスト>/inject.sh` を root で実行する（障害を起こす）
5. プレイヤーが調査を始める。成功条件（`goals`）を数秒おきに判定し、すべて満たしたらクリア

setup.sh と inject.sh を分けておくのは、自動検証が「障害の前は正常に動くはず」「障害の後は成功条件を満たさないはず」を区別して確かめられるようにするためです。

### ホストに置くファイル

`files/` の下には、コンテナのルート（`/`）からのパスでファイルを置きます。`files/etc/myapp.conf` はコンテナの `/etc/myapp.conf` になります。シンボリックリンクは置けません。ステージのディレクトリ全体で 8MiB・500 ファイルまでです。

### setup.sh と inject.sh

どちらも bash のスクリプトで、コンテナの中で root として動きます。ホスト（手元のパソコン）では動きません。

- setup.sh は 9 秒以内、inject.sh は 10 秒以内に終わるようにします。データベースへの大量投入のような重い準備は inject.sh に回すか、出来上がったデータを `files/` に置きます。
- 1 ファイル 256KiB まで、UTF-8 で書きます。
- inject.sh が無いホストでは、障害の注入はしません。setup.sh だけで障害の状態を作ることもできます。
- 注入の順番を決めたいときは、`level.yaml` の `injects` に並べます（10 章）。

サンプルの inject.sh は 1 行だけです。

```bash
#!/bin/bash
# 朝の更新作業の事故: 差し替えたトップページが root だけ読める権限になる
chmod 600 /usr/share/nginx/html/index.html
```

スクリプトのコメントはプレイヤーには見えません。ただし、障害の原因をファイルとしてコンテナ内に残すと（たとえば「ここで権限を壊した」と書いたメモを置くと）、`grep -r` で答えが見つかってしまいます。原因を示す痕跡は、本物の障害でも残るもの（ログ・設定・権限）だけにします。

## 3. level.yaml を書く

最小限の `level.yaml` は次の形です。サンプルから必要なところを書き換えるのが早道です。

```yaml
schema_version: 1
number: 101
title: 社内ポータルが開けない            # 症状で書く（原因を書かない）
fault_name: 公開ファイルの権限不足        # 作者用のメモ。画面には出ない
target_minutes: 5
brief: |
  【10:15 総務部より】
  社内ポータル (http://web01/) を開くと「403 Forbidden」と表示されます。
entry:
  - host: web01
    text: 左のペインで `ssh web01` と入力するとログインできる
hosts:
  - name: web01
    role: 社内ポータル Webサーバー
    image: el9
goals:
  - text: http://web01/ が HTTP 200 で社内ポータルのトップページを返す
    check:
      http:
        url: http://web01/
        status: 200
        contains: 社内ポータル
hints:
  - "まずは `curl -I http://localhost/` で、サーバー自身から見ても同じ応答かを確かめよう。"
  - "nginx のエラーログには、断られた理由が書いてある。"
  - "nginx が動いているユーザーと、ファイルの権限を見比べよう。"
solution:
  steps:
    - on: web01
      user: root
      run: chmod 644 /usr/share/nginx/html/index.html
  explanation: |
    【原因】index.html の権限が 600 になっていて、nginx が読めませんでした。
```

`title` と `brief` には、`fault_name` の文字列をそのまま含めることはできません（LV-E20）。プレイヤーが最初に読む文で答えが分からないようにするための検査です。

## 4. 検証して遊ぶ

### level check で書式を確かめる

```sh
sysfail level check <ステージを並べたディレクトリ>
```

Docker を使わず、数秒で終わります。問題があると次のように出ます。

```
エラー LV-E06 [L005-mine number] 自作ステージの番号は 101〜999 の範囲である必要があります
OK L101 社内ポータルが開けない
検証: エラー 1 件。エラーのあるステージは遊べません
```

エラーのあるステージは一覧に出ず、遊べません。警告（`LV-W…`）があっても遊べますが、多くは書き漏れの知らせなので直しておきましょう。終了コードは、すべて通れば 0、エラーがあるかステージが 1 件も無ければ 1 です。

### verify で中身を確かめる

`level check` は書式しか見ません。障害が本当に起きるか、模範解答で本当に直るかは、`verify` で確かめます。Docker で実際に環境を作るので、1 ステージ数分かかります。

```sh
sysfail --levels-dir <dir> verify 101          # 1 ステージ
sysfail --levels-dir <dir> verify 101-110      # 範囲
sysfail --levels-dir <dir> verify --all        # ディレクトリ内の全部
sysfail --levels-dir <dir> verify --with-reset 101   # やり直し後も確かめる
```

verify が確かめることは 4 つです。

| 段階 | 確かめること | 落ちたときに疑うところ |
|---|---|---|
| 開始直後 | 成功条件を満たしていない | inject.sh が効いていない。成功条件が緩すぎる |
| 模範解答の後 | 成功条件をすべて満たす（`hold_seconds` の間続く） | 模範解答の手順の漏れ。成功条件が厳しすぎる。待ち時間（`verify.timeout_seconds`）不足 |
| その場しのぎ | `anti_solutions` の手順ではクリアにならない | 成功条件が回避策でも満たせてしまう |
| 外部到達 | ステージの環境にパソコンの外から届かない | 通常は作者の書き方では起きない |

`--difficulty` は自作ステージでは使えません。自作ステージには難易度の帯が無く、一覧では「自作」と表示されます。

### 遊ぶ

```sh
sysfail --levels-dir <dir>        # 選択画面（自作ステージだけが並ぶ）
sysfail --levels-dir <dir> 101    # 番号を指定して始める
```

クリアの記録は同梱レベルとは別のファイル（状態ディレクトリの `progress-custom-<12 桁の英数字>.json`）に残ります。ファイルは `--levels-dir` に渡したディレクトリごとに分かれるので、別の作者の L101 と記録が混ざることはありません。教育版（`--edu`）は自作ステージでは使えません。

## 5. 他人が作ったステージを遊ぶときの注意

自作ステージの setup.sh・inject.sh・模範解答は、すべてゲームのコンテナの中で動きます。sysfail があなたのパソコンで作者のコマンドを直接実行することはありません。コンテナには、同梱レベルと同じ制限（特権なし、ホストのディレクトリやポートを使わない、許可した権限だけ）がかかっています。

それでも、次の点は作者を信頼するしかありません。

- **コンテナの中身は作者のコードです。** Docker の隔離が破られる不具合（カーネルの脆弱性など）があれば、そこを突くコードを書くこともできます。
- **資源を浪費させることはできます。** 決められた範囲で CPU・メモリを使い切ったり、時間のかかる処理を仕込んだりできます。
- **文章で誘導されることがあります。** brief やヒントに「パソコンのターミナルでこのコマンドを実行して」と書かれていても、ゲームの外では従わないでください。sysfail の障害対応は、すべてゲームの画面の中で完結します。

遊ぶ前に `level.yaml` と `hosts/*/setup.sh`・`inject.sh` を読み、何をするステージなのかを確かめてください。中身を読めないもの・作者が分からないものは遊ばないのが安全です。一覧と選択画面には、自作ステージであることを示す注意書きが常に表示されます。

## 6. 成功条件（goals）の種類

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

### 成功条件を書くこつ

- 「直った状態」をプレイヤーの目線で書きます。「nginx の設定を直す」ではなく「http://web01/ が 200 を返す」。
- 回避策では満たせないようにします。サンプルの `contains: 社内ポータル` は、index.html を別のページに置き換えただけでは条件を満たさないようにするための欄です。
- 時間をかけて確かめたいときは `hold_seconds`（0〜180 秒）を使い、どれかの `text` に「60秒」「2分」のような時間を書きます（書かないと LV-E31）。

## 7. ヒントの書き方

ヒントは 1〜8 件（3 件未満は警告 LV-W01）で、プレイヤーが求めた順に 1 つずつ出ます。

**ヒントに答えを書かない**のが一番の作法です。sysfail のヒントは「どこを見ればよいか」「何と何を比べればよいか」を示すもので、直し方そのもの（`chmod 644 …` のようなコマンド）は書きません。答えは模範解答の解説（`solution.explanation`）に書きます。

1 つ目は調べ始めの方向、2 つ目は手がかりの場所、3 つ目は比べる対象、と少しずつ近づけると、自分で気づく余地が残ります。

| 避ける | こう書く |
|---|---|
| index.html の権限を 644 に直そう。 | nginx が動いているユーザーと、ファイルの権限を見比べよう。 |
| 設定ファイルの 12 行目の listen が間違っている。 | `nginx -t` で、設定ファイルのどこで止まるかを確かめよう。 |

障害の名前（`fault_name`）を、ヒントや `title`・`brief`・成功条件の `text` に書かないのも同じ理由です。

## 8. 模範解答と解説

`solution.steps` は verify が実行する手順で、プレイヤーが降参したときにも表示されます。

```yaml
solution:
  steps:
    - on: web01              # ホスト名か toolbox
      user: root             # 省略時はホストでは root、toolbox ではプレイヤー
      run: chmod 644 /usr/share/nginx/html/index.html
      comment: nginx のワーカーが読めるように権限を戻す
      timeout_seconds: 30    # 1〜120。省略時 30
  explanation: |
    【原因】…
    【復旧】…
    【実務でのポイント】…
```

前の手順の出力を後の手順で使いたいときは `capture` を使います。`capture: {PID: 'pid=([0-9]+)'}` と書くと、後の手順の `run` で `{{var:PID}}` として参照できます。

`explanation` は 1500 文字まで。「原因」「復旧」「実務でのポイント」の 3 段落に分けると、同梱レベルと同じ構成になります。

## 9. 最終確認とその場しのぎの検査

### 最終確認（challenge）

成功条件を満たした後、クリアの前に「本当に直ったか」を試す仕組みです。サービスの再起動やコンテナの再起動をかけて、それでも条件が保たれるかを確かめます。その場でプロセスを起動しただけで、再起動すると元に戻ってしまう直し方を見分けられます。

```yaml
challenge:
  text: 再起動後もポータルが表示されるか確かめています
  steps:
    - on: web01
      run: systemctl restart nginx
      timeout_seconds: 30
  settle_seconds: 5     # 手順の後に待つ秒数（0〜120、既定 5）
  hold_seconds: 20      # その後、条件を満たし続ける秒数（1〜300）
```

`steps` は 1〜10 件、片付け用の `cleanup` は 0〜5 件です。`{{var:…}}` は使えません。

### その場しのぎの検査（anti_solutions）

`verify.anti_solutions` に、クリアにしてはいけない直し方を最大 5 件書きます。verify は模範解答の代わりにこの手順を実行し、クリアにならないことを確かめます。

```yaml
verify:
  anti_solutions:
    - name: 別のページで置き換える
      steps:
        - on: web01
          user: root
          run: rm -f /usr/share/nginx/html/index.html && echo ok > /usr/share/nginx/html/index.html
      expect: not_cleared
```

回避策ごとに `timeout_seconds` を追加すると、判定の待ち時間を短くできます。既定は省略し、実測の根拠があるときだけ書いてください。正の値の下限は `max(60, 判定条件の metric.window_seconds の最大 + 30)`（metric が無ければ 60）、上限は既定値を埋めた後の `verify.timeout_seconds` です。レベルの待ち時間が 60 秒未満なら上書きできません。省略または 0 以下は従来の待ち時間、正なら `min(上書き値, レベルの実効値)` を使います。

上書き値に達しても、直前の評価で未決定の条件が残っていれば従来の待ち時間まで延長します。`verify --ignore-anti-timeout` または `SYSFAIL_VERIFY_IGNORE_ANTI_TIMEOUT=1` で上書きを無視できます（CI の定期実行と verify-diff でも使います）。この新しいキーを含む定義は、旧バージョンのバイナリでは未知のキーとして拒否され、読み込めません。

「プレイヤーがやりそうな雑な直し方」を思いつくたびに足しておくと、成功条件の甘さに早く気づけます。

## 10. level.yaml の全項目

「必須」の欄が無いとエラーになります。文字数は全角も半角も 1 文字と数えます。

### 最上位

| 項目 | 型 | 必須 | 内容・制限 | 例 |
|---|---|---|---|---|
| `schema_version` | 整数 | 必須 | 常に 1 | `1` |
| `number` | 整数 | 必須 | 101〜999。ディレクトリ名の番号と同じ | `101` |
| `title` | 文字列 | 必須 | 1〜40 文字、1 行。症状で書く | `社内ポータルが開けない` |
| `fault_name` | 文字列 | 必須 | 1〜60 文字。作者用で画面に出ない | `公開ファイルの権限不足` |
| `tags` | 文字列の配列 | 任意 | `C K H O T B R D` から。重複なし。分類用 | `[K]` |
| `target_minutes` | 整数 | 必須 | 1〜60。評価（S〜C）の基準時間 | `5` |
| `brief` | 文字列 | 必須 | 1〜600 文字、改行可。症状だけを書く | — |
| `domain` | 文字列 | 任意 | 小文字の FQDN。`{{fqdn:…}}` を使うなら必須 | `example.internal` |
| `networks` | 配列 | 任意 | 1〜4 件。省略時は `lan` の 1 つ | 下表 |
| `hosts` | 配列 | 必須 | 1〜6 件（5 件以上は警告） | 下表 |
| `entry` | 配列 | 必須 | 1〜4 件。調査の入口 | 下表 |
| `injects` | 配列 | 任意 | 注入の順番。省略時は hosts の順に inject.sh があるものすべて | 下表 |
| `goals` | 配列 | 必須 | 1〜8 件。6 章 | — |
| `hold_seconds` | 整数 | 任意 | 0〜180。成功条件を満たし続ける秒数 | `60` |
| `resolver` | 文字列 | 任意 | DNS の役を担うホスト名 | `dns01` |
| `metrics` | 文字列の配列 | 任意 | 独自の metric 名の宣言（16 件まで）。動作保証外 | — |
| `hints` | 文字列の配列 | 必須 | 1〜8 件、各 1〜300 文字。7 章 | — |
| `solution` | オブジェクト | 必須 | 8 章 | — |
| `verify` | オブジェクト | 任意 | `timeout_seconds`（10〜1200）、`anti_solutions` | — |
| `challenge` | オブジェクト | 任意 | 9 章 | — |
| `ssh_exemption` | オブジェクト | 任意 | `reason`（1〜200 文字）。ssh できるホストが 1 つも無いときだけ | — |

`tags` の記号は、障害が何を模しているかの分類です。C はクラウド・外部サービス、K はカーネル・資源（ディスク・メモリ・時刻）、H はハードウェア、O は大きな基盤（Kubernetes 等）、T は長い周期の現象、B はブラウザ側の現象、R は現実の制約、D は Docker とホストの境界です。

### networks の各項目

| 項目 | 型 | 必須 | 内容・制限 |
|---|---|---|---|
| `name` | 文字列 | 必須 | 英小文字で始まる 15 文字まで。`default`・`host`・`none`・`bridge` は使えない |
| `mtu` | 整数 | 任意 | 576〜9000 |
| `routed` | 真偽 | 任意 | ホストがネットワーク間を NAT なしで中継するとき。使うなら 2 つ以上に付ける |

どのホストも入らないネットワークはエラーです（LV-E45）。

### hosts の各項目

| 項目 | 型 | 必須 | 内容・制限 | 例 |
|---|---|---|---|---|
| `name` | 文字列 | 必須 | 英小文字で始まる 31 文字まで。`toolbox`・`probe`・`reach`・`localhost`・`sysfail`・`watchdog` は使えない | `web01` |
| `role` | 文字列 | 必須 | 1〜30 文字。画面に出る役割 | `Webサーバー` |
| `image` | 文字列 | 必須 | `el9`・`el9-empty`・`el9-repo`（pack.yaml の `images` にも宣言） | `el9` |
| `ssh` | 真偽 | 任意 | ssh でログインできるか（既定 true） | `false` |
| `visible` | 真偽 | 任意 | 画面のホスト一覧に出すか（既定 true） | `false` |
| `networks` | 文字列の配列 | 任意 | 入るネットワーク（1〜4）。省略時は最初のネットワーク | `[front, back]` |
| `address` | 整数かオブジェクト | 任意 | ネットワーク内のアドレスの末尾（2〜239）。複数ネットワークなら `{front: 10}` | `10` |
| `aliases` | 文字列の配列 | 任意 | 別名（4 件まで） | `[shop]` |
| `limits` | オブジェクト | 任意 | `memory`（32m〜512m）、`cpus`（0.1〜1.0）、`pids`（64〜2048） | `{memory: 256m}` |
| `tmpfs` | オブジェクト | 任意 | パス → 大きさ（1m〜128m、合計 256m）。ディスクを埋める障害はここで | `{/var/log/app: 16m}` |
| `caps` | 文字列の配列 | 任意 | `NET_RAW`・`SYS_PTRACE` だけ | `[SYS_PTRACE]` |
| `netctl` | 真偽 | 任意 | iptables・tc・ip route などで通信を操作するとき true | `true` |
| `sysctls` | オブジェクト | 任意 | `net.ipv4.*`・`net.ipv6.*`・`net.core.somaxconn` など名前空間付きのものだけ | — |
| `env` | オブジェクト | 任意 | 環境変数（16 件まで）。`SF_`・`SYSFAIL_` で始まる名前は使えない | `{APP_ENV: prod}` |

`privileged`・`volumes`・`ports`・`devices` などコンテナの権限を広げる欄は書けません（LV-E10）。

構築用の `el9-empty` は Rocky 9 と既存の sf-init（systemd 互換）を使い、SSH/sudo と
init に必要な Python などの基本ツールだけを導入しています。nginx・MariaDB・chrony・cron
などのミドルウェアは未導入です。dnf の接続先は `http://repo01/repo/` だけです。
同じネットワークに次の隠しホストを置き、pack.yaml の `images` に
`[el9-empty, el9-repo]`（ほかに使うホストイメージがあればそれも）を宣言してください。
既存の core/shell パックの許可リストは変わりません。

```yaml
# level.yaml の hosts に追加（ネットワーク省略時は最初のネットワーク）
- name: repo01
  role: ローカルパッケージミラー
  image: el9-repo
  ssh: false
  visible: false
```

`el9-repo` はビルド時に `images/el9-repo/packages.txt` の RPM と依存パッケージを
取得し、署名を検証してリポジトリを作ります。実行時は外部通信なしで HTTP の 80 番を
使います。ホスト側へのポート公開やレベルごとの setup.sh は不要です。
収録範囲は build パックの基礎 1〜20 用です（nginx、MariaDB、chrony、logrotate、
firewalld/nftables、cronie、rsync、OpenSSH、policycoreutils、httpd-tools、Python など）。

### entry の各項目

| 項目 | 型 | 必須 | 内容 |
|---|---|---|---|
| `host` | 文字列 | 必須 | 表示するホスト名か `toolbox`（左の隔離シェル） |
| `text` | 文字列 | 必須 | 1〜100 文字。入り方の案内 |

### injects の各項目

| 項目 | 型 | 必須 | 内容 |
|---|---|---|---|
| `host` | 文字列 | 必須 | ホスト名 |
| `script` | 文字列 | 必須 | `hosts/<host>/` からの相対パス。`.sh` で終わり、`..` を含まない |

同じホストとスクリプトの組を 2 回書くとエラーです（LV-E43）。

### solution・verify・challenge

8 章・9 章の例のとおりです。数の上限は次のとおりです。

| 項目 | 制限 |
|---|---|
| `solution.steps` | 1〜40 件。`run` は 4000 文字まで、`timeout_seconds` は 1〜120（既定 30）、`capture` は 1 手順 4 個・全体 8 個まで |
| `solution.explanation` | 1〜1500 文字 |
| `verify.timeout_seconds` | 10〜1200。`hold_seconds + 10` 以上。既定は `hold_seconds + 120`。「60 ＋ 模範解答の各手順の timeout の合計 ＋ この値」がパックの verify 予算以下（既定 1700 秒） |
| `verify.anti_solutions` | 0〜5 件。各 `name`（1〜60 文字）・`steps`・`expect: not_cleared`・任意の `timeout_seconds`（範囲と注意は「その場しのぎの検査」） |
| `challenge.steps` | 1〜10 件。各 `timeout_seconds` は必須（1〜300）。全体で 600 秒以内 |

### YAML の書き方の制限

アンカー（`&`）・別名（`*`）・マージキー（`<<`）・`!!str` のような明示タグは使えません（LV-E49）。同じキーを 2 回書くこともできません（LV-E51）。`level.yaml` は 256KiB までです。

## 11. よくある失敗

**`level check` で LV-E06 が出る。** 番号が 1〜100 になっています。自作ステージは 101〜999 です。ディレクトリ名と `number` の両方を直します。

**LV-E04 が出る。** ディレクトリ名の番号と `number` が違うか、ディレクトリ名が `L101-name` の形になっていません（大文字・空白・アンダースコアは使えません）。

**LV-E57 が出る。** ステージを並べたディレクトリに `_common` があります。共通のファイルは同梱のものが自動で入るので、自作ステージには置けません。各ホストの `files/` に置いてください。

**verify の「開始直後」で落ちる。** 障害が起きていません。inject.sh が 10 秒以内に終わっているか、実行権限やパスが合っているかを、ゲームを始めて実際にホストに入って確かめます。

**verify の「模範解答の後」で落ちる。** 手順が途中で失敗しているか、サービスの起動に時間がかかっています。`hold_seconds` を使っているなら、`verify.timeout_seconds` が足りているかも見ます。

**その場しのぎで直してもクリアになってしまう。** 成功条件がゆるすぎます。サンプルで `status: 200` だけを見ていたら、index.html を `echo ok` で作り直すだけでクリアできてしまいます。ページの中身（`contains`）や設定の状態（`file`・`unit`）まで見るか、`challenge` で再起動後の状態を確かめます。

**ヒントだけで解けてしまう。** ヒントに直し方そのものを書いています。7 章の表のように「見る場所」に書き換えます。

**一覧に出ない。** `level check` でエラーが出ていないか確かめます。エラーのあるステージは一覧から外れます。`--levels-dir` に渡すのは、`L101-…` を並べた親のディレクトリです（ステージのディレクトリそのものではありません）。

## 用語

| 用語 | 意味 |
|---|---|
| ステージ | `L<番号>-<名前>/` のディレクトリ 1 つ。同梱のものは「レベル」と呼ぶ |
| 成功条件（goals） | クリアに必要な状態。プレイ中に数秒おきに判定される |
| 模範解答（solution） | 成功条件を満たす手順。verify が実行し、降参したプレイヤーにも見せる |
| 最終確認（challenge） | 成功条件を満たした後、クリアの前に行う再起動などの確認 |
| その場しのぎの検査（anti_solutions） | クリアにしてはいけない直し方。verify だけが実行する |
| toolbox | 画面左の隔離シェル。プレイヤーが最初にいる場所 |

### pack.yaml の verify 予算

`pack.yaml` に任意の `verify_budget_seconds`（整数、1〜3600 秒）を指定できます。
省略時は従来どおり 1700 秒です。例:

```yaml
verify_budget_seconds: 3600
```

LV-E52 は `60 + 模範解答の手順 timeout の合計 + challenge の時間 + verify.timeout_seconds`
をこの予算と比較します。手順の timeout を省略すると 30 秒として数えます。
回避策は 1 件ずつ、模範解答の手順と待ち時間をその回避策の実効値に置き換えて
同じ予算で検査します。`verify.timeout_seconds` 自体の上限 1200 秒は変わりません。
範囲外は LV-E17、整数以外は LV-E03 です。CI は予算が 1700 秒を超えるパックで
1 シャードのレベル数を予算に応じて減らします。
