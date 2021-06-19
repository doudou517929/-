# 一、赋值运算符函数

思路：先用new分配新内容，再用delete释放已有内容。

```c++
CMyString& CMyString::operator=(const CMyString &str)
{
    // m_pData是类的一个私有成员属性
    if (this != &str) {
        CMyString strTemp(str); // 拷贝构造str放入strTemp
        char* pTemp = strTemp.m_pData; // 取出str的m_pData暂存
        strTemp.m_pData = m_pData; // 把要抛弃的值存入strTemp
        m_pData = pTemp; // 把暂存的值放入m_pData
    }
    return *this; // 函数结束，strTemp被自动释放
}
```

# 二、实现单例模式

思路：利用静态构造函数。解决创建时机过早的问题。

```c#
public sealed class Singleton5
{
    Singleton5()
    {
    }
    public static Singleton5 Instance
    {
        get
        {
            return Nested.instance;
        }
    }
    // 第一次通过属性Singleton5.Instance得到实例时，会走动调用Nested创建instance
    class Nested
    {
        static Nested()
        { 
        }
        internal static readonly Singleton5 instance = new Singleton5();
    }
}
```

# 三、数组中重复的数字

描述：长度n，数字在0~n-1。

#### 题目一、可以修改数组

描述：在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。请找出数组中任意一个重复的数字。

思路：检查每个下标i，看numbers[i] == i, 如果不是，把numbers[i]和numbers[numbers[i]]交换。如果numbers[i]==i,说明找到了。

时间O(n) 空间O(1)

```c++
int findRepeatNumber(vector<int>& nums) {
    int n = nums.size();
    int i;
    for (i = 0; i < n; ++i) {
        while (nums[i] != i) {
            if (nums[i] == nums[nums[i]]) {
                return nums[i];
            }
            swap(nums[i], nums[nums[i]]);
        }
    }
    return nums[i];
}
```

#### 题目二、不修改数组

思路：长度为n，数字范围0~n-1，取中间数字n/2，统计前半段比n/2小的数字的个数。出现次数大于n/2说明前半段有重复数字，继续分半。

```c++
int getDuplication(const int* numbers, int length)
{
    if (numbers == nullptr || length <= 0)
        return -1;
    int start = 1;
    int end = length - 1;
    while (end >= start) {
        int middle = ((end - start) >> 1) + start; // 取中位数
        int count = countRange(numbers, length, start, middle); // 统计数组中值在中位数前的个数
        // 首尾指向同一个位置，如果count>1说明有超过一个值为end的数，即为结果
        if (end == start) {
            if (count > 1)
                return start;
            else
                break;
        }
        // 如果count在前半段出现的次数比前半段从start到middle的数字多说明前半段有重复数组
        if (count > (middle - start + 1))
            end = middle;
        else
            start = middle + 1;
    }
    return -1;
}
int countRange(const int* numbers, int length, int start, int end)
{
    if (numbers == nullptr)
        return 0;
    int count = 0;
    for (int i = 0; i < length; ++i)
        if (numbers[i] >= start && numbers[i] <= end)
            ++count;
    return count;
}
```

# 四、二维数组中的查找

描述：每一行从左到右递增。每一列从上到下递增。输入一个整数，判断数组中是否含有该整数

思路：每次选取查找范围内的右上角数字。如果大于number则这一列不会有number，剔除。以此类推。如果小于number说明只会出现在右侧，但是右侧都已经被剔除了，所以只会在下面，剔除这一行。以此类推。

```c++
bool Find(int* matrix, int rows, int columns, int number)
{
    bool found = false;
    if (matrix != nullptr && rows > 0 && columns > 0) {
        int row = 0;
        int column = columns - 1;
        while (row < rows && column >= 0) {
            if (matrix[row * columns + column] == number) {
                found = true;
                break;
            } else if (matrix[row * columns + column] > number) {
                --column;
            } else {
                ++row;
            }
        }
    }
    return found;
}
```

# 五、替换空格

思路：先遍历一遍字符串统计空格总数。准备两个指针P1，P2。P1指向原始字符串末尾，P2指向替换后的字符串末尾。向前移动P1逐个复制到P2，直到碰到空格。碰到空格后，P1向前1格，P2向前移动3格插入"%20"。

时间O(n)

```c++
// length为字符数组的总容量
void ReplaceBlank(char string[], int length)
{
    if (string == nullptr || length <= 0)
        return;
    int originalLength = 0; // 字符串实际长度
    int numberOfBlank = 0;
    int i = 0;
    // 统计空格数量
    while (string[i] != '\0') {
        ++originalLength;
        if (string[i] == ' ')
            ++numberOfBlank;
        ++i;
    }
    int newLength = originalLength + numberOfBlank * 2;
    if (newLength > length)
        return; // 超过总容量
    int indexOfOriginal = originalLength; // P1
    int indexOfNew = newLength; // P2
    while (indexOfOriginal >= 0 && indexOfNew > indexOfOriginal) {
        // P1遍历到了空格
        if (string[indexOfOriginal] == ' ') {
            string[indexOfNew--] = '0';
            string[indexOfNew--] = '2';
            string[indexOfNew--] = '%';
        } else {
            string[indexOfNew--] = string[indexOfOriginal];
        }
        --indexOfOriginal;
    }
}
```

# 六、从尾到头打印链表

思路：栈，后进先出

```c++
struct ListNode
{
    int m_nKey;
    ListNode* m_pNext;
};
void PrintListReversingly_Iteratively(ListNode* pHead)
{
    std::stack<ListNode*> nodes;
    ListNode* pNode = pHead;
    while (pNode != nullptr) {
        nodes.push(pNode);
        pNode = pNode->m_pNext;
    }
    while (!nodes.empty()) {
        pNode = nodes.top();
        printf("%d\t", pNode->m_nValue);
        nodes.pop();
    }
}
```

# 七、重建二叉树

描述：输入二叉树前序和中序的结果，重建二叉树。结果中不含重复数字。

思想：前序第一个数字是根节点，在中序中找到它。根前面的就是左子树的值

```c++
struct BinaryTreeNode
{
    int m_nValue;
    BinaryTreeNode* m_pLeft;
    BinaryTreeNode* m_pRight;
};
BinaryTreeNode* Construct(int* preorder, int* inorder, int length)
{
    if (preorder == nullptr || inorder == nullptr || length <= 0)
        return nullptr;
    return ConstructCore(preorder, preorder + length - 1, inorder, inorder + length - 1);
}
// 递归构建树
BinaryTreeNode* ConstructCore(int* startPreorder, int* endPreorder, int* startInorder, int* endInorder)
{
    // 前序第一个数字是根
    int rootValue = startPreorder[0];
    BinaryTreeNode* root = new BinaryTreeNode();
    root->m_nValue = rootValue;
    root->m_pLeft = root->m_pRight = nullptr;
    // 递归出口：前序遍历到头，检查此节点是否为叶节点
    if (startPreorder == endPreorder) {
        if (startInorder == endInorder && *startPreorder == *startInorder)
            return root;
        else
            throw std::exception("Invalid input");
    }
    // 在中序遍历中找到根节点的值
    int* rootInorder = startInorder;
    while (rootInorder <= endInorder && *rootInorder != rootValue)
        ++rootInorder;
    // 没找到根节点
    if (rootInorder == endInorder && *rootInorder != rootValue)
        throw std::exception("Invalid input.");
    // 左子树长度
    int leftLength = rootInorder - startInorder;
    int* leftPreorderEnd = startPreorder + leftLength;
    if (leftLength > 0) {
        // 构建左子树
        root->m_pLeft = ConstructCore(startPreorder + 1, leftPreorderEnd, startInorder, rootInorder - 1);
    }
    if (leftLength < endPreorder - startPreorder) {
        // 构建右子树
        root->m_pRight = ConstructCore(leftPreorderEnd + 1, endPreorder, rootInorder + 1, endInorder);
    }
    return root;
}
```

```c++
class Solution {
public:
    vector<int> Preorder ;
    map<int,int> dic;
    
    TreeNode* build(int pre_root ,int in_left ,int in_right){
        //如果左边界大于右边界说明到过了叶子
        if(in_left > in_right){
            return NULL;
        }
        //pre_root 是先序里面的索引 ！！
        TreeNode* root = new TreeNode(Preorder[pre_root]);
        //获取先序中的节点在中序中的节点， 即index 左边就是这节点的左子树，index右边就是节点的右子树
        int index = dic[Preorder[pre_root]];
        //当前节点左树即为先序索引+1 （没了话会在下一次迭代返回NULL）
        root->left = build(pre_root+1,in_left,index-1);
        //当前节点右树即为 根结点在前序中的索引+左树所有节点数（即节点在中序中的索引）-左边界+1 ，下一次的左边界为根在中序的索引+1  
        root->right = build(pre_root+index-in_left+1,index+1 ,in_right);
        return root;
    }

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        //赋值至外部变量
        Preorder = preorder;
        //使用map映射inorder的值和索引，提高找到索引效率
        for(int i=0;i<inorder.size();i++){
            dic[inorder[i]] = i;
        }
        return build(0,0,preorder.size()-1);
    }
};
```



# 八、二叉树的下一个节点

描述：给定一棵二叉树和其中的一个节点，找出中序下一个节点。(parent,left,right)

思路：①如果节点有右子树，下一个节点就是其右子树中最左子节点。②如果没有右子树，而且它是父节点的左子节点，下一个节点就是其父节点。③如果没有右子树，它还是它父节点的右子节点，则沿着父节点指针向上遍历，直到找到一个它是父节点的左子节点的节点。

```c++
BinaryTreeNode* GetNext(BinaryTreeNode* pNode)
{
    if (pNode == nullptr)
        return nullptr;
    BinaryTreeNode* pNext = nullptr;
    // 右子树不为空
    if (pNode->m_pRight != nullptr) {
        BinaryTreeNode* pRight = pNode->m_pRight;
        // 寻找右子树的最左子节点
        while (pRight->m_pLeft != nullptr)
            pRight = pRight->m_pLeft;
        pNext = pRight;
    } else if (pNode->m_pParent != nullptr) {
        BinaryTreeNode* pCurrent = pNode;
        BinaryTreeNode* pParent = pNode->m_pParent;
        // 向上寻找它是其父节点最左子节点的节点
        while (pParent != nullptr && pCurrent == pParent->m_pRight) {
            pCurrent = pParent;
            pParent = pParent->m_pParent;
        }
        pNext = pParent;
    }
    return pNext;
}
```

# 九、用两个栈实现队列

描述：实现appendTail, deleteHead

思路：先进后出->先进先出。stack2不空时，栈顶是最先进入队列的元素可以弹出。stack2空时，把stack1中逐个压入stack2，弹出

```c++
class CQueue {
    stack<int> stack1,stack2;
public:
    CQueue() {
        while (!stack1.empty()) {
            stack1.pop();
        }
        while (!stack2.empty()) {
            stack2.pop();
        }
    }
    
    void appendTail(int value) {
        stack1.push(value);
    }
    
    int deleteHead() {
        // 如果第二个栈为空
        if (stack2.empty()) {
            while (!stack1.empty()) {
                stack2.push(stack1.top());
                stack1.pop();
            }
        } 
        if (stack2.empty()) {
            return -1;
        } else {
            int deleteItem = stack2.top();
            stack2.pop();
            return deleteItem;
        }
    }
};
```

