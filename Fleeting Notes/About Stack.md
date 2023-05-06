#algorithm #data-structure
Stack helps to implement search and conquer algorithms like a traversal of maze in attempt to find the exit. As we traverse the maze we are going either left or right, if we hit the end we pop from the stack and continue.
The examples of stack used in the software dev:
* the memory relies on the stack as we keep reference in memory of the local block;
* Android fragments kept in the stack structure;

In the Stack we can only add/remove from one side. It is a data structure that implements last in first out. You can not remove the bottom plate without removing plates above it.

# Identify if the string is balanced
Given string `))((hello` is unbalanced. Given string `((hello))` is balanced.
Iterate through the string. Check if the char is `(` and add it to the stack. Check if the char is `)` and pop the stack. If the stack is empty as we check `)` it means that  the string unballenced as we have not yet encountered `)`.
```kotlin
fun String.isBalanced() : Boolean {
  val stack = Stack<Char>()

  for (char in this) {
	  when (char) {
		  is '(' -> stack.push(char)
		  is ')' -> when (stack.isEmpty) {
			  true -> return false
			  else -> stack.pop()
		  }
	  }
  }
  
  return stack.isEmpty
  }
}
```
