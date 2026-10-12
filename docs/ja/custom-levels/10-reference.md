# 10. level.yaml の全項目

「必須」の欄が無いとエラーになります。文字数は全角も半角も 1 文字と数えます。

## 最上位

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
| `goals` | 配列 | 必須 | 1〜8 件。[6 章](06-goals.md) | — |
| `hold_seconds` | 整数 | 任意 | 0〜180。成功条件を満たし続ける秒数 | `60` |
| `resolver` | 文字列 | 任意 | DNS の役を担うホスト名 | `dns01` |
| `metrics` | 文字列の配列 | 任意 | 独自の metric 名の宣言（16 件まで）。動作保証外 | — |
| `hints` | 文字列の配列 | 必須 | 1〜8 件、各 1〜300 文字。[7 章](07-hints.md) | — |
| `solution` | オブジェクト | 必須 | [8 章](08-solution.md) | — |
| `verify` | オブジェクト | 任意 | `timeout_seconds`（10〜1200）、`anti_solutions` | — |
| `challenge` | オブジェクト | 任意 | [9 章](09-challenge.md) | — |
| `ssh_exemption` | オブジェクト | 任意 | `reason`（1〜200 文字）。ssh できるホストが 1 つも無いときだけ | — |

`tags` の記号は、障害が何を模しているかの分類です。C はクラウド・外部サービス、K はカーネル・資源（ディスク・メモリ・時刻）、H はハードウェア、O は大きな基盤（Kubernetes 等）、T は長い周期の現象、B はブラウザ側の現象、R は現実の制約、D は Docker とホストの境界です。

## networks の各項目

| 項目 | 型 | 必須 | 内容・制限 |
|---|---|---|---|
| `name` | 文字列 | 必須 | 英小文字で始まる 15 文字まで。`default`・`host`・`none`・`bridge` は使えない |
| `mtu` | 整数 | 任意 | 576〜9000 |
| `routed` | 真偽 | 任意 | ホストがネットワーク間を NAT なしで中継するとき。使うなら 2 つ以上に付ける |

どのホストも入らないネットワークはエラーです（LV-E45）。

## hosts の各項目

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

## entry の各項目

| 項目 | 型 | 必須 | 内容 |
|---|---|---|---|
| `host` | 文字列 | 必須 | 表示するホスト名か `toolbox`（左の隔離シェル） |
| `text` | 文字列 | 必須 | 1〜100 文字。入り方の案内 |

## injects の各項目

| 項目 | 型 | 必須 | 内容 |
|---|---|---|---|
| `host` | 文字列 | 必須 | ホスト名 |
| `script` | 文字列 | 必須 | `hosts/<host>/` からの相対パス。`.sh` で終わり、`..` を含まない |

同じホストとスクリプトの組を 2 回書くとエラーです（LV-E43）。

## solution・verify・challenge

[8 章](08-solution.md)・[9 章](09-challenge.md)の例のとおりです。数の上限は次のとおりです。

| 項目 | 制限 |
|---|---|
| `solution.steps` | 1〜40 件。`run` は 4000 文字まで、`timeout_seconds` は 1〜120（既定 30）、`capture` は 1 手順 4 個・全体 8 個まで |
| `solution.explanation` | 1〜1500 文字 |
| `verify.timeout_seconds` | 10〜1200。`hold_seconds + 10` 以上。既定は `hold_seconds + 120`。「60 ＋ 模範解答の各手順の timeout の合計 ＋ この値」がパックの verify 予算以下（既定 1700 秒） |
| `verify.anti_solutions` | 0〜5 件。各 `name`（1〜60 文字）・`steps`・`expect: not_cleared`・任意の `timeout_seconds`（範囲と注意は「その場しのぎの検査」） |
| `challenge.steps` | 1〜10 件。各 `timeout_seconds` は必須（1〜300）。全体で 600 秒以内 |

## YAML の書き方の制限

アンカー（`&`）・別名（`*`）・マージキー（`<<`）・`!!str` のような明示タグは使えません（LV-E49）。同じキーを 2 回書くこともできません（LV-E51）。`level.yaml` は 256KiB までです。
