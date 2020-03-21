# LeetCode刷题笔记

## 刷题想法

- 漫无目的的刷--早期刷的几道题，没有按照固定套路或者有意识的分类去刷，但是这些题有些想法非常不错，或者用到了一些STL的高级用法等等，还是需要总结一下
- 根据公众号以及网上其他资源，分门别类的刷，例如：二分法/动态规划等

## 漫无目的的刷 

### No.1 两数之和

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

#### 一些基本思路

- 基本解法：两层遍历，但是值得注意的是，内层遍历变量j不需要从0开始遍历，只需要从i+1开始遍历
- 高阶解法：原题等价于对于某个数组元素a[i]，数组中是否存在值为target - a[i]。是否存在这种典型场景，可以使用set或者map；由于还要返回下标，所以使用map。循环每个元素a[i]， 判断target - a[i]是否已经在map中，如果存在，返回map中存储的下标以及当前下标i；如果不存在，在map中缓存(a[i],i)

#### 示例代码
```c++
#include <algorithm>
#include <vector>
#include <map>
class Solution {
public:
	vector<int> twoSum(vector<int>& nums, int target) {
		vector<int> result;
		map<int, int> valueIndex;
		int currentValue = 0;
		int leftValue = 0;
		for (int i = 0; i < nums.size(); i++) {
			currentValue = nums[i];
			leftValue = target - currentValue;
			if (valueIndex.count(leftValue))
			{
				result.push_back(valueIndex[leftValue]);
				result.push_back(i);
				break;
			} else {
				valueIndex[currentValue] = i;
			}
		}
		return result;

	}
};
```

### No.2 两数相加

给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

#### 一些基本思路

循环两条单链表，但是要注意链表长度可能不一样，需要额外处理。为了代码书写方便，可以先建一个头节点，后面再删掉

#### 示例代码
```C++
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode* head = new ListNode(-1);
        ListNode* pre = head;
        int added = 0;
        while (l1 || l2) {
             int sum = 0;
             if (l1) {
                  sum += l1->val;
                  l1 = l1->next;
            }
             if (l2) { 
                 sum += l2->val;
                 l2 = l2->next;
            }
            sum += added;
            pre->next = new ListNode(sum % 10);
            added = sum / 10;
            pre = pre->next;
        }
        if (added) {
            pre->next = new ListNode(added % 10);
        }
        ListNode* result = head->next;
        delete head;
        head = nullptr;
        return result;
    }
};
```

### No.3 无重复字符的最长子串

给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。

#### 一些基本思路

我自己想出来的思路：用一个129个的长度的flag数组标识每个字符上次出现的下表位置；从头开始遍历，维护几个变量：lenMax（最长长度），index（当前判断的连续字串的起始处）。如果当前循环到的元素根据flag数组判断已经存在且那个位置有效（即在本次判断连续字符范围内），则需要更新lenMAx， index以及flag[a[i]]。原来这就是滑动窗口

#### 示例代码
```C++
#include <cstring>

class Solution {
public:
	int lengthOfLongestSubstring(string s) {
		int flag[129];
		memset(flag, -1, sizeof flag);
		int index = 0;
		int lenMax = 0;
		int lenNow = 0;

		const char *p = s.c_str();
		for (int i = 0; i < s.length(); i++) {
			char ch = s.at(i);
			if (flag[ch] >= index){
				lenMax = lenNow > lenMax ? lenNow : lenMax;
				lenNow = i - flag[ch];
				index = flag[ch] + 1;
				flag[ch] = i;	
			}
			else {
				lenNow++;
				flag[ch] = i;
			}
		}
		lenMax = lenNow > lenMax ? lenNow : lenMax;
		return lenMax;
	}
};
```

### No.5 最长回文子串

给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

#### 一些基本思路

- 遍历串中每一个开始i和每一个结束j。判断i...j的串是否是回文串，效率会很差
- 遍地开花，枚举每一个回文串的中心。但是回文串长度是偶数时比较难办，最好的方法是预处理一番。将原字符用一个特定字符分开。这样，遍历每一个预处理后串的元素，以他为中心，想两边扩展，直到不能扩展，刷新最长回文串起始位置imax和长度max，对应原串位置imax/2和 max/2
- 马拉车算法，专门讲解一下。

#### 示例代码

```C++
#include <algorithm>
#include <cstring>

class Solution {
public:
	string longestPalindrome(string s) {
		unsigned length = s.length()*2 + 2;
		char *str = new char[length];
		memset(str, 0, length);
		str[0] = '#';
		for (int i = 0 ; i < s.length(); i++) {
			str[2 * i + 1] = s.at(i);
			str[2 * i + 2] = '#';
		}
		int max = 0;
		int max_i = -1;
		int max_j = -1;
		for (int index = 1; index < length; index++)
		{
			int i = index - 1; 
			int j = index + 1;
			int len = 1;
			while(i >= 0 && j < length && str[i] == str[j]) {
				len += 2;
				i--;
				j++;
			}
			if (len > max) {
				max = len;
				max_i = i + 1;
				max_j = j - 1 ;
			}
		}
        return s.substr(max_i/2, max/2);

	}
};
```

