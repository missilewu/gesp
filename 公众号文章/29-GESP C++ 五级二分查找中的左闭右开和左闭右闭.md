
# GESP C++ 五级二分查找中的左闭右开和左闭右闭

GESP C++ 五级真题的客观题中出现二分查找（以及答案）题型，主要考察是以下内容：
1. mid 的计算
2. left 和 righ 两个边界的收敛方向和算法
3. lower_bound 和 upper_bound 的意义

下面是2026年3月，以及2025年真题的代码，请先仔细思考答案。

## 一、 真题和变形
### 1. binarySearch

```cpp
int binarySearch(int arr[], int left, int right, int target) {
	while (left <= right) {
		________________________________ // 在此处填入代码
		if (arr[mid] == target)
			return mid;
		else if (arr[mid] < target)
			left = mid + 1;
		else
			right = mid - 1;
	}
	return -1;
}
```

**答案：**
```cpp
int mid = left + (right - left) / 2;
```


上题，变化一下 `while` 循环边界为：`l < r`
```cpp
int binarySearch(int arr[], int left, int right, int target) {
	while (left < right) {
		int mid = left + (right - left) / 2;
		________________________________ // 在此处填入代码
	}
	return -1;
}
```

**答案：**
```cpp
if (arr[mid] == target)
    return mid;
else if (arr[mid] < target)
    left = mid + 1;
else
    right = mid - 1;
```

### 2. lowerBound
```cpp
int lowerBound(const vector<int>& a, int x) {
	int l=0, r=a.size();
	while(l<r) {
		int mid = l + (r - l)/2;
		if(a[mid] >= x) _____________;
		else l = mid + 1;
	}
	return l;
}
```

**答案：**

```cpp
r = mid;
```



### 3. 切单根木头

⼩杨要把⼀根长度为 L 的⽊头切成 K 段，使得每段长度⼩于等于 x 。已知每切⼀⼑只能把⼀段⽊头分成两段，他⽤⼆分法找到满⾜条件的最⼩ x （ x 为正整数），则横线处应填写（ ）。
```cpp
// 判断：在不超过 K 次切割内，是否能让每段长度 <= x
bool check(int L, int K, int x) {
	int cuts = (L - 1) / x;
	return cuts <= K;
}
// 二分查找最小可行的 x
int binary_cut(int L, int K) {
	int l = 1, r = L;
	while (l < r) {
		int mid = l + (r - l) / 2;
		________________________________ // 在此处填入代码
	}
	return l;
}
int main() {
	int L = 10; // 木头长度
	int K = 2; // 最多切 K 刀
	cout << binary_cut(L, K) << endl;
	return 0;
}
```

**答案：**

```cpp
if (check(L, K, mid)) 
    r = mid; 
else 
    l = mid + 1;
```


上题，变化一下 `while` 循环边界为：`l <= r`

```cpp
int binary_cut(int L, int K) {
	int l = 1, r = L;
	while (l <= r) {
		int mid = l + (r - l) / 2;
		________________________________ // 在此处填入代码
	}
	return l;
}
```

**答案：**

```cpp
if (check(L, K, mid)) 
    r = mid - 1; 
else 
    l = mid + 1;
```


### 4. 切多根木头

给定 n 根⽊头，第 i 根长度为 `a[i]` 。要切成不少于 m 段等长⽊段，求最⼤可能长度，则横线上应填写（ ）。

```cpp
const int MAXN = 100005;
long long a[MAXN];
int n, m;
bool check(long long x) {
	long long cnt = 0;
	for(int i = 1; i <= n; i++) {
		if(x == 0) return true;
		cnt += a[i] / x;
		if(cnt >= m) return true;
	}
	return false;
}
int main() {
	cin >> n >> m;
	long long mx = 0;
	for(int i = 1; i <= n; i++) {
		cin >> a[i];
		mx = max(mx, a[i]);
	}
	long long l = 1, r = mx;
	long long ans = 0;
	while(l <= r) {
		long long mid = l + (r - l) / 2;
		if(check(mid)) {
			ans = mid;
			______________________
		} else {
			______________________
		}
	}
	cout << ans << endl;
	return 0;
}
```

