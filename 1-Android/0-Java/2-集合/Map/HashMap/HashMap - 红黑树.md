# 前置知识

**二叉查找树：**二叉查找树（ BST）是一棵二叉树，其中每个结点都含有一个 Comparable 的键（以及相关联的值）且每个结点的键都大于其左子树中的任意结点的键而小于右子树的任意结点的键。

![图片](images/HashMap - 红黑树/640-20240408174323182)

**递归：**掌握递归的核心就是相信递归。

# 红黑树介绍

红黑树是平衡查找树的一种。什么是平衡查找树呢？举个例子：我们从头构造一棵BST，从 1 到 5 按顺序插入节点，得到的BST如下：

![图片](images/HashMap - 红黑树/640-20240408174426928)

对于这样的BST，它的查找性能就退化成与链表一致了。我们希望一个有N个节点的BST，它的查找能保持在 lgN 以内。

平衡查找树就可以达到上面的要求，但是插入的时候要保持完美的平衡代价太高了，我们退而求其次，稍微放松平衡的要求，红黑树就是这样的一种树。

# 红黑树性质

《算法导论》描述了红黑树的如下几个性质：

1. Every node is either red or black.
2. The root is black.
3. Every leaf (NIL) is black.
4. If a node is red, then both its children are black.
5. For each node, all simple paths from the node to descendant leaves contain the same number of black nodes.

前面3条是固定约束，很好理解，后面2条可以推理出来。我们画几个图来解释一下：

![图片](images/HashMap - 红黑树/640-20240408175032924)

在红黑树上，一条边上不能出现两个红色节点，一红一黑，两黑都行。

那么，**如果一个节点是红色，那么它的孩子节点只能是黑色，由于NULL节点也算黑色，所以它的孩子节点只能是黑色节点或者NULL节点，如下图：**

![图片](images/HashMap - 红黑树/640-20240408175228065)

红黑树中，计算树高，红色节点的高度为0，黑色节点的高度为 1。一个典型的红黑树如下：

![图片](images/HashMap - 红黑树/640-20240408175704177)

可以看到，每条到叶子节点的路径上，黑色节点的个数是相等的，都为2，该值叫做黑高。

对于红黑树的每个节点来说，左子树的黑高与右子树的黑高是相等的。这里体现出来的弱平衡就是只要求黑色节点的数量一致。

# 红黑树本质

为什么红黑树可以做到 lgN 的查找速度呢？

我们假想一下，红黑树最坏的情况就是红黑间隔，这样树高从 lgN 变成了 2lgN，这是在常数级别之内，并没有退化为 N。

其实红黑树的本质是 2-3-4 树，什么是 2-3-4 树呢？就是树的每个节点的孩子数量可以为 2、3、4。

![图片](images/HashMap - 红黑树/640-20240408180256696)

由这3种节点混合构成的树就叫 2-3-4 树。

我们将红色节点与其父节点看作是一个节点，我们就得到如下变换：

![图片](images/HashMap - 红黑树/640-20240408180547574)

看下面的一个红黑树：

![图片](images/HashMap - 红黑树/640-20240408180658633)

转换为 2-3-4 树就是：

![图片](images/HashMap - 红黑树/640-20240408180832730)

掌握了红黑树的本质之后，我们来分析其核心功能，插入与删除。

# 红黑树插入

红黑树的插入与二叉查找树的插入差不多，也是分为几种情况讨论。

插入只发生在叶子节点上，且待插入的节点需要被当作是红色节点（不影响子树的黑高）。

这里又要分两种情况，该叶子节点是红色还是黑色。

**叶子节点是黑色**

如果是黑色，直接插入就好了，该子树的黑高并没有被破坏。

![图片](images/HashMap - 红黑树/640-20240408181123402)

**叶子节点是红色**

如果是红色，插入节点后破坏了红黑树的性质：

![图片](images/HashMap - 红黑树/640-20240408181315133)

这个时候就需要进行旋转，旋转的规则很简单：

首先，先找3个节点，我们假设待插入节点叫 x，叶子节点叫p，p 的 parent 叫 g。根据红黑树的性质，g 肯定是黑色。

然后，有了3个节点后找**到 x p g 3个节点的中序遍历的中间节点，将这个节点往上提即可**，如下图：

![图片](images/HashMap - 红黑树/640-20240408181749677)

