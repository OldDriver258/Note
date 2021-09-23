# Hello world!

# 一级标题

## 二级标题

### 三级标题

每写完一个段落要隔一行空行。

就像这样，隔了一行空行。

---

分割线

**重点加粗**

*斜体*

~~删除线~~

==高亮==

---

# 列表：

* 无序列表
  * 嵌套无序列表
  * 嵌套无序列表
* 无序列表
* 无序列表

1. 有序列表 1
   1. 嵌套有序列表 1
   2. 嵌套有序列表 2
2. 有序列表 2
3. 有序列表 3
   
任务列表
- [x] 已经完成的事 1
- [x] 已经完成的事 2
- [x] 已经完成的事 3
- [ ] 已经完成的事 4
- [ ] 已经完成的事 5


---

# 引用文本：

>引用别人说的话
>就这样写
>By. OrangeX4

---

# 代码

这是`行内代码`语法.

代码块语法:

``` python
print("Hello,world")
```

请将'替换成`

代码行数的显示:

``` javascript {.line-numbers}
function add(x, y) {
  return x + y
}
```

注意要将 ' 替换成 `.

---

# 表格:

| 表头 | 表头 |
| ---- | ---- |
| 内容 | 内容 |
| 内容 | 内容 |

拓展语法:

| 表头 | 表头 |
| ---- | ---- |
| 内容 | 内容 |
| >    | 内容 |

| 表头 | 表头 |
| ---- | ---- |
| 内容 | 内容 |
| ^    | 内容 |

自动表格对齐: `Shift + Alt + F`

---

# 超链接和图片

[超链接名称](连接地址)

![图片提示语](图片地址)

例如,图片可以使用网址和图床