#### 马拉车算法

可以参照https://mp.weixin.qq.com/s/Fu_MZ0FO2yJ3zsf8VfJGJw
马拉车算法的原理其实是上面遍地开花的进阶版，但是每次遍地开花时可以利用之前遍地开花时的结果，即利用已经获取到信息，类似于KMP算法。
- 需要一个辅助数组p，记录下循环当前元素之前的各个元素为中心的回文串的情况,p[i]时半径，就是从中心点开始，扩展的步数
- 维护一个中心center和右边界maxRight。maxRight是已经扫描了的最右边的元素下标，center是那次扫描对应的中心点。那么每次循环处理到一个新元素s[i]可以做以下分析：
  - 如果i >= maxRight，只能继续遍地开花。因为i后面的元素是啥情况一无所知
  - 如果i < maxRight， 取到i关于center的对称值mirror（mirror = 2\*center-i）则继续分情况讨论:
    - 如果p[m]值较小，小于maxRight-i，则p[i]肯定等于p[m]
    - 如果p[m]刚好等于maxRight-i，需要继续从maxRight开始向右扩展
    - 如果p[m]的值大于maxRight-i,则p[i]肯定只能等于maxRight-i。否则当初maxRight肯定可以再扩展嘛
结合这三种情况，i < maxRight时，可以参考p[m]和maxRight-i的最小值，并且再尝试扩展，这样的话代码相对好些一点

#### 马拉车解法代码

```C++
string longestPalindrome(string s) {
		unsigned length = s.length()*2 + 2;
		char *str = new char[length];  
		memset(str, 0, length);
		str[0] = '#';
		for (int i = 0 ; i < s.length(); i++) {
		    str[2 * i + 1] = s.at(i);
		    str[2 * i + 2] = '#';
		}
        // pre-handle
        int maxRight = 0;
        int center = 0;
        int *p = new int[length];
        memset(p, 0, length * sizeof(int));
	int max = 0;
	int max_i = 0;
        for (int i = 0; i < length; i++) {
            if (i < maxRight) // point 1
            {
                int m = 2 * center - i;
                p[i] = min(p[m], maxRight - i); // point 2
            }
            
            int ileft = i - (p[i] + 1) ;    // point 3, not the i - (p[i] + 1)/2
            int iright = i + (p[i] + 1);
            while (ileft >= 0 && iright < length && str[ileft] == str[iright]) {
                ileft--;
                iright++;
                p[i]++;
            }
            if (i + p[i] > maxRight)
            {
                maxRight = i + p[i];    // point 4
                center = i;
            }
            if (p[i] > max)
            {
                max = p[i];
                max_i = ileft + 1;
            }   
        }
        delete []str;
        delete []p;
        return s.substr(max_i/2, max);
	}
```

### No.7 整数反转

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。 假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−2^31,  2^31 − 1]。请根据这个假设，如果反转后整数溢出那么就返回0

####一些基本思路

这里的重点是考虑溢出。在可能导致溢出的地方加判断。另外注意INT_MAX和INT_MIN这几个宏的使用，这几个宏来自于c中的limits.h

####示例代码

```C++
class Solution {
public:
    int reverse(int x) {
        int result = 0;
        bool isPostive = (x > 0);
        while (x) {

            if ((isPostive && result > (INT_MAX - x % 10) / 10) ||
                (!isPostive && result < (INT_MIN - x % 10) /10))
            {
                return 0;
            } 
            result = result * 10 + x % 10;
            x /= 10;
        }
        return result;
    }
};
```
### No.9 回文数

判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

#### 一些基本思路

- 直接将数字取反形成一个新的数字再比较相等会导致溢出，理论上溢出也能保证答案正确，但是leetCode会报错，应该是有什么编译参数在校验
- 无脑的做法是把各位写在一个vector中再判断
- 事实上，可以按照思路1，但是不需要把数字完全取反，只取到一半就可以了

#### 示例代码

```C++
class Solution
{
public:
    bool isPalindrome(int x)
    {

        if (x < 0 || (x % 10 == 0 && x)) // 結尾是0的只有0本身是回文數字
        {
            return false;
        }

        int reverse = 0;
        while (x > reverse) //  直到判断超过了一般
        {
            reverse = reverse * 10 + x % 10;
            x /= 10;
        }
        return x == reverse || x == reverse / 10; // 要么相等 要么错一位
    }
};
```

### No.14 最长公共前缀

编写一个函数来查找字符串数组中的最长公共前缀。

#### 一些基本思路

- 分治法：其实只能当成分治法的一种练习，效率并不高
- 我自己更认同的算法，所有的串逐位比较，这样每个串的每个字符都被比较了一次，没有重复比较。

#### 示例代码

