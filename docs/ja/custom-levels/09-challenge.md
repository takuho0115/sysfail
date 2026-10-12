# 9. 最終確認とその場しのぎの検査

## 最終確認（challenge）

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

## その場しのぎの検査（anti_solutions）

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
