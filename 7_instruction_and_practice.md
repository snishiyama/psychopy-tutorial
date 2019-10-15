# はじめに

本記事は，「PsychoPy Coderによる心理学実験作成チュートリアル」の第7回の記事です。[第6回](https://qiita.com/snishym/items/5e01641150581c632bea)では参加者情報の取得方法を紹介しました。そこで実験プログラムは完成しましたが，もう少し実験プログラムを作り込んで行きます。今回は課題の教示，課題の練習をこれまでのコードに追加します。それに合わせて関数の作成について解説します。これがPsychoPyのチュートリアルとしては最後の記事なります。

文量が多いように見えますが，完成版のコードを記載していることが原因です。内容はそんなに多くないと思います。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/8b52db0d901cf5744463)をご一読ください。

# 教示・長文テキスト

参加者が課題を行うにあたって，どのように進めればいいか説明する必要があります。といっても，これまで使い続けてきた`visual.TextStim`を使えばいいだけです[^1]。

[^1]: ただし，教示文の改行や配置（左寄せ・中央寄せ）にこだわりがあったり，図なども教示に表示したいという場合は，パワーポイントなどで教示用の画像を作って`visual.ImageStim`で提示させるほうがいいでしょう。

```python:show_instruction.py
from psychopy import visual, event

win = visual.Window()

inst_stim = visual.TextStim(win) # 教示用のTextStimを作成
inst_text = "教示です。\nスペースキーを押してください。"

inst_stim.setText(inst_text)
inst_stim.draw()
win.flip()
event.waitKeys(keyList = ['space'])
win.close()
```

このコードについて特に説明することはありません。これを課題の前に入れれば，教示に続けて課題が実行されます。しかしながら，「教示です」なんで教示はありません。もっと課題の概要・取り組み方を説明しなければなりません。

本シリーズで作成してきたサイモン課題は，まず「キー押しの課題です」。具体的には，「LとRがランダムに提示されます」。そして，参加者がしなければならないのは，提示された刺激が「『L』なら左方向キーを押し，『R』なら右方向キーを押す」ということです。適切に改行しながらこれを反映すると，`inst_text`は以下のようになります。

```python
inst_text = "これからキー押し課題を行ってもらいます。\n「L」「R」がランダムに表示されるので，\n\n「L」が表示されたら左方向キー\n「R」が表示されたら右方向キー\n\nを押してください。\n準備ができたらスペースキーを押して課題を始めてください。"
```

これで教示としてはOKになりました。改行を2つ重ねていると空白行ができるので，段落などのまとまりを作って，見やすくしたりすることができます。しかし，実験を作っている立場からすると，実行してみるまでどのように表示されるのか想像がつきにくいです。改行記号を打つのも面倒です。

Pythonでは，クォーテーション`'`（あるいはダブルクォーテーション`"`）を3つ並べて`'''文字列'''`(`"""文字列"""`)を用いて以下のように書くことで，先程の教示のような長文テキストをより簡単に作成することができます。

```python
inst_text = """\
これからキー押し課題を行ってもらいます。
「L」「R」がランダムに表示されるので，

「L」が表示されたら左方向キー
「R」が表示されたら右方向キー

を押してください。
準備ができたらスペースキーを押して課題を始めてください。"""
```

改行記号も書かず，実際の提示のされ方に近い形でプログラム上で書くことができました。これは，`'''...'''`内の改行がそのまま反映されるからです。なお，一行目`"""\`のバックスラッシュは，そのテキスト内での改行を無視するために使用されています。`\`の有無でどのように変わるか教示の表示が変わるのか試してもらうと分かりやすいです。いろんな行末に`\`をつけてみてください。

# 課題の練習

実際の実験では，分析用のデータを取る（つまり本番）の前に，練習課題を行います。参加者の教示の理解を確認したり，反応のマッピングを生成したりするために重要です。そこで，練習課題用のブロックも作成しましょう。素朴に，課題ブロックを複製すればいいです。

```python:simon_with_practice.py
from psychopy import visual, core, event
import random