```C++
// 分治算法
class Solution2 {
public:
    string findLongestCommonPrefix(vector<string>& strs, int begin, int end){
        if (begin == end)
        {
            return strs[begin];
        }
        int mid = (begin + end) / 2;
        string str1 =  findLongestCommonPrefix(strs, begin, mid);
        string str2 =  findLongestCommonPrefix(strs, mid + 1, end);
        string s;
        for (int i = 0; i < str1.length() && i < str2.length() && str1[i] == str2[i]; i++)
        {
            s.push_back(str1[i]);
        }
        return s;
    }
    string longestCommonPrefix(vector<string>& strs) {
        if (strs.empty())
        {
            return "";
        }
        
        return findLongestCommonPrefix(strs, 0, strs.size()-1);
    }
};

// 更高效的算法
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if (strs.empty())
        {
            return "";
        }
        int result = -1;
        while (true)
        {
            bool isBreak = false;
            int at = result + 1;
            if (at >= strs[0].length())
            {
                break;
            }
            char atch = strs[0][at];
            
            for (int j = 1; j < strs.size(); j++) {
                if (at >= strs[j].length() || strs[j][at] != atch)
                {
                    isBreak = true;
                    break;
                }
                
            }
            if(isBreak) {
                break;
            }
            result++;
        }
        if (result + 1 > 0)
        {
            return strs[0].substr(0, result + 1);
        } else {
            return "";
        }     
    }
};
```

### No.406 根据身高重建队列

假设有打乱顺序的一群人站成一个队列。 每个人由一个整数对(h, k)表示，其中h是这个人的身高，k是排在这个人前面且身高大于或等于h的人数。 编写一个算法来重建这个队列。

#### 一些基本思路

- 看题目描述，基本是需要排序。重点是排序后怎么输出正确的原始队列
- 每一次都保证，子队列是满足条件的，那么最终插入完成后是满足条件的

#### 示例代码
```C++
class Solution
{
public:
    static bool cmp(vector<int> v1, vector<int> v2)
    {
        if (v1[0] != v2[0])
        {
            return v1[0] > v2[0];
        }
        else
        {
            return v1[1] < v2[1];
        }
    }

    // 先按照体重降序，体重相等的再根据个数升序
    // 这样，再将排序后的队列进行插入即可
    // 每一次插入时，插入在beg+vector[1]的位置都能保证，当前为止的顺序是正确的
    // 例如，最大的元素时第一个被插入，对应的vector[1]为0，插入在beg+0
    // 继续插入第二大元素，如果vector[1]为0，肯定插入在beg+0；否则vector[1]为1，插入在beg+1

    vector<vector<int>> reconstructQueue(vector<vector<int>> &people)
    {
        sort(people.begin(), people.end(), cmp);

        vector<vector<int>> result;
        //result.resize(people.size());
        for (auto &v : people)
        {
            result.insert(result.begin() + v[1], v);
        }
        result.resize(people.size());
        return result;
    }
};
```

## 二分法

针对二分法，要转变原先的思维，将二分法转向夹逼的思想。每次去除一半的可能空间。要深刻理解这句话，明确解空间是什么，取中位数和排除的逻辑，注意不要进入死循环。建议详细参考公众号以下文档：https://mp.weixin.qq.com/s/gjXOjOt32d8oAbb40tN5_Q

### No.704 二分查找

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的target，如果目标值存在返回下标，否则返回 -1。

#### 一些基本思路

- 最简单的二分场景
- 或者使用stl的lower_bound/upper_bound
  - lower_bound：Returns an iterator pointing to the first element in the range [first, last) that is not less than (i.e. greater or equal to) value, or last if no such element is found.
  - upper_bound:Returns an iterator pointing to the first element in the range [first, last) that is greater than value, or last if no such element is found.
  
#### 示例代码

```C++
class Solution {
public:
    // Your runtime beats 95.16 % of cpp submissions
    // Your memory usage beats 74 % of cpp submissions (10.9 MB)
    int search(vector<int>& nums, int target) {
        int begin = 0;
        int end = nums.size() - 1;

        while (begin < end)
        {
            int mid = (begin + end) / 2;
            if (nums[mid] < target)
            {
                begin = mid + 1;
            } else {
                end = mid;
            }
        }
        if (nums[begin] == target)
        {
            return begin;
        } else {
            return -1;
        } 
    }

    // Your runtime beats 85.43 % of cpp submissions
    // Your memory usage beats 70 % of cpp submissions (11 MB)
    int search2(vector<int>& nums, int target) {
		auto it = lower_bound(nums.begin(), nums.end(), target);
		if (it != nums.end() && *it == target)
		{
			return it - nums.begin();
		}
		else
		{
			return -1;
		}
	}
};
```

### No.69 x 的平方根

计算并返回 x 的平方根，其中 x 是非负整数。
由于返回类型是整数，结果只保留整数的部分，小数部分将被舍去。

#### 一些基本思路

- 解空间：0-x
- 排除的逻辑：如果m\*m > x， 则x的平方根肯定比m小

#### 示例代码

```C++
class Solution {
public:
    int mySqrt(int x) {
        int begin = 0;
        int end = x;
        while (begin < end) {
            int mid = (begin + end) / 2 + 1;
            if (mid > x / mid) {  // 这样能防止溢出
                end = mid - 1;
            } else {
                begin = mid;
            }
        }
        return begin;
        
    }
};
```
### No.287 寻找重复数

