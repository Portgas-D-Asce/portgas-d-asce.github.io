## AVL树认知
AVL树是一种平衡二叉搜索树，它满足性质： 
- 对于任意节点，其子树的左右子树的高度差，不会超过 1；

思考1：
- 高度为 $H$ 的二叉搜索树，所包含的节点数量在什么范围内？
  - 二叉搜索树退化为链表时，其所包含的节点数量最少，下界为 $H$；
  - 二叉搜索树为满二叉搜索树时，其所包含的节点数量最多，上界为 $2^H - 1$
- 高度为 $H$ 的 AVL 树，所包含的节点数量在什么范围内？
  - 设 $low(H)$ 为高度为 $H$ 的 AVL 树包含的最少节点个数，下界为 $low(H) = low(H - 1) + low(H - 2) + 1$；
  - 同二叉搜索树，其上界为 $2^H - 1$；

思考2：AVL 树大概长什么样子？高度为 $H$ 的 AVL 树，第 1 到 $H\ -\ 2$ 层一定是一棵满二叉树吗？
- 答案：不一定。
- 想明白上面的问题，对更加深入地理解 AVL 树有一定帮助。

## 左/右旋
**左/右旋的本质：** 在不破坏二叉搜索树性质的前提下改变左右子树的高度。

**抓着谁旋转：** 回溯过程中，第一次违反 AVL 树性质的那个节点；

