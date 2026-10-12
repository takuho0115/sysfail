# 3. level.yaml を書く

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
