---
title: "数理最適化定式化のコツ"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python", "matplotlib"]
published: false
---

# 配送計画で経路が０個以上のサイクルからなる

- 地点$l$に向かう移動は、高々１つしか許されない

$$
\sum_{k \in K}x_{dkl} \le 1 \quad (d \in [D], l \in K)
$$

ここで$K$は地点の集合、$D$は計画立案期間の日数、$x_{dkl} \in {0, 1}$は、$d(\in [D])$日に地点$k(\in K)$から$l(\in K)$への移動があれば１、そうれなければ０。

- 地点$l$に向かう移動の有無は地点$l$を出る移動の有無と一致する

$$
\sum_{k \in K}x_{dkl} = \sum_{k \in K}x_{dlk} \quad (d \in [D], l \in K)
$$

この２つの制約で移動経路が「０個以上のサイクルからなる」ことが保証されるが、配送拠点$o$を通らないようなサイクルも存在してしまう。
これを防ぐために、補助変数$u_{dk}$を利用して、配送拠点$o$を通らないサイクルの発生を禁止する。
このテクニックは巡回セールスマン問題のMTZ定式化（Miller-Tucker-Zemlin）で用いられる。

$$
u_{do} = 0 \\
u_{dk} \ge 1 \quad (k \in K\backslash \{o\}) \\
u_{dk} + 1 \le u_{dl} + (|K| - 1)(1 - x_{dkl}) \quad (k, l \in K\backslash \{o\})
$$

ここで、$u_{dk}$は、$u_{dk} \in [0, |K| - 1] \quad (d \in [D], k \in K)$である（$[0, |K| - 1]$は$0$以上$|K|-1$以下の実数全体の集合を表す）。
$d$日の移動経路において、$k_1, k_2, k_3 \in K\backslash \{o\}$がサイクルをなしていると仮定する。
このとき$x_{dk_{1}k_{2}} = x_{dk_{2}k_{3}} = x_{dk_{3}k_{1}} = 1$なので、上記制約の３番目の式から次の不等式が得られる。

$$
u_{dk_{1}} + 1 \le u_{dk_{2}} \\
u_{dk_{2}} + 1 \le u_{dk_{3}} \\
u_{dk_{3}} + 1 \le u_{dk_{1}}
$$

これらを足し合わせると

$$
\sum_{i=1,2,3}u_{dk_{i}} + 3 \le \sum_{i=1,2,3}u_{dk_{i}}
$$

となり、これは$u$をどのようにとっても満たすことができない。したがって、$k_1, k_2, k_3 \in K\backslash \{o\}$がサイクルをなすことが禁止されいていることがわかる。任意の長さの$o$を通らないサイクルが禁止されることも、同様の議論でわかる。
次に、例えば$K={o, k_1, k_2, k_3, k_4, k_5, k_6}$で$o, k_1, k_2, k_3$だけがサイクルをなす場合に、実際に制約違反とならない$u_{dk}$の値の取り方があることを確認する。
$x_{dkl}=0$のときは$u_{dk}+1 \le u_{dl} + |K| - 1$となるが、$u_{dk}(k \ne o)$は１以上$|K|-1$以下の値をとるという条件から常に成り立ち、制約として有効にならない。
さらに、$k=0$か$l=o$について、そもそも$k, l in K\backslash \{o\}$のため対象外。
したがって上記制約の３番目の式から得られる不等式は次の２式である。

$$
u_{dk_{1}} + 1 \le u_{dk_{2}} \\
u_{dk_{2}} + 1 \le u_{dk_{3}}
$$

ここで次のように$u$に値を割り振ると、制約違反とならない$u_{dk}$の値の振り方となり、$x_{dkl}$が実行可能であることがわかる。
実行可能解では、$u_{dk}$の値は各地点に訪問する順序を表す。


# 配送計画で巡回セールスマン問題を複数日同時に解くのを避ける

## 素直なモデリングだが解けない（計算量大）パターン

- パラメータ
  - $D \in N$：計画立案の対象日数。$D \ge 1$
  - $K$：地点の集合
  - $t_{kl}(k, l \in K)$：移動時間。ただし$t_{kl} \ge 0, t_{kk} = 0$
  - $o \in K$：自社拠点を表す地点
  - $R$：注文の集合
  - 注文$r \in R$について
    - $k_r(\in K)$：注文$r$の届け先地点
    - $d_r^0, d_r^1 \in [D]$：配送指定期間。$d_r^0 \le d_r^1$
    - $w_r$：重量。$w_r \ge 0$
    - $f_r$：配送の外部委託費用。$f_r \ge 0$
  - $H^0$：所定労働時間。$H^0 > 0$
  - $W$：トラックの最大積載量。$W > 0$
  - $c$：1時間当たりの残業代。$c > 0$
  - $H^1$：最大残業時間。$H^1 \ge 0$
- 決定係数
  - $x_{dkl} \in \{0, 1\} \quad (d \in [D], k, l \in K)$：自社トラックの移動
  - $y_{dr} \in \{0, 1\} \quad (d \in [D], r \in R)$：自社トラックによる配送
  - $u_{dk} \in [0, |K|-1] \quad (\in \mathbb{R})$：移動の順序を表す補助変数
  - $h_d \in \mathbb{R}_+ \quad (d \in [D])$：残業時間を表す補助変数
- 制約条件
  - $\sum_{k \in K}x_{dkl} \le 1 \quad (d \in [D], l \in K)$
  - $\sum_{k \in K}x_{dkl} = \sum_{k \in K}x_{dlk} \quad (d \in [D], l \in K)$
  - $u_{do} = 0 \quad (d \in [D])$
  - $u_{dk} \ge 1 \quad (d \in [D], k \in K \backslash \{o\})$
  - $u_{dk}+1 \le u_{dl}+(|K|-1)(1-x_{dkl}) \quad (k, l \in K\backslash \{o\})$
  - $\sum_{d}y_{dr} \le 1 \quad (r \in R)$
  - $y_{dr} \le \sum_{k} x_{dkk_{r}} (d \in [D], r \in R)$
  - $\sum_{r} w_r y_{dr} \le W \quad (d \in [D])$
  - $\sum_{k, l \in K}{t_{kl}x_{dkl}} - H^0 \le h_d \quad (d \in [D])$
  - $h_d \le H^1 \quad (d \in [D])$
  - $\sum_{d \in [D]\backslash [d_r^0, d_r^1]} y_{dr} = 0 \quad (r \in R)$
- 目的関数（最小化）
  - $c\sum_{d\in [D]}h_d + \sum_{r\in R}{f_r (1-\sum_{d\in [D]}y_{dr})}$

## 日ごとにスケジュールを事前に列挙しておくことで可能となるモデリング


# 絶対値の値を最小化したい場合には、非負の補助変数を導入して上下で挟む

[こちらのブログ](https://yamaimo.hatenablog.jp/entry/2022/12/11/220000)より。

$$
\min_a |a| \Leftrightarrow \min m, \quad \rm{sub. to} -m_i \le a \le m, \quad m \ge 0
$$