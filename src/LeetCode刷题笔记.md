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

    

## 二分法

### No. 4寻找两个有序数组的中位数