给定一个包含 n + 1 个整数的数组 nums，其数字都在 1 到 n 之间（包括 1 和 n），可知至少存在一个重复的整数。假设只有一个重复的整数，找出这个重复的数。
- 不能更改原数组（假设数组是只读的）。
- 只能使用额外的 O(1) 的空间。
- 时间复杂度小于 O(n^2) 。
- 数组中只有一个重复的数字，但它可能不止重复出现一次。

### 一些基本思路

- 根据要求无法使用set/map等对象辅助，不能更改数组即不能排序
- 这个地方比较精巧的使用二分法，以往都是二分数组下标；但是二分的精髓时对解空间二分。我们知道，重复数字肯定在1-n之间（解空间）。排除的逻辑时什么呢？假设没有重复数据，对于每个数字x，小于等于x的数字刚好等于x；回到原场景，如果对于中位数m，如果小于等于m的个数大于m，证明0-m肯定存在重复数字

### 示例代码

```C++
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int beg = 1;
        int end = nums.size() - 1;
        while (beg < end)
        {
            int mid = (beg + end) / 2;
            int cnt = 0;
            for (int i = 0; i < nums.size(); i++) {
                if (nums[i] <= mid) {
                    cnt++;
                }
            }
            if (cnt > mid) // this is important
            {
                end = mid;
            } else {
                beg = mid + 1;
            }  
        }
        return beg;
    }
};
```

### No.4 寻找两个有序数组的中位数

寻找两个有序数组的中位数。

#### 一些基本思路

- 将两个有序链表合并然后再取中位数的方法一般都会想到，但是复杂度不满足要求，
- 根据题目要求的复杂度，明显是想要二分法。这道题在看了题解时恍然大悟。其实可以用二分的思路解决，主要时以下思路：
  - 中位数的特点是将数组分成两部分，个数相等（偶数个元素）或者只差1
  - 尝试将两个数组分割，对于第一个数组的每一次分割（长度较小的数组，可以少分一点），第二个数组有一个唯一对应的分割（要满足第一个条件）
  - 所以只需要遍历数组有分割点，满足二分。关键要想清楚，每次如何排处一半的空间以及排除的条件。

#### 示例代码

```C++
class Solution
{
    double findMedianSortedArrays(vector<int> &nums1, vector<int> &nums2)
    {
        // devide two arrays A[0,...A.len -1] => A[0,...i-1], A[i,...A.len-1]
        //                   B[0,...B.len-1]  => B[0,...j-1], B[j,...B.len-1]
        // if i, j is correct answer, it's will follow this rules:
        // j = (m + n + 1) / 2 - i 
        // A[i-1] <= B[j]
        // B[j-1] <= A[i]

        //Your runtime beats 99.35 % of cpp submissions
        //Your memory usage beats 94.29 % of cpp submissions (9.5 MB)

        int m = nums1.size();
        int n = nums2.size();

        if (m > n) // 少分几次，选少的数组进行二分
        {
            return findMedianSortedArrays(nums2, nums1);
        }
        int iMin = 0;
        int iMax = m;
        while (iMin < iMax)
        {
            /* code */
            int i = (iMin + iMax) / 2;
            int j = (m + n + 1) / 2 - i;

            if (i != m && j != 0 && nums2[j - 1] > nums1[i]) // 分少了
            {
                iMin = i + 1;
            }
            else
            {
                iMax = i;
            }
        }
        int i = iMax;
        int j = (m + n + 1) / 2 - i;

        int maxLeft;
        if (i == 0)
        {
            maxLeft = nums2[j - 1];
        }
        else if (j == 0)
        {
            maxLeft = nums1[i - 1];
        }
        else
        {
            maxLeft = max(nums1[i - 1], nums2[j - 1]);
        }

        if ((m + n) & 1)
        {
            return maxLeft;
        }

        int minRight;
        if (i == m)
        {
            minRight = nums2[j];
        }
        else if (j == n)
        {
            minRight = nums1[i];
        }
        else
        {
            minRight = min(nums1[i], nums2[j]);
        }
        return (maxLeft + minRight) / 2.0;
    }
};
```


### No.153 寻找旋转排序数组中的最小值

假设按照升序排序的数组在预先未知的某个点上进行了旋转。( 例如，数组 [0,1,2,4,5,6,7]  可能变为 [4,5,6,7,0,1,2] )。请找出其中最小的元素。
 
#### 一些基本思路

使用二分法时，要搞清楚每一次排除一半可能性是怎么排除的。原来的有序数组被旋转，那么此时从左至右是两段上升序列（中间就是那个最低点，最小值）且第二段的最大值肯定要比第一段的最小值都要小。每次拿中间元素跟最后一个元素比较，如果中间元素大，那么这个突降点肯定在后半段

