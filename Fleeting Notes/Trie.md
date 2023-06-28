#data-structure
### Summary
* Very effective at prefix matching
* Memory efficient, because many words share the same prefixes, thus we reuse some of the nodes
* Both insert and remove O(N) where N is the number of elements in the word you are looking/removing in Trie
### General
Trie allows us to build up tree dictionaries. With a trie we can implement search suggestions feature.
To represent Trie we use following node structure:
```kotlin
/**  
 * key is optional, because the root of the trie has no key 
 * a reference to the parent helps to remove() the node 
 * */
 * class TrieNode<Key : Any>(  
    val key: Key? = null,  
    val parent: TrieNode<Key>? = null,  
) {  
    val children = HashMap<Key, TrieNode<Key>>()  
  
    /** The indicator of the end of the collection */    
    var isTerminating = false  
}
```
As we can see one node has a reference to the list of children. It means that one level can be a host to the multiple node references.
### Removal
The tricky part while working with a Trie is to ensure that when we remove the word that is usually represented as collection of char. We need to backtrace through parent and remove any references from children map.
The easiest way to understand the removal. Is to imagine a Trie that contains 'cat', 'car' and 'carts'. This trie is 5 levels deep.
* c -> a -> t.
* c -> a -> r. -> t -> s.
The dot repsents the terminating flag. When we remove the word 'carts' we need to keep both 'cat' and 'car', but remove 's', then 't'.
1. We need find the last terminating node for the word 'carts'. This will be 's'.
2. Then we need travers back to the parent and check children for the 't' and remove it.
3. When we reach 'r' we see that it is a terminating node, so we exit the backtracking.
```kotlin
fun remove(keys: List<Key>) {  
    var current = root  
  
    keys.forEach { element ->  
        val child = current.children[element] ?: return  
        current = child  
    }  

	// we found 's' let's unmark it as terminated
    if (!current.isTerminating) return  
  
    current.isTerminating = false  
    var parent = current.parent // now we grab its parent which is 't'
    while (parent != null && current.children.isEmpty() && !current.isTerminating) {  
        parent.children.remove(current.key)  // we remove from parent node 's'
        current = parent  // now 't' needs to be removed from parent
        parent = current.parent // we are going up and to 'r' and then remove 't'
    }  
}
```
### Search of prefixes
With a Trie we can find all words prefixed with the arbitrary word. For the example, trie with word 'car', 'carper', 'carry' all prefixed with 'car'.
We are working with a trie, so to effectively traverse a data structure we need stack. The stack allows us to visit tree level by level. As we reach each branch we accumulate prefixes and the terminating flag reached and then we add word to the prefixes collection.
The time complexity is O(k * m) where `k` is longest collection matching the prefix and the `m` the number of collections that match the prefix.
```kotlin
fun allPrefixesNonRecursive(prefix: List<Key>): List<List<Key>> {  
    var current = root  
    prefix.forEach { element ->  
        val child = current.children[element] ?: return emptyList()  
        current = child // we need to find the last node that represents the last
        // element in prefix list
        // By traversing the trie using the `keys` sequence, the code ensures that the `current` node at the end of the traversal represents the last character in the sequence. This is important because it is at this node that we check if the word is terminating (`isTerminating`). If it is not terminating, the word is not present in the trie, and we can return early.
    }  
  
    val results = mutableListOf<List<Key>>()  
    val stack = Stack<Pair<List<Key>, TrieNode<Key>>>().also {  
        it.push(prefix to current)  
    }  
  
    while (stack.size > 0) {  
        val (currentPrefix, currentNode) = stack.pop()  
  
        if (currentNode.isTerminating) {  
            results.add(currentPrefix)  
        }  
  
        currentNode.children.forEach { (key, node) ->  
            val subPrefix = currentPrefix + key  
            stack.push(subPrefix to node)  
        }  
    }  
  
    return results  
}
```