补充：两个队列实现一个栈

# 十、斐波那契数列

```c++
long long Fibonacci(unsigned n)
{
    int result[2] = {0, 1};
    if (n < 2)
        return result[n];
    long long fibNMinusOne = 1;
    long long fibNMinusTwo = 0;
    long long fibN = 0;
    for (unsigned int i = 2; i <= n; ++i) {
        fibN = fibNMinusOne + fibNMinusTwo;
        fibNMinusTwo = fibNMinusOne;
        fibNMinusOne = fibN;
    }
    return fibN;
}
```

#### 题目二、青蛙跳台

描述：一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 `n` 级的台阶总共有多少种跳法。

```c++
int numWays(int n) {
    long long res = 0, tmp1 = 1, tmp2 = 1;
    if (0 == n || 1 == n) {
        return 1;
    } else {
        for (int i = 2; i <= n; ++i) {
            res = (tmp1 + tmp2) % 1000000007;
            tmp2 = tmp1;
            tmp1 = res;
        }
    }
    return res;
}
```

# 十一、旋转数组的最小数字

描述：旋转数组->把一个数组最开始的若干个元素搬到数组的末尾。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 `[3,4,5,1,2]` 为 `[1,2,3,4,5]` 的一个旋转，该数组的最小值为1。

思路：二分查找找到两个子数组分界线。两个指针分别指向首位。①一般情况，首指针>=尾指针。如果中间元素>=首指针，说明最小元素位于中间元素后面，更新首指针。同理更新尾指针，不断缩小范围。最终会指向两个相邻元素，尾指针指向的就是目标。②旋转后为排序数组本身，第一个元素就是目标。所以把indexMid初始化为首指针。③10111，11101采取顺序查找

```c++
int minArray(vector<int>& numbers) {
    int low = 0;
    int high = numbers.size() - 1;
    while (low < high) {
        int pivot = low + (high - low) / 2;
        // 最小值在pivot左侧
        if (numbers[pivot] < numbers[high]) {
            high = pivot;
        }
        // 最小值在pivot右侧
        else if (numbers[pivot] > numbers[high]) {
            low = pivot + 1;
        }
        // low = high时
        else {
            high -= 1;
        }
    }
    return numbers[low];
}
```

# 十二、矩阵中的路径

描述：判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。

思路：回溯法。

```c++
public:
	bool exist(vector<vector<char>>& board, string word) {
        rows = board.size();
        cols = board[0].size();
        // 遍历所有的格子，以它们为起点
        for(int i = 0; i < rows; i++) {
            for(int j = 0; j < cols; j++) {
                if(dfs(board, word, i, j, 0)) return true;
            }
        }
        return false;
    }
private:
    int rows, cols;
    bool dfs(vector<vector<char>>& board, string word, int i, int j, int k) {
        if(i >= rows || i < 0 || j >= cols || j < 0 || board[i][j] != word[k]) return false;
        if(k == word.size() - 1) return true;
        // 经过并且采纳的格子不能重复经过
        board[i][j] = '\0';
        bool res = dfs(board, word, i + 1, j, k + 1) || dfs(board, word, i - 1, j, k + 1) || dfs(board, word, i, j + 1, k + 1) || dfs(board, word, i , j - 1, k + 1);
        // 把之前标记为经过的格子恢复
        board[i][j] = word[k];
        return res;
    }
```

# 十三、机器人的运动范围

描述：地上有一个m行n列的方格，从坐标 [0,0] 到坐标 [m-1,n-1] 。一个机器人从坐标 [0, 0] 的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），也不能进入行坐标和列坐标的数位之和大于k的格子。例如，当k为18时，机器人能够进入方格 [35, 37] ，因为3+5+3+7=18。但它不能进入方格 [35, 38]，因为3+5+3+8=19。请问该机器人能够到达多少个格子？

思路：回溯法

```c++
// 求一个数字的位数和
int get(int x) {
    int res = 0;
    for (; x; x /= 10) {
        res += x % 10;
    }
    return res;
}
int movingCount(int m, int n, int k) {
    if (!k) {
        return 1;
    }
    // vis存储1/0, 表示这个格子能否到达。
    vector<vector<int>> vis(m, vector<int>(n, 0));
    int ans = 1;
    vis[0][0] = 1;
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if ((i == 0 && j == 0) || get(i) + get(j) > k) {
                continue;
            }
            // 边界判断。更新当前格子，看是否能到达。
            if (i - 1 >= 0) {
                vis[i][j] |= vis[i - 1][j];
            }
            if (j - 1 >= 0) {
                vis[i][j] |= vis[i][j - 1];
            }
            // 如果当前格子可以到达，vis[i][j]为1，更新结果
            ans += vis[i][j];
        }
    }
    return ans;
}
```

# 十四、剪绳子

描述：给你一根长度为 n 的绳子，请把绳子剪成整数长度的 m 段（m、n都是整数，n>1并且m>1），每段绳子的长度记为 k[0],k[1]...k[m-1] 。请问 k[0]\*k[1]*...\*k[m-1] 可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。

思路：贪婪算法。当n>=5时，尽可能多的剪长度为3的绳子。当剩下长度为4时，把剩下的剪成两段2.

```c++
int cuttingRope(int n) {
    if (n <= 3) {
        return n - 1;
    }
    // 尽可能多的剪去长度为3的绳子段
    int timesOf3 = n / 3;
    // 当剩下长度为4时
    if (n - timesOf3 * 3 == 1) {
        --timesOf3;
    }
    int timesOf2 = (n - timesOf3 * 3) / 2;
    return pow(3, timesOf3) * pow(2, timesOf2);
}
```

# 十五、二进制中1的个数

描述：请实现一个函数，输入一个整数（以二进制串形式），输出该数二进制表示中 1 的个数。例如，把 9 表示成二进制是 1001，有 2 位是 1。因此，如果输入 9，则该函数输出 2。

思路：如果一个二进制数!=0说明至少一位是1.①最右边一位是1，减去1时相当于给最后一位取反。②最后一位是0，减去1会把最右边的1变成0.

```c++
int hammingWeight(uint32_t n) {
    int count = 0;
    while (n) {
        ++count;
        n = (n - 1) & n;
    }
    return count;
}
```

# 十六、数值的整数次方

描述：实现函数double Power(double base, int exponent)，求base的exponent次方。不得使用库函数，同时不需要考虑大数问题。

思路：如果求x的32次方，可以把16次方做平方。以此类推，32次方需要5次乘法。
$$
a^n = 
\begin{cases}
a^{n/2} \cdot a^{n/2} & \quad \text{如果 } n \text{为偶数}\\
a^{(n-1)/2} \cdot a^{(n-1)/2}\cdot a & \quad \text{如果 } n \text{为奇数}
\end{cases}
$$

```c++
double myPow(double x, int n) {
    if (0 == x) {
        return 0;
    }
    long b = n;
    double res = 1.0;
    // 指数为负数，n次方为倒数
    if (b < 0) {
        x = 1 / x;
        b = -b;
    }
    while (b > 0) {
        // 右移后当前最右边为1则更新结果.如果为1说明是奇数，需要*基数
        if (1 == (b & 1)) {
            res *= x;
        }
        x *= x; // 随着b右移，更新base
        // 每右移一次相当于当前base的平方。
        b >>= 1;
    }
    return res;
}
```

# 十七、打印从1到最大的n位数

描述：输入数字 `n`，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的 3 位数 999。

思路：如果n范围很大，long类型也会溢出。需要用字符串模拟数字加法。首先在字符串表达的数字上模拟加法，再把字符串表达的数字打印出来。String 类型的数字的进位操作效率较低，生成的列表实际上是 n位0 - 9的 全排列 ，因此可避开进位操作，通过递归生成数字的 String 列表。基于分治算法的思想，先固定高位，向低位递归，当个位已被固定时，添加数字的字符串。

```c++
void printNumbers(int n, int index, string& str, vector<int> &res) {
    // 递归出口
    if (index == n) {
        // 把str转为整数,加入结果集
        int num = atoi(str.c_str());
        if (num != 0) {
            res.push_back(num);
        }
        return;
    }
    // 每一位 从0到9依次排列
    for (int i = 0; i < 10; ++i) {
        str[index] = i + '0';
        printNumbers(n, index + 1, str, res);
    }
}
vector<int> printNumbers(int n) {
    vector<int> res;
    string str;
    str.resize(n);
    printNumbers(n, 0, str, res);
    return res;
}
```

# 十八、删除链表的节点

#### 题目一、删除给定节点

描述：给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。返回删除后的链表的头节点。

```c++
class Solution {
public:
    ListNode* deleteNode(ListNode* head, int val) {
        if(head->val == val) return head->next;
        ListNode *pre = head, *cur = head->next;
        while(cur != nullptr && cur->val != val) {
            pre = cur;
            cur = cur->next;
        }
        if(cur != nullptr) pre->next = cur->next;
        return head;
    }
};
```

#### 题目二、删除链表中重复节点

描述：给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。

```c++
ListNode* deleteDuplicates(ListNode* head) {
    ListNode* p = head;
    ListNode* q = head;
    if (head == nullptr) {
        return nullptr;
    }
    while (p->next != nullptr) {
        while (p->next != NULL && p->next->val != p->val) {
            p = p->next;               
        }
        if (p->next != nullptr) {
            q = p->next;
            p->next = q->next;
            q->next = nullptr;
            delete q;
        }
    }        
    return head;
}
```

# 十九、正则表达式匹配

描述：请实现一个函数用来匹配包含'. '和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（含0次）。在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但与"aa.a"和"ab*a"均不匹配。

思路：如果p中ch是'.',可以匹配任意字符。如果不是'.',s中的也是ch，也匹配。继续。①p中第二个不是\**，如果第一个相互匹配，同时后移，继续。第一个不匹配，false。②第二个字符是\**，选择一：p后移两个字符，相当于*和它前面的字符被忽略了。选择二：保持不变

```c++
bool isMatch(string s, string p) {
    s=" "+s;//防止该案例：""\n"c*"
    p=" "+p;
    int m = s.size(), n = p.size();
    // 矩阵，存储所有的匹配可能。因为s和p都加上了一个空格，所以长度+1
    bool dp[m+1][n+1];
    memset(dp, false, (m+1)*(n+1));
    // 初始两个第一个都设置为空格
    dp[0][0] = true;
    for (int i = 1; i <= m; ++i) {
        for (int j = 1; j <= n; ++j) {
            // m, n存储在这之前是否成功
            if (s[i - 1] == p[j - 1] || p[j - 1] == '.') {
                // 匹配成功。如果前面的是成功的，则成功。如果前面失败了，也失败
                dp[i][j] = dp[i - 1][j - 1];
            } else if (p[j - 1] == '*') {
                if (s[i - 1] != p[j - 2] && p[j - 2] != '.') {
                    // *匹配了0个字符，*是第j-1个，如果*前面的字符匹配成功，则*所在的位置也成功
                    // abbb  abc*a
                    dp[i][j] = dp[i][j - 2];
                } else {
                    // *匹配了*前面0个或多个字符。j-1表示一个，j-2表示0个，i-1表示多个
                    dp[i][j] = dp[i][j-1] || dp[i][j-2] || dp[i-1][j];
                }
            }
        }
    }
    return dp[m][n];
}
```