```c++
class Solution
{
public:
    int findMin(vector<int> &nums)
    {
        int len = nums.size();
        if (!len)
        {
            return 0;
        }
        int left = 0;
        int right = len - 1;
        while (left < right)
        {
            int mid = left + (right - left) / 2;
            if (nums[mid] > nums[right])
            { // 找这个突降点，因为原数组时升序的，所以...
                left = mid + 1;
            }
            else
            {
                right = mid;
            }
        }
        return nums[left];
    }
};
```
### No.154 寻找旋转排序数组中的最小值II

假设按照升序排序的数组在预先未知的某个点上进行了旋转。( 例如，数组 [0,1,2,4,5,6,7]  可能变为 [4,5,6,7,0,1,2] )。请找出其中最小的元素。
但是有重复数字
 
#### 一些基本思路
这道题的解法，是参考了网上的题解。主要是单独处理相等的情况。我怎么就没想出来呢？ 首先应该基于153，先思考：原先的排除场景是否有效，即nums[mid
] > nums[r]成立时，l = mid + 1这肯定是正确的；接下来再处理原先代码处理不了的场景：无非就是<和==，<就是原先的逻辑；==的场景比较难办，直接r--。但这样效率最差情况下比较差

```c++
class Solution
{
public:
    int findMin(vector<int> &nums)
    {
        int len = nums.size();
        if (!len)
        {
            return 0;
        }
        int left = 0;
        int right = len - 1;
        while (left < right)
        {
            int mid = left + (right - left) / 2;
            if (nums[mid] > nums[right])
            { // 找这个突降点，因为原数组时升序的，所以...
                left = mid + 1;
            }
            else if (nums[mid] < nums[right])
            {
                right = mid;
            }
            else
            {
                right--;
            }
        }
        return nums[left];
    }
};
```

### [1095] 山脉数组中查找目标值

给你一个 山脉数组 mountainArr，请你返回能够使得 mountainArr.get(index) 等于 target 最小 的下标 index
值
 
#### 一些基本思路

先求出顶峰 peak；然后试图在前半部分寻找，然后试图在后半部分寻找，最共使用三次二分。

```c++
class Solution
{
public:
    map<int, int> Cache;
    int getValue(int index, MountainArray &mountainArr)
    {
        if (!Cache.count(index))
        {
            int v = mountainArr.get(index);
            Cache[index] = v;
        }
        return Cache[index];
    }
    int findInMountainArray(int target, MountainArray &mountainArr)
    {

        // step1 find the peak
        int length = mountainArr.length();

        int l = 0;
        int r = length - 1;
        int peak = r;
        while (l < r)
        {
            peak = l + (r - l) / 2;
            if (getValue(peak, mountainArr) < getValue(peak + 1, mountainArr))
            {
                l = peak + 1;
            }
            else
            {
                r = peak;
            }
        }
        peak = l;

        // step2 find index in pre-half
        l = 0;
        r = peak;
        while (l < r)
        {
            int m = l + (r - l) / 2;
            if (getValue(m, mountainArr) < target)
            {
                l = m + 1;
            }
            else
            {
                r = m;
            }
        }
        if (getValue(l, mountainArr) == target)
        {
            return l;
        }

        // step3 find index in post-half
        l = peak;
        r = length - 1;
        while (l < r)
        {
            int m = l + (r - l) / 2;
            if (getValue(m, mountainArr) > target)
            {
                l = m + 1;
            }
            else
            {
                r = m;
            }
        }
        if (getValue(l, mountainArr) == target)
        {
            return l;
        }

        return -1;
    }
};
```












### [1095] 山脉数组中查找目标值


找到 K 个最接近的元素
 
#### 一些基本思路

- 寻找lower_bound(二分)
- 比较lower_bound两边，选取k个数

```
class Solution
{
public:
    vector<int> findClosestElements(vector<int> &arr, int k, int x)
    {
        // find lower_bound x
        int l = 0;
        int r = arr.size() - 1;
        while (l < r)
        {
            int m = l + (r - l) / 2;
            if (arr[m] < x)  // careful
            {
                l = m + 1;
            }
            else
            {
                r = m;
            }
        }
        int begin = l - 1;
        int end = l;
        int kbk = k;
        while (k--)
        {
            if (begin < 0 || end < arr.size() && arr[end] - x < x - arr[begin])
            {
                end++;
            }
            else
            {
                begin--;
            }
        }
        begin++;
        if (begin < 0)
        {
            begin = 0;
        }
        vector<int> result;
        result.insert(result.begin(), arr.begin() + begin, arr.begin() + begin + kbk);
        return result;
    }
};
```


## 动态规划

有固定的模板和思维方式，背包算法等一些经典题目要格外注意
https://mp.weixin.qq.com/s/erJPc8Xx9BBXY1ZiEXVvKg

### No.70 爬楼梯

假设你正在爬楼梯。需要 n 阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

注意：给定 n 是一个正整数。

#### 一些基本思路

最简单的动态规划题目，f(n) = f(n-1) + f(n-2)。正推或者反推，但要注意，反推时使用类似缓存的方法，避免重复计算