最后，移动 x p g 各自的孩子节点，保持二叉搜索树的不等式性质：left child < node < right child 。

**有了这张图，再也不用搞不清楚左旋和右旋讲的是什么东西了。**

所以，x p g 3个节点的位置有如四种情况，我们先看第一种：

![图片](images/HashMap - 红黑树/640-20240408181803019)

在旋转前，我们有这样的不等式：

A < X < B < P < C < G < D

旋转后，节点 G 占了 C 的位置，那么为了保持不等式的正确，可以将 C 变成 G 的左孩子。

搞定了不等式之后，还需要搞定该子树的黑高问题。

如上图，旋转后，左孩子的黑高比右孩子的黑高少了1，**所以需要重新染色。**

染色需要保持该子树的黑高不变，有两种可选方案，第一种是 p 染成黑色，x 与 g 染成红色。但是这样还需要处理 x 与 g 的孩子的颜色问题。

第二种方案是 p 染成 红色，x 与 g 染成黑色，这样 x 与 g 就不用考虑孩子节点的颜色问题。但是 p 需要考虑它父节点的颜色问题。这样就形成一个向上的递归。

显然第二种方案要简单。

同样的，后面3种情况如下：

![图片](images/HashMap - 红黑树/640-20240408181823416)

**插入代码展示**

```
Node insertParent = findInsertLeaf(node);
insertLeaf(node, insertParent);

// check color if need rotate
Node x = node;
Node p = x.getParent();
Node g = p.getParent();

// 根节点是黑色，会终结循环
while (p.isRed() && !isRoot(x)) {
    x = rotate(x, p, g);
    p = x.getParent();
    g = p.getParent();

    // make x color red, left and right child black
    recolor(x, x.getLeftChild(), x.getRightChild());
}
```

# 红黑树删除

红黑树的删除也是要分情况讨论，其中比较核心的就是双黑节点，这个会稍微麻烦点。但是代码写起来还是比较简单的。

对于书中的任意一个节点来说，它分3种情况，一种是没有孩子节点，一种是只有一个孩子节点，一种是有两个孩子节点。

**没有孩子节点**

我们先看最简单的，也是最复杂的情况。这里还需要再分两种，该节点是黑色还是红色。

**如果该节点是红色**，直接删除即可。

![图片](images/HashMap - 红黑树/640-20240408181850139)

**如果该节点是黑色**，删除后，需要将该节点记下，将它看成一个双黑节点，只有这一种情况会出现双黑节点，我们后面再讨论。

![图片](images/HashMap - 红黑树/640-20240408181855491)

**有一个孩子节点**

根据红黑树的性质，有一个孩子节点，那么该节点必定是一个黑色节点，且孩子节点为红色。

![图片](images/HashMap - 红黑树/640-20240408181902946)

另一种情况是对称的，图就不上了。这里我们只需要删除该黑色节点，让红色节点补位，然后将他的颜色改成黑色即可。

代码如下：

```
child.setColor(Color.BLACK);
replaceNode(x, child);
```

**有两个孩子节点**

这里我们可以想一下 BST 的删除过程。

要删除该节点，首先我们需要找到该节点的后继节点，也就是在树种最接近它且比它大的节点。

这里算法导论里面又分了两种情况，但是本质其实是一样的，只不过代码写起来稍微不一样。

**我们将要删除的节点记为x，其后继节点记为 s。**

第一种是，x 的右孩子的左孩子为NULL，也就是说 x 的右孩子就是 s：

![图片](images/HashMap - 红黑树/640-20240408181917492)

这种情况下，我们的不等式为 a < x < s < b。将 x 与 s 先交换位置，不等式就变为了 a < s < x < b，但是 x 会被删除，所以不等式最终还是真确的。

为了保持子树与其parent 的颜色正确性，所以还需要将 x 与 s 再次交换颜色。

上图最终的结果，想要删除x，其实就回到了我们上面说的只有一个孩子节点的情况了。代码如下：

```
replaceNode(x, s);
s.setLeftChild(xLeft);
x.setRightChild(s.getRightChild());
x.setLeftChild(null);
s.setRightChild(x);

continue;
```

第二种就是上面的一种泛化情形：

![图片](images/HashMap - 红黑树/640-20240408181929090)

