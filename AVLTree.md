# AVL Trees

While challenging myself to work with trees, I was having major issues getting AVL trees
to stick. However often I implemented and re-implemented them the logic just would not
click. That is probably because I never took the time to sit down and really explain to
myself how they work. This post is an exercise in doing that with corresponding code examples 
in Java.

Even if AVLs have become passé I think they are a great foundation to have in tree rotation
and tree balancing.

## What are AVL trees

AVL trees are the first-ever invented self-balancing binary search trees (BST). BST are trees
where the values on the left hand side of a node are always smaller than the values on the right
hand side. In mathematical terms it looks something like this:

```
left < node's value < right
```

For simplicity I'm skipping the situation where there might be duplicate values in the tree,
but there are a few ways to store those.

```
       (5)
      /   \
	 /     \
   (4)     (7)
	       /
	      / 
	    (6)
```

BSTs are great. They provide O(log _n_) running time on average for search, inserts and deletion.

But BSTs have a major downside: they can become very unbalanced.

```
   (5)
      \
	   (6)
	     \
		  (7)
```

This is a perfectly valid BST. If `(8)` is inserted into this tree it will be added as a right
child of `(7)`. This tree has the search complexity of O(_n_) and that makes sense if you look
at it: the BST has degraded into a linked list.

This is where AVL trees come in. They are designed to prevent a situation like the one above
from happening by re-balancing the tree during the insert or delete operation which will maintain
the O(log _n_) running time we like so much from BSTs. This is done by rotating the tree around
an unbalanced node.

### Balance Factor

The type of rotation picked depends on the balance factor of the tree. The balance factor
indicates how balanced or unbalanced the tree is and on which side it is unbalanced.

The balance factor is calculated as by withdrawing the height of the left sub-tree from the
height of the right sub-tree, or in mathematical terms:

```
BalanceFactor(Node) = Height(Node.Right) - Height(Node.Left)
```

Or in Java

``` java
int balanceFactor(Node n) {
	return height(n.right) - height(n.left);
}
```

If the balance factor is less than -1, the tree is considered "left heavy". If it is more than
1 it is considered "right heavy". A AVL tree is one where every node has a balance factor of
1, 0 or -1.

```
Left heavy if BalanceFactor(Node) < -1
Right heavy if BalanceFactor(Node) > 1
```

Or in Java

```java
boolean isLeftHeavy(Node n) {
	return balanceFactor(n) < -1;
}

boolean isRightHeavy(Node n) {
	return balanceFactor(n) > 1;
}
```

This tree is perfectly balanced:

```
             (b)  B(b) = height(b.right) - height(b.left) = 1 - 1 = 0
	        /   \
B(a) = 0  (a)   (c) B(c) = 0
```

This tree is left heavy, but still a valid AVL tree.

```
	        (b)    B(b) = height(b.right) - height(b.left) = 1 - 2 = -1
		   /   \
B(a) = 0 (a)    (c) B(c) = 0 - 1 = -1
		          \
                   (d) B(d) = 0
```

This tree is left heavy _and_ unbalanced. It will need to be re-balanced.

```
	        (b)    B(b) = height(b.right) - height(b.left) = 1 - 3 = -2
		   /   \
B(a) = 0 (a)    (c) B(c) = 0 - 2 = -2
		          \
                   (d) B(d) = 0 - 1 = -1
				     \
					  (e) B(e) = 0
```

This tree is right heavy, but still a valid AVL tree.

```
	        (c)    B(c) = height(b.right) - height(b.left) = 2 - 1 = 1
		   /   \
B(b) = 1 (b)    (d) B(c) = 0
	    /
      (a)
```

This tree is right heavy _and_ unbalanced. It will need to be re-balanced.

```
	        (d)    B(c) = height(b.right) - height(b.left) = 3 - 1 = 2
		   /   \
B(c) = 2 (c)    (e) B(c) = 0
	    /
      (b)
	 /
   (a)
```


## Types of rotations

AVL trees describe four types of rotations: "Simple Left Rotation", "Simple Right Rotation",
"Right-Left Rotation" and "Left-Right Rotation".

### Simple Left Rotation

A tree should be rotated with a Simple Left Rotation when the root (a) is right heavy and the
root's right child is not left heavy.

```
   (a)                   (c)
      \                 /   \
	   (c)     ->     (a)   (d)
	  /  \               \    \
	(b)   (d)            (b)  (e)
	        \
			(e)
```

* In order traversal before the change: a b c d e
* In order traversal after the change:  a b c d e

