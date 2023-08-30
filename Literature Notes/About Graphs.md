#data-structure 
Notes based on [[Data structures & algorithms in Kotlin, I. Galata, M. Braun, M. Suica]] page 349.
Useful representation of data composed of vertices and edges. There are weighted edges or those with assigned value (e.g price of flying from Tokyo to New York). The vertexes represent the object the edge relationships between the objects.

Directional when one direction is possible. Non-directional or bi-directional graphs where the movement possible in both directions.
The graph can be represented as an adjacency list. Where the vertex is a key to the list of edges coming out of the vertex.
The graph can be represented as adjacency matrix. Where the wightes represented by two-dimensional array. Both rows and columns represent the underlying graph structure.
## Adjacency List
The addition of vertex in the adjacency list is $O(1)$ operation.
```kotlin
private val adjacencies = mutableMapOf<Vertex<T>, ArrayList<Edge<T>>>()

fun createVertex(data: T) {
	val vertex = Vertex<T>(adjacencies.count(), data)
	adjacencies.put(vertex, arrayListOf<Edge<T>>())
}
```

The addition of edge is also $O(1)$ operation.
```kotlin
fun createEdge(source: Vertex<T>, destination: Vertex<T>, weight: Double) {
  adjacencies[source].add(Edge(source, destination, weight))
}
```
To list all edges is $O(1)$ operation.
```kotlin
fun edges(source: Vertex<T>): List<Edge<T>> {
	return adjacencies[source]
}
```

The computation of weigh between two vertices is $O(n)$.
```kotlin
fun weight(source: Vertex<T>, destination: Vertex<T>): Double? {
   return adjacencies[source].firstOrNull { it.destination == destination }?.weight
}
```
## Adjacency Matrix
The addition of vertex in the adjacency matrix is $O(V^2)$. As we add vertex we need to add a column of every row and as we know the matrix in our case is quadratic.

```kotlin
private val vertices = arrayListOf<Vertex<T>>()  
private val weights = arrayListOf<ArrayList<Double?>>()

fun createVertex(data: T) {
	val vertex = Vertex<T>(adjacencies.count(), data)
	vertices.add(vertex)

	// add column
	weights.forEach { it.add(null) }

	// add row
	val row = ArrayList<Double?>(vertices.count())
	repeat(vertices.count()) { row.add(null) } 
	weights.add(row)
}
```
The addition of an edge is a process of populating a weigh for the intersection between 2 vertices represented as indices in the 2, thus $O(1)$.
```kotlin
fun createEdge(source: Vertex<T>, destination: Vertex<T>, weight: Double) {
  weights[source.index][destination.index] = weight
```
To list all edges is $O(V)$ operation as we need to loop through all columns of the source vertex.
```kotlin
fun edges(source: Vertex<T>): List<Edge<T>> {
	val edges = mutableListOf<Edge<T>>()
	(0 until weights.count()).forEach { column ->
	  val weight = weights[source.index][column]
	  if (weight != null) {
		  edges.add(Edge(source, vertices[column], weight))
	  }
	}
	return edges
}
```
To find the weight between two vertices is `O(1)` operation.
```kotlin
fun weight(source: Vertex<T>, destination: Vertex<T>): Double? {
   return weights[source.index][destination.index]
}
```