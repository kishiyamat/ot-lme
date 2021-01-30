Description
================

## tableau

Rの data.frame で`data/1.tab`を読ませるとこんな感じになる。 適当に岸山が
/ba.ba/ と /bab.a/ 以外は作成. rankは1–0のランク, input と candidate
はそのまま、 `c_`始まりは制約(constraints). その下の 1は`*`,
2は`**`みたいなもの。 tableau感は…ありますかね？

``` r
tab_1 = read.table("data/1.tab", header = T)
tab_1
```

    ##   rank  input candidate c_onset c_no_coda
    ## 1  1.0 /baba/   /ba.ba/       0         0
    ## 2  0.7 /baba/  /ba.ban/       0         1
    ## 3  0.2 /baba/ /ban.ban/       0         2
    ## 4  0.0 /baba/   /bab.a/       1         1

同じ制約のセットを持った別の例を `data/` にどんどん足していきたい。
下の例は勝手に作成.

``` r
tab_2 = read.table("data/2.tab", header = T)
tab_2
```

    ##   rank  input candidate c_onset c_no_coda
    ## 1  1.0 /mama/   /ma.ma/       0         0
    ## 2  0.6 /mama/  /ma.man/       0         1
    ## 3  0.0 /mama/   /mam.a/       1         1

簡単にデータはくっつけられる。

``` r
tabs = rbind(tab_1, tab_2)
tabs
```

    ##   rank  input candidate c_onset c_no_coda
    ## 1  1.0 /baba/   /ba.ba/       0         0
    ## 2  0.7 /baba/  /ba.ban/       0         1
    ## 3  0.2 /baba/ /ban.ban/       0         2
    ## 4  0.0 /baba/   /bab.a/       1         1
    ## 5  1.0 /mama/   /ma.ma/       0         0
    ## 6  0.6 /mama/  /ma.man/       0         1
    ## 7  0.0 /mama/   /mam.a/       1         1

## Model

rankを制約から予測して、制約の重みを推定させる。

``` r
reg_formula = rank ~ c_onset + c_no_coda
model = lm(reg_formula, tabs)
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = reg_formula, data = tabs)
    ## 
    ## Residuals:
    ##          1          2          3          4          5          6          7 
    ## -1.429e-02  7.857e-02 -2.857e-02  1.908e-17 -1.429e-02 -2.143e-02  1.908e-17 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  1.01429    0.02901   34.96 4.00e-06 ***
    ## c_onset     -0.62143    0.03746  -16.59 7.73e-05 ***
    ## c_no_coda   -0.39286    0.02649  -14.83  0.00012 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.04432 on 4 degrees of freedom
    ## Multiple R-squared:  0.9931, Adjusted R-squared:  0.9897 
    ## F-statistic: 288.2 on 2 and 4 DF,  p-value: 4.75e-05

tableauのConstraintsの左右と重みは1:1の関係にある。 例えば c\_onset
に違反すると(c\_onset==1) Estimate は 0.62、 c\_no\_coda
に違反すると(c\_no\_coda==1) Estimate は 0.39 rank が下がる。これは
c\_onset の方が違反した時のペナルティーが重いことを示す。
このペナルティーの重さが大きいものをtableauでは人手で左に寄せている。
つまり重さ=Constraintsの順序になる。

## ToDo

1.  rank は1(Best)と0(Worst) を入れる. 不明なときは直観で良い
2.  input は input
3.  candidate は candidate
4.  それより右には constraints を書く
    1.  constraints の種類は揃える
    2.  ただし多くても少なくても問題ない. NAとなるだけ

後はRで複数のconstraintの重みを自動で推定する。
自動で推定した重みはそのままtableau上の制約の優先度になる。

``` r
tab_more = read.table("data/more.tab", header = T)
tab_less = read.table("data/less.tab", header = T)
```

    ## Warning in read.table("data/less.tab", header = T): incomplete final line found
    ## by readTableHeader on 'data/less.tab'

``` r
dplyr::bind_rows(tab_1, tab_more, tab_less)
```

    ##    rank  input candidate c_onset c_no_coda c_more
    ## 1   1.0 /baba/   /ba.ba/       0         0     NA
    ## 2   0.7 /baba/  /ba.ban/       0         1     NA
    ## 3   0.2 /baba/ /ban.ban/       0         2     NA
    ## 4   0.0 /baba/   /bab.a/       1         1     NA
    ## 5   1.0 /mama/   /ma.ma/       0         0      0
    ## 6   0.6 /mama/  /ma.man/       1         0      0
    ## 7   0.0 /mama/   /mam.a/       1         1      0
    ## 8   1.0 /mama/   /ma.ma/      NA         0     NA
    ## 9   0.6 /mama/  /ma.man/      NA         0     NA
    ## 10  0.0 /mama/   /mam.a/      NA         1     NA