In this situation, (c) becomes the new root. (c)'s left children become (a)'s right children and
(a) becomes (c)'s left child. Or in code:

```java
Node simpleLeftRotation(Node a) {
	Node c = a.right;        // This is c
	Node c_left = c.left;    // This is c's left child
	c.left = a;              // a becomes c's left child
	a.right = c_left;        // c's left child becomes a's right child
	return c;                // Return the new root
}
```

### Simple Right Rotation

A tree should be rotated with a Simple Right Rotation when the root (e) is left heavy and the
root's left child is not right heavy. This operation is the mirror operation of the Simple Left
Rotation described above.

```
            (e)            (c)
		   /              /   \
	     (c)      ->    (b)    (e)
	    /  \           /      /
	  (b)  (d)        (a)   (d)
	 /
   (a)
```

* In order traversal before the change: a b c d e
* In order traversal after the change:  a b c d e

In this situation (c) becomes the new root. (c)'s right children become (e)'s left children and
(e) becomes (c)'s right child. Or in code:

```java
Node simpleRightRotation(Node e) {
	Node c = e.left;         // This is c
	Node c_right = c.right;  // This is c's right child
	c.right = e;             // e becomes c's right child
	e.left = c_right;        // c's right child becomes e's left child
	return c;                // Return the new root
}
```

### Left-Right Rotation

A tree should be rotated with a Left-Right rotation when the root is left heavy _but the root's right
node is right heavy_.

```
            (e)
		   /
	     (b)
	    /  \
	  (a)  (c)
             \
             (d)
```

* In order traversal: a b c d e

Looking at that tree, two things stick out:

* (e) is right heavy
* (b) is left heavy

Which makes it a candidate for Left-Right rotation. Here we first rotate the right heavy sub-tree
and then rotate the now changed left heavy sub-tree. Lets rotate the right heavy sub-tree as we've
done before using Simple Left Rotation.

```
	     (b)               (c)
	    /  \              /   \
	  (a)  (c)    ->    (b)   (d)
             \         /
             (d)     (a)
```

This sub-tree is no longer right heavy, but has become left heavy. What does the whole tree look
like now?

```
         (e)
	    /
	  (c)
     /   \
   (b)   (d)
  /
(a)
```

As with the Right-Left Rotation, this tree has been primed for a rotation we know how to do. In this
case we need to apply Simple Right Rotation.

```
         (e)               (c)
	    /                 /   \
	  (c)               (b)   (e)
     /   \        ->    /    /
   (b)   (d)          (a)   (d)
  /
(a)
```

The whole rotation looks like this:

```
     (e)               (e)           (c)
     /                 /            /   \
   (b)      ->       (c)     ->   (b)   (e)
  /  \              /   \         /     /
(a)  (c)          (b)   (d)     (a)   (d)
       \         /
       (d)     (a)
```

* In order traversal before: a b c d e
* In order traversal after:  a b c d e

We first rotate the right heavy sub-tree using (1) Simple Left Rotation,
and then rotate the left heavy sub-tree using (2) Simple Right rotation.

Hence the name, Left-Right rotation!

Or in code

```java
Node leftRightRotation(Node e) {
	e.right = simpleLeftRotation(e.right); // Rotate the sub-tree
	return simpleLeftRotation(e);          // Rotate e
}
```

### Right-Left Rotation

A tree should be rotated with a Right-Left Rotation when the root (a) is right heavy _but the
root's left node is left heavy_.

```
   (a)
      \
	   (d)
	  /  \
	(c)   (e)
   /
 (b)
```

In order traversal before the change: a b c d e

Looking at that tree, two things stick out:

* (a) is right heavy
* (d) is left heavy

Which makes it a candidate for Right-Left Rotation. Here we first rotate the left heavy sub-tree
and then rotate the now changed right heavy sub-tree. Let's rotate the left-heavy sub-tree as we've
done before using Simple Right Rotation.

```
	   (d)            (c)
	  /  \           /   \
	(c)   (e)  ->  (b)   (d)
   /                        \
 (b)                        (e)
```

* In order traversal before: b c d e
* In order traversal after:  b c d e

This sub-tree is no longer left heavy, but has now become right heavy. What does the whole tree
look like now?

```
   (a)
      \
   	  (c)
	 /   \
   (b)   (d)
           \
           (e)
```

Well! Now we have a tree where (a) is left heavy, as before, but (c) is not right heavy. We can
therefore apply Simple Left Rotation here.

