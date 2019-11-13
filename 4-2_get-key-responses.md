# はじめに

本記事は，「PsychoPy Coderによる心理学実験作成チュートリアル」の第4回の補足記事です。[第4回](https://qiita.com/snishym/items/0b10e714363656094181)では`event.waitKeys()`を利用したキー反応の取得を紹介しました。この補足では，`event.waitKeys()`を使わないキー反応の取得方法について紹介します。合わせて，Pythonの基本構文の一つである`while`について紹介します。

# `event.waitKeys()`では困る場面

`event.waitKeys()`は反応があるまで処理を待機する関数になります。本チュートリアルのサイモン課題のように利用すれば，参加者のキー反応を待ち，反応が得られるとともに次の試行に進みます。しかし，反応とともに次の試行に移ってしまうため，反応の有無に関わらず刺激を一定時間提示したり，刺激の提示中に複数回の反応を取得したりしたいという場合には`event.waitKeys()`は利用できません。
そのような場合に有用なキー反応取得方法として今回は

- `core.wait()` + `event.getKeys()`
- `while` + `event.getKeys()`

の2つを紹介します。1つ目のほうがシンプルですが，私個人は2つ目のほうを利用しています（理由は後述）。

# `core.wait()` + `event.getKeys()`

まず初めに，`core.wait()` と `event.getKeys()` を使ったキー反応の取得方法を紹介します[1]。`core.wait()`は[第1回](https://qiita.com/snishym/items/5c33710aca0fea3d41ec)，[第2回](https://qiita.com/snishym/items/31c980f08e62190469f9)，[第3回](https://qiita.com/snishym/items/1bdf90cf51ba28cc4ef3)で出てきました。`()`内に指定した秒数だけ処理を待機する関数です。
`event.getKeys()`は名前の通りキー入力を取得する関数です。ただし，`event.waitKeys()`と違い，キー入力を待ちませんので，この関数を単独で使っても一瞬で次の処理に進んでしまうので，キーを取得するのはかなり難しいです。次のコードを実行してみましょう。

[1]: この方法はwindow描画のバックエンドで`pyglet`を利用する場合にしか利用できないようです（[公式リファレンス](https://www.psychopy.org/api/core.html#psychopy.core.wait)）。描画のバックエンドはPsychoPyの設定>一般>winTypeで確認できます[十河先生のサイト](http://www.s12600.net/psy/python/18-2.html)（ちょっと古いですが参考になるはずです）。デフォルトでは`pyglet`になっているはずです。現在利用できるバックエンドは[3種類あります](https://www.psychopy.org/api/visual/window.html#psychopy.visual.Window)。

```python:get_key_response_severe.py
from psychopy import visual, core, event

win = visual.Window()

instruction = visual.TextStim(win, "いずれかのキーを押してください")
instruction.draw()
win.flip()

resp = event.getKeys() # 押されたキーをrespという名前で利用する
print(resp) # 出力

win.close()
```

どのような出力が得られたでしょうか。多くの人は`None`が得られたのではないでしょうか。そもそも一瞬で画面の提示が終わってしまったはずです。
このままでは全く使い物になりません。それでは`core.wait()`を組み合わせて5秒待つようにした以下のコードを実行してみましょう。だいたい1秒間隔でキー入力をしてみてください。なお，あとの説明のために反応時間も取得するようにしています。

```python:get_key_response_bad.py
from psychopy import visual, core, event

win = visual.Window()
instruction = visual.TextStim(win, "いずれかのキーを押してください")

stopwatch = core.Clock() # stopwatchという名前の時計を用意

instruction.draw()
win.flip()

stopwatch.reset() # 時計の時間をリセットする
core.wait(5) # 5秒待つ
resp = event.getKeys(timeStamped = stopwatch) # timeStampedに用意した時計を指定
print(resp)

win.close()
```

さて，どのような出力が得られたでしょうか。`[['space', 4.8015793999657035], ['space', 4.802112399949692], ['space', 4.80252439994365], ['space', 4.802825400023721]]`みたいな結果が得られたのではないでしょうか。確かに1秒，2秒，3秒，4秒時点で反応していたとしたら4つ反応が得られていることに間違いはないですが，出力されている秒数がどれもだいたい4.8秒ほどになってしまっています。全く意図したように動いておらず，これも使い物になりません。
それでは，次のコードを実行してみましょう。同様に1秒間隔でキー入力をしてみてください。

```python:get_key_response_good.py
from psychopy import visual, core, event

win = visual.Window()
instruction = visual.TextStim(win, "いずれかのキーを押してください")

stopwatch = core.Clock() # stopwatchという名前の時計を用意

instruction.draw()
win.flip()

stopwatch.reset() # 時計の時間をリセットする
core.wait(5, hogCPUperiod=5.0) # 5秒待つ
resp = event.getKeys(timeStamped = stopwatch) # timeStampedに用意した時計を指定
print(resp)

win.close()
```

どうでしょうか。いい感じに反応を取得できたのではないでしょうか。先程のコードとの違いは， `core.wait()`で`hogCPUperiod=5.0`としている点です。`core.wait()`は最初に指定した秒数だけPsychoPyの処理を止めているのですが，その次に`hogCPUperiod=秒数`という引数を指定することで，待ち時間の終了何秒前からバックグラウンドの処理を始めるかを決めることができます。
それではなぜ，先程のコードでは，反応時間が4.8程度だと出力されたのでしょうか。そのポイントは，`event.getKeys()`のキー入力の処理方法と`core.wait()`の`hogCPUperiod`のデフォルト値にあります。
まず，PsychoPyにおいて，キー入力は`eventbuffer`というところで保持されます。そこで保持されたキー入力は`event.getKeys()`が実行された時点で処理されます。この引数のデフォルトは`hogCPUperiod=0.2`となっています（[公式リファレンス](https://www.psychopy.org/api/core.html)）。このため，先程のコードでは，処理されずに保持されていた