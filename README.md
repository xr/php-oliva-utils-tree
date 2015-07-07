# php-tree-utils
Simple utility for handling tree structures in PHP (Nette).

Useful for handling **tree data stored in database** or for performing **search** and **filter** operations on tree structures.

## Usage
Each tree has a root node. Each node allows getting/setting of children and parent, thus creating a tree structure.

### Node
**NodeBase** is an abstract implementation of the **INode** interface. **Node** is a full-featured implementation allowing creation of trees with arbitrary data or objects.
```php
$node = new Node();
$node->title = 'item one';
$child = $node->addChild(new Node(['title' => 'first child of one']));
$node->addChildren([new Node(['title' => 'second child of one']), new Node(['title' => 'third child of one'])]);

$child->getParent() === $node; // TRUE
$siblings = $child->getSiblings();
$allChildren = $node->getChildren();
$child->isFirst(); // TRUE - first of the siblings
$child->isLast(); // FALSE - last of the siblings

$child->getLevel(); // 1 - since it only has one parent, the root (the root is level 0)

$child->detach(); // detach from the tree, becomes root
$child->isRoot(); // TRUE

// implements IteratorAggregate interface
foreach($node as $key => $node){...}

// can be created from arbitrary objects
$myObject = new MyObject();
$node = new Node($myObject);
$node->aMethodOfTheobject(); // the call is forwarded to the object
```

### Iterations, search, filtering
Create a tree.
```php
$tree = new Tree($rootNode);
```

Iterate through all the nodes, with nodes on the current level first (**breadth-first** iteration).
```php
$it = $tree->getIterator(TreeIterator::BREADTH_FIRST_RECURSION); // the default
foreach($it as $node) {...}
// or also this
foreach($tree as $node) {...}
```

Iterate through all the nodes, in **depth-first** manner.
```php
$it = $tree->getIterator(TreeIterator::DEPTH_FIRST_RECURSION);
foreach($it as $node) {...}
```

Iterate through the nodes on the **current level only**.
```php
$it = $tree->getIterator(TreeIterator::NON_RECURSIVE);
foreach($it as $node) {...}
```

**Search** for a node with a specific key-value pair.
```php
// find a red node
$redNode = $tree->find('color', 'red');
```

**Filter** specific nodes.
```php
// filter all big ripe oranges
$oranges = $orangeTree->getFilterIterator(['status' => 'ripe', 'size' => 'big'], TreeFilterIterator::MODE_AND);

// filter apples and pears
$filtered = $fruitTree->getFilterIterator(['type' => 'apple', 'type' => 'pear'], TreeFilterIterator::MODE_OR);

// filter very specific nodes
$it = $tree->getFilteringCallbackIterator(function(\Oliva\Tree\NodeBase $node, $index) {
    try {
    	return $index === '002' || $node->id === 4;
    } catch (MemberAccessException $e) {
    	return FALSE;
    }
});

```

### Recursive tree
A trivial tree where each data-node has a pointer to its parent.

| ID        | parent    | title|
|:----------|:----------|:-------|
|1          | NULL| the root|
|2|1|first child of root
|3|1|second child of root
|4|2|third level
|5|2|third level, second child
|6|NULL| second root

Get this structure from a database as an array.
Tell the tree that "parent" is the member where the parent's "id" is found.
Create the tree.
```php
$tree = new RecursiveTree($data, 'parent', 'id');
```

### Path tree
A tree where the position of a node in a tree is represented by a position (hierarchy) string. This allows ordering of leafs. Each level is represented by 3 (in this case) characters of the string representing the position of the node in the current sub-tree.

Path trees are useful for storing menu hierarchy or similar structures where position of a node within a sub-tree matters.

| ID        | position    | title|
|:----------|:----------|:-------|
|1          | 001| the root|
|2|001001|first child of root
|3|001002|second child of root
|4|001001001|third level
|5|001001002|third level, second child
|6|002|second root

Get this structure from a database as an array.
Tell the tree that you are using 'position' as the hierarchy column and there are 3 characters for each level.
Create the tree.
```php
$tree = new PathTree($data, 'position', 3);
```

>> Note: if you have more than one root, the tree won't work. This example is only to demostrate the possibility of having multiple roots.

### Transformations
Allows transformation to 1D arrays.
```php
$tree = new Tree($rootNode);
$linear = $tree->getBreadthFirst();
$linearWithDeepestFirst = $root->getDepthFirst();
```


## TODO

* remove dependency on Nette Framework
* materialized path tree (id variant)
* allow users to create own classes when using tree's transform() method