win = visual.Window()

letter_stim = visual.TextStim(win) 
letter_pos = [
    ["L",-0.3],
    ["R",0.3],
    ["L",0.3],
    ["R",-0.3]
]
prac_list = letter_pos * 6 # 練習用のリスト
main_list = letter_pos * 30 # 本番用のリスト
random.shuffle(prac_list)
random.shuffle(main_list)

trialID = 0

stopwatch = core.Clock()

# 練習用のブロック
for letter, x_axis in prac_list: # 練習用のリストを使う
    letter_stim.setText(letter)
    letter_stim.setPos([x_axis, 0])
    letter_stim.draw()
    win.flip()

    stopwatch.reset()
    resp = event.waitKeys(keyList = ['left', 'right', 'escape'], timeStamped = stopwatch)
    key, rt = resp[0]
    if key == 'escape':
        core.quit()
    
    trialID = trialID + 1

# 本番用のブロック
for letter, x_axis in main_list: # 本番用のリストを使う
    letter_stim.setText(letter)
    letter_stim.setPos([x_axis, 0])
    letter_stim.draw()
    win.flip()

    stopwatch.reset()
    resp = event.waitKeys(keyList = ['left', 'right', 'escape'], timeStamped = stopwatch)
    key, rt = resp[0]
    if key == 'escape':
        core.quit()
    
    trialID = trialID + 1

win.close()
```

長いです。実は，ダイアログボックスやデータの保存の部分は省略したので，これを加えるとより長く，見にくくなります。もちろんこれでも構いません[^2]。

[^2]: 私も長い間このように長いコードを使って実験してきました。

しかし，これでは非常に不便です。第2回でも述べましたが，同じ処理のコピペはコードの管理の点から望ましくありません。ある変更をコピペしたコードすべてに加える必要があります。例えば，今回の記事では後で注視点やITIを足しますが，それらを練習用と本番用のブロックの同じ場所に追加する必要があります。変更のたびにこのようなことをしていると，いつかどこかでミスを犯します。そしてその修正も同様に面倒です。

この問題を解決するために，課題ブロックを関数化します。

# 関数

関数は処理をまとめたものです。例えば`print()`は値を表示するための関数です。この裏では値をいい感じに表示したり，意図しない使われ方をしたときにはエラーを吐いたりする処理が実行されています[^3]。もしその処理を毎回書かないといけないなら，ユーザーの負担は相当重いし，コードもとても読みにくくなってしまうでしょう。面倒な処理がまとめられて`print()`という名前で提供されているため，ユーザーは気軽にコード内の値を何度も出力できるようになっています。そして，このような処理のまとまりはユーザーが自分で作ることも可能です。

[^3]: ソースコードを見たことはありません。

関数は以下のように`def 関数名():`で作成できます。

```python:func_odd_even.py
def judge_odd_even(num):
    if num % 2 == 0:
        print('偶数です')
    else:
        print('奇数です')

judge_odd_even(2)
judge_odd_even(3)
```

2では割った余りが0(つまり偶数)かどうかを判定するif文の処理が`judge_odd_even()`という名前の関数としてまとめられています。そのおかげで，最後の2行では`judge_odd_even(数字)`と宣言するだけで偶数か奇数かが出力されます。

`def judge_odd_even(num):`の`num`は引数と呼ばれます。ここに入れた値は関数の中で`num`という名前で使用されます。また，関数の前に定義した値は関数の中で使用することができます[^4]。

[^4]: 本来はクラスとして定義したほうがスコープの点でいいのですが，説明が大変なので，グローバル変数を使った関数の作成を紹介してます。

```python:func_var_outside.py
bad_num = 1 # 関数外での定義

def judge_odd_even(num):
    num = num + bad_num # 引数にbad_numを足す
    if num % 2 == 0:
        print('偶数です')
    else:
        print('奇数です')

