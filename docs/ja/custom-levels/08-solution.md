# 8. 模範解答と解説

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
