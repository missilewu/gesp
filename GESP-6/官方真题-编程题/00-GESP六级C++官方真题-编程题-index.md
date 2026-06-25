
## 提示词
----
你是GESP, NOI, IOI的考试和竞赛的培训老师，正在总结附件中GESP六级C++考试真题的考点和相关的知识点。详细的分析以下内容：
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
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
int N;
vector<int> a, b;
int ans = 1e9;
int main() {
	cin >> N;
	a.resize(N);
	b.resize(N);
	for (int i = 0; i < N; ++i) {
		cin >> a[i];
	}
	for (int i = 0; i < N; ++i) {
		cin >> b[i];
	}
	vector<int> permutation;
	permutation.resize(N);
	for (int i = 0; i < N; i ++)
		permutation[i] = i;
	do {
		int curr_len = N;
		for (int i = 1; i < N; ++i) {
			curr_len += max(b[permutation[i - 1]], a[permutation[i]]);
		}
		ans = min(ans, curr_len);
	} while(next_permutation(permutation.begin(), permutation.end()));
	cout << ans << endl;
	return 0;
}
```