```C++
class Solution
{
public:
    int climbStairs(int n)
    {
        int f0 = 1;
        int f1 = 1;
        if (n < 2)
        {
            return 1;
        }
        n -= 1;
        while (n--)
        {
            int t = f1;
            f1 = f1 + f0;
            f0 = t;
        }
        return f1;
    }
};
```
```C++
class Solution
{

public:
    map<int, int> Cache;
    int climbStairs(int n)
    {
        if (!Cache.count(n))
        {
            int result;
            if (n < 2)
            {
                result = 1;
            }
            else
            {
                result = climbStairs(n - 1) + climbStairs(n - 2);
            }
            Cache[n] = result;
        }
        return Cache[n];
    }
}
```

### No.120 三角形最小路径和

给定一个三角形，找出自顶向下的最小路径和。每一步只能移动到下一行中相邻的结点上。

#### 基本思路
从最后一层往上反推，注意这里有滚动数组的思想

```C++
class Solution
{
public:
    int minimumTotal(vector<vector<int>> &triangle)
    {
        vector<int> result = triangle.back();
        for (int layer = triangle.size() - 2; layer > -1; layer--)
        {
            for (int i = 0; i < triangle[layer].size(); i++)
            {
                if (result[i] < result[i + 1])
                {
                    result[i] = triangle[layer][i] + result[i];
                }
                else
                {
                    result[i] = triangle[layer][i] + result[i + 1];
                }
            }
        }
        return result[0];
    }
};
```

### 53 最大子序和

#### 基本思路

- 直接根据逻辑遍历 贪心的思想

```C++
class Solution
{
public:
    int maxSubArray(vector<int> &nums)
    {
        if (nums.empty())
        {
            return -1;
        }
        int maxSum = nums[0];
        int sumForNow = nums[0];

        for (size_t i = 1; i < nums.size(); i++)
        {
            if (sumForNow < 0 && nums[i] > sumForNow)
            {
                sumForNow = nums[i];
            }
            else
            {
                sumForNow += nums[i];
            }

            if (sumForNow > maxSum)
            {
                maxSum = sumForNow;
            }
        }
        return maxSum;
    }
};
```
- 动态规划
```C++
class Solution
{
public:
    int maxSubArray(vector<int> &nums)
    {
        vector<int> dp;
        dp.reserve(nums.size());
        dp[0] = nums[0];
        int max = dp[0];
        for (int i = 1; i < nums.size(); i++)
        {
            if (dp[i - 1] > 0)
            {
                dp[i] = nums[i] + dp[i - 1];
            }
            else
            {
                dp[i] = nums[i];
            }
            if (dp[i] > max)
            {
                max = dp[i];
            }
        }
        return max;
    }
};
```

### 62不同路径

一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。
机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。问总共有多少条不同的路径？

#### 基本思路

动态规划

```C++
class Solution
{
public:
    int uniquePaths(int m, int n)
    {
        vector<vector<int>> dp;
        dp.resize(m);
        for (int i = 0; i < m; i++)
        {
            (dp[i]).resize(n);
        }
        for (int i = 0; i < n; i++)
        {
            dp[0][i] = 1;
        }
        for (int j = 0; j < m; j++)
        {
            dp[j][0] = 1;
        }

        for (int i = 1; i < m; i++)
        {
            for (int j = 1; j < n; j++)
            {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        return dp[m - 1][n - 1];
    }
};
```

### 63不同路径II
考虑有障碍物的场景

#### 基本思路

这里一开始理解上有偏差，我认为当前这个节点是障碍，则当前这个节点是可以到达的；但是下一个节点不能从这个节点过去，这样思考将问题复杂化了，一般这类题目，当前节点是障碍，则无法到达这个节点，依次处理问题将会简化。
```C++
class Solution
{
public:
    int uniquePathsWithObstacles(vector<vector<int>> &obstacleGrid)
    {
        int m = obstacleGrid.size();
        int n = obstacleGrid[0].size();
        cout << m << " " << n << endl;
        vector<vector<long long>> dp;
        dp.resize(m);
        for (int i = 0; i < m; i++)
        {
            (dp[i]).resize(n);
        }

        if (obstacleGrid[0][0])
            return 0;
        dp[0][0] = 1;
        for (int i = 1; i < n; i++)
        {
            if (obstacleGrid[0][i])
            {
                dp[0][i] = 0;
            }
            else
            {
                dp[0][i] = dp[0][i - 1];
            }
        }
        for (int j = 1; j < m; j++)
        {
            if (obstacleGrid[j][0])
            {
                dp[j][0] = 0;
            }
            else
            {
                dp[j][0] = dp[j - 1][0];
            }
        }

        for (int i = 1; i < m; i++)
        {
            for (int j = 1; j < n; j++)
            {
                cout << i << " " << j << " ";
                if (obstacleGrid[i][j])
                    dp[i][j] = 0;
                else
                    dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        return dp[m - 1][n - 1]; // careful m - 1 & n -1
    }
};
```

### 64最小路径和
和之前的三角形最小路径和类似。

