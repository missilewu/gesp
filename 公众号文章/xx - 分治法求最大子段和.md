
# 分治法求最大子段和（最大子数组和）
## 一、问题定义
给定整数数组 `nums`，找出**连续子数组**，使其元素之和最大，返回该和。
例：`[-2,1,-3,4,-1,2,1,-5,4]`，答案为 `6`（子数组 `[4,-1,2,1]`）。

## 二、分治核心原理

将数组 `[l, r]` 从中点 `mid=(l+r)//2` 切为左右两段，全局最大子段和只有**三种可能**：
1. **完全在左区间 `[l, mid]`**：递归求解左区间最大值 `leftMax`
2. **完全在右区间 `[mid+1, r]`**：递归求解右区间最大值 `rightMax`
3. **跨越中点 mid**：左半取**以mid结尾的最大后缀和** + 右半取**以mid+1开头的最大前缀和**，相加得 `crossMax`

最终答案 = `max(leftMax, rightMax, crossMax)`

### 跨中点 `crossMax` 计算步骤
1. 从 `mid` 向左累加，记录过程中最大和 `leftCross`（左半最大后缀）
2. 从 `mid+1` 向右累加，记录过程中最大和 `rightCross`（右半最大前缀）
3. `crossMax = leftCross + rightCross`

### 递归终止条件
区间只剩一个元素（`l == r`），直接返回该元素。

## 三、复杂度分析
- 递归式：$T(n) = 2T(\frac{n}{2}) + O(n)$
- 时间复杂度：$O(n\log n)$
- 空间复杂度：$O(\log n)$（递归调用栈深度）

对比：暴力$O(n^2)$、DP(Kadane)$O(n)$（最优），分治多用于算法教学、线段树拓展。

## 四、完整代码实现
### 1. Python 基础版（最易懂）
```python
def maxSubArray(nums):
    def divide(l, r):
        # 递归终止：单个元素
        if l == r:
            return nums[l]
        mid = (l + r) // 2
        # 1. 左区间最大值
        left_max = divide(l, mid)
        # 2. 右区间最大值
        right_max = divide(mid + 1, r)

        # 3. 计算跨中点最大和 cross_max
        # 左半：从mid向左找最大后缀
        left_cross = float('-inf')
        cur = 0
        for i in range(mid, l - 1, -1):
            cur += nums[i]
            left_cross = max(left_cross, cur)
        # 右半：从mid+1向右找最大前缀
        right_cross = float('-inf')
        cur = 0
        for i in range(mid + 1, r + 1):
            cur += nums[i]
            right_cross = max(right_cross, cur)
        cross_max = left_cross + right_cross

        # 三者取最大
        return max(left_max, right_max, cross_max)

    return divide(0, len(nums) - 1)

# 测试
print(maxSubArray([-2,1,-3,4,-1,2,1,-5,4]))  # 输出6
print(maxSubArray([-1]))  # 输出-1
```

### 2. C++ 版本
```cpp
#include <vector>
#include <algorithm>
using namespace std;

class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        return divide(nums, 0, nums.size()-1);
    }
private:
    int divide(vector<int>& nums, int l, int r) {
        if (l == r) return nums[l];
        int mid = (l + r) / 2;
        int leftMax = divide(nums, l, mid);
        int rightMax = divide(nums, mid+1, r);

        // 跨中点
        int leftCross = INT_MIN, cur = 0;
        for(int i = mid; i >= l; i--){
            cur += nums[i];
            leftCross = max(leftCross, cur);
        }
        int rightCross = INT_MIN; cur = 0;
        for(int i = mid+1; i <= r; i++){
            cur += nums[i];
            rightCross = max(rightCross, cur);
        }
        int crossMax = leftCross + rightCross;
        return max({leftMax, rightMax, crossMax});
    }
};
```

### 3. 进阶分治（线段树思想，维护4个区间信息）
每个区间存储4个值：
- `sum`：区间总和
- `lmax`：区间以左端点开头最大前缀和
- `rmax`：区间以右端点结尾最大后缀和
- `mmax`：区间内部最大子段和

合并规则：
```
sum = 左.sum + 右.sum
lmax = max(左.lmax, 左.sum + 右.lmax)
rmax = max(右.rmax, 右.sum + 左.rmax)
mmax = max(左.mmax, 右.mmax, 左.rmax + 右.lmax)
```
适合多次区间查询场景（线段树模板）。

## 五、示例演算
数组：`[-2,1,-3,4,-1,2,1,-5,4]`
1. 中点 `mid=4`（值`-1`）
2. 左区间`[-2,1,-3,4,-1]`递归得最大值`4`
3. 右区间`[2,1,-5,4]`递归得最大值`3`
4. 跨中点计算：
   - mid向左累加：`-1 → 4-1=3 → -3+3=0 → 1+0=1 → -2+1=-1`，最大后缀`3`
   - mid+1向右累加：`2 → 3 → -2 → 2`，最大前缀`3`
   - crossMax = 3+3=6
5. `max(4,3,6)=6`，得到答案。

## 六、优缺点
### 优点
1. 分治思想标准，适合学习递归、分治算法
2. 可拓展为线段树，支持任意区间最大子段查询
### 缺点
1. 时间 $O(n\log n)$ 不如动态规划 $O(n)$ 高效
2. 递归栈存在空间开销，大数据量易栈溢出

需要我把**分治和Kadane动态规划**做一份对比表格吗？