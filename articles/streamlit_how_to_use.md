---
title: "Streamlitの備忘録"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Streamlit", "Python", "MachineLearning", "DataScience"]
published: true
---

Streamlitはデータサイエンスに適したWebアプリを簡単に作れるPythonのフレームワークです．  
このページはStreamlitの使い方の備忘録として随時更新していきます.

# アプリの実行

ミニマムなアプリのソースコードは以下

```py
import streamlit as st
import numpy as np
import pandas as pd

st.title("My first app")
```

これを実行するには，スクリプトのファイル名がyour_app_name.pyだとして以下

```
streamlit run your_app_name.py
```

Python Launcherを使っている場合は，

```
py -m streamlit run your_app_name.py
```

# サイドバー

サイドバーにウィジェットを追加するとサイドバーが現れます．

```py
st.sidebar.selectbox(
    "How would you like to be contacted?",
    ("Email", "Home phone", "MObile phone")
)
```

# イベントハンドリング

よくあるフレームワークではイベントハンドラなどでボタン押下イベントなどをハンドリングしますが，Streamlitでは何かイベントが発生するたびにスクリプト全体が再実行されます．このとき，例えばあるボタンが押されたときは，そのボタンウィジェットの戻り値がTrueになるため，以下のように書くことで，ボタン押下に関連した処理を実行することができます．  
（後述するようにon_change引数を使ってイベントをハンドリングすることもできます）

```py
if st.button("Button1"):
   st.write("Button1 clicked")

if st.button("Button2"):
   st.write("Button2 clicked")
```

# 複数ページ

Streamlitではイベント発生時にスクリプト全体が再実行されると述べました．なので，ページの記載内容を関数内に記載し，イベントに応じて呼び出す関数を切り替えることで，複数ページの切り替えを実現することができます．

```py
def page1():
   st.write("Here is page 1.")

def page2():
   st.write("Here is page 2.")

if st.sidebar.button("Button1"):
   page1()

if st.sidebar.button("Button2"):
   page2()
```

# 状態の保持

Streamlitはイベントが発生するたびにスクリプトが上から再実行されるため，変数の値などを保持することがそのままではできません．例えば以下のコードでは，ボタンを何度押してもhogeの値は1より大きくなりません．ボタンが押されるたびにhoge=0と初期化されるためです．

```py
hoge = 0

if st.button("Button1"):
   hoge += 1

st.write("hoge is:", hoge)
```

これはSession State（st.settion_state）を利用することで対応できます．  
settion_stateは辞書のように利用することができ（プロパティのようにアクセスすることもできます），settion_state内の変数はスクリプトが再実行されても値が保持されます．以下のように書くことで，ボタンを押すごとにhogeの値がインクリメントされるようになります．

```py
if "hoge" not in st.session_state:
   st.session_state.hoge = 0
   # st.settion_state["hoge"] = 0 でもOK

if st.button("Button1"):
   st.session_state.hoge += 1

st.write("hoge is:", st.session_state.hoge)
```

# コールバック

ウィジェットの引数on_changeを使うと，ウィジェットが更新されたときにコールバックを受け取ることができます．つまりウィジェットが更新されたときに特定のメソッドを呼び出すことができます．このメソッド呼び出しはスクリプト全体の実行よりも先に実行されます．  
このとき，例えばテキストインプットウィジェット（text_input）に入力された値は，session_stateに，ウィジェットのkey引数で指定した名前で格納されます．

```py
def update_txtbox():
   st.write(st.session_state.txtbox1)

st.text_input("Textbox", key="txtbox1", on_change=update_txtbox)
```

# テキスト

## 改行

テキストの改行は￥ｎでできます．st.writeだけでなく，st.markdown内での改行も￥ｎです．
