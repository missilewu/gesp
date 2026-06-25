
同学们好！我是你们的GESP/NOI/IOI竞赛教练。初等数论是算法竞赛的基石，尤其在GESP高组和NOIP普及组/提高组中频繁出现。

以下是核心知识点的简明梳理，请重点掌握**性质**与**代码实现**。

### 1. 基础概念速查
*   **素数与合数**：
    *   **素数**：只有1和自身两个约数的自然数（$n > 1$）。注意：**1既不是素数也不是合数**。
    *   **判定**：$O(\sqrt{n})$ 试除法。
*   **约数与倍数**：
    *   若 $a | b$，则 $b = k \cdot a$。
    *   **约数个数定理**：若 $n = p_1^{c_1} p_2^{c_2} \dots p_k^{c_k}$，则约数个数 $d(n) = (c_1+1)(c_2+1)\dots(c_k+1)$。
*   **质因数分解**：任何大于1的整数都能唯一地写成素数的乘积形式。这是解决大多数数论问题的突破口。
*   **奇偶性**：
    *   奇 $\pm$ 奇 = 偶，奇 $\times$ 奇 = 奇。
    *   常用于构造题或排除法（如：若 $a^2 + b^2 = c^2$，则 $a,b$ 不能同为奇数）。
*   **同余与模运算**：
    *   $a \equiv b \pmod m \iff m | (a-b)$。
    *   **重要性质**：$(a+b)\%m = ((a\%m)+(b\%m))\%m$，$(a\times b)\%m = ((a\%m)\times(b\%m))\%m$。**除法没有直接分配律**，需使用**乘法逆元**。

---

### 2. 核心算法与定理

#### (1) 最大公约数 (GCD) 与 最小公倍数 (LCM)
*   **欧几里得算法 (辗转相除法)**：
    *   **原理**：$\gcd(a, b) = \gcd(b, a \bmod b)$。
    *   **复杂度**：$O(\log \min(a, b))$。
    *   **代码模板**：
        ```cpp
        int gcd(int a, int b) { 
	        return b == 0 ? a : gcd(b, a % b); 
		}
        ```
*   **LCM 公式**：
    *   $\text{lcm}(a, b) = \frac{a \times b}{\gcd(a, b)}$。
    *   **注意**：计算时先除后乘防止溢出：`a / gcd(a, b) * b`。

#### (2) 唯一分解定理 (算术基本定理)
*   **内容**：任意大于1的整数 $N$ 可唯一分解为 $N = p_1^{a_1} p_2^{a_2} \dots p_k^{a_k}$，其中 $p_i$ 为素数，$a_i$ 为正整数。
*   **应用**：
    *   求 $\gcd$：取各素因子指数的**最小值**。
    *   求 $\text{lcm}$：取各素因子指数的**最大值**。
    *   推导约数个数、约数和公式。

---

### 3. 素数筛法 (高频考点)

在需要处理 $1 \sim N$ 范围内大量素数时，必须使用筛法。

#### (1) 埃拉托斯特尼筛法 (埃氏筛 Eratosthenes)
*   **原理**：从小到大枚举，若 $i$ 是素数，则标记其所有倍数 $2i, 3i, \dots$ 为合数。
*   **复杂度**：$O(N \log \log N)$。
*   **特点**：代码简单，对于 $N \le 10^7$ 足够快。
*   **代码片段**：
    ```cpp
    bool is_prime[MAXN];
    void sieve_eratosthenes(int n) {
        fill(is_prime, is_prime + n + 1, true);
        is_prime[0] = is_prime[1] = false;
        for (int i = 2; i * i <= n; ++i) {
            if (is_prime[i]) {
                for (int j = i * i; j <= n; j += i) // 从 i*i 开始优化
                    is_prime[j] = false;
            }
        }
    }
    ```

#### (2) 线性筛法 (欧拉筛 Euler Sieve)
*   **原理**：保证每个合数只被其**最小质因子**筛掉一次。利用 `i % prime[j] == 0` 时跳出循环。
*   **复杂度**：$O(N)$。
*   **特点**：NOI/IOI 级别必备，处理 $N \ge 10^7$ 或需要同时求积性函数（如 $\phi, \mu$）时使用。
*   **代码模板**：
    ```cpp
    int primes[MAXN], cnt;
    bool is_prime[MAXN];
    void sieve_linear(int n) {
        fill(is_prime, is_prime + n + 1, true);
        is_prime[0] = is_prime[1] = false;
        for (int i = 2; i <= n; ++i) {
            if (is_prime[i]) primes[++cnt] = i;
            for (int j = 1; j <= cnt && i * primes[j] <= n; ++j) {
                is_prime[i * primes[j]] = false;
                if (i % primes[j] == 0) break; // 关键：保证只被最小质因子筛除
            }
        }
    }
    ```