#### 基本思路
最简单的动态规划题目，注意细心一点
```C++
class Solution
{
public:
    int minPathSum(vector<vector<int>> &grid)
    {
        if (grid.empty() || grid[0].empty())
        {
            return -1;
        }
        int m = grid.size();
        int n = grid[0].size();
        vector<vector<int>> dp;
        dp.resize(m);
        for (int i = 0; i < m; i++)
        {
            dp[i].resize(n);
        }
        dp[0][0] = grid[0][0];
        for (int j = 1; j < n; j++)
        {
            dp[0][j] = dp[0][j - 1] + grid[0][j];   // careful !!!
        }
        for (int i = 1; i < m; i++)
        {
            dp[i][0] = dp[i - 1][0] + grid[i][0];
        }

        for (int i = 1; i < m; i++)
        {
            for (int j = 1; j < n; j++)
            {
                dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
            }
        }
        return dp[m - 1][n - 1];
    }
};
```

### 221正方形面积
在一个由 0 和 1 组成的二维矩阵内，找到只包含 1 的最大正方形，并返回其面积。

#### 基本思路
这道题的递推公式比较难找，记录下以每个坐标为右下角的正方形的最大边长，则
dp[i][j] = min(dp[i-1][j-1], dp[i-1][j], dp[i][j-1]) + 1
在纸上画一画确实是这样，但是自己思考确实有些难度
```C++
class Solution
{
public:
    int maximalSquare(vector<vector<char>> &matrix)
    {
        if (matrix.empty() || matrix[0].empty())
        {
            return 0;
        }
        int m = matrix.size();
        int n = matrix[0].size();
        vector<vector<int>> dp;
        dp.resize(m);
        int maxLength = 0;
        for (int i = 0; i < m; i++)
        {
            dp[i].resize(n);
            dp[i][0] = matrix[i][0] - '0';
            if (dp[i][0] > maxLength)
                maxLength = dp[i][0];
        }
        for (int j = 0; j < n; j++)
        {
            dp[0][j] = matrix[0][j] - '0';
            if (dp[0][j] > maxLength)
                maxLength = dp[0][j];
        }

        for (int i = 1; i < m; i++)
        {
            for (int j = 1; j < n; j++)
            {
                if (matrix[i][j] - '0')
                {
                    dp[i][j] = min(dp[i - 1][j], min(dp[i][j - 1], dp[i - 1][j - 1])) + 1;
                    if (dp[i][j] > maxLength)
                        maxLength = dp[i][j];
                }
                else
                {
                    dp[i][j] = 0;
                }
            }
        }
        return maxLength * maxLength;
    }
};
```


### [887] 鸡蛋掉落

你将获得 K 个鸡蛋，并可以使用一栋从 1 到 N  共有 N 层楼的建筑。
每个蛋的功能都是一样的，如果一个蛋碎了，你就不能再把它掉下去。
你知道存在楼层 F ，满足 0 <= F <= N 任何从高于 F 的楼层落下的鸡蛋都会碎，从 F 楼层或比它低的楼层落下的鸡蛋都不会破。
每次移动，你可以取一个鸡蛋（如果你有完整的鸡蛋）并把它从任一楼层 X 扔下（满足 1 <= X <= N）。
你的目标是确切地知道 F 的值是多少。
无论 F 的初始值如何，你确定 F 的值的最小移动次数是多少？

#### 基本思路

这道题目时看讲解做出来的 https://mp.weixin.qq.com/s/xn4LjWfaKTPQeCXR0qDqZg
这里思维上觉得比困难的点在于：每层楼碎与不碎，背后都是整个结果，即能够遍历所有的可能性。这种情况下，遍历区间，选出一个最小的结果

```C++
class Solution
{
public:
    int superEggDrop(int K, int N)
    {
        return dp(K, N);
    }
    int dp(int K, int N)
    {
        pair pr{K, N};
        if (memo.count(pr))
        {
            return memo[pr];
        }

        int &result = memo[pr];
        result = 0;
        if (K == 1)
        {
            result = N;
        }
        else if (N == 0)
        {
            result = 0;
        }
        else
        {
            int temp = N;
            for (int i = 1; i <= N; i++)
            {
                temp = min(temp,
                           max(dp(K, N - i), dp(K - 1, i - 1)) + 1);
            }
            result = temp;
        }
        return result;
    }
    map<pair<int, int>, int> memo;
};
```