# 二十、表示数值的字符串

描述：请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100"、"5e2"、"-123"、"3.1416"、"-1E-16"、"0123"都表示数值，但"12e"、"1a3.14"、"1.2.3"、"+-5"及"12e+5.4"都不是。

思路：有限状态机

```c++
class Solution {
public:
    // 状态：初始、符号、整数、小数点、没有整数的小数点、小数、指数、指数符号、指数数值、结束状态
    enum State {
        STATE_INITIAL,
        STATE_INT_SIGN,
        STATE_INTEGER,
        STATE_POINT,
        STATE_POINT_WITHOUT_INT,
        STATE_FRACTION,
        STATE_EXP,
        STATE_EXP_SIGN,
        STATE_EXP_NUMBER,
        STATE_END,
    };
	// 字符是数字、e、小数点、符号、空格、非法
    enum CharType {
        CHAR_NUMBER,
        CHAR_EXP,
        CHAR_POINT,
        CHAR_SIGN,
        CHAR_SPACE,
        CHAR_ILLEGAL,
    };
	// 检查并返回每个字符的类型
    CharType toCharType(char ch) {
        if (ch >= '0' && ch <= '9') {
            return CHAR_NUMBER;
        } else if (ch == 'e' || ch == 'E') {
            return CHAR_EXP;
        } else if (ch == '.') {
            return CHAR_POINT;
        } else if (ch == '+' || ch == '-') {
            return CHAR_SIGN;
        } else if (ch == ' ') {
            return CHAR_SPACE;
        } else {
            return CHAR_ILLEGAL;
        }
    }

    bool isNumber(string s) {
        // 键值对：当前状态-{出现的字符类型-转换后的所处状态}
        unordered_map<State, unordered_map<CharType, State>> transfer{
            {
                // 初始状态可能变为：空格-初始；数字-整数；小数点-.前没有数字；符号-符号数字
                STATE_INITIAL, {
                    {CHAR_SPACE, STATE_INITIAL},
                    {CHAR_NUMBER, STATE_INTEGER},
                    {CHAR_POINT, STATE_POINT_WITHOUT_INT},
                    {CHAR_SIGN, STATE_INT_SIGN},
                }
            }, {
                // 符号状态可能变为：数字-数字；小数点-.前没有数字
                STATE_INT_SIGN, {
                    {CHAR_NUMBER, STATE_INTEGER},
                    {CHAR_POINT, STATE_POINT_WITHOUT_INT},
                }
            }, {
                // 数字状态可能变为：数字-数字；指数-指数；小数点-小数点；空格-结束
                STATE_INTEGER, {
                    {CHAR_NUMBER, STATE_INTEGER},
                    {CHAR_EXP, STATE_EXP},
                    {CHAR_POINT, STATE_POINT},
                    {CHAR_SPACE, STATE_END},
                }
            }, {
                // 小数点状态可能变为：数字-小数；指数-指数；空格-结束
                STATE_POINT, {
                    {CHAR_NUMBER, STATE_FRACTION},
                    {CHAR_EXP, STATE_EXP},
                    {CHAR_SPACE, STATE_END},
                }
            }, {
                // .前没有整数：数字-小数
                STATE_POINT_WITHOUT_INT, {
                    {CHAR_NUMBER, STATE_FRACTION},
                }
            }, {
                // 小数：数字-小数；指数-指数；空格-结束
                STATE_FRACTION,
                {
                    {CHAR_NUMBER, STATE_FRACTION},
                    {CHAR_EXP, STATE_EXP},
                    {CHAR_SPACE, STATE_END},
                }
            }, {
                // 指数：数字-指数数字；符号-指数符号
                STATE_EXP,
                {
                    {CHAR_NUMBER, STATE_EXP_NUMBER},
                    {CHAR_SIGN, STATE_EXP_SIGN},
                }
            }, {
                // 指数符号：数字-指数
                STATE_EXP_SIGN, {
                    {CHAR_NUMBER, STATE_EXP_NUMBER},
                }
            }, {
                // 指数数字：数字-指数；空格-结束
                STATE_EXP_NUMBER, {
                    {CHAR_NUMBER, STATE_EXP_NUMBER},
                    {CHAR_SPACE, STATE_END},
                }
            }, {
                // 结束：空格-结束
                STATE_END, {
                    {CHAR_SPACE, STATE_END},
                }
            }
        };

        int len = s.length();
        State st = STATE_INITIAL;
		// 从头开始遍历
        for (int i = 0; i < len; i++) {
            CharType typ = toCharType(s[i]);
            // 如果当前状态下，出现了键值对中无法匹配的字符，没有下一步状态转换。失败
            if (transfer[st].find(typ) == transfer[st].end()) {
                return false;
            } else {
                // 更新状态
                st = transfer[st][typ];
            }
        }
        // 结束在合法状态
        return st == STATE_INTEGER || st == STATE_POINT || st == STATE_FRACTION || st == STATE_EXP_NUMBER || st == STATE_END;
    }
};
```

# 二十一、调整数组顺序使奇数位于偶数前面

描述：输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

思路：如果改调整标准，不需要修改大的逻辑框架。把判断的标准变成一个函数指针。

```c++
static bool isEven(int n) {
    return 0 == (n & 1);
}
void Reorder(vector<int>& nums, unsigned int len, bool(*func)(int)) {
    if (0 == len) {
        return;
    }
    int i = 0, j = len - 1;
    while (i < j) {
        while (i < j && !func(nums[i])) {
            ++i;
        }
        while (i < j && func(nums[j])) {
            --j;
        }
        if (i < j) {
            swap(nums[i], nums[j]);
        }
    }
}
vector<int> exchange(vector<int>& nums) {
    Reorder(nums, nums.size(), isEven);
    return nums;
}
```

# 二十二、链表中倒数第k个节点

描述：输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。

```c++
ListNode* getKthFromEnd(ListNode* head, int k) {
    if (head == NULL) {
        return NULL;
    }
    ListNode* former = head, *latter = head;
    int i = 1;
    while (i < k && latter->next != NULL) {
        latter = latter->next;
        ++i;
    }
    while (latter->next != NULL) {
        latter = latter->next;
        former = former->next;
    }
    return former;
}
```

# 二十三、链表中环的入口节点

思路：①确定链表中包含环：两个指针，一个指针一次走一步，另一个指针一次走两步。如果走得快的追上了走得慢的，证明包含环。如果走得快的到了末尾也没追上慢的，不包含环。②找入口：p1p2指向头，如果链表中的环有n个节点，则p1走n步，然后两个指针同时开始走。当p2到环入口时，p1已经绕环走了一圈。③确定n数目：①中两指针相遇的节点在环中，从此节点出发，一边移动一边计数，再次回到这节点时，就可得到数目。

```c++
ListNode* hasCycle(ListNode *head) {
    if (head == NULL) {
        return NULL;
    }
    ListNode* pSlow = head->next;
    if (pSlow == NULL) {
        return NULL;
    }
    ListNode* pFast = pSlow->next;
    while (pFast != NULL && pSlow != NULL) {
        if (pFast == pSlow) {
            return pFast;
        }
        pSlow = pSlow->next;
        pFast = pFast->next;
        if (pFast != NULL) {
            pFast = pFast->next;
        }
    }
    return NULL;
}
ListNode *detectCycle(ListNode *head) {
    ListNode* meetingNode = hasCycle(head);
    if (meetingNode == NULL) {
        return NULL;
    }
    // 得到环中节点数目
    int nodeInLoop = 1;
    ListNode* pNode1 = meetingNode;
    while (pNode1->next != meetingNode) {
        pNode1 = pNode1->next;
        ++nodeInLoop;
    }
    // 移动pNode1,次数为环中节点数目
    pNode1 = head;
    for (int i = 0; i < nodeInLoop; ++i) {
        pNode1 = pNode1->next;
    }
    // 移动两个节点
    ListNode* pNode2 = head;
    while (pNode1 != pNode2) {
        pNode1 = pNode1->next;
        pNode2 = pNode2->next;
    }
    return pNode1;
}
```

# 二十四、反转链表

```c++
ListNode* reverseList(ListNode* head) {
    ListNode* pReversedHead = NULL;
    ListNode* p = head;
    ListNode* prev = NULL;
    while (p != NULL) {
        ListNode* pNext = p->next;
        if (pNext == NULL) {
            pReversedHead = p;
        }
        p->next = prev;
        prev = p;
        p = pNext;
    }
    return pReversedHead;
}
```

# 二十五、合并两个排序的链表

```c++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode* dum = new ListNode(0);
        ListNode* cur = dum;
        while (l1 != NULL && l2 != NULL) {
            if (l1->val < l2->val) {
                cur->next = l1;
                l1 = l1->next;
            } else {
                cur->next = l2;
                l2 = l2->next;
            }
            cur = cur->next;
        }
        cur->next = l1 != NULL ? l1 : l2;
        return dum->next;
    }
};
```

# 二十六、树的子结构

描述：输入两棵二叉树A和B，判断B是不是A的子结构。(约定空树不是任意一个树的子结构)。B是A的子结构， 即 A中有出现和B相同的结构和节点值。

```c++
bool hasSubTree(TreeNode* tmpP, TreeNode* tmpQ) {
    if (tmpQ == NULL) {
        return true;
    }
    if (tmpP == NULL) {
        return false;
    }
    if (tmpP->val != tmpQ->val) {
        return false;
    }
    return hasSubTree(tmpP->left, tmpQ->left) && hasSubTree(tmpP->right, tmpQ->right);
}
bool isSubStructure(TreeNode* A, TreeNode* B) {
    bool result = false;
    if (A != NULL && B != NULL) {
        if (A->val == B->val) {
            result = hasSubTree(A, B);
        }
        if (!result) {
            result = isSubStructure(A->left, B);
        }
        if (!result) {
            result = isSubStructure(A->right, B);
        }
    }
    return result;
}
```

# 二十七、二叉树的镜像

```c++
void mirrorRecursively(TreeNode* root) {
    if (root == NULL) {
        return;
    }
    if (root->left == NULL && root->right == NULL) {
        return;
    }
    TreeNode* tmp = root->left;
    root->left = root->right;
    root->right = tmp;
    if (root->left) {
        mirrorTree(root->left);
    }
    if (root->right) {
        mirrorTree(root->right);
    }
}
TreeNode* mirrorTree(TreeNode* root) {
    TreeNode* p = root;
    mirrorRecursively(p);
    return root;
}
```

# 二十八、对称的二叉树

思路：比较二叉树的前序遍历和对称前序遍历判断。如果两个序列一样，就是对称的

```c++
bool isSymmetrical(TreeNode* r1, TreeNode* r2) {
    if (r1 == NULL && r2 == NULL) {
        return true;
    }
    if (r1 == NULL || r2 == NULL) {
        return false;
    }
    if (r1->val != r2->val) {
        return false;
    }
    return isSymmetrical(r1->left, r2->right) && 
        						isSymmetrical(r1->right, r2->left);
}
bool isSymmetric(TreeNode* root) {
    return isSymmetrical(root, root);
}
```