答案：

```cpp
  l = mid + 1;
  r = mid - 1;
```


-----

## 二、循环条件和区间的开闭分析

通过真题可以看到，对`循环的终止条件，以及left 和 righ 两个边界的收敛方向和算法`是考察的重点，而且是细节。

### 1. 先明确两个基础概念
#### (1) 区间符号含义

- **左闭右闭 $[l,\ r]$**：包含端点 `l` 和 `r`，区间内有效下标：$l,\ l+1,\dots,r$
- **左闭右开 $[l,\ r)$**：包含端点 `l`，**不包含**端点 `r`，区间内有效下标：$l,\ l+1,\dots,r-1$

#### (2) 循环条件的本质

`while(条件)` 的含义：**只要条件成立，区间内就还有元素，继续查找**；条件不成立时，**区间已无待查元素**，停止循环。停止循环后，循环变量的数据还是有效的，指向的位置可为后续程序逻辑所用。例如：answer。

---

### 2. 其次是理解`while`循环和开闭区间的映射关系

#### （1） `while(left < right)` → 左闭右开 $[\boldsymbol{left,\ right})$

临界状态分析：当 `left < right` 不成立时，结果一定是：
$$\boldsymbol{left = right}$$

此时如果是**有效下标**，必须满足：下标 $\boldsymbol{< right}$。
也就是说：**当前待查的所有元素，都在 $[\boldsymbol{left,\ right})$ 这个范围里**，`right` 本身不是待查元素。但是，注意这个`但是`, 当循环结束时，`left`是指向了`right`，也就是`right`就是要找的答案。这就是为什么区间收敛的时候`right = mid`的原因，因为
- `mid`是一个潜在的答案
- 如果 `[left, right)`范围内没有找到比`mid`更合适的答案，那么`mid`最终就会被选中
- 如果找到了，那么就继续缩小边界。 

---

#### (2) `while(left <= right)` → 左闭右闭 $[\boldsymbol{left,\ right}]$

临界状态分析：当 `left <= right` 不成立时，结果一定是：
$$\boldsymbol{left > right}$$
更准确的说是： `left = right + 1`;

此时如果是**有效下标**，必须满足：下标 $\boldsymbol{\le right}$。
也就是说：**当前待查的所有元素，都在 $[\boldsymbol{left,\ right}]$这个范围里**，`right` 本身就是待查元素。 当循环结束时，`left`是指向了`right+1`，也就是`right+1`就是要找的答案，注意这个`right+1` 就是 `mid`。这就是为什么区间收敛的时候`right = mid - 1`的原因，因为
- `mid`是一个潜在的答案，但是在下个循环中不参与 `判断`
- 如果 `[left, right]`范围内没有找到比`mid`更合适的答案，那么 `mid` 最终就会被选中
- 如果找到了，那么就继续缩小边界


总之，抛开开闭区间理论，我是这么理解的。数据被切分成：`left, mid, right`时，`mid`是一个潜在的答案（ans），需要根据循环终止条件合理的安排 right 指向的位置，保证循环终止时，left 最终指向 mid，本质上用的利用C++语言的特性实现的。

## 三、关键结论

不是人为规定 `left < right` 对应左闭右开，而是**循环的终止逻辑 + 变量代表的查找范围**，自然推导出了区间形态。两套写法必须配套使用，不能混用更新语句。

### 情况1：`while(left < right)` （$[l,r)$）

目标元素一定在区间内部，更新规则必须匹配区间：
1. $arr[mid] < target$：目标在右侧 → 舍弃 mid，`left = mid + 1`
2. $arr[mid] > target$：目标在左侧 → 新区间是 $[l,\ mid)$，所以 `right = mid`
✅ 全程保证：待查区间永远是 $[l,\ r)$，循环结束时：`l == r == mid`

### 情况2：`while(left <= right)` （$[l,r]$）

1. $arr[mid] < target$ → `left = mid + 1`
2. $arr[mid] > target$ → 新区间是 $[l,\ mid-1]$，所以 `right = mid - 1`
✅ 全程保证：待查区间永远是 $[l,\ r]$，循环结束时：`l == r + 1 == mid`


加油💪，满分！！！
