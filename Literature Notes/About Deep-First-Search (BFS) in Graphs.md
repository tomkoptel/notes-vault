#algorithm #graph
Notes based on [[Data structures & algorithms in Kotlin, I. Galata, M. Braun, M. Suica]] page 469.
# Applications
* Topological sorting
* Detecting a cycle
* Pathfinding in a maze
* Finding connected components in a sparse graph
# Key Points
In the deep first search we are visiting the node first neighbour until reaching the end of the chain. The easiest way to achieve this is by keeping the iterator of the edges for the each next level of the vertex.
Its time complexity is `O(V + E)` as we eventually visit all vertices and all edges per vertex.
```kotlin
fun <T : Any> Graph<T>.dfs(): List<Vertex<T>> {
  val sources = allVertices.firstOrNull() ?: return
  val stack = LinkedList<Iterator<Edge<T>>()
  val visited = mutableSetOf<Vertex<T>>()
  val order = mutableListOf<Vertex<T>>()
  
  visited.add(source)
  order.add(source)
  stack.push(edges(source).iterator())

  while (stack.isNotEmpty()) {
    val edgeIterator = stack.element()
    
	if (edgeIterator.hasNex()) {
	  val edge = edgeIterator.next()
	  val destination = edge.destination
	  if (destination !in visited) {
	    visited.add(destination)
	    order.add(destination)
	    stack.push(edges(destination).iterator())
	  }
	} else {
	   stack.pop()  
	}
  }

  return order
}
```
We don't pop immediately, but wait until all edges exhausted. This way we are eventually backtrack to the vertex on the initial levels to visit other neighbours which were not visited.
# Find a cycle
We can find the cycle in the graph with the help of slightly modified DFS.
```kotlin
fun <T : Any> Graph<T>.hasCycle(): List<Vertex<T>> {
  val sources = allVertices.firstOrNull() ?: return
  val stack = LinkedList<Vertex<T> to Iterator<Edge<T>>()
  val visited = mutableSetOf<Vertex<T>>()
  
  visited.add(source)
  stack.push(source to edges(source).iterator())

  while (stack.isNotEmpty()) {
    val (vertex, edgeIterator) = stack.element()
    
	if (edgeIterator.hasNex()) {
	  val edge = edgeIterator.next()
	  val destination = edge.destination
	  if (destination !in visited) {
	    visited.add(destination)
	    stack.push(destination to edges(destination).iterator())
	  } else {
		  return true
	  }
	} else {
	   stack.pop() 
	   visited.remove(vertex)
	}
  }

  return false
}
```
We need to remove the vertex from visited, because we have exhausted its neighbours and have not reached it from the other neighbours direction. Think about this a crossroad where each of the route that spawned from was visited without returning back meaning that there is no cycle between crossroad and its routes.