# 二十九、顺时针打印矩阵

描述：输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字。

```c++
void printMatrixInCircle(vector<vector<int>>& matrix, int columns, int rows, int start, vector<int>& res) {
    int endX = columns - 1 - start;
    int endY = rows - 1 - start;
    // 从左到右打印一行
    for (int i = start; i <= endX; ++i) {
        res.push_back(matrix[start][i]);
    }
    // 从上到下打印一列
    if (start < endY) {
        for (int i = start + 1; i <= endY; ++i) {
            res.push_back(matrix[i][endX]);
        }
    }
    // 从右到左打印一行
    if (start < endX && start < endY) {
        for (int i = endX - 1; i >= start; --i) {
            res.push_back(matrix[endY][i]);
        }
    }
    // 从下到上打印一列
    if (start < endX && start < endY - 1) {
        for (int i = endY - 1; i >= start + 1; --i) {
            res.push_back(matrix[i][start]);
        }
    }
}
vector<int> spiralOrder(vector<vector<int>>& matrix) {
    vector<int> res;
    int rows = matrix.size();
    if (0 == rows) {
        return res;
    }
    int columns = matrix[0].size();       
    if (0 == columns) {
        return res;
    }       
    int start = 0;
    while (columns > start * 2 && rows > start * 2) {
        printMatrixInCircle(matrix, columns, rows, start, res);
        ++start;
    }
    return res;
}
```

# 三十、包含min函数的栈

描述：定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。

思路：用辅助栈储存最小值。

```c++
class MinStack {
public:
    /** initialize your data structure here. */
    stack<int> data, mindata;
    MinStack() {
    }
    
    void push(int x) {
        data.push(x);
        if (0 == mindata.size() || x < mindata.top()) {
            mindata.push(x);
        } else {
            mindata.push(mindata.top());
        }
    }
    
    void pop() {
        if (data.size() > 0 && mindata.size() > 0) {
            mindata.pop();
            data.pop();
        }
    }
    
    int top() {
        return data.top();
    }
    
    int min() {
        return mindata.top();
    }
};
```

# 三十一、栈的压入 弹出序列

描述：输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列 {1,2,3,4,5} 是某栈的压栈序列，序列 {4,5,3,2,1} 是该压栈序列对应的一个弹出序列，但 {4,3,5,1,2} 就不可能是该压栈序列的弹出序列。

思路：辅助栈，把输入的第一个序列中的数字依次压入辅助栈，按照第二个序列的顺序依次弹出。如果下一个数字是栈顶，弹出。如果不是，则把序列中还没有入栈的数字压入辅助栈，直到遇到目标数字，如果所有数字都入栈扔没有找到，则不是弹出序列。

```c++
bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
    bool res = false;
    size_t n = pushed.size();
    size_t i = 0, j = 0;
    stack<int> tmp;
    while (i < n && j < n) {           
        tmp.push(pushed[i]);
        ++i;         
        while (i < n && j < n && !tmp.empty() && tmp.top() != popped[j]) {         
            tmp.push(pushed[i]);
            ++i;
        }
        while (j < n && !tmp.empty() && tmp.top() == popped[j]) {          
            tmp.pop();
            ++j;
        }           
    }
    if (tmp.empty() && i == n && j == n) {
        return true;
    }
    return false;
}
```

# 三十二、从上到下打印二叉树

```c++
vector<int> levelOrder(TreeNode* root) {
    vector<int> res;
    if (root == NULL) {
        return res;
    }
    deque<TreeNode*> tmp;
    tmp.push_back(root);
    while (!tmp.empty()) {
        TreeNode* p = tmp.front();
        tmp.pop_front();
        res.push_back(p->val);
        if (p->left) {
            tmp.push_back(p->left);
        }
        if (p->right) {
            tmp.push_back(p->right);
        }
    }
    return res;
}
```

#### 扩展1：二叉树的每一行单独打印到一行里

思路：需要两个变量，一个表示当前层中还没有打印的节点数，另一个表示下一层节点的数目。

```c++
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (root == NULL) {
        return res;
    }
    queue<TreeNode*> tmp;
    tmp.push(root);
    vector<int> vec;
    int nextLevel = 0;
    int toBeRes = 1;
    while (!tmp.empty()) {           
        TreeNode* p = tmp.front();
        vec.push_back(p->val);
        if (p->left) {
            tmp.push(p->left);
            ++nextLevel;
        }
        if (p->right) {
            tmp.push(p->right);
            ++nextLevel;
        }
        tmp.pop();
        --toBeRes;
        // 当前层打印完毕
        if (0 == toBeRes) {
            res.push_back(vec);
            vec.clear();
            toBeRes = nextLevel;
            nextLevel = 0;
        }
    }
    return res;
}
```

#### 扩展2：之字形顺序打印二叉树

思路：两个栈。打印某一层节点时，把下一层节点保存到栈里。如果当前打印奇数层，则先保存左子节点再保存右子节点到第一个栈。如果是偶数层，先存右子节点再存左子节点到第二个栈里。

```c++
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (root == NULL) {
        return res;
    }
    stack<TreeNode*> levels[2];
    int current = 0;
    int next = 1;
    levels[current].push(root);
    vector<int> tmp;
    while (!levels[0].empty() || !levels[1].empty()) {
        TreeNode* p = levels[current].top();
        levels[current].pop();
        tmp.push_back(p->val);
        if (0 == current) {
            if (p->left) {
                levels[next].push(p->left);
            }
            if (p->right) {
                levels[next].push(p->right);
            }
        } else {
            if (p->right) {
                levels[next].push(p->right);
            }
            if (p->left) {
                levels[next].push(p->left);
            }
        }
        if (levels[current].empty()) {
            res.push_back(tmp);
            tmp.clear();
            current = 1 - current;
            next = 1 - next;
        }
    }
    return res;
}
```

# 三十三、二叉搜索树的后序遍历序列

描述：输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果。如果是则返回 `true`，否则返回 `false`。假设输入的数组的任意两个数字都互不相同。

思路：后序遍历中最后一个数是根。递归确定数组每一部分对应的子树的结构。根左边比它小，右边比它大。

```c++
bool verify(vector<int>& postorder, int start, int len) {
    // 出界
    if (len <= start || start == postorder.size()) {
        return false;
    }
    int root = postorder[len - 1];
    // 搜索左子树节点
    int i = start;
    for (; i < len - 1; ++i) {
        // i最后保留的是根右子节点的位置
        if (postorder[i] > root) {
            break;
        }
    }
    // 验证右子树节点是否都大于根
    int j = i;
    for (; j < len - 1; ++j) {
        if (postorder[j] < root) {
            return false;
        }
    }
    // 判断左子树是不是二叉搜索树
    bool left = true;
    if (i > start) {
        // i为左子树的起点+长度
        left = verify(postorder, start, i);
    }
    // 判断右子树是不是二叉搜索树
    bool right = true;
    if (i < len - 1) {
        right = verify(postorder, i, len - 1);
    }
    return (left && right);
}
bool verifyPostorder(vector<int>& postorder) {
    int n = postorder.size();
    if (0 == n) {
        return true;
    }
    return verify(postorder, 0, n);
}
```

# 三十四、二叉树中和为某一值的路径

描述：输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。

思路：遍历是从根节点出发到叶节点，选择前序遍历。每访问一个节点就添加到路径栈中。每当从子节点回到父节点，需要在路径上删除子节点。

```c++
void FindPath(TreeNode* root, int sum, vector<vector<int>>& res, vector<int>& tmp, int curSum) {
    curSum += root->val;
    tmp.push_back(root->val);
    // 如果是叶节点，和为目标
    bool isLeaf = root->left == NULL && root->right == NULL;
    if (curSum == sum && isLeaf) {
        res.push_back(tmp);
    }
    // 否则遍历它的子节点
    if (root->left) {
        FindPath(root->left, sum, res, tmp, curSum);
    }
    if (root->right) {
        FindPath(root->right, sum, res, tmp, curSum);
    }
    // 返回父节点之前，在路径上删除当前节点
    tmp.pop_back();
}
vector<vector<int>> pathSum(TreeNode* root, int sum) {
    vector<vector<int>> res;
    if (root == NULL) {
        return res;
    }
    vector<int> tmp;
    int curSum = 0;
    FindPath(root, sum, res, tmp, curSum);
    return res;
}
```

# 三十五、复杂链表的复制

描述：在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

思路：①复制节点N，创建N'，把每N'链接在N后面。②设置random，S'是S的next指向的节点。③把这个长链表拆分成两个链表，奇数位置的节点用next链接起来就是原始链表，偶数位置链接起来就是复制出来的链表。

```c++
void CloneNodes(Node* head) {
    Node* p = head;
    while (p) {
        Node* pNew = new Node(p->val);
        pNew->next = p->next;
        pNew->random = NULL;
        p->next = pNew;
        p = pNew->next;
    }
}
void ConnectRandom(Node* head) {
    Node* p = head;
    while (p) {
        Node* pNew = p->next;
        if (p->random) {
            pNew->random = p->random->next;
        }
        p = pNew->next;
    }
}
Node* ReconnectNodes(Node* head) {
    Node* p = head;
    Node* newHead = NULL;
    Node* pNew = NULL;
    if (p) {
        newHead = pNew = p->next;
        p->next = pNew->next;
        p = p->next;
    }
    while (p) {
        pNew->next = p->next;
        pNew = pNew->next;
        p->next = pNew->next;
        p = p->next;
    }
    return newHead;
}
Node* copyRandomList(Node* head) {
    CloneNodes(head);
    ConnectRandom(head);
    return ReconnectNodes(head);
}
```

# 三十六、二叉搜索树与双向链表

描述：输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的循环双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

思路：原先指向左子节点的指针调整为链表中指向前一个节点的指针。原先指向右子节点的指针调整为链表中指向后一个节点的指针。用中序遍历，因为中序是从小到大顺序遍历二叉树的每个节点。当遍历到根节点，左子树已经转换成一个排序的链表了，把最后一个(最大的节点)和根节点链接起来，接着遍历转换右子树，并把根节点和右子树中最小的节点链接起来。递归。

```c++
// last为左侧链表的最大节点
void ConvertNode(Node* proot, Node** plast) {
    if (proot == NULL) {
        return;
    }
    // 保存当前根节点
    Node* pcur = proot;
    // 如果根有左子树，递归转换为双向链表
    if (pcur->left) {
        ConvertNode(proot->left, plast);
    }
    // 当pcur的左子树为空或转换完毕，把左侧链表的最后一个节点和根节点链接起来
    pcur->left = *plast;
    // 如果左侧链表最后一个节点不为空，完成和根节点的链接
    if (*plast) {
        (*plast)->right = pcur;
    }
    // 把根节点设为左侧链表的最后一个节点(因为根节点是根节点的右子节点左侧链表的最大节点)
    *plast = pcur;
    // 如果当前根节点有右子节点，把右子节点的左子树递归转换为双向链表
    if (pcur->right) {
        ConvertNode(pcur->right, plast);
    }
}
Node* treeToDoublyList(Node* root) {
    if (root == NULL) {
        return NULL;
    }
    Node* p = NULL;
    ConvertNode(root, &p);
    // p指向链表尾结点
    Node* newHead = p;
    while (newHead && newHead->left) {
        newHead = newHead->left;
    }
    p->right = newHead;
    newHead->left = p;
    return newHead;
}
```