x的右孩子不为NULL，这里我们可以先通过递归的方式找到 x 的后继，就是从 x 的右孩子开始，一直往左边走，知道节点的左孩子为空，该节点就是 s，如上图的最左边。

找到之后，交换位置，然后交换颜色。

然后再删除x，这里其实就是又回到了上面讨论的没有孩子节点的情况或者是只有一个孩子节点的情况。代码如下：

```
Node succussor = s;
while (succussor != null && succussor.getLeftChild() != null) {
    succussor = succussor.getLeftChild();
}
s = succussor;
sRight = s.getRightChild();

Node sParent = s.getParent();

x.setRightChild(sRight);
x.setLeftChild(null);
s.setLeftChild(xLeft);
s.setRightChild(xRight);

replaceNode(x, s);
sParent.setLeftChild(x);

continue;
```

**双黑节点**

上面我们讨论到，当一个黑色节点被删除的时候，该节点需要被当作一个双黑节点，目的就是为了将该黑节点转移到其他的节点上去，好让它可以直接删除。

**双黑节点未删除时，左右时平衡的，删除后，也要是平衡的，双黑节点通过转移的方法，将待删除节点的黑色转移到别的地方，达到平衡。**

**为了更好的理解，也可以把双黑节点当作一个类似指针的标识。**

调整双黑节点需要4个节点的参与：

- 该节点 x 的父节点，p
- p 的另一个孩子节点，也就是 x 的兄弟节点 s
- s 的孩子节点，离 x 近的，n
- s 的孩子节点，离 x 远的，f

其中，n f 可以是 NULL。

这4个节点，根据红黑树的性质，按照颜色的组合，一共有 9 种可能性：

![图片](images/HashMap - 红黑树/640-20240408181948872)

我们先看3种特殊的情况，0xF,0xB,0x7。

- **0xF**

![图片](images/HashMap - 红黑树/640-20240408182004490)

![图片](images/HashMap - 红黑树/640-20240408182007414)

首先，我们需要明白，双黑节点只是我们设置了一个标识，并不是真正的改动了红黑树的节点内容。

所以上面贴了两张图，下面的一张是解释每个节点的黑高。

双黑节点往 parent 进行转移，为了维持左右平衡，右子树黑高需要减一，将 s 染成红色即可。p 成为了新的双黑节点，然后进行递归。它会变成其他的几种情况之一。

- **0xB**

![图片](images/HashMap - 红黑树/640-20240408182020616)

![图片](images/HashMap - 红黑树/640-20240408182029617)

这里将 p s f 3个节点进行了旋转，然后交换 s 与 p 的颜色。就转变成了其他的情况。

- **0x7**

![图片](images/HashMap - 红黑树/640-20240408182035057)

![图片](images/HashMap - 红黑树/640-20240408182040307)

这种情况比较简单，将双黑节点向 p 转移，将p染成黑色，但是右子树黑高增加了，为了平衡，将 s 染成红色。

- **其他6种情况**

![图片](images/HashMap - 红黑树/640-20240408182046397)

n 和 f 有一个为红色，或者都为红色，就有3种情况。

p 为 红色或者黑色，有两种情况，一共有6种情况。

这6种情况的变换都一样，都是先旋转，再交换颜色，再转移黑色。

**状态转移图**

上面9种情况的状态转移图如下：

![图片](images/HashMap - 红黑树/640-20240408182052588)

代码如下：

```
while (db != root()) {
  switch (psfnColor) {
      case 0xf:
          // float up
          db = p;
          s.setColor(Color.RED);
          continue;
      case 0xb:
          rotate(f, s, p);
          s.setColor(Color.BLACK);
          p.setColor(Color.RED);
          continue;
      case 0x7:
          p.setColor(Color.BLACK);
          s.setColor(Color.RED);
          break;
      case 0x4:
      case 0x5:
      case 0xc:
      case 0xd:
          rotate(n, s, p);
          p.setColor(Color.BLACK);
          s.setColor(Color.BLACK);
          n.setColor(pColor);
          break;
      case 0x6:
      case 0xe:
          rotate(f, s, p);
          p.setColor(Color.BLACK);
          s.setColor(pColor);
          f.setColor(Color.BLACK);
          break;

      default:
          Check.shouldNotReachHere();
          break;
  }
  break;
```

# 手写红黑树

有了上面的基础，自己从0到1写一个红黑树也就不是难事了，赶紧动手巩固一下吧，毕竟只有写出代码才能真正算得上理解了。