这种解法超时了。再考虑优化：在K，N固定时，dp(K, N - i)随i递减，dp(K-1,i-1)随i递增，最终的结果就是交点或则和交点附近。所以这有点像求谷值的性质，所有可以用二分法加速计算过程，这个地方的思想也比较灵活。
```C++
class Solution
{
public:
    int superEggDrop(int K, int N)
    {
        return dp(K, N);
    }
    int dp(int K, int N)
    {
        pair pr{K, N};
        if (memo.count(pr))
        {
            return memo[pr];
        }

        int &result = memo[pr];
        result = 0;
        if (K == 1)
        {
            result = N;
        }
        else if (N == 0)
        {
            result = 0;
        }
        else
        {
            int temp = N;

            int left = 1;
            int right = N;
            int mid = 0;
            while (left < right)
            {
                mid = left + (right - left + 1) / 2;
                int broke = dp(K - 1, mid - 1) + 1;
                int notBroke = dp(K, N - mid) + 1;
                if (broke > notBroke)
                {
                    right = mid - 1;
                    temp = min(temp, broke);
                }
                else
                {
                    left = mid;
                    temp = min(temp, notBroke);
                }
            }
            result = temp;
        }
        return result;
    }
    map<pair<int, int>, int> memo;
};
```
新的解法是：改变状态的定义，dp[i][j]重新定位为有i个鸡蛋，允许你仍j次情况下，你所能测试的最高楼层。于是，dp[k][m] = 1 + dp[k][m-1] + dp[k-1][m-1]。那么题目其实是再找一个m，使得dp[k][m]不小于N，很牛x
```C++
class Solution
{
public:
    int superEggDrop(int K, int N)
    {
        vector<vector<int>> dp;
        dp.resize(K + 1);
        for (int i = 0; i < dp.size(); i++)
        {
            dp[i].resize(N + 1);
        }

        int m = 0;
        while (dp[K][m] < N)
        {
            m++;
            for (int k = 1; k <= K; k++)
            {
                dp[k][m] = 1 + dp[k - 1][m - 1] + dp[k][m - 1];
            }
        }
        return m;
    }
};
```

### [0321] 词语接龙

- 基本想法： 就是DFS，但是一开始对于循环回路等想的太复杂了
```C++
const int NODE_SIZE = 52;
class Solution0321
{
public:
	int getLongestNum(vector<string>& strVec)
	{
		if (strVec.empty()) {
			return 0;
		}
		memset(graph, 0, sizeof(int) * NODE_SIZE * NODE_SIZE);
		memset(visit, 0, sizeof(int) * NODE_SIZE);
		for (auto& str : strVec) {
			graph[str.front() - 'A'][str.back() - 'A'] += 1;
		}
		for (int i = 0; i < NODE_SIZE; i++) {
			if (visit[i] == 0) {
				(void)dfs(i);
			}
		}
		int result = 0;
		for (int i = 0; i < NODE_SIZE; i++) {
			result = (result < visit[i]) ? visit[i] : result;
		}
		return result;

	}
	int dfs(int begin) {
		int max = 0;
		for (int i = 0; i < NODE_SIZE; i++) {
			if (graph[begin][i]) {  // have edge from begin-->i
				int temp = 0;
				if (visit[i] != 0) {
					temp = visit[i] + 1;
				}
				else {
					graph[begin][i]--;	
					temp = dfs(i) + 1;
					graph[begin][i]++;
				}
				if (max < temp) {
					max = temp;
				}
			}
		}
		max;
		visit[begin] = max;
		return max;
	}

	int graph[NODE_SIZE][NODE_SIZE];
	int visit[NODE_SIZE]; // the longest from position i
	
};
int main()
{
	vector<string> strVec{ "AC", "CD","DE","EA","AD","DA" };
	for (int i = 6; i < 100; i++) {
		strVec.push_back(strVec[i - 6]);
	}
	auto start = chrono::steady_clock::now();
	cout << Solution0321().getLongestNum(strVec) << endl;
	auto end = chrono::steady_clock::now();
	auto diff = end - start;
	cout << chrono::duration <double, milli>(diff).count() << " ms" << endl;
	return 0;
}

```


## 拓扑排序

拓扑排序的基本概念就是给定一些有向边，是否存在一个合理的序列满足这些关系。那么显然，如果存在环路肯定是不行的。如果没有环路，那么必定可以。在一条链上的节点先后有关系，保证先后顺序即可。

###  [210] 课程表 II

现在你总共有 n 门课需要选，记为 0 到 n-1。

在选修某些课程之前需要一些先修课程。 例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示他们: [0,1]

给定课程总量以及它们的先决条件，返回你为了学完所有课程所安排的学习顺序。

可能会有多个正确的顺序，你只要返回一种就可以了。如果不可能完成所有课程，返回一个空数组。

```C++
const int STATE_VISITED = 1;
const int STATE_BLANK = 0;
const int STATE_VISITING = -1;
class Solution
{
public:
    vector<int> findOrder(int numCourses, vector<vector<int>> &prerequisites)
    {
        state.resize(numCourses);
        graph.resize(numCourses);
        for (int i = 0; i < numCourses; i++)
        {
            graph[i].resize(numCourses);
        }
        for (auto &pair : prerequisites)
        {
            graph[pair[0]][pair[1]] = 1;
        }

        for (int i = 0; i < numCourses; i++)
        {
            if (state[i] != STATE_VISITED && !dfs(i))
                return vector<int>{};
        }
        return result;
    }
    bool dfs(int begin)
    {
        if (state[begin] == STATE_VISITING)
        {
            return false; // loop occure here!
        }

        state[begin] = STATE_VISITING;
        for (int i = 0; i < state.size(); i++)
        {
            if (graph[begin][i] && state[i] != STATE_VISITED && !dfs(i))
            {
                return false;
            }
        }
        state[begin] = STATE_VISITED;
        result.push_back(begin);
        return true;
    }
    vector<int> result;
    vector<int> state;
    vector<vector<int>> graph;
};
```

