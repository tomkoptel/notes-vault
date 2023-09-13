#algorithm #graph
Notes based on [[Data structures & algorithms in Kotlin, I. Galata, M. Braun, M. Suica]] page 456.
# Applications
* Generate a minimum-spanning tree.
* Find potential paths between vertices.
* Find the shortest path between two vertices.
# About
Breadth first search is a queue based algorithm that visits the vertex and then all its direct neighbours before going to the next level in the graph.
```kotlin
fun Graph<Vertex<T>.bfs(): List<Vertex<T>> {
  val source = allVertices.firstOrNull() ?: return emptyList()

  val queue = LinkedList<Vertex<T>>()
  val enqueued = mutableSetOf<Vertex<T>>()
  val visited = mutableListOf<Vertex<T>>

  queue.add(source)

  while (queue.isNotEmpty()) {
    val vertex = queue.poll()
    visited.add(vertex)
	
	val neighbours = edges(vertex)
	neighbours.forEach { neighbour ->
	  val destination = neighbour.destination
	  if (destination !in enqueued) {
	     queue.add(destination)
	     enqueued.add(destination)
	  }
	}
  }
  
  return visited
}
```
When traversing the tree we do visit all vertices which is $O(V)$. Then we visit each edge of the vertex which is $O(E)$. In summary, we have $O(V + E)$.
	The space complexity is $O(3 * V)$ as we create 3 structures to hold information about queue, those that were enqueued and visit order. We disregard a constant and this gives us $O(V)$.