# 三十七、序列号二叉树

描述：请实现两个函数，分别用来序列化和反序列化二叉树。

思路：①如果二叉树的序列号是从根节点开始的，那么相应的反序列化在根节点数值读出来的时候就可以开始了。根据前序遍历的顺序来序列号二叉树。遇到空指针输出'$',节点的数值之间用','隔开。②第一个数字是根节点，第二个是左子节点，如果连续两个'$'，说明是一个叶节点，回退重建右子节点。如果右子节点是空，说明这个树左右子树已经构建完毕，回到根节点，反序列化根节点的右子树。

```c++
class Codec {
public:
    string serialize(TreeNode* root) {
        if (!root) return "X";
        auto l = "(" + serialize(root->left) + ")";
        auto r = "(" + serialize(root->right) + ")";
        return  l + to_string(root->val) + r;
    }

    inline TreeNode* parseSubtree(const string &data, int &ptr) {
        ++ptr; // 跳过左括号
        auto subtree = parse(data, ptr);
        ++ptr; // 跳过右括号
        return subtree;
    }

    inline int parseInt(const string &data, int &ptr) {
        int x = 0, sgn = 1;
        if (!isdigit(data[ptr])) {
            sgn = -1;
            ++ptr;
        }
        while (isdigit(data[ptr])) {
            x = x * 10 + data[ptr++] - '0';
        }
        return x * sgn;
    }

    TreeNode* parse(const string &data, int &ptr) {
        if (data[ptr] == 'X') {
            ++ptr;
            return nullptr;
        }
        auto cur = new TreeNode(0);
        cur->left = parseSubtree(data, ptr);
        cur->val = parseInt(data, ptr);
        cur->right = parseSubtree(data, ptr);
        return cur;
    }

    TreeNode* deserialize(string data) {
        int ptr = 0;
        return parse(data, ptr);
    }
};
```

# 三十八、字符串的排列

描述：输入一个字符串，打印出该字符串中字符的所有排列。

思路：看成两步。第一步求所有可能出现在第一个位置的字符，即把第一个字符和后面所有字符交换。第二步，固定第一个字符，求后面字符的排列。重复。

```c++
void permutation(vector<string>& res, string& str, int i, int n) {
    if (i == n - 1) {
        res.push_back(str);
    } else {
        set<char> tmp;
        for (int j = i; j < n; ++j) {
            // 如果当前位置已经有同样的字符出现过则跳过
            if (tmp.count(str[j])) {
                continue;
            }
            tmp.insert(str[j]);
            swap(str[i], str[j]);
            permutation(res, str, i + 1, n);
            swap(str[i], str[j]);
        }
    }
}
vector<string> permutation(string s) {
    vector<string> res;
    int n = s.size();
    if (0 == n) {
        return res;
    }
    permutation(res, s, 0, n);
    return res;
}
```

# 三十九、数组中出现次数超过一半的数字

```c++
int majorityElement(vector<int>& nums) {
    size_t n = nums.size();
    int res = nums[0];
    int count = 1;
    for (size_t i = 1; i < n; ++i) {
        if (res == nums[i]) {
            ++count;
        } else {
            if (0 < count) {
                --count;
            } else {
                res = nums[i];
                ++count;
            }
        }
    }
    return res;
}
```

# 四十、最小的k个数

描述：输入整数数组 `arr` ，找出其中最小的 `k` 个数。例如，输入4、5、1、6、2、7、3、8这8个数字，则最小的4个数字是1、2、3、4。

```c++
vector<int> getLeastNumbers(vector<int>& arr, int k) {
    sort(arr.begin(), arr.end());
    vector<int> res(arr.begin(), arr.begin() + k);
    return res;
}
```

如果数组不可更改，可以用一个k大小的最大堆或者红黑树

# 四十一、数据流中的中位数

描述：如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。

思路1：平衡二叉搜索树(AVL树)。平衡因子改为左右子树节点数目差。即可用O(logn)时间添加新节点。用O(1)时间得到中位数。

思路2：P1指向左边最大，P2指向右边最小。保证左边数据都小于右边数据，这样即使左右两边内部没有排序也可以根据左边最大和右边最小得到中位数。采用最大堆实现左边数据容器，最小堆实现右边数据容器。插入O(logn), 取中位数O(1)。细节：保证数据平均分配，在数据总数是偶数时把新数据插入最小堆，否则插入最大堆。如果该数字应该插入最小堆，可它比最大堆中一些数据小，可以先把它放最大堆，接着把最大堆中最大的数字插入最小堆。反之同理。

```c++
class MedianFinder {
public:
    //升序队列
    priority_queue <int,vector<int>,greater<int> > qLeft;
    //降序队列
    priority_queue <int,vector<int>,less<int> >qRight;
    
    /** initialize your data structure here. */
    MedianFinder() {

    }
    
    void addNum(int num) {
        if (0 == ((qLeft.size() + qRight.size()) & 1)) {
            // 偶数时把新数据插入最小堆
            if (qRight.size() > 0 && num < qRight.top()) {
                // 如果该数字应该插入最小堆，可它比最大堆中一些数据小，可以先把它放最大堆，接着把最大堆中最大的数字插入最小堆
                qRight.push(num);
                qLeft.push(qRight.top());
                qRight.pop();
            } else {
                qLeft.push(num);
            }
        } else {
            // 奇数时把新数据插入最大堆
            if (qLeft.size() > 0 && num > qLeft.top()) {
                qLeft.push(num);
                qRight.push(qLeft.top());
                qLeft.pop();
            } else {
                qRight.push(num);
            }
        }
    }
    
    double findMedian() {
        int n = qLeft.size() + qRight.size();
        if (0 == n) {
            return NULL;
        }
        if (1 == (n & 1)) {
            return qLeft.top();
        } else {
            return ((double)qRight.top() + (double)qLeft.top()) / 2.0;
        }
    }
};
```

# 四十二、连续子数组的最大和

描述：输入一个整型数组，数组中的一个或连续多个整数组成一个子数组。求所有子数组的和的最大值。要求时间复杂度为O(n)。

思路：距离分析数组规律。初始化和为0，逐个累加。如果累加后的值比最后一个加数还小(也就是加数之前的和是负数)，那么此加数之前的都被抛弃。

```c++
int maxSubArray(vector<int>& nums) {
    int n = nums.size();
    int res = INT_MIN, cur = INT_MIN;
    for (int i = 0; i < n; ++i) {
        if (cur <= 0) {
            cur = nums[i];
        } else {
            cur += nums[i];
        }
        if (cur > res) {
            res = cur;
        }
    }
    return res;
}
```

# 四十三、1~n整数中1出现的次数

思路：每次去掉最高位进行递归，递归的次数和位数相同。n有O(logn)位。因此时间O(logn)

```c++
int PowerBase10(unsigned int n) {
    int res = 1;
    for (unsigned int i = 0; i < n; ++i) {
        res *= 10;
    }
    return res;
}
// "21345"
int NumberOf1(const char* strn) {
    if (!strn || *strn < '0' || *strn > '9' || *strn == '\0') {
        return 0;
    }
    int first = *strn - '0'; // 2
    unsigned int length = static_cast<unsigned int>(strlen(strn));
    // 0
    if (1 == length && 0 == first) {
        return 0;
    }
    // 1~9
    if (1 == length && 0 < first) {
        return 1;
    }
    // strn = "21345", numFirstDigit=10000~19999中第一位中的数目
    int numFirstDigit = 0;
    if (1 < first) {
        // 10000~19999=10000
        numFirstDigit = PowerBase10(length - 1);
    } else if (1 == first) {
        // if strn="11345", strn+1=>strn[1]+1=>1346
        numFirstDigit = atoi(strn + 1) + 1;
    }
    // numOtherDigits=1346~21345除第一位之外的数位中的数目
    int numOtherDigits = first * (length - 1) * PowerBase10(length - 2);
    // numRecursive=1~1345
    int numRecursive = NumberOf1(strn + 1);
    return numRecursive + numOtherDigits + numFirstDigit;
}
int countDigitOne(int n) {
    // 数字转为字符串
    char strn[50];
    sprintf(strn, "%d", n);
    return NumberOf1(strn);
}
```

# 四十四、数字序列中某一位的数字

描述：数字以0123456789101112131415…的格式序列化到一个字符序列中。在这个序列中，第5位（从下标0开始计数）是5，第13位是1，第19位是4，等等。请写一个函数，求任意第n位对应的数字。

```
    // 第一个m位数
    int beginNumber(int digits) {
        if (1 == digits) {
            return 0;
        }
        return (int)pow(10, digits - 1);
    }
    // m位的数字总共有多少个
    int countOfIntegers(int digits) {
        if (1 == digits) {
            return 10;
        }
        int count = (int)pow(10, digits - 1);
        return 9 * count;
    }
    // 要找的数字位于某m位数之中
    int digitAtIndex(int index, int digits) {
        int number = beginNumber(digits) + index / digits;
        int indexFromRight = digits - index % digits;
        for (int i = 1; i < indexFromRight; ++i) {
            number /= 10;
        }
        return number % 10;
    }
    int findNthDigit(int n) {
        unsigned long digits = 1;
        while (true) {
            // m位的数字总共有多少个
            // digits=1,numbers=10(0~9).digits=2, numbers=90(10~99)
            unsigned long numbers = countOfIntegers(digits);
            unsigned long tmp = numbers * digits;
            if (n < tmp) {
                // 要找的数字位于某m位数之中
                return digitAtIndex(n, digits);
            }
            n -= digits * numbers;
            ++digits;
        }
        return -1;
    }
```

# 四十五、把数组排成最小的数

描述：输入一个非负整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

思路：找到排序规则。数组根据这个规则排序后能排成最小的数字。对于两个数m，n。如果mn<nm，应该打印出mn。反之nm。拼接数字考虑int范围，因此转换成字符串，按照字符串大小的比较规则即可。

```c++
class Solution {
public:
    string minNumber(vector<int>& nums) {
        vector<string> strs;
        for(int i = 0; i < nums.size(); i++)
            strs.push_back(to_string(nums[i]));
        quickSort(strs, 0, strs.size() - 1);
        string res;
        for(string s : strs)
            res.append(s);
        return res;
    }
private:
    void quickSort(vector<string>& strs, int l, int r) {
        if(l >= r) return;
        int i = l, j = r;
        while(i < j) {
            while(strs[j] + strs[l] >= strs[l] + strs[j] && i < j) j--;
            while(strs[i] + strs[l] <= strs[l] + strs[i] && i < j) i++;
            swap(strs[i], strs[j]);
        }
        swap(strs[i], strs[l]);
        quickSort(strs, l, i - 1);
        quickSort(strs, i + 1, r);
    }
};
```

# 四十六、把数字翻译成字符串

