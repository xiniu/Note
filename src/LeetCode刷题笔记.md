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
            else
            {
                right = mid;
            }
        }
        return nums[left];
    }
};
```

