![5.png](http://localhost:8080/image/5)

## 平衡遭到破坏的情况

设:
- $K1$ 为回溯过程中，第一次发现失衡的节点；
- $ha$（$hb$， $hc$， $hd$）表示从 $K1$ 节点到 $A$（$B$， $C$， $D$）子树叶子结点的最长路径长度；

### LL 型

![6.png](http://localhost:8080/image/6)

**LL 型：** $K1$ 的左子树更高 且 $K2$ 的左子树更高；
- $ha$, $hb$, $hc$, $hd$ 之间的关系为： $ha = hb + 1 = max(hc, hd) + 2$;
  - 如果 $hb = ha$，那么说明在插入/删除之前 $K1$ 子树已经不平衡了；因此 $ha = hb + 1$ 一定成立；
  - 因为是回溯到 $K1$ 节点的时候，才发现失衡，因此 $ha = max(hc, hd) + 2$ 一定成立；

如何解决 LL 型失衡呢？
- 抓着 $K1$ “大右旋”；

### LR 型
![7.png](http://localhost:8080/image/7)

**LR 型：** $K1$ 的左子树更高 且 $K2$ 的右子树更高；
- $ha$, $hb$, $hc$, $hd$ 之间的关系为： $ha + 1 = max(hb, hc) = hd + 2$（原因参照 LL 型）；

如何解决 LR 型失衡？
- 先抓着 $K2$ "小左旋" 使得 $ha = max(hb, hc) + 1$，也就是转化成 LL 型；
- 然后抓着 $K1$ “大右旋”；

### RR 型
![8.png](http://localhost:8080/image/8)

**RR 型：** $K1$ 的右子树更高 且 $K3$ 的右子树更高；
- $ha$, $hb$, $hc$, $hd$ 之间的关系为： $max(ha, hb) + 2 = hc + 1 = hd$（原因参照 LL 型）；

如何解决 RR 型失衡？
- 抓着 $K1$ “大左旋”；

### RL 型
![9.png](http://localhost:8080/image/9)

**RL 型：** $K1$ 的右子树更高 且 $K2$ 的左子树更高；
- $ha$, $hb$, $hc$, $hd$ 之间的关系为： $ha + 2 = max(hb, hc) = hd + 1$（原因参照 LL 型）;

如何解决 RL 型？
- 转着 $K2$ “小右旋” 使得 $max(hb + hc) + 1 = hd$，也就是转化成 RR 型；
- 抓着 $K1$ “大左旋”;

### 总结
思路整理：实际上只需要记一种 LL 型就够了：
- RR 型跟 LL 型是一样的，只需要把 LL 型的 “大右旋” 变成 “大左旋” 即可；
- LR 型可以对其 左孩子进行一次 “小左旋” 即可将其转化为 LL 型；
- 同理，RL 型也是可以转化为 RR 型；

记忆：
- LL 型：L + L = R（大右旋）；
- RR 型：R + R = L （大左旋）；
- LR 型：抓着 左孩子 “小左旋”（抓着第一位朝着第一位方向小旋，把第二位变成第一位）把问题转化为 LL 型；
- RL 型：抓着 右孩子 “小右旋”（抓着第一位朝着第一位方向小旋，把第二位变成第一位）把问题转化为 RR 型；

思考：结合 左/右旋 的本质，针对四种情况，分别考虑下到底是为什么 “那么处理一下”，AVL 树就能恢复平衡？

## 实现
```cpp
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int val, h;
    struct Node *left, *right;
} Node;

//用 NIL 替换掉 NULL？为了简化代码实现，使实现更加优雅的一个小技巧。
//一下方式，避免了声明一个全局的 struct Node 指针，程序执行完后，NIL会自动被释放。
struct Node NIL_NODE;
#define NIL (&NIL_NODE)
__attribute__((constructor))
void init_nil() {
    NIL->val = 0;
    NIL->h = -1;
    NIL->left = NIL->right = NIL;
}


Node *init(int val) {
    Node *p = (Node *)malloc(sizeof(Node));
    p->val = val;
    p->left = p->right = NIL;
    p->h = NIL->h + 1;
    return p;
}

void clear(Node *root) {
    if(root == NIL) return;
    clear(root->left);
    clear(root->right);
    free(root);
}

//如果没有 NIL，需要空，代码逻辑复杂，不优雅
void update_height(Node *root) {
    root->h = (root->left->h > root->right->h ? root->left->h : root->right->h) + 1;
}

Node *left_rotate(Node *root) {
    Node *temp = root->right;
    root->right = temp->left;
    temp->left = root;
    update_height(root);
    update_height(temp);
    return temp;
}

Node *right_rotate(Node *root) {
    Node *temp = root->left;
    root->left = temp->right;
    temp->right = root;
    update_height(root);
    update_height(temp);
    return temp;
}

//如果没有 NIL，需要空，代码逻辑复杂，不优雅
Node *maintain(Node *root) {
    if(abs(root->left->h - root->right->h) < 2) return root;
    if(root->left->h > root->right->h) {
        //LR型
        if(root->left->right->h > root->left->left->h) {
            //先 “小左旋” 转化成 LL型
            root->left = left_rotate(root->left);
        }

        //LL型 “大右旋”
        root = right_rotate(root);
    } else {
        //RL型
        if(root->right->left->h > root->right->right->h) {
            //先 “小右旋” 转化成 RR型
            root->right = right_rotate(root->right);
        }
        
        //RR型 “大左旋”
        root = left_rotate(root);
    }
    return root;
}

Node *insert(Node *root, int val) {
    if(root == NIL) return init(val);
    if(root->val == val) return root;
    if(val < root->val) {
        root->left = insert(root->left, val);
    } else {
        root->right = insert(root->right, val);
    }
    update_height(root);
    root = maintain(root);
    return root;
}

Node *predecessor(Node *root) {
    Node *temp = root->left;
    while(temp->right != NIL) temp = temp->right;
    return temp;
}

Node *erase(Node *root, int val) {
    if(root == NIL) return NIL;
    if(root->val == val) {
        if(root->left == NIL || root->right == NIL) {
            Node *temp = root->left != NIL ? root->left : root->right;
            free(root);
            return temp;
        } else {
            Node *temp = predecessor(root);
            root->val = temp->val;
            root->left = erase(root->left, temp->val);
            return root;
        }
    }
    if(val < root->val) {
        root->left = erase(root->left, val);
    } else {
        root->right = erase(root->right, val);
    }
    update_height(root);
    root = maintain(root);
    return root;
}

int search(Node *root, int val) {
    if(root == NIL) return 0;
    if(root->val == val) return 1;
    if(val < root->val) {
        return search(root->left, val);
    }
    return search(root->right, val);
}

void output(Node *root) {
    if(root == NIL) return;

    int left = root->left != NIL ? root->left->val : 0;
    int right = root->right != NIL ? root->right->val : 0;
    printf("(%d[%d], %d, %d)\n", root->val, root->h, left, right);

    output(root->left);
    output(root->right);
}

int main() {
    int op = 0, val = 0;
    Node *root = NIL;
    while(scanf("%d%d", &op, &val) != EOF) {
        switch(op) {
            case 0:
                printf("searching %d, result = %d\n", val, search(root, val));
                break;
            case 1:
                root = insert(root, val);
                break;
            case 2:
                root = erase(root, val);
                break;
        }
        if(op) {
            output(root);
            printf("----------\n");
        }
    }
    return 0;
}
```