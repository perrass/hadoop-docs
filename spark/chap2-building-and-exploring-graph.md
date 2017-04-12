## Building and Exploring GraphX

### Graph builders

#### Graph factory method

Using the `apply` method in the companion object.

```scala
def apply[VD, ED] (vertices: RDD[(VertexId, VD)], 
                   edges: RDD[Edge[ED]],
                   defaultVertexAttr: VD = null): Graph[VD, ED]
```

The `Graph` factory method is convenient when the RDD collections of edges and vertices are readily available

#### edgeListFile

Only two lines in a txt file, the first elem in a line is `srcId`, and the second elem in a lien is `destId`. The API is `GraphLoader.edgeListFile`. But **how can we add attr of edges? Using the `mapEdge` method ?**

```scala
def edgeListFile(
	sc: SparkContext, 
	path: String,...): Graph[Int, Int]  // The type is [Int, Int], and default is 1
```



#### fromEdges, fromEdgeTuples

```scala
def fromEdges[VD, ED](edges: RDD[Edge[ED]])): Graph[VD, ED]
def fromEdgeTuples[VD](rawEdges: RDD[(VertexId, VertexId)]): Graph[VD, Int]  // Type is [VD, Int]
```

所以，还是那个问题，增量数据怎么处理，正确的方式应该是新增到库，再导入到`GraphX`中，而不是利用`GraphX`处理新增数据

### Building Graphs

```scala
    val myVertices = sc.makeRDD(Array((1L, "Ann"), (2L, "Bill"), (3L, "Charles"),
                                      (4L, "Diane"), (5L, "Went to gym this morning")))

    val myEdges = sc.makeRDD(Array(Edge(1L, 2L, "is-friends-with"), Edge(2L, 3L, "is-friends-with"),
                                  Edge(3L, 4L, "is-friends-with"), Edge(4L, 5L, "Like-status"),
                                  Edge(3L, 5L, "Wrote-status")))

    val myGraph = Graph(myVertices, myEdges)
      
    // 找到srcId是1的所有dstId
    // 返回类型是Array[org.apache.spark.graphx.VertexId]
    myGraph.edges.filter(_.srcId == 1).map(_.dstId).collect()
```

#### Building a bipartite graph (二分图)

A bipartite graph is composed of two sets of nodes. The nodes within the same set cannot be connected but only the pairs belonging to the different sets can be.