描述：给定一个数字，我们按照如下规则把它翻译为字符串：0 翻译成 “a” ，1 翻译成 “b”，……，11 翻译成 “l”，……，25 翻译成 “z”。一个数字可能有多个翻译。请编程实现一个函数，用来计算一个数字有多少种不同的翻译方法。

思路：翻译12258可以分解成两个子问题，1,2258；12,258。2258分解为2,258；22,58.递归从最大的问题开始自上而下解决，我们也可以从最小的子问题开始自下而上解决，消除重复的子问题。从数字的末尾开始，从右到左翻译并计算不同翻译的数目。

```c++
int translateNum(int num) {
    if (num == 0) return 1;
    return f(num);
}
int f(int num) {
    if (num < 10) {
        return 1;
    }
    // 余数为10~25=>f(按两个个位数算)+f(按一个两位数算)
    if (num % 100 < 26 && num % 100 > 9) {
        return f(num / 10) + f(num / 100);
    } else {
        // 余数为26~99，拆分成两个个位数
        return f(num / 10);
    }
}
```

# 四十七、礼物的最大价值

描述：在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

思路：动态规划。f(i, j) = max[f(i, j - 1), f(i - 1, j)] + grid(i, j)
$$
dp(i, j) = 
  \begin{cases}
    grid(i, j)    & \quad \text{if } i = 0, j = 0 \\
    grid(i, j) + dp(i, j - 1)    & \quad \text{if } i = 0, j ≠ 0 \\
    grid(i, j) + dp(i - 1, j)    & \quad \text{if } i ≠ 0, j = 0 \\
    grid(i, j) + max[dp(i - 1, j), dp(i, j - 1)]    & \quad \text{if } i ≠ 0, j ≠ 0
  \end{cases}
$$
分别为起始元素；矩阵第一行元素(只可从左边到达)；第一列元素(只可从上边到达)；可从左边或上边到达。

返回值：
$$
dp[m - 1][n - 1]
$$
空间优化：将原grid矩阵用作dp矩阵。即直接在grid修改。

```c++
int maxValue(vector<vector<int>>& grid) {
    // 处理第一行
    for (int i = 0, j = 1; j < grid[0].size(); ++j) {
        grid[i][j] += grid[i][j - 1];
    }
    //处理第一列
    for (int i = 1,j=0; i < grid.size(); i++) {
        grid[i][j] += grid[i-1][j];
    }
    for (int i = 1; i < grid.size(); i++) {
        for (int j = 1; j < grid[0].size(); j++) {
            if (grid[i - 1][j] >= grid[i][j - 1])	grid[i][j] += grid[i - 1][j];
            else grid[i][j] += grid[i][j - 1];
        }
    }
    return grid[grid.size()-1][grid[0].size()-1];
}
```

# 四十八、最长不含重复字符的子字符串

```c++
int lengthOfLongestSubstring(string s) {
    int n = s.size();
    if (0 == n) {
        return 0;
    }
    int rk = -1, ans = 0;
    unordered_set<char> cnt;
    // i表示当前左指针位置。rk表示当前右指针位置。
    for (int i = 0; i < n; ++i) {
        // 从第二次以后的循环，每次都把左指针右移
        if (0 != i) {
            cnt.erase(s[i - 1]);
        }
        // 不断把字符添加到集合中，直到出现重复字符
        while (rk + 1 < n && !cnt.count(s[rk + 1])) {
            cnt.insert(s[++rk]);  
        }
        // 更新最大长度
        ans = max(ans, rk - i + 1);
    }
    return ans;
}
```

# 四十九、丑数

描述：我们把只包含质因子 2、3 和 5 的数称作丑数（Ugly Number）。求按从小到大的顺序的第 n 个丑数。

思路：创建数组保存已经找到的丑数，里面的丑数都是排好序的。每个丑数都是前面的丑数\*2/3/5得到的。保证排序：数组中已有最大的丑数M，下一个丑数一定是前面某个丑数\*2/3/5的结果。首先把已有的每个丑数\*2，得到若干个<=M的结果，忽略它们。得到第一个>M的结果记为M2，再把已有的每个丑数\*3/5，得到的第一个>M的结果M3/M5，下一个丑数是这三个里最小的。

优化：对于\*2而言，肯定存在某一丑数T2，排在它之前的每个丑数\*2都会小于已有最大丑数，在它之后的每个丑数\*2得到的结果都会太大，只要记下这个丑数的位置，同时每次生成新的丑数时更新这个T2。T3/T5同理。

```c++
class Solution {
public:
    // 找出T2,T3,T5最小的
    int Min(int number1, int number2, int number3) {
        int min = (number1 < number2) ? number1 : number2;
        min = (min < number3) ? min : number3;
        return min;
    }
    int nthUglyNumber(int n) {
        if (n <= 0) {
            return 0;
        }
        // 存储从1到n的所有丑数
        int *pUglyNumbers = new int[n];
        // 
        pUglyNumbers[0] = 1;
        int nextUglyIndex = 1;
        int *pMultiply2 = pUglyNumbers;
        int *pMultiply3 = pUglyNumbers;
        int *pMultiply5 = pUglyNumbers;
        while (nextUglyIndex < n) {
            int min = Min(*pMultiply2 * 2, *pMultiply3 * 3, *pMultiply5 * 5);
            pUglyNumbers[nextUglyIndex] = min;
            while (*pMultiply2 * 2 <= pUglyNumbers[nextUglyIndex]) {
                ++pMultiply2;
            }
            while (*pMultiply3 * 3 <= pUglyNumbers[nextUglyIndex]) {
                ++pMultiply3;
            }
            while (*pMultiply5 * 5 <= pUglyNumbers[nextUglyIndex]) {
                ++pMultiply5;
            }
            ++nextUglyIndex;
        }
        int ugly = pUglyNumbers[nextUglyIndex - 1];
        delete[] pUglyNumbers;
        return ugly;
    }
};
```

# 五十、第一个只出现一次的字符

描述：在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。 s 只包含小写字母。

思路：用一个容器存放每个字符出现的次数，根据字符来查找它出现的次数。哈希表。字符char长度为8，因此总共有256中可能。创建长度为256的数组，以ASCII码为key。

```c++
char firstUniqChar(string s) {
    int n = s.size();
    if (0 == n) {
        return ' ';
    }
    // 存放ASCII码每个字符的出现次数
    vector<int> hashTable(256);
    // 统计每个字符的出现次数
    for (int i = 0; i < n; ++i) {
        ++hashTable[s[i]];
    }
    // 找出目标
    for (int i = 0; i < n; ++i) {
        if (1 == hashTable[s[i]]) {
            return s[i];
        }
    }
    return ' ';
}
```

### 题目二、字符流中第一个只出现一次的字符

思路：定义一个数据容器来保存字符第一次在字符流中读出来的位置。如果再次读出来它，就把它在容器里保存的值更新成一个特殊的值。用字符的ASCII码做key，字符对应的位置做值。

```c++
class CharStatistics
{
private:
    // 统计ASCII出现次数
    int occurrence[256];
    int index;
public:
    // 每个字符出现次数初始化为-1
    ChaStatics(): index(0) {
        for (int i = 0; i < 256; ++i) {
            occurrence[i] = -1;
        }
    }
    // 插入一个字符
    void Insert(char ch) {
        if (occurrence[ch] == -1) {
            occurrence[ch] = index;
        } else if (occurrence[ch] >= 0) {
            // 已经出现超过一次，标记为-2
            occurrence[ch] = -2;
        }
        // 记录在字符流中出现的位置
        ++index;
    }
    // 返回第一次出现的字符
    char FirstAppearingOnce() {
        char ch = '\0';
        int minIndex = numeric_limits<int>::max();
        for (int i = 0; i < 256; ++i) {
            if (occurrence[i] >= 0 && occurrence[i] < minIndex) {
                ch = (char)i;
                minIndex = occurrence[i];
            }
        }
        return ch;
    }
};
```

# 五十一、数组中的逆序对

描述：在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。

思路：考虑先比较两个相邻的数字。先把数组分解成两个，再继续拆分子数组。一边合并相邻子数组一边统计逆序对数目。类似归并排序。

```c++
class Solution {
public:
    int InversePairsCore(vector<int>&nums, vector<int>& copy, int start, int end) {
        // 子数组长度已经为1，没有逆序对
        if (start == end) {
            copy[start] = nums[start];
            return 0;
        }
        // 子数组长度
        int length = (end - start) / 2;
        // 分别递归求解左右子数组中的逆序对数量。copy是已经排好序的数组，当做原数组nums传入。把原数组放在copy当做待排序数组
        int left = InversePairsCore(copy, nums, start, start + length);
        int right = InversePairsCore(copy, nums, start + length + 1, end);
        // i初始化为前半段最后一个数字的下标
        int i = start + length;
        // j初始化为后半段最后一个数字的下标
        int j = end;
        // 排序好的左右两个数组指向最右边的坐标，
        int indexCopy = end;
        // 当前计数初始化为0
        int count = 0;
        // 不断遍历，如果右边指针所指的值比左边指针所指的值小，则把右边数组起始到j的长度加到count，并把排好序的值赋给copy。否则直接赋值。
        while (i >= start && j >= start + length + 1) {
            if (nums[i] > nums[j]) {
                copy[indexCopy--] = nums[i--];
                count += j - start - length;
            } else {
                copy[indexCopy--] = nums[j--];
            }
        }
        // 如果左边数组或者右边数组已经遍历结束说明这两个子数组已经完成排序，把没有遍历结束的数组剩余元素赋值给copy。
        for (; i >= start; --i) {
            copy[indexCopy--] = nums[i];
        }
        for (; j >= start + length + 1; --j) {
            copy[indexCopy--] = nums[j];
        }
        // 返回左右数组中逆序对的数量加上本次合并找到的逆序对数量
        return left + right + count;
    }
    
    int reversePairs(vector<int>& nums) {
        int n = nums.size();
        if (0 == n) {
            return 0;
        }
        vector<int> copy(nums);
        int count = InversePairsCore(nums, copy, 0, n - 1);
        return count;
    }
};
```

# 五十二、两个链表的第一个公共节点

思路：如果两个链表有公共节点，那么这两个链表从某一节点开始，他们的next都指向同一个节点。考虑分别把两个链表的节点放入两个栈里，这样两个链表的尾节点就位于两个栈的栈顶。接下来比较栈顶是否相同，如果相同，则弹出继续比较下一个栈顶，直到找到最后一个相同的节点。

优化：首先遍历两个链表得到他们的长度，第二次遍历的时候在较长链表先走若干步，接着同时在两个链表上遍历，找到的第一个相同的节点就是他们的第一个公共节点。

