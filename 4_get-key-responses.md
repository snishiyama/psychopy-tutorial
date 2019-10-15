# はじめに

本記事は，「PsychoPy Coderによる心理学実験作成チュートリアル」の第4回の記事です。[第3回](https://qiita.com/snishym/items/1bdf90cf51ba28cc4ef3)では刺激の提示位置を変更しながら系列提示を行う方法を紹介しました。今回は，キー反応の取得を行います。記事に余裕があるのでついでに刺激のランダマイズも行います。詳細は右側の見出しリストをご覧ください。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/8b52db0d901cf5744463)をご一読ください。

# キー反応の受付

これまでは`core.wait()`を使って，指定した秒数だけ刺激を提示していました。これを，「何かキーが押されるまで刺激を提示し続ける」に変更するためには，`core.wait()`を`event.waitKeys()`に変えます。ただし，新たに`event`というツールを使うことになるので，最初に追加して読み込むことを忘れないようにしてください。

```python:allow_key_response.py
from psychopy import visual, core, event # 新たにeventというツール（モジュール）を使用

win = visual.Window()

instruction = visual.TextStim(win, "いずれかのキーを押してください")
instruction.draw()
win.flip()

event.waitKeys() # 何かしらのキーを押すまで待機

win.close()
```

何かキーを押したらウィンドウが閉じたと思います。
`keyList = `に続けて受け付けるキーのリストを指定することで受け付けるキーを限定することができます。

```python:allow_spacebar.py
from psychopy import visual, core, event

win = visual.Window()

instruction = visual.TextStim(win, "いずれかのキーを押してください")
instruction.draw()
win.flip()

event.waitKeys(keyList = ['space']) # スペースキーを押すまで待機

win.close()
```

スペースキー以外では画面が閉じなかったはずです。

# キー反応を取得する

サイモン課題では，押したキーをデータとして保存する必要があります。そのため，キーを受け付けるようにするだけでなく，あとで保存できるようにプログラム内で扱えるようにしなければなりません。

といっても，大したことはありません。実は`event.waitKeys()`は常に押されたキーを返してくれているので，それに名前をつけて受け止めてあげるだけです。以下のコードでは`resp`という名前をつけています。

```python:get_key_response.py
from psychopy import visual, core, event

win = visual.Window()

instruction = visual.TextStim(win, "いずれかのキーを押してください")
instruction.draw()
win.flip()

resp = event.waitKeys() # 押されたキーをrespという名前で利用する
print(resp) # 出力

win.close()
```

下部出力エリアを見てください。`['space']`のような出力が得られるはずです。`[]`で囲まれている通り，押されたキーはリストに格納されます。なお，同じプログラムを実行して複数のキーを同時押しすると，`['s', 'd']`のように押したキーのいくつかが格納されたリストが出力されます。複数キーを取得するタイミングのボーダーラインはわかりませんが，通常のタイピングスピードでは複数のキーが取得されることはなさそうです。

もちろん`keyList = `で受け付けるキーを指定すれば，受け付けたキーのうち，入力されたキーだけが出力されます。試してみてください。

# 反応時間を取得する

サイモン課題では反応の正誤だけでなく，反応時間も有用な指標です。そこで，反応時間も取得できるようにしましょう。
反応時間を計測するための時計を用意して，`event.waitKeys(timeStamped = 時計の名前)`で時計を指定します。

```python:get_resp_with_rt.py
from psychopy import visual, core, event

win = visual.Window()
instruction = visual.TextStim(win, "いずれかのキーを押してください")

stopwatch = core.Clock() # stopwatchという名前の時計を用意

instruction.draw()
win.flip()

stopwatch.reset() # 時計の時間をリセットする
resp = event.waitKeys(timeStamped = stopwatch) # timeStampedに用意した時計を指定
print(resp)

win.close()
```

コードを実行してキーを押すと，`[['j', 0.4960438920]]`のようなリストのリストが出力されます。最初の要素が押されたキーで，次の要素が時計がリセットされてからキー入力までにかかった時間になります。刺激の提示直後に時間をリセットしているので，得られた時間を刺激への反応にかかった時間として解釈することができます[^1]。

[^1]: ある試行の反応時間をどのタイミングから測定するのかは課題によって変わると思います。その場合は，`.reset()`の位置を調整して対応しましょう。

# サイモン課題におけるキー反応の取得

ということで，今回紹介したキー取得のコードをこれまでのコードに取り入れます。

- 指定したキーが入力されるまで待機する
- 刺激が提示されてからキーが入力されるまでにかかった時間を計測する
- 入力されたキーと計測した時間をとりあえず`print()`で出力する

下記のコードに目を通す前にご自身で書いてみてください。

Lだったらどのキーで反応させるか，Rだったらどのキーで反応させるか。**使いたいキーをどのように指定したらいいかわからない場合は，上記のすべてのキーを受け付ける状態で使いたいキーを入力した結果を確認するのが，一番簡単な確認方法です**。

```python:fixed_simon.py
from psychopy import visual, core, event # eventを追加し忘れない

win = visual.Window()

letter_stim = visual.TextStim(win) 
letter_pos = [
    ["L",-0.3], # 一致
    ["R",0.3], # 一致
    ["L",0.3], # 不一致
    ["R",-0.3] # 不一致
]

stopwatch = core.Clock()

for letter, x_axis in letter_pos:
    letter_stim.setText(letter)
    letter_stim.setPos([x_axis, 0])
    letter_stim.draw()
    win.flip()

    stopwatch.reset()
    resp = event.waitKeys(keyList = ['left', 'right'], timeStamped = stopwatch)
    print(resp)

win.close()
```

4試行だけですが，もう立派なサイモン課題ですね。上記のコードでは，左右の方向キーで反応を取得するようにしました。エラーが発生したという場合は，最初に`event`を追加し忘れているかもしれません。

今回の主な内容は以上ですが，ランダマイズの話を最後にします。今回の内容とは直接関わってきませんが，1回分を費やしてまでする内容ではなかったので，ここで取り扱うことにしました。

# 刺激の提示順序をランダマイズする

刺激の提示順序をランダマイズするためには，いくつか方法がありますが，刺激のリストの中身の順番をランダムにシャッフルするのが一番簡単です。Pythonに標準で装備されている`random`モジュールの`shuffle()`という関数を使用します。`()`内にリストを入れると，そのリストをシャッフルしてくれます。

```python:randomize_list.py
import random # randomというツール（モジュール）を使用する

num_list = [1,2,3,4,5]

random.shuffle(num_list) # リストの中身をシャッフルする
print(num_list)
```

サイモン課題ではリストのリストを使っていますが，同じ方法でシャッフルして問題ありません。

```python:randomize_list_list.py
import random

letter_pos = [
    ["L",-0.3], # 一致
    ["R",0.3], # 一致
    ["L",0.3], # 不一致
    ["R",-0.3] # 不一致
] * 2 # ランダマイズの結果を見やすくするために刺激を2倍の長さにしておく

random.shuffle(letter_pos)
print(letter_pos)
```

文字と位置の組み合わせがうまくランダマイズされたと思います。ランダマイズは以上になります。サイモン課題のコードに付け足すのもさほど難しくないので，ここでは紹介しません。もしうまく行かなかったという場合は，次回以降のコードに登場してくるので，そこで確認してください。

# おわりに

今回はキー入力およびその反応時間の取得方法を紹介しました。合わせて刺激のランダマイズ方法も紹介しました。これでサイモン課題自体は完成しました。

しかし，今回取得した反応データはCoderの「出力」に表示されただけでした。このままでは分析はほぼ不可能です。そこで次回は実験データをファイルに保存する方法を紹介します。

[【第5回】データの保存](https://qiita.com/snishym/items/f80607cbe462f8a4e1d4)