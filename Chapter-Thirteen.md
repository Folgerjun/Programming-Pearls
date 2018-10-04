## 第十三章 搜索

**<h4 id = "1">1. [答案 12.9](Chapter-Twelve.md#9) 描述了生成有序随机整数集合的 Bob Floyd 算法。你能否用本章的几种 IntSet 实现该算法？这些结构在 Floyd 算法生成的非随机分布上性能如何？</h4>**

可以用如下 IntSet 类实现：
```
void genfloyd(int m, int maxval) {
    int *v = new int[m];
    IntSetSTL S(m, maxval);
    for (int j = maxval-m; j < maxval; j++) {
        int t = bigrand() % (j+1);
        int oldsize = S.size();
        S.insert(t);
        if (S.size() == oldsize) // t already in S
            S.insert(j);
    }
    S.report(v);
    for (int i = 0; i < m; i++)
        cout << v[i] << "\n";
}
```
当 m 和 maxval 相等时，元素按升序插入，这正是二分搜索树的最坏情况。

**<h4 id = "2">2. 如何修改简单的 IntSet 接口使其更健壮？</h4>**

应进行错误检查，以确保待插入的整数在正确的范围内，且数据结构还没有被填满。此外，还应该用一个析构函数来返回所分配的存储空间。

**<h4 id = "3">3. 为集合类增加一个 find 函数，该函数用于判断给定的元素是否在集合中。你能否让该函数比 inset 更高效？</h4>**

使用二分搜索来测试某个元素是否在有序数组中。

**<h4 id = "4">4. 为链表、箱和二分搜索树的递归插入函数重写相应的迭代版本，并度量运行时间的差别。</h4>**

下面的链表迭代插入算法比对应的递归算法长一些，因为它把在 head 后面插入结点和后来在链表中插入结点的实例分析各写了一遍：
```
void insert(t)
    if head->val == t
        return
    if head->val > t
        head = new node(t, head)
        n++
        return
    for (p = head;p->next->val < t;p = p->next)
        ;
    if p->next->val == t
        return
    p->next = new node(t, p->next)
    n++
```
下面的简化代码通过使用指向指针的指针来去除重复：
```
void insert(t)
    for (p = &head; (*p)->val < t; p = &((*p)->next))
        ;
    if (*p)->val == t
        return
    *p = new node(t, *p)
    n++
```
这段代码的速度跟前一版本一样快。只要对其稍作修改即可用于箱。[答案 7](#7) 将这一方法用到了二分搜索树上。

**<h4 id = "5">5. 9.1 节和[答案 9.2](Chapter-Nine#2) 描述了 Chris Van Wyk 如何通过将可用结点保存在自己的结构中来避免多次调用存储分配器。说明如何将这一思想应用到链表、箱和二分搜索树实现的 IntSet 上。</h4>**

为了用一次存储分配来取代多次分配，我们需要有一个指向下一个可用结点的指针： `node *freenode;` 在构造类的时候就分配出足够的空间： `freenode = new node[maxelms]` 然后在插入函数中根据需要加以使用：
```
if (p == 0)
    p = freenode++
    p->val = t
    p->left = p->right = 0
    n++
else if ...
```
同样的方法可以应用到箱中。[答案 7](#7) 将其用到了二分搜索树上。

**<h4 id = "6">6. 在各种 IntSet 实现上对下面的代码段计时，能够发现什么？</h4>**
```
IntSetImp S(m, n);
for (int i = 0; i < m; i++)
    S.insert(i);
```

按升序插入结点可以度量数组和链表的搜索开销，而且只会引入很小的插入开销。而对于箱和二分搜索树，该代码会导致最坏情况。

**<h4 id = "7">7. 我们的数组、链表和箱都使用了哨兵。说明如何将哨兵用于二分搜索树。</h4>**

把以前的 null 指针都指向哨兵结点，哨兵在构造函数中进行初始化： `root = sentinel = new node` 插入代码先将目标值 t 放入哨兵结点，然后用一个指向指针的指针（见[答案 4](#4)）来自顶向下遍历树直至找到 t。接着使用[答案 5](#5) 的方法插入一个新结点。
```
void insert(t)
    sentinel->val = t
    p = &root
    while (*p)->val != t
        if t < (*p)->val
            p = &((*p)->left)
        else
            p = &((*p)->right)
    if *p == sentinel
        *p = freenode++
        (*p)->val = t
        (*p)->left = (*p)->right = sentinel
        n++
```
其中结点变量声明并初始化如下： `node **p = &root;`

**<h4 id = "8">8. 说明如何通过同时在很多位上进行操作来加速位向量的初始化和输出操作。这种方法在操作 char、short、int、long 或某种其他类型时是不是最有效的？</h4>**

**<h4 id = "9">9. 说明如何通过使用低开销的逻辑移位替代高开销的除法运算来对箱进行加速。</h4>**

为了用移位取代除法，我们用类似下面的伪代码对变量进行初始化：
```
goal = n/m
binshift = 1
for (i = 2; i < goal; i *= 2)
    binshift++
nbins = 1 + (n >> binshift)
```
插入函数从该结点开始： `p = &(bin[t >> binshift])`

**<h4 id = "10">10. 在完成类似于生成随机数的任务时，可以使用其他哪些数据结构来表示整数集合？</h4>**

可以通过混合并匹配多种数据结构来表示随机集合。例如，由于我们很清楚每个箱中将包含多少项，因此可以用 13.2 节的知识，使用小数组来表示大多数箱中的项（当箱太满时可以将剩下的元素放到一个链表中）。Don Knuth 在 1986 年 5 月《ACM 通讯》的“编程珠玑”专栏中描述了一种“有序散列表”来解决这一问题，以展示他的文档化 Pascal 程序 Web 系统。该论文也是他 1992 年出版的 Literate Programming 一书的第 5 章。

**<h4 id = "11">11. 实现一个最快的完整函数来生成一个有序的随机整数数组，不允许重复。（可以使用前面介绍的任何接口来表示集合。）</h4>**
