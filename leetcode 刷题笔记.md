

# leetcode 刷题笔记



## 一、贪心

#### T517超级洗衣机



### T50,T29快速幂

~~~java
class Solution {
    public double myPow(double x, int n) {
        long N = n;
        return N>0?quickMul(x, N):1.0/quickMul(x, -N);
    }
    public double quickMul(double x, long n){
        if(n==0){
            return 1.0;
        }
        double y = quickMul(x, n/2);
        return n%2==0?y * y:y*y*x;
    }
}
~~~

正常求幂的复杂度位O(n)，直接x乘n次

快速幂就是利用分治思想

![image-20211012112755819](C:\Users\wy\Desktop\笔记\images\image-20211012112755819.png)

T29快速加

## 二、二分

### Offer_1_53

控制nusm[mid]>target 中的等于号可以控制二分查找到的数组中target的位置

当nusm[mid]>=target  时可以找到nums中target的左边界

当nusm[mid]>target 时可以找到nums中target的右边界 

```java
    int left = 0, right = nums.length - 1, ans = nums.length;
    while (left <= right) {
        int mid = (left + right) / 2;
        if (nums[mid] > target || (lower && nums[mid] >= target)) {
            right = mid - 1;
            ans = mid;
        } else {
            left = mid + 1;
        }
    }
    return ans;
```

## 三、数据结构

### 1、双向链表

一定要创建head和tail节点，要不然在头尾插入时需要大量的边界判断。

### 2、单调栈

单调栈中存放的数据应该是有序的，所以单调栈也分为**单调递增栈**和**单调递减栈**

- 单调递增栈：单调递增栈就是从栈底到栈顶数据是从大到小
- 单调递减栈：单调递减栈就是从栈底到栈顶数据是从小到大

```
stack<int> st;
//此处一般需要给数组最后添加结束标志符，具体下面例题会有详细讲解
for (遍历这个数组)
{
	if (栈空 || 栈顶元素大于等于当前比较元素)
	{
		入栈;
	}
	else
	{
		while (栈不为空 && 栈顶元素小于当前元素)
		{
			栈顶元素出栈;
			更新结果;
		}
		当前数据入栈;
	}
}

```