judge_odd_even(2) # 奇数です
judge_odd_even(3) # 偶数です
```

上記のコードでは，奇偶の出力が反転します。関数外で定義した`bad_num`が引数に与えられた数字に加算されているからです。もちろん，`bad_num`が2,4,...なら出力は反転しません。**ただし，関数中で値を変更するものは関数内に入れましょう**。以下のコードはエラーになります。

```python
bad_num = 1 # 関数外での定義

def judge_odd_even(num):
    num = num + bad_num
    if num % 2 == 0:
        print('偶数です')
    else:
        print('奇数です')
    
    bad_num = bad_num + 1 # 関数内で変更

judge_odd_even(2)
```

ということで，サイモン課題のブロックを一つの関数としてまとめましょう。ただし，サンプルのコードを短くするために，参加者情報の取得，データの保存については省略しています。

```python:simon_task_function.py
from psychopy import visual, core, event
import random

win = visual.Window()
letter_stim = visual.TextStim(win)
stopwatch = core.Clock()

def run_simon_task(phase, stim_list): # 課題の関数を作成
    trialID = 0
    
    for letter, x_axis in stim_list:
        letter_stim.setText(letter)
        letter_stim.setPos([x_axis, 0])
        letter_stim.draw()
        win.flip()

        stopwatch.reset()
        resp = event.waitKeys(keyList = ['left', 'right', 'escape'], timeStamped = stopwatch)
    
        key, rt = resp[0]
        if key == 'escape':
            core.quit()
    
        trialID = trialID + 1

# 刺激の準備
letter_pos = [
    ["L",-0.3],
    ["R",0.3],
    ["L",0.3],
    ["R",-0.3]
]
prac_list = letter_pos * 6 # 練習用のリスト
main_list = letter_pos * 30 # 本番用のリスト
random.shuffle(prac_list)
random.shuffle(main_list)

# ここでサイモン課題関数が実行される
run_simon_task(phase = 'prac', stim_list = prac_list) # 練習
win.flip()
core.wait(3) # 区切り
run_simon_task(phase = 'main', stim_list = main_list) # 本番
```

前回までに作った課題ブロックの部分をまとめただけです。ただし，練習と本番では使用するリストが違う（`simon_with_practice.py`中段付近の`prac_list`と`main_list`）ので，これに対応できるように，`for letter, x_axis in stim_list:`とし，引数で`stim_list`を指定できるようにしました。練習・本番どちらのデータかを区別できるように，引数に`phase`を設け，データファイルにphase情報を保存するようにしました。

これで，効率よく課題ブロックを増やしたりできるようになりました。もし課題ブロックに変更点があっても，定義した関数を変更するだけで済みます。練習が終わったあとも，本番の前に「それでは本番です」のような教示を表示する必要があります。教示表示用の関数も作成してみましょう。

# サイモン課題の完成

それでは，サイモン課題を完成させましょう。やることは，教示と練習課題の追加です。ただしそのために，教示表示用の関数，サイモン課題の関数を作成します。

サイモン課題の最初に注視点を表示したり，試行が終わるごとに空白画面を出す（ITI）ようにしてください。特に説明はしませんでしたが，これまで説明したことしか使わないです。

```python:simon_exp.py
from psychopy import visual, core, event, gui
import random
import pathlib

subj_info = {"参加者ID":'', "年齢":'', "性別": ["男性", "女性"]}

info_dlg = gui.DlgFromDict(subj_info)

if not info_dlg.OK:
   core.quit()

# 以下，自作関数に使っているものだけを先に書く（提示刺激のリストは関数のあとに移動）

subjID = subj_info["参加者ID"]
age = subj_info["年齢"]
sex = subj_info["性別"]

win = visual.Window()

letter_stim = visual.TextStim(win)
inst_stim = visual.TextStim(win) # 教示用のTextStimを作成
fixation_stim = visual.TextStim(win, "+") # 注視点

