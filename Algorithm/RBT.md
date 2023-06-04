## BST(Binary search tree)
![image](https://upload.wikimedia.org/wikipedia/commons/d/da/Binary_search_tree.svg)
- Binary search trees keep their keys in sorted order.  
- The key in each node must be greater than or equal to any key stored in the left sub-tree, and less than or equal to any key stored in the right sub-tree.

#### BST Lookup
```
T key; // a search key 
Node root; // point to the root of the BST

while(true){
    if(null == root){
       break;
    }
    if(key.equals(root.value)){
        return root;
    }
    if(key.compareTo(root.value) < 0){
        root = root.left;
    }else{
        root = root.right;
    }
}
return null;
```
#### BST Insertion

```
Node node = new Node(); // create a new node with special value
Node root = Tree.root(); // point to the root of BST
Node parent=null;

while(true){
    if(null == root){
        break;
    }
    parent = root;
    if(node.value.compareTo(root.value)<= 0){
        root = root.left;
    }else{
        root = root.right;
    }
}
if(null != parent){
    if(node.value.compareTo(parent.value) <=0){
        parent.left = node;
    }else{
        parent.right = node;
    }
}
```
#### BST Deletion
- Deleting a node with no children: simply remove the node from the tree.
- Deleting a node with one child: remove the node and replace it with its child.
- Deleting a node with two children: call the node to be deleted D. Do not delete D. Instead, choose either its in-order predecessor node or its in-order successor node as replacement node E (s. figure). Copy the user values of E to D. If E does not have a child simply remove E from its previous parent G. If E has a child, say F, it is a right child. Replace E with F at E's parent.

![image](https://upload.wikimedia.org/wikipedia/commons/3/36/AVL-tree-delete.svg)


---

## RBT(Red–black tree)
### Properties:
1. Each node is either red or black.
2. The root is black. This rule is sometimes omitted. Since the root can always be changed from red to black, but not necessarily vice versa, this rule has little effect on analysis.
3. All leaves (NIL) are black.   
4. If a node is red, then both its children are black.
5. Every path from a given node to any of its descendant NIL nodes contains the same number of black nodes.

![image](https://upload.wikimedia.org/wikipedia/commons/6/66/Red-black_tree_example.svg)


```
public class Node<T extends Comparable<T>> {
    private Node<T> parent;
    private Node<T> left;
    private Node<T> right;
    private boolean isRed;
    private T value;

    public Node<T> getParent() {
        return parent;
    }

    public void setParent(Node<T> parent) {
        this.parent = parent;
    }

    public Node<T> getLeft() {
        return left;
    }

    public void setLeft(Node<T> left) {
        this.left = left;
    }

    public Node<T> getRight() {
        return right;
    }

    public void setRight(Node<T> right) {
        this.right = right;
    }

    public boolean isRed() {
        return isRed;
    }

    public void setRed(boolean red) {
        isRed = red;
    }

    public T getValue() {
        return value;
    }

    public void setValue(T value) {
        this.value = value;
    }
}


public class BRT<T extends Comparable<T>> {

    /**
     * 获取父节点
     *
     * @param node
     * @return node
     */
    public Node<T> getParent(Node<T> node) {
        if (null != null) {
            return node;
        }
        return null;
    }

    /**
     * 获取祖父节点
     *
     * @param node
     * @return node
     */
    public Node<T> getGrandParent(Node<T> node) {
        Node<T> p = getParent(node);
        if (null == p) {
            return null;
        }
        return p;
    }

    /**
     * 获取兄弟节点
     *
     * @param node
     * @return
     */
    public Node<T> getSibling(Node<T> node) {
        Node<T> p = getParent(node);
        if (null == p) {
            return null;
        }
        if (node == p.getLeft()) {
            return p.getRight();
        } else {
            return p.getLeft();
        }
    }

    /**
     * 获取叔叔节点
     *
     * @param node
     * @return node
     */
    public Node<T> getUncle(Node<T> node) {
        Node<T> p = getParent(node);
        if (null == p) {
            return null;
        }
        return getSibling(p);
    }

    public Node<T> insert(Node<T> n, Node<T> root) {

        // 递归插入新点解
        insertRecursively(root, n);

        // 修复RBT
        insertRepairTree(n);
        root = n;
        while (null != getParent(root)) {
            root = getParent(root);
        }
        return root;
    }

    private void insertRepairTree(Node<T> node) {

        if (null == getParent(node)) {
            node.setRed(false);
        } else if (!getParent(node).isRed()) {
            return;
        } else if (null != getUncle(node) && getUncle(node).isRed()) {
            getParent(node).setRed(false);
            getUncle(node).setRed(false);
            getGrandParent(node).setRed(true);
            insertRepairTree(getGrandParent(node));
        } else {
            Node<T> p = getParent(node);
            Node<T> g = getGrandParent(node);
            if (node == p.getLeft() && g.getLeft() == p) {
                rotateLeft(node);
                node = node.getLeft();
            }
            if (node == p.getLeft()) {
                rotateRight(g);
            } else {
                rotateLeft(g);
            }
            p.setRed(false);
            g.setRed(true);
        }

    }

    /**
     * 左旋
     *
     * @param node 需要旋转的节点
     */
    public void rotateLeft(Node<T> node) {
        Node<T> tempNode = node.getRight();
        Node<T> p = getParent(node);
        assert null != tempNode;
        node.setRight(tempNode.getLeft());
        tempNode.setLeft(node);
        node.setParent(tempNode);

        if (null != node.getRight()) {
            node.getRight().setParent(node);
        }

        if (null != p) {
            if (node == p.getLeft()) {
                p.setLeft(tempNode);
            } else if (node == p.getRight()) {
                p.setRight(tempNode);
            }
        }
        tempNode.setParent(p);
    }

    /**
     * 节点右旋
     *
     * @param node
     */
    public void rotateRight(Node<T> node) {

        Node<T> tempNode = node.getLeft();
        Node<T> p = node.getParent();
        assert null != tempNode;
        node.setLeft(tempNode.getRight());
        tempNode.setRight(node);
        node.setParent(tempNode);
        if (null != node.getLeft()) {
            node.getLeft().setParent(node);
        }

        if (null != p) {
            if (node == p.getLeft()) {
                p.setLeft(tempNode);
            } else if (node == p.getRight()) {
                p.setRight(tempNode);
            }
        }
        tempNode.setParent(p);
    }

    /**
     * 循环插入新节点
     *
     * @param root 根节点
     * @param node 需要插入的新节点
     */
    private void insertLoop(Node<T> root, Node<T> node) {
        Node<T> parent = root;
        while (true) {
            if (null == root) {
                break;
            }
            if (node.getLeft().getValue().compareTo(root.getValue()) <= 0) {
                parent = root.getLeft();
            } else {
                parent = root.getLeft();
            }
        }
        if (null != parent) {
            if (node.getValue().compareTo(parent.getValue()) <= 0) {
                parent.setLeft(node);
            } else {
                parent.setRight(node);
            }
        }
    }

    /**
     * 递归插入新的node
     *
     * @param root 根节点
     * @param node 需要加入的新节点
     */
    private void insertRecursively(Node<T> root, Node<T> node) {
        if (null != root && node.getLeft().getValue().compareTo(root.getValue()) <= 0) {
            if (null != root.getLeft()) {
                insertRecursively(root.getLeft(), node);
                return;
            } else {
                root.setLeft(node);
            }
        } else if (null != root) {
            if (null != root.getRight()) {
                insertRecursively(root.getRight(), node);
                return;
            } else {
                root.setRight(node);
            }
        }
        node.setParent(root);
        node.setLeft(null);
        node.setRight(null);
        node.setRed(true);
    }
}
```

