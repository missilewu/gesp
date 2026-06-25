# 主定理 完整解析（对应式子：$\boldsymbol{T(n)=2T\left(\dfrac{n}{2}\right)+O(n)}$）
## 一、主定理通用公式
针对分治递推标准形式：
$$
\boldsymbol{T(n) = a\,T\left(\frac{n}{b}\right) + f(n)}
$$
- $a \ge 1$：分解后的**子问题个数**
- $b>1$：原问题缩小为原来的 $\boldsymbol{\dfrac{1}{b}}$
- $f(n)$：分解 + 合并子问题的时间代价
- 关键比较项：$\boldsymbol{n^{\log_b a}}$

---
## 二、代入本题参数
$$T(n)=2T\left(\frac{n}{2}\right)+O(n)$$
1. 取值：
$a=2,\ b=2,\ f(n)=O(n)$

2. 计算临界幂次：

$$
\log_b a = \log_2 2 = 1
$$
$$
n^{\log_b a}=n^1 = n
$$

3. 阶数对比：
$$f(n)=O(n) \quad=\quad n^{\log_b a}$$

---
## 三、主定理三种情况（背诵版）
设 $\varepsilon>0$ 为极小常数
1. **情况一**：$f(n) < n^{\log_b a}$
$T(n)=\boldsymbol{O\left(n^{\log_b a}\right)}$

2. **情况二**：$\boldsymbol{f(n) \;=\; n^{\log_b a}}$
$T(n)=\boldsymbol{O\left(n^{\log_b a}\log n\right)}$

3. **情况三**：$f(n) > n^{\log_b a}$
$T(n)=\boldsymbol{O(f(n))}$

---
## 四、本题判定与结果
本题满足**主定理第二种情况**：
$$f(n)=\Theta(n)=\Theta(n^{\log_2 2})$$
直接得：
$$
\boldsymbol{T(n)=O(n\log n)}
$$

---
## 五、对应算法场景
该递推式是经典平衡分治复杂度：
- 快速排序 **最优/平均情况**
- 归并排序 全局复杂度
- 二叉树层序分治、二分+遍历类算法

---
需要我顺带把**快排最坏 $T(n)=T(n-1)+O(n)$** 为什么**不能用主定理**也讲一下吗？
-