```
   (a)                   (c)
      \                 /   \
	   (c)     ->     (a)   (d)
	  /  \               \    \
	(b)   (d)            (b)  (e)
	        \
			(e)
```

The whole rotation looks as such

```
   (a)                (a)                 (c)
      \                  \               /   \
	   (d)     ->        (c)       ->  (a)   (d)
	  /  \              /   \             \    \
	(c)   (e)         (b)   (d)           (b)  (e)
   /                          \
 (b)                          (e)
```

* In order traversal before the change: a b c d e
* In order traversal after the change:  a b c d e

We first rotate the left heavy sub-tree using (1) Simple Right Rotation,
and then rotate the right heavy sub-tree using (2) Simple Left Rotation.

Hence the name, Right-Left rotation!

Or in code

```java
Node rightLeftRotation(Node a) {
	a.left = simpleRightRotation(a.left); // Rotate the sub-tree
	return simpleLeftRotation(a);         // Rotate a
}
```

# Inserting into a AVL tree

Now that we know how inserts can unbalance the tree and how the unbalanced
tree can be re-balanced so that it remains a valid AVL tree, we need an insert
method that takes care of this.

Inserts into a AVL tree starts out just like an insert into a BST:

```java
Node insertIntoBst(Node node, int value) {
	if (node == null) {
		return new Node(value);
	}
	if (node.value > value) {
		node.left = insertIntoBst(node.left, value);
	} else if (node.value < value) {
		node.right = insertIntoBst(node.right, value);
	}
	return node;
}
```

This is a recursive call that will insert into either the left or right child
depending on the value (remember how BST are ordered). Once the base case is
hit, where `node` is `null` a new node is added. The call stack looks as such.

```
insert(a, c)
(a) insert(a.right, c)       -> (a)
  \                               \
  (b) insert(b.right, c)          (b)
                                    \
									(c)
```

Since `b.right` is `null`, it's value is set to (c). This will create unbalanced
trees fast as new items are inserted, and therefore we know we need to re-balance.

The trick to writing a recursive algorithm like this is to ignore the recursive
nature of it and think of only a handful of cases. In case of AVL trees we only
care about four potential cases:

1. The node needs to be rotated using Simple Left Rotation after insert if the
   node is right heavy and the node's right child is not left heavy;
1. The node needs to be rotated using Simple Right Rotation after insert if the
   node is left heavy and the node's left child is not right heavy;
1. The node needs to be rotated using Left-Right Rotation after insert if the
   root is left heavy but the root's right node is right heavy;
1. The node needs to be rotate using Right-Left Rotation after insert if the node
   is right heavy but the node's left child is left heavy.

We have already described the cases that cause the condition that trigger these
rotations so what we need to do is to _check for them after the insert_. In our
case we need to check for them after the recursive call to insert. Let's give
it a try.

```java
Node insertIntoAvl(Node node, int value) {
	// Start with a normal BST insert, but we can ignore that when dealing with
	// AVL specific logic, as it only applies after insert.
	if (node == null) {
		return Node(value);
	}
	if (node.value > value) {
		node.left = insertIntoBst(node.left, value);
	} else if (node.value < value) {
		node.right = insertIntoBst(node.right, value);
	}
	
	// Check if the tree is unbalanced. If it is not, return
	int balanceFactor = balanceFactor(node);
	if (balanceFactor >= -1 && balanceFactor <= 1) {
		return node;
	}
	// Check for condition (1) described above
	if (isRightHeavy(n) && !isLeftHeavy(n.right)) {
		return simpleLeftRotation(n);
	}
	// Check for condition (2) described above
	if (isLeftHeavy(n) && !isRightHeavy(n.left)) {
		return simpleRightRotation(n);
	}
	// Check for condition (3) described above
	if (isRightHeavy(n) && isLeftHeavy(n.right)) {
		return leftRightRotation(n);
	}
	if (isLeftHeavy(n) && isRightHeavy(n.right)) {
		return rightLeftRotation(n);
	}
	// Nothing matches? It's balanced so just return
	return node;
}
```

A beautiful thing about AVL trees is that the balance operation takes place
in constant time if properly implemented. That part is actually missing from
the implementation above for clarity, but adding the height of the node to
the node would not be a major change. That means that the big-oh insert complexity
of a AVL tree is O(log _n_ + 1) or simply O(log _n_) in the best and worst case.
That's a hell of a lot better than BSTs worst case of O(_n_)!

And that's how you insert into AVL trees. Deletions from AVL trees follow a
somewhat different set of rules, which I might write about in another post.
