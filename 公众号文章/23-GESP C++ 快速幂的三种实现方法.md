
GESP C++ 快速幂的三种实现方法

快速幂是算法竞赛中处理大数幂运算的“标配”工具，尤其是在涉及取模运算时。它的核心作用是将计算 $a^b$ 的时间复杂度从线性的 $O(b)$ 降低到对数级的 $O(\log b)$。这在指数 $b$ 非常大（比如 $10^9$ 甚至更大）时至关重要，否则程序会因为超时而无法通过。


## 朴素的幂取模运算

例如：下面的例子中，100个3相乘，远远大于了long long的表示范围，需要使用高精度整数的方式实现，复杂度高。

$$3^{100} \mod 10 = 3\times3\times3\times3 \text{....} \times3\times3 \mod 10$$

为了解决数据精度问题，我们可以根据模运算性质：
$$(A \times B) \pmod P = ((A \pmod P) \times (B \pmod P)) \pmod P$$

防止数字无限变大，在每一次乘法后都进行取模操作，保证单次运算的数据精度都是较小的。

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

// 计算 (base^exp) % mod 的函数（即 base 的 exp 次方对 mod 取模）
ll power(ll base, ll exp, ll mod) {
	ll result = 1;  // 初始化结果为 1
	// 通过循环 exp 次，每次将结果乘以 base 并对 mod 取模
	for (ll i = 0; i < exp; i++) {
		// 将当前结果 result 乘以底数 base，然后对 mod 取余，更新 result 的值
		// 这个是关键：跟上面的模运算性质对照着看
		result = result * base % mod;
	}
	return result;
}

int main() {
	// 从标准输入读取底数(base)、指数(exp)和模数(mod)
	ll base, exp, mod;
	cin >> base >> exp >> mod;	
	cout << power(base, exp, mod) << endl;	
	
	return 0;  // 程序正常结束
}
```

测试1：
```
输入：
3 100 10
结果：
1
```

测试2：
```
输入：
2 100 10
结果:
6
```


这个朴素的计算方法，虽然时间复杂度是 `O(n)`，但是当指数exp非常大的时候，万亿次的计算也是一个不小的开销，需要进行优化。这类优化的算法叫做**快速幂**，采用了`分治`的思想，把时间复杂度降到`O(logn)`。主要有两种实现：递归快速幂（二分法）和非递归快速幂（二进制拆分法）

---
## 递归快速幂（二分法）

### 逻辑是分治的思想：
- 如果指数 $b$ 是偶数：$a^b = (a)^{b/2} \times (a)^{b/2}$
- 如果指数 $b$ 是奇数：$a^b = (a)^{b/2} \times (a)^{b/2} \times a$

通过每次将指数除以 2，问题规模迅速缩小，时间复杂度从普通循环的 O(n) 降到了 O(log⁡n) 。n是指 b 的大小。

### 递归的结束条件是：
* b 为 0 是，$a^b$ 为 1

```cpp
// 递归实现快速幂：计算 (base^exp) % mod
ll quick_pow_recursive(ll base, ll exp, ll mod) {
	// 递归终止条件：任何数的 0 次方都等于 1
	if(exp == 0) {
		return 1;
	}
	
	// 分治思想：先递归计算出 base 的 (exp/2) 次方对 mod 取模的结果
	ll half = quick_pow_recursive(base, exp/2, mod);
	
	// 根据指数运算法则：base^exp = (base^(exp/2))^2
	// 先计算 half * half 并对 mod 取模，防止中间结果溢出
	ll result = half * half % mod;
	
	// exp & 1 是位运算，等价于 exp % 2，用来判断指数 exp 是否为奇数
	if(exp & 1) {
		// 如果 exp 是奇数，还需要额外乘上一个底数 base，并再次取模
		result = result * base % mod;
	}

	// 返回最终计算结果
	return result;
}
```

**位运算小技巧**：  
    `if(exp & 1)` 是 C++ 竞赛和底层开发中非常常见的写法。`&` 是按位与运算，`exp & 1` 的结果如果是 1，说明 `exp` 的二进制最后一位是 1（即为奇数）；如果是 0，说明是偶数。它的效果和 `if(exp % 2 == 1)` 完全一样，但运行效率稍高一些。

---
## 非递归快速幂（二进制拆分法）

### 逻辑是利用了**二进制**的思想

* 任何一个整数 $b$ 都可以拆分为二进制形式。

例如，假设我们要计算 $3^{13}$。
指数 $13$ 的二进制是 $1101$，这意味着：
$$13 = 1 \times 2^3 + 1 \times 2^2 + 0 \times 2^1 + 1 \times 2^0 = 8 + 4 + 0 + 1$$

所以：
$$3^{13} = 3^{(8+4+0+1)} = 3^8 \times 3^4 \times 3^0 \times 3^1$$

**快速幂的逻辑是：**
我们不需要计算 13 次乘法，而是通过不断对底数“平方”，得到 $3^1, 3^2, 3^4, 3^8 \dots$ 这些项。如果指数 $b$ 的当前二进制位是 1，我们就把对应的项乘入结果中。该方法的时间复杂度是`O(logn)`。在竞赛中，**迭代版（非递归）** 更常用，因为它效率稍高且不会因为递归层数过深导致栈溢出。

```cpp
ll quick_pow(ll base, ll exp, ll mod) {
	// 初始化结果变量，任何数的0次方为1
	ll result = 1;
	// 先将底数取模，防止初始底数就大于模数导致溢出
	base = base % mod;
	
	// 当指数大于0时，持续循环
	while(exp > 0) {
		// 判断当前指数的二进制最低位是否为1（即判断指数是否为奇数）
		if(exp & 1) {
			// 如果当前位是1，说明需要累乘当前的底数到结果中，并取模防止溢出
			result = result * base % mod;
		}
		// 底数不断自乘（平方），对应二进制位的权重（a^1, a^2, a^4, a^8...），并取模
		base = base * base % mod;
		// 将指数右移一位（等价于除以2），处理二进制的下一位
		exp >>= 1;
	}
	// 返回最终计算结果
	return result;
}
```
## 总结

- **朴素算法：** 循环 $b$ 次乘法，时间复杂度 $O(b)$。如果 $b=10^9$，计算机大概需要运行 1-2 秒甚至更久，容易超时。
- **快速幂：** 循环次数取决于 $b$ 的二进制位数，即 $\log_2 b$。如果 $b=10^9$，$\log_2(10^9) \approx 30$。只需要运算 30 次左右，速度极快（远小于 1 毫秒）。

## 全部代码

无注释，适合代码阅读理解

```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

ll power(ll base, ll exp, ll mod) {
	ll result = 1; 
	for (ll i = 0; i < exp; i++) {
		result = result * base % mod;
	}
	return result;
}

ll quick_pow_recursive(ll base, ll exp, ll mod) {
	if(exp == 0) {
		return 1;
	}
	ll half = quick_pow_recursive(base, exp/2, mod);
	ll result = half * half % mod;
	if(exp&1) {
		result = result*base%mod;
	}
	return result;
}

ll quick_pow(ll base, ll exp, ll mod) {
	ll result = 1;
	base = base % mod;
	while(exp > 0) {
		if(exp & 1) {
			result = result * base % mod;
		}
		base = base * base % mod;
		exp >>= 1;
	}
	return result;
}

int main() {
	ll base, exp, mod;
	cin >> base >> exp >> mod;
	cout << "朴素：" << power(base, exp, mod) << endl;
	cout << "递归：" << quick_pow_recursive(base, exp, mod) << endl; 
	cout << "迭代：" << quick_pow(base, exp, mod) << endl; 
	return 0;
}
```
