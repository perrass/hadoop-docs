## GraphX Basics

The basic graph class in GraphX is called `Graph`, which contains two RDDs: one for edges and one for vertices. Once a graph has been constructed, you can access these collections via the `edges()` and `vertices()` accessors of `Graph`.

`Graph` defines a property graph, each edge and each vertex carries its own custom properties, described by user-definded classes.

Run a `.scala` file

> spark-shell -i file.scala  (local file)

### Functions

* `triplets()` joins vertexs and edges together, and returns an RDD of `EdgeTriplet[VD, ED]`, which is a subclass of `Edge[ED]`, and key fields of `EdgeTriplet` is
  * `Attr`, attribute data for the edge
  * `srcId`, vertex id of the edge's source vertex
  * `srcAttr`, attribute data for the edge's source vertex
  * `dstId`
  * `dstAttr`
* `mapTriplets()` transforms each edge attribute a partition at a time using the map function

```scala
myGraph.mapTriplets(t => (t.attr, t.attr == "is-friends-with" && t.srcAttr.toLowerCase.contains("a"))).triplets.collect
```

* `mapVertices()` transform the `Vertex` class
* `aggregateMessages[A](sendMsg: (EdgeContext[VD, ED, A]) => Unit, mergeMsg: (A, A) => A)` aggregates values from the neighboring edges and vertices of each vertex. The **user-supplied** `sendMsg` function is invoked on each edge of the graph, generating 0 or more messages to be sent to either vertex in the edge. The `mergeMsg` function is then used to combine all messages destined to the same vertex. **In the following code**, The class of `_` in `_.sendToSrc()` is `EdgeContext`
  * `sendMsg` is a method that takes an `EdgeContext` as parameter and returns nothing, `EdgeContext` class is similar to `EdgeTriplet`, but provides two more methods
    * `sendToSrc`, sends a message of type `Msg` to the source vertex
    * `sendToDst`, sends a message of type `Msg` to the destination vertex
  * All the messages for each vertex are collected together and delivered to the `mergeMsg` method, and return a `VertexRDD[Int]` class . `VertexRDD` is an RDD containing `Tuple2s` consisting of the `VertexId`and the `mergeMsg` result for that vertex.

```Scala
myGraph.aggregateMessages[Int](_.sendToSrc(1), _ + _).collect
```
### Save and Load

Read and write a standard Hadoop sequence file, which is a binary file containing a sequence of serialized objects

```scala
myGraph.vertices.saveAsObjectFile("myGraphVertices")
myGraph.edges.saveAsObjectFile("myGraphEdges")
val myGraph2 = Graph(
	sc.objectFile[Tuple2[VertexId, String]]("myGraphVertices"),
	sc.objectFile[Edge[String]]("myGraphEdges"))
```

The result is a dictory with lots of small files, whose performances is poor in Hadoop. Hence, a merge is required

> hdfs dfs -getmerge 	/user/xxx/myGraphVertices myGraphVertices

Or using Hadoop Java API

```scala
import org.apache.hadoop.fs.{FileSystem, FileUtil, Path}
val conf = new org.apache.hadoop.conf.Configuration
conf.set("fs.defaultFS", "hdfs://localhost")
val fs = FilsSystem.get(conf)
FileUtil.copyMerge(fs, new Path("/user/xxx/myGraphVertices/"), 
                   fs, new Path("/user/xxx/myGraphVerticesFile"),
                   false, conf, null)
```

Or saving a `triplet` instead of `vertices` and `edges`

### Summary

* `Pregel` and `aggregateMessages()` are the **cornerstones of graph processing in GraphX**