上面简单的贴了一些代码片段，是为了说明，红黑树的代码没有想象中的那么麻烦。虽然我们讨论的情况确实很多，但是代码写起来要简单很多。

我使用 java 实现了一个自己的红黑树，核心代码在 300 行左右。项目工程已提交到了 github，有任何问题可以提 issue。项目里面我还提供了一些测试用例，可以便于理解。

https://github.com/aprz512/red-black-tree

# 测试红黑树

代码实现完成后，我们还需要对代码进行测试。为了方便测试各种case，我们需要能够随意构造红黑树，所以使用字符串的方式来构造。如下：

**![图片](images/HashMap - 红黑树/640-20240408182100365)**

对于这样的两个字符串，我们希望它能构造出如下的红黑树：

![图片](images/HashMap - 红黑树/640-20240408182104602)

对于节点 key 的解析，我们使用 stack 来辅助（下面的代码经过了简化，主打一个核心思路）：

```
private Node buildTree(String treeString) {

  Stack<Node> stack = new Stack<>();

  while (i < treeString.length()) {
      ch = treeString.charAt(i);
      if (ch == '(') {

          Node node = new Node();
          stack.push(node);

          continue;
      } else if (ch == ')') {

          Node top = stack.pop();
          Node parent = stack.peek();

          parent.setChild(top);

      } else if (ch == '#') {
          parent.setChild(top);

      } else {
          i++;
          continue;
      }
  }

  return null;
}
```

遇到左括号，就push一个节点到 stack 中，遇到右节点，就pop出来，将他设置到 parent 的 leftChild 或者 rightChild 上。遇到叶子节点也是如此处理。

对于颜色的解析就更简单了，直接使用递归：

```
private int colorTreeDfs(Node tree, String colorString, int index) {
    if (tree == null) {
        Check.checkEquals('#', colorString.charAt(index));
        return index;
    }

    if (colorString.charAt(index) == 'R') {
        tree.setColor(Color.RED);
    } else if (colorString.charAt(index) == 'B') {
        tree.setColor(Color.BLACK);
    }

    index = colorTreeDfs(tree.getLeftChild(), colorString, index + 1);
    // index 传递给参数，累加
    index = colorTreeDfs(tree.getRightChild(), colorString, index + 1);

    return index;
}
```

就是按照前序遍历的方式染色。

红黑树构造好了之后，我们需要校验三个方面的东西：

第一就是每个节点的左右子树的黑高是否相等，

第二个就是不等式性质，left < key < right。我自己写的代码，是采用的 left < key ≤ right 的性质。由于节点会旋转，所以最终可能会出现 left ≤ key ≤ right 的情况。

第三个就是颜色约束，红色节点的child必须是黑色的。

检验如下：

```
private static int verifyBlackHeight(Node node, int blackHeight) {
    if (node == null) {
        return 0;
    }

    Node left = node.getLeftChild();
    Node right = node.getRightChild();

    int leftBlackHeight = verifyBlackHeight(left, blackHeight);
    int rightBlackHeight = verifyBlackHeight(right, blackHeight);

    Check.checkEquals(leftBlackHeight, rightBlackHeight);

    if (node.isRed()) {
        if (left != null) {
            Check.checkEquals(left.getColor(), Color.BLACK);
        }
        if (right != null) {
            Check.checkEquals(right.getColor(), Color.BLACK);
        }
        blackHeight = leftBlackHeight;
    } else if (node.isBlack()) {
        blackHeight = leftBlackHeight + 1;
    } else {
        Check.shouldNotReachHere("color is not red or black, color = " + node.getColor());
    }

    // 旋转之后，left child 可能与 node 相等
    if (left != null) {
        Check.checkLessEquals(left.getKey(), node.getKey());
    }

    if (right != null) {
        Check.checkLessEquals(node.getKey(), right.getKey());
    }

    return blackHeight;
}
```

# 总结

恭喜你看到这里，这说明你已经对树这种结构的理解已经到了登堂入室的境界了。

掌握红黑树是非常有必要的，因为它的使用很广泛，像虚拟内存的管理，数据库等地方都有使用到。



# 参考

[HashMap - 红黑树](https://mp.weixin.qq.com/s/bJ7ufMLAwsPRrd-SSbJQFw)