```c++
class Solution {
public:
    unsigned int GetListLength(ListNode* pHead) {
        unsigned int len = 0;
        ListNode* p = pHead;
        while (p != nullptr) {
            ++len;
            p = p->next;
        }
        return len;
    }
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        // 得到两个链表的长度
        unsigned int len1 = GetListLength(headA);
        unsigned int len2 = GetListLength(headB);
        int diff = len1 - len2;
        ListNode* pListHeadLong = headA;
        ListNode* pListHeadShort = headB;
        if (len2 > len1) {
            pListHeadLong = headB;
            pListHeadShort = headA;
            diff = len2 - len1;
        }
        // 先在长链表走，再同时遍历
        for (int i = 0; i < diff; ++i) {
            pListHeadLong = pListHeadLong->next;
        }
        while ((pListHeadLong != nullptr) && (pListHeadShort != nullptr) && (pListHeadLong != pListHeadShort)) {
            pListHeadLong = pListHeadLong->next;
            pListHeadShort = pListHeadShort->next;
        }
        // 得到第一个公共节点
        ListNode* pFirstCommonNode = pListHeadLong;
        return pFirstCommonNode;
    }
};
```

# 五十三、在排序数组中查找数字

描述：统计一个数字在排序数组中出现的次数。

思路：使用二分法分别找到 左边界 left和 右边界 right，易得数字 target 的数量为 right - left - 1

```c++
class Solution {
public:
    // 查找数字tar在数组nums中的插入点，若数组中存在值相同的元素，则插入到这些元素的右边。
    int helper(vector<int>& nums, int tar) {
        int i = 0, j = nums.size() - 1;
        while (i <= j) {
            int m = (i + j) / 2;
            if (nums[m] <= tar) {
                i = m + 1;
            } else {
                j = m - 1;
            }
        }
        return i;
    }
    int search(vector<int>& nums, int target) {
        return helper(nums, target) - helper(nums, target - 1);
    }
};
```

#### 题目二、0~n-1中缺失的数字

思路：转换成在排序数组中找出第一个值和下标不相等的元素

```c++
int missingNumber(vector<int>& nums) {
    int left = 0;
    int right = nums.size() - 1;
    while (left <= right) {
        int m = (right + left) >> 1;
        if (nums[m] != m) {
            if (0 == m || nums[m - 1] == m - 1) {
                return m;
            }
            right = m - 1;
        } else {
            left = m + 1;
        }
    }
    if (left == nums.size()) {
        return nums.size();
    }
    return -1;
}
```

#### 题目三、数组中数值和下标相等的元素

描述：数组单调递增，元素值唯一。

思路：二分查找。数字m>下标i，那么它右边的数字都大于对应的下标。下一轮只找它左边的即可。

# 五十四、二叉搜索树的第k大节点

思路：中序遍历。

```c++
class Solution {
    int res, k;
public:
    void dfs(TreeNode* root) {
        if (root == nullptr) {
            return;
        }
        dfs(root->right);
        if (0 == k) {
            return;
        }
        if (--k == 0) {
            res = root->val;
        }
        dfs(root->left);
    }
    int kthLargest(TreeNode* root, int k) {
        this->k = k;
        dfs(root);
        return res;
    }
};
```

# 五十五、二叉树的深度

```c++
int maxDepth(TreeNode* root) {
    if (root == nullptr) {
        return 0;
    }
    int nleft = maxDepth(root->left);
    int nright = maxDepth(root->right);
    return max(nleft, nright) + 1;
}
```

#### 题目二、平衡二叉树

描述：输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

思路：后序遍历，在遍历到一个节点之前已经遍历了它的左右子树。只要在遍历每个节点的时候记录它的深度即可。

```c++
class Solution {
public:
    bool isBalanced(TreeNode* root, int* depth) {
        if (root == nullptr) {
            *depth = 0;
            return true;
        }
        int left, right;
        if (isBalanced(root->left, &left) && isBalanced(root->right, &right)) {
            int diff = left - right;
            if (diff <= 1 && diff >= -1) {
                *depth = 1 + max(left, right);
                return true;
            }
        }
        return false;
    }
    bool isBalanced(TreeNode* root) {
        int depth = 0;
        return isBalanced(root, &depth);
    }
};
```

# 五十六、数组中数字出现的次数

描述：给定一个整数数组 `nums`，其中恰好有两个元素只出现一次，其余所有元素均出现两次。 找出只出现一次的那两个元素。你可以按 **任意顺序** 返回答案。

```c++
vector<int> singleNumbers(vector<int>& nums) {
    long int temp = 0;
    //求出异或值
    for (int x : nums) {
        temp ^= x;
    }
    //保留最右边的一个 1
    long int group = temp & (-temp);
    vector<int> arr(2);
    for (int y : nums) { 
        //分组位为0的组，组内异或
        if ((group & y) == 0) {
            arr[0] ^= y;
            //分组位为 1 的组，组内异或   
        } else {
            arr[1] ^= y;
        }
    }
    return arr;
}
```

#### 题目二

描述：给定一个**非空**整数数组，除了某个元素只出现一次以外，其余每个元素均出现了三次。找出那个只出现了一次的元素。

```c++
int singleNumber(vector<int>& nums) {
    /*
        为了区分出现一次的数字和出现三次的数字，使用两个位掩码：seen_once 和 seen_twice。
        仅当 seen_twice 未变时，改变 seen_once。
        仅当 seen_once 未变时，改变seen_twice。
        */
    int seenOnce = 0, seenTwice = 0;
    for (int num : nums) {
        // 第一次出现，添加到seenOnce
        // 第二次出现，从seenOnce移除，添加到seenTwice
        // 第三次出现，从seenTwice移除
        seenOnce = ~seenTwice & (seenOnce ^ num);
        seenTwice = ~seenOnce & (seenTwice ^ num);
    }
    return seenOnce;
}
```

# 五十七、和为s的数字

描述：输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，则输出任意一对即可。(两数之和)

```c++
vector<int> twoSum(vector<int>& nums, int target) {
    int n = nums.size();
    int left = 0, right = n - 1;
    vector<int> res;
    while (nums[right] >= target) {
        --right;
    }
    while (left < right) {
        if (nums[left] + nums[right] < target) {
            ++left;
        } else if (nums[left] + nums[right] > target) {
            --right;
        } else {
            res.push_back(nums[left]);
            res.push_back(nums[right]);
            return res;
        }
    }
    return res;
}
```

#### 题目二、和为s的连续正数序列

```c++
vector<vector<int>> findContinuousSequence(int target) {
    vector<vector<int>> vec;
    vector<int> res;
    // 当l>=r的时候说明数字比target大，后面的不用考虑了
    for (int l = 1, r = 2; l < r;) {
        // l~r的和。
        int sum = (l + r) * (r - l + 1) / 2;
        // 如果相等，把这些元素放进结果集。l前移
        if (sum == target) {
            res.clear();
            for (int i = l; i <= r; ++i) {
                res.emplace_back(i);
            }
            vec.emplace_back(res);
            ++l;
        } else if (sum < target) {
            ++r;
        } else {
            ++l;
        }
    }
    return vec;
}
```

# 五十八、翻转字符串

描述：输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。为简单起见，标点符号和普通字母一样处理。例如输入字符串"I am a student. "，则输出"student. a am I"。

```c++
class Solution {
public:
    void reverseStr(string& s) {
        int l = 0, r = s.size() - 1;
        while (l < r) {
            swap(s[l], s[r]);
            ++l;
            --r;
        }
    }
    string reverseWords(string s) {
        int i = 0;
        // 过滤掉开头的空格
        while (i < s.size() && s[i] == ' ') {
            ++i;
        }
        if (i == s.size()) {
            return "";
        }
        vector<string> slist;
        while (i < s.size()) {
            if (s[i] == ' ') {
                ++i;
            } else {
                // 按空格分割，把每个单词存到slist
                string tmp;
                while (i < s.size() && s[i] != ' ') {
                    tmp.push_back(s[i]);
                    ++i;
                }
                slist.push_back(tmp);
            }
        }
        // 逆转每个单词
        for (i = 0; i < slist.size(); ++i) {
            reverseStr(slist[i]);
        }
        // 清空s，重新拼接字符串。
        s.clear();
        s = slist[0];
        for (i = 1; i < slist.size(); ++i) {
            s.append(" ");
            s.append(slist[i]);
        }
        // 再次逆转
        reverseStr(s);
        return s;
    }
};
```

#### 题目二、左旋字符串

描述：字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。

```c++
class Solution {
public:
    // 逆转字符串
    void reverseStr(string& s) {
        int l = 0, r = s.size() - 1;
        while (l < r) {
            swap(s[l], s[r]);
            ++l;
            --r;
        }
    }
    string reverseLeftWords(string s, int n) {
        int len = s.size();
        // 寻找逆转字符串的分界点
        n = n % len;
        // 分成两个子串，分别逆转。拼接在一起再次逆转
        string s1 = s.substr(0, n);
        string s2 = s.substr(n, s.size());
        reverseStr(s1);
        reverseStr(s2);
        s = s1;
        s.append(s2);
        reverseStr(s);
        return s;
    }
};
```

# 五十九、队列的最大值

#### 题目一、滑动窗口最大值

```c++
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    int n = nums.size();
    vector<int> res;
    if (0 == n || 0 == k || k > n) {
        return res;
    }
    deque<int> index;
    // 如果双向队列不空，且当前元素比队尾元素大，出队。直到队尾元素比其大。这样窗口尺寸内队中的队头最大队尾最小。
    for (unsigned int i = 0; i < k; ++i) {
        while (!index.empty() && nums[i] >= nums[index.back()]) {
            index.pop_back();
        }
        // 队尾入队。保证队列中元素顺序
        index.push_back(i);
    }
    // 从窗口k大小的位置开始遍历，当前队头是最大的，加入结果集
    for (unsigned int i = k; i < n; ++i) {
        res.push_back(nums[index.front()]);
        // 同上，循环出队
        while (!index.empty() && nums[i] >= nums[index.back()]) {
            index.pop_back();
        }
        // 如果队头的坐标在窗口之外了，出队
        if (!index.empty() && index.front() <= (int)(i - k)) {
            index.pop_front();
        }
        // 当前元素入队
        index.push_back(i);
    }
    // 把最后一次的结果入队
    res.push_back(nums[index.front()]);
    return res;
}
```

#### 题目二、队列的最大值

描述：请定义一个队列并实现函数 max_value 得到队列里的最大值，要求函数max_value、push_back 和 pop_front 的均摊时间复杂度都是O(1)。若队列为空，pop_front 和 max_value 需要返回 -1

```c++
class MaxQueue {
    struct InternalData {
        int number;
        int index;
    };
    deque<InternalData> data;
    deque<InternalData> maximums;
    int currentIndex;
public:
    MaxQueue() {

    }
    
    int max_value() {
        if (maximums.empty()) {
            return -1;
        }
        return maximums.front().number;
    }
    
    void push_back(int value) {
        // 和上题思想一样，比当前元素小的队尾元素全部弹出，然后从队尾入队
        while (!maximums.empty() && value >= maximums.back().number) {
            maximums.pop_back();
        }
        InternalData internalData = {value, currentIndex};
        data.push_back(internalData);
        maximums.push_back(internalData);
        ++currentIndex;
    }
    
    int pop_front() {
        if (maximums.empty()) {
            return -1;
        }
        if (maximums.front().index == data.front().index) {
            maximums.pop_front();
        }
        int res = data.front().number;
        data.pop_front();
        return res;
    }
};
```

# 六十、n个骰子的点数

描述：把n个骰子扔在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值出现的概率。你需要用一个浮点数数组返回答案，其中第 i 个元素代表这 n 个骰子所能掷出的点数集合中第 i 小的那个的概率。n<15