[OrangeX4's Blog](https://orangex4.cool/)

![OrangeX's Avatar](https://orangex4.cool/images/icons/profile.jpg)


也可以在本地用相对地址:

[Other](other.md)

![OrangeX's Avatar](images/profile.jpg)

剪切板工具的使用,通过**Paste Image**插件

![粘贴快捷键](images/2021-06-10-10-40-35.png)

---

# 注释:

<!-- 你看不见我 -->

<!-- 多行注释
就像这样 -->

---

# 数学公式

行内公示:

单位圆 $x^2+y^2=1$

公式快:

$$
\begin{cases}
  x=\rho\cos\theta  \\
  y=\rho\sin\theta  \\  
\end{cases}
$$

1. 上标和下标

   上标 $x^2 + y^{12} = 1$

   下标 $x_1 + y_{12} = 1$

2. 分式

    较小的行内行分数 $\frac{1}{2}$

    展示型的分式 $\displaystyle\frac{x+1}{x-1}$

3. 根式

    开平方 $\sqrt{2}$
    开 $n$ 次方 $\sqrt[n]{2}$

4. 空格

   紧贴 $a\!b$

   没有空格 $ab$

   小空格 $a\,b$

   中等空格 $a\;b$

   大空格 $a\ b$

   quad 空格 $a\quad b$

   两个 quad 空格 $a\qquad b$

5. 累加,累乘和积分

    累加 $\sum_{k=1}^n\frac{1}{k} \quad \displaystyle\sum_{k=1}^n\frac{1}{k}$

    累乘 $\prod_{k=1}^n\frac{1}{k} \quad \displaystyle\prod_{k=1}^n\frac{1}{k}$

    积分 $\displaystyle \int_0^1x{\rm d}x \quad \iint_{D_{xy}} \quad \iiint_{\Omega_{xyz}}$

6. 括号修饰

   圆括号 $\displaystyle \left(\sum_{k=1}^{n}\frac{1}{k} \right)^2$

   方括号 $\displaystyle \left[\sum_{k=1}^{n}\frac{1}{k} \right]^2$

   花括号 $\displaystyle \left\{\sum_{k=1}^{n}\frac{1}{k} \right\}^2$

   尖括号 $\displaystyle \left\langle\sum_{k=1}^{n}\frac{1}{k} \right\rangle^2$

7. 多行算式对齐

   居中:

   $$
   \begin{aligned}
   y &=(x+5)^2-(x+1)^2 \\
   &=(x^2+10x+25)-(x^2+2x+1) \\
   &=8x+24 \\
   \end{aligned}
   $$

   左对齐:

   $
   \begin{aligned}
   y &=(x+5)^2-(x+1)^2 \\
   &=(x^2+10x+25)-(x^2+2x+1) \\
   &=8x+24 \\
   \end{aligned}
   $

8. 方程组

   $$
   \begin{cases}
   k_{11}x_1+k_{12}x_2+\cdots+k_{1n}x_n=b_1 \\
   k_{21}x_1+k_{22}x_2+\cdots+k_{2n}x_n=b_2 \\
   \cdots \\
   k_{n1}x_1+k_{n2}x_2+\cdots+k_{nn}x_n=b_n \\
   \end{cases}
   $$

9. 矩阵

   矩阵:

   $$
   \begin{pmatrix}
       1 & 1 & \cdots & 1 \\
       1 & 1 & \cdots & 1 \\
       \vdots & \vdots & \ddots & \vdots \\
       1 & 1 & \cdots & 1 \\
   \end{pmatrix}
   \quad
   \begin{bmatrix}
   1 & 1 & \cdots & 1 \\
   1 & 1 & \cdots & 1 \\
   \vdots & \vdots & \ddots & \vdots \\
   1 & 1 & \cdots & 1 \\
   \end{bmatrix}
   $$

   行列式: 

   $$
   \begin{vmatrix}
   1 & 1 & \cdots & 1 \\
   1 & 1 & \cdots & 1 \\
   \vdots & \vdots & \ddots & \vdots \\
   1 & 1 & \cdots & 1 \\
   \end{vmatrix}
   $$

10. 特殊字符
   
   ![特殊字符表](images/2021-06-10-11-22-57.png)
   更多特殊符号可以 [上网查询](https://oeis.org/wiki/List_of_LaTeX_mathematical_symbols)

   可以搜索 “Latex 符号表”

11. 零碎重要语法
* 点乘 $\cdot$, 叉乘 $\times$, 异或 $\otimes$, 直和 $\oplus$, 加减 $\pm$, 复合 $\circ$. 除$\div$
* 小于等于 $\leq$, 大于等于 $\geq$, 不等 $\neq$, 恒等 $\equiv$, 约等 $\approx$, 等价 $\cong$, 相似 $\sim$, 相似等于 $\simeq$, 点等 $\doteq$.
* 逻辑与 $\land$, 逻辑或 $\lor$, 逻辑非 $\lnot$, 蕴涵 $\to$, 等价 $\leftrightarrow$.
* 因为 $\because$, 所以 $\therefore$, 存在 $\exist$, 任意 $\forall$.
* 左小箭头 $\leftarrow$, 右小箭头 $\rightarrow$, 左大箭头 $\Leftarrow$, 右大箭头 $\Rightarrow$, 右长箭头 $\xrightarrow[fgh]{abcde}$.
* 属于 $\in$, 包含于 $\subset$, 真包含于 $\subseteq$, 交 $\cap$, 并 $\cup$, 空集 $\empty$
* 短向量 $\vec{x}$, 长向量 $\overrightarrow{AB}$, 上横线 $\overline{p}$.
* 无限 $\infty$, 极限 $\lim$, 微分 ${\rm d}$, 偏导 $\partial$, 点求导 $\dot{y}$, 点二阶导 $\ddot{y}$, 变化量 $\Delta$, 梯度 $\nabla$.
* 横省略 $\cdots$, 竖省略 $\vdots$, 斜省略 $\ddots$.
* 常见函数 $\sin$, $\cos$, $\tan$, $\arcsin$, $\arccos$, $\arctan$, $\ln$, $\log$, $\exp$.


12. 科学计算
    快捷键为 `Shift + Ctrl + Alt + E`
   * $\frac{d}{dx}(x^3+x^2+1)=x \left(3 x + 2\right)$