### 💡 教练寄语
1.  **数据范围决定算法**：$N=10^5$ 可用 $O(N\sqrt{N})$ 或埃氏筛；$N=10^7$ 必须用线性筛。
2.  **防爆意识**：中间过程涉及乘法时，务必考虑 `long long` 或及时取模。
3.  **多练手撕**：不要只背结论，GESP和NOI常考手写 `gcd` 或 `sieve` 的变体。

加油，数论是逻辑的体操，多刷题自然通神！




# 初等数论 极简C++模板（GESP/NOI 直接复制用）
所有代码**无冗余、比赛通用、编译运行稳定**，适配 C++11 及以上。

## 1. 欧几里得算法（求最大公约数 gcd）
```cpp
#include <iostream>
using namespace std;

// 辗转相除法求 gcd
int gcd(int a, int b) {
    while (b != 0) {
        int temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}

// 最小公倍数 lcm
int lcm(int a, int b) {
    return a / gcd(a, b) * b; // 先除后乘，防止溢出
}

int main() {
    cout << gcd(12, 18) << endl;  // 输出 6
    cout << lcm(12, 18) << endl;  // 输出 36
    return 0;
}
```

## 2. 质因数分解（试除法）
```cpp
#include <iostream>
#include <vector>
using namespace std;

vector<int> factor(int n) {
    vector<int> res;
    // 从 2 试除到 sqrt(n)
    for (int i = 2; i * i <= n; i++) {
        while (n % i == 0) {
            res.push_back(i); // 记录质因子
            n /= i;
        }
    }
    if (n > 1) res.push_back(n); // 剩余的大质因子
    return res;
}

int main() {
    vector<int> ans = factor(60); // 2 2 3 5
    for (int x : ans) cout << x << " ";
    return 0;
}
```

## 3. 埃氏筛法（求素数表）
```cpp
#include <iostream>
#include <cstring>
using namespace std;

const int MAXN = 1e6 + 5;
bool is_prime[MAXN];

// 埃氏筛
void sieve(int n) {
    memset(is_prime, true, sizeof(is_prime));
    is_prime[0] = is_prime[1] = false;
    for (int i = 2; i * i <= n; i++) {
        if (is_prime[i]) {
            // 标记 i 的所有倍数
            for (int j = i * i; j <= n; j += i)
                is_prime[j] = false;
        }
    }
}

int main() {
    sieve(100);
    cout << is_prime[13] << endl; // 1 表示是素数
    return 0;
}
```

## 4. 线性筛（欧拉筛，最优素数筛）
**NOI 标准筛法，每个数只筛一次，速度最快**
```cpp
#include <iostream>
#include <cstring>
#include <vector>
using namespace std;

const int MAXN = 1e6 + 5;
bool is_prime[MAXN];
vector<int> primes;

// 线性筛（欧拉筛）
void linear_sieve(int n) {
    memset(is_prime, true, sizeof(is_prime));
    is_prime[0] = is_prime[1] = false;
    for (int i = 2; i <= n; i++) {
        if (is_prime[i]) primes.push_back(i);
        // 用当前素数去筛
        for (int p : primes) {
            if (i * p > n) break;
            is_prime[i * p] = false;
            if (i % p == 0) break; // 保证只被最小质因子筛去
        }
    }
}

int main() {
    linear_sieve(100);
    cout << is_prime[97] << endl; // 1
    return 0;
}
```

## 5. 唯一分解定理（求约数个数 / 约数和）
```cpp
#include <iostream>
using namespace std;

// 求 n 的正约数个数
int count_divisors(int n) {
    int res = 1;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            int cnt = 0;
            while (n % i == 0) {
                cnt++;
                n /= i;
            }
            res *= (cnt + 1);
        }
    }
    if (n > 1) res *= 2;
    return res;
}

int main() {
    cout << count_divisors(12) << endl; // 6 个约数
    return 0;
}
```

---

### 总结
1. **求gcd/lcm** → 用欧几里得算法
2. **分解质因数** → 试除法
3. **快速找素数** → 埃氏筛（简单）/ 线性筛（更快）
4. **约数个数/和** → 唯一分解定理模板

需要我把这些模板**合并成一个头文件**，比赛直接`#include`调用吗？