思路：用两个数组存储骰子点数的每个总数出现的次数。第一轮：第一个数组中的第n个数字表示骰子和为n出现的次数。下一轮：加上一个新的骰子，此时和为n的骰子出现的次数为上一轮中骰子点数和为n-1~n-6的次数的总和。所以把另一个数组的第n个数字和设为前一个数组对应的第n-1~n-6个数字之和。

P(k)=k出现的次数/总次数。我们的目的就是 **计算出投掷完 n枚骰子后每个点数出现的次数**。

使用动态规划解决问题一般分为三步：

1. 表示状态：那最后一个阶段很显然就是：当投掷完 n枚骰子后，各个点数出现的次数。
2. 找出状态转移方程
3. 边界处理

```c++
/*
首先用数组的第一维来表示阶段，也就是投掷完了几枚骰子。
然后用第二维来表示投掷完这些骰子后，可能出现的点数。
数组的值就表示，该阶段各个点数出现的次数。
*/
// 状态转移
for (第n枚骰子的点数 i = 1; i <= 6; i ++) {
    dp[n][j] += dp[n-1][j - i]
}
// n 表示阶段，j表示投掷完 n枚骰子后的点数和，i表示第 n枚骰子会出现的六个点数。
// 边界
for (int i = 1; i <= 6; i ++) {
    dp[1][i] = 1;
}
```

优化：每个阶段的状态都只和它前一阶段的状态有关，因此我们不需要用额外的一维来保存所有阶段。用一维数组来保存一个阶段的状态，然后对下一个阶段可能出现的点数 j从大到小遍历，实现一个阶段到下一阶段的转换。

```c++
vector<double> dicesProbability(int n) {
    int dp[70];
    memset(dp, 0, sizeof(dp));
    /*
    首先用数组的第一维来表示阶段，也就是投掷完了几枚骰子。
    然后用第二维来表示投掷完这些骰子后，可能出现的点数。
    数组的值就表示，该阶段各个点数出现的次数。
    */
    // 边界(初始)
    for (int i = 1; i <= 6; i ++) {
        dp[i] = 1;
    }
    // 投完了i个骰子
    for (int i = 2; i <= n; ++i) {
        // 投完后，可能出现的点数j
        for (int j = 6 * i; j >= i; --j) {
            // 各个点数出现的次数dp[j]
            dp[j] = 0;
            // 把另一个数组的第n个数字和设为前一个数组对应的第n-1~n-6个数字之和。
            for (int cur = 1; cur <= 6; ++cur) {
                // 可能出现的点最小为全1，也就是i。对前一次的投掷来说是i-1
                if (j - cur < i - 1) {
                    break;
                }
                dp[j] += dp[j - cur];
            }
        }
    }
    // 所有n次投出来的点数
    int all = pow(6, n);
    vector<double> res;
    for (int i = n; i <= 6 * n; i ++) {
        res.push_back(dp[i] * 1.0 / all);
    }
    return res;
}
```

# 六十一、扑克牌中的顺子

描述：从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。

思路：首先把数组排序，其次统计0个数，最后统计排序后数组中相邻数字之间的空缺总数。如果<=0个数，连续。否则不连续。如果数组中非0重复出现，不连续。

```c++
bool isStraight(vector<int>& nums) {
    int n = nums.size();
    sort(nums.begin(), nums.end());
    int zeroCount = 0, diff = 0;
    int i = 0;
    // 统计0个数，i移动到第一个不是0的地方
    while (i < n && 0 == nums[i]) {
        ++i;
        ++zeroCount;
    }
    // j为i后面的数字，如果两个相同牌就不可能是顺子。统计两个数的累计的差
    int j = i + 1;
    while (j < n) {
        if (nums[j] == nums[i]) {
            return false;
        } else if (nums[j] > nums[i] + 1) {
            diff += (nums[j] - nums[i] - 1);
        }
        ++i;
        ++j;
    }
    if (zeroCount >= diff) {
        return true;
    }
    return false;
}
```

# 六十二、圆圈中最后剩下的数字

描述：0,1,···,n-1这n个数字排成一个圆圈，从数字0开始，每次从这个圆圈里删除第m个数字（删除后从下一个数字开始计数）。求出这个圆圈里剩下的最后一个数字。

思路：定义一个函数f(n, m)=y, y为最终留下的元素的序号。有n个数时，花掉了下标为(m-1)%n的数字。划完这个数，往后数x+1下，下标为(m-1)%n+x+1. 得到下一次的函数是f(n-1, m)=x, 答案为[(m-1)%n+x+1]%n. 

f(n, m) = (m % n + x) % n = (m + x) % n

```perl
f(n,m)=[(m-1)%n+x+1]%n
      =[(m-1)%n%n+(x+1)%n]%n
      =[(m-1)%n+(x+1)%n]%n
      =(m-1+x+1)%n
      =(m+x)%n
```

```c++
int lastRemaining(int n, int m) {
    int last = 0;
    for (int i = 2; i <= n; ++i) {
        last = (last + m) % i;
    }
    return last;
}
```

# 六十三、股票的最大利润

描述：假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？

```c++
int maxProfit(vector<int>& prices) {
    int n = prices.size();
    if (0 == n) {
        return 0;
    }
    int diff = 0, maxdiff = 0;
    int buy = prices[0];
    for (int i = 1; i < n; ++i) {
        diff = prices[i] - buy;
        if (diff < 0) {
            buy = prices[i];
        } else {
            maxdiff = max(diff, maxdiff);
        }
    }
    return maxdiff;
}
```

# 六十四、求1+2+...+n

描述：求 `1+2+...+n` ，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

思路：①利用构造函数求解：先定义一个类型，创建n个该类型的实例，将累加代码放构造函数里。②利用虚函数求解：定义两个函数一个充当递归函数角色，另一个处理终止递归的情况。n转为bool变量!!n，非零的n为true，0位false。③函数指针：纯c下用函数指针模拟虚函数。④利用模板类型：让编译器帮助完成类似于递归的计算。

```c++
class Temp {
    static int N;
    static int Sum;
public:
    Temp() {
        ++N;
        Sum += N;
    }
    static void Reset() {
        N = 0;
        Sum = 0;
    }
    static int GetSum() {
        return Sum;
    }
};
int Temp::N = 0;
int Temp::Sum = 0;
class Solution {
public:
    int sumNums(int n) {
        Temp::Reset();
        Temp *a = new Temp[n];
        delete []a;
        a = nullptr;
        return Temp::GetSum();
    }
};
```

# 六十五、不用加减乘除做加法

思路：加法分三步：相加不进位->进位->把前面两个结果相加。转为位运算。第一步不考虑进位，和异或结果一样。第二步想象成两个数位与运算再左移一位。第三步把前两个步骤结果相加，也是重复前两步，直到不产生进位。

```c++
int add(int a, int b) {
    while(b != 0) { // 当进位为 0 时跳出
        //C++中负数不支持左移位，因为结果是不定的
        int c = (unsigned int)(a & b) << 1;  // c = 进位
        a ^= b; // a = 非进位和
        b = c; // b = 进位
    }
    return a;
}
```

# 六十六、构建乘积数组

描述：给定一个数组 A[0,1,…,n-1]，请构建一个数组 B[0,1,…,n-1]，其中 B[i] 的值是数组 A 中除了下标 i 以外的元素的积, 即 B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]。不能使用除法。

思路：B可以用一个矩阵来创建。定义C[i] = A[0] * ... * A[i-1], D[i] = A[i+1] * ... * A[n-1], C[i] = C[i-1] * A[i-1], D[i] = D[i+1] * A[i+1]

```c++
vector<int> constructArr(vector<int>& a) {
    int n = a.size();
    vector<int> b(n);
    if (0 == n) {
        return b;
    }
    // b[i]=a[0]*...*a[i-1]
    b[0] = 1;
    for (int i = 1; i < n; ++i) {
        b[i] = b[i - 1] * a[i - 1];
    }
    // b[i]=b[i]*tmp,tmp=a[i+1]*...*a[n-1]
    int tmp = 1;
    for (int i = n - 2; i >= 0; --i) {
        tmp *= a[i + 1];
        b[i] *= tmp;
    }
    return b;
}
```

# 六十七、把字符串转换成整数

思路：丢弃无用的开头字符。第一个为+-时，作为正负号。忽略尾部多余字符。

```c++
int strToInt(string str) {
    // 当前数字
    int res = 0;
    // 标志位，默认为整数
    int sign = 1;
    int n = str.size();
    if (n <= 0)
    {
        return res;
    }
    int i = 0;
    // 先找到第一个数字
    while (str[i] == ' ')
    {
        ++i;
        // 全部字符都是空格
        if (i == n)
        {
            return res;
        }
    }
    // 判断标点符号
    sign = str[i] == '-' ? -1 : 1;
    if (str[i] == '-' || str[i] == '+')
    {
        ++i;
    }
    // 边界的数字 2147483647/10
    int edge = 214748364;
    // 继续找数字
    while (i < n) {
        // 无效字符
        if (str[i] < '0' || str[i] > '9')
        {
            break;
        }
        // 判断是否超过范围，超过则按照符号来输出 INT_MIN 或 INT_MAX
        int currNum = str[i] - '0';
        if ((res > edge) || (res == edge && currNum > 7))
        {
            cout << res << " " << currNum << endl;
            return sign == 1 ? INT_MAX : INT_MIN;
        }

        res = res*10 + currNum;
        ++i;
    }
    return res * sign;
}
```

# 六十八、树中两个节点的最低公共祖先

#### 题目一、二叉搜索树

描述：给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。

```c++
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    TreeNode* r = root;
    while (r) {
        // pq都在r左边，继续往左找
        if (r->val > p->val && r->val > q->val) {
            r = r->left;
        } else if (r->val < p->val && r->val < q->val) {
            // pq都在r右边，继续往右找
            r = r->right;
        } else {
            // pq在r一左一右，是第一个公共祖先
            return r;
        }
    }
    return r;
}
```

#### 题目二、二叉树

描述：给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

思路：如果有指向父节点的指针，考虑用两个链表公共节点的方式。如果没有，用一个辅助内存保存前序遍历的路径。求出两条路径的最后一个公共节点。

```c++
class Solution {
    TreeNode* ans;
    // 深度优先遍历，相当于前序遍历
    bool dfs(TreeNode* root, TreeNode* p, TreeNode* q) {
        // 递归出口，遇到叶子节点
        if (root == nullptr) {
            return false;
        }
        // 递归遍历左右子树，寻找公共节点
        bool lson = dfs(root->left, p, q);
        bool rson = dfs(root->right, p, q);
        // 如果有一个点，同时处于p，q的路径上，就是目标节点。或者这个点位于p或者q，同时位于q或者p的路径上也是目标节点
        if ((lson && rson) || ((root->val == p->val || root->val == q->val) && (lson || rson))) {
            ans = root;
        }
        // 返回值，如果找到了一个root和p或者q在同一处返回true，表示找到了它的路径。然后它的所有父节点都是true
        return lson ||rson || (root->val == p->val || root->val == q->val);
    }
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        dfs(root, p, q);
        return ans;
    }
};
```

