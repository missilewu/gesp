
## 提示词
----
你是GESP, NOI, IOI的考试和竞赛的培训老师，正在总结附件中GESP五级C++考试真题的考点和相关的知识点。详细的分析以下内容：
1. 题意的理解
2. 题目核心要求
3. 解题思路
4. 样例的分析
5. 完整的代码和逻辑解析
6. 参考代码的逻辑解析和注释
7. 易错点和注意事项
   
---
## 官方代码参考代码

```cpp
#include <cstdio>

using namespace std;

int a, p;
int ans;

int fpw(int b, int e) {
	if (e == 0)
		return 1;
	int r = fpw(b, e >> 1);
	r = 1ll * r * r % p;
	if (e & 1)
		r = 1ll * r * b % p;
	return r;
}

void check(int e) {
	if (fpw(a, e) == 1)
		ans = 0;
}

int main() {
	int T;
	scanf("%d", &T);
	while (T--) {
		scanf("%d%d", &a, &p);
		ans = 1;
		int phi = p - 1, r = phi;
		for (int i = 2; i * i <= phi; i++)
			if (phi % i == 0) {
				check(phi / i);
				while (r % i == 0)
					r /= i;
			}
		if (r > 1)
			check(phi / r);
		printf(ans ? "Yes\n" : "No\n");
	}
}
```