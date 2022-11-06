---
title: "matplotlibの備忘録"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SQL"]
published: false
---

Pythonのライブラリ、matplotlibの備忘録です。

# plt.plotか、ax.plotか

```python
import matplotlib.pyplot as plt
```

した後、簡単なグラフを書くには大きく2つの方法があります。一つ目は、

```python
plt.plot(...)
plt.sho()
```

とする方法です。もう一つは、

```python
fig, ax = plt.subplots()
ax.plot(...)
plt.show()
```

と書く方法です。
グラフを１つ描画する場合にはどちらでもよいのですが、前者は後者の簡易的な記述方法ですので、後者の書き方で覚えておいたほうが応用が利くと思います。
後者の書き方を理解するにあたり、matplotlibの描画領域について理解しておく必要があります。
matplotlibでは、描画領域全体をfigure、その中の一つ一つのグラフの描画領域がaxesになります（下図参照）。

![](/images/matplotlib/matplotlib_figure_axes_axis.png)