current_folder = pathlib.Path(__file__).parent
new_filename = "{}_simon.csv".format(subjID)
new_filepath = current_folder/"data"/new_filename

datafile = open(new_filepath, mode = 'a')
datafile.write('参加者ID,年齢,性別,フェイズ,trialID,刺激,位置,反応キー,反応時間\n') # 列名を増やしておく

stopwatch = core.Clock()



def show_instruction(sentence): # 教示の関数を作成
    inst_stim.setText(sentence)
    inst_stim.draw()
    win.flip()
    key = event.waitKeys(keyList = ['space', 'escape'])
    if key[0] == 'escape': # 入力がescapeなら実験を終了する
        datafile.close()
        core.quit()


def run_simon_task(phase, stim_list): # 課題の関数を作成
    # 教示後の空白画面
    win.flip()
    core.wait(1)
    
    # 注視点
    fixation_stim.draw()
    win.flip()
    core.wait(0.5)

    trialID = 0
    for letter, x_axis in stim_list:
        # 刺激の提示
        letter_stim.setText(letter)
        letter_stim.setPos([x_axis, 0])
        letter_stim.draw()
        win.flip()

        # 反応の取得
        stopwatch.reset()
        resp = event.waitKeys(keyList = ['left', 'right', 'escape'], timeStamped = stopwatch)
    
        # データの保存
        key, rt = resp[0]
        if key == 'escape': # 入力キーがescapeなら実験を終了する
            datafile.close() # ファイルを閉じるのを忘れない
            core.quit()
    
        data = '{},{},{},{},{},{},{},{},{}\n'.format(subjID, age, sex, phase, trialID, letter, x_axis, key, rt)
        datafile.write(data)
    
        trialID = trialID + 1
        
        # ITI
        win.flip()
        core.wait(0.5)


# 各教示文の準備
inst_first = '''\
これからキー押し課題を行ってもらいます。
「L」「R」がランダムに表示されるので，

「L」が表示されたら左方向キー
「R」が表示されたら右方向キー

を押してください。

まず練習を行います。
準備ができたらスペースキーを押して
課題を始めてください。'''

inst_go_exp = '''\
それでは本番です。
準備ができたらスペースキーを押して
課題を始めてください。'''

# 刺激の準備
letter_pos = [
    ["L",-0.3],
    ["R",0.3],
    ["L",0.3],
    ["R",-0.3]
]
prac_list = letter_pos * 6 # 練習用のリスト
main_list = letter_pos * 30 # 本番用のリスト
random.shuffle(prac_list)
random.shuffle(main_list)

### 実際に課題を実行している部分 ###

show_instruction(inst_first)

run_simon_task(phase = 'prac', stim_list = prac_list) # 練習

show_instruction(inst_go_exp)

run_simon_task(phase = 'main', stim_list = main_list) # 本番

datafile.close()
win.close()
```

自作関数の作成にともない，これまで序盤に持ってきていた`letter_pos`の定義を自作関数の定義のあとに移動させました。その方が，自作関数で利用されているものとそうでないものの区別がつきやすく，コードの管理が容易になるからです。課題の関数内に教示後の空白画面と，注視点，ITIが増えているのもよく見ておいてください。

# おわりに

これでサイモン課題実験が完成しました。お疲れ様です。ぜひ，身近な人に実施してみてください。

もちろん，チュートリアルということで，PsychoPyの機能のほんの一部しか紹介していません。ウィンドウの全画面表示の方法すら紹介していません。しかし，ここまでやり遂げたみなさんなら，[PsychoPyの公式リファレンス](https://www.psychopy.org/api/api.html)を見ながら，ご自身の研究課題に合わせてコードを書くことができるはずです。

これでPsychoPyのチュートリアルは終わりです。一応番外編として，次回は実験で生成された参加者ごとの実験データを集約する方法について紹介します。

[【第8回】実験データの集約](https://qiita.com/snishym/items/b4cf8b8a4ac2b8b69bc5)