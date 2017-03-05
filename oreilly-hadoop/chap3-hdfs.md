Chap3 The Hadoop Distribute File System
===

#### Definition

HDFS is a filesystem designed for storing very large files with streaming data access patterns (the pattern is write-once, read-many-times pattern), running on clusters of commodity hardware.

**Drawbacks**

* Low-latency data access: HDFS is optimized for delivering a high throughput of data, and this may be at the expense of latency. HBase is a proxy
* Lots of small files: the reason is **the namenode holds filesystem metadata in memory and 150 bytes for one file**
* Multiple writers, arbitrary file modifications: Files in HDFS may be written to by a single writer. Writes are always made at the end of the file, in **append-only** fashion

### HDFS Concepts

#### Blocks

HDFS blocks are large compared to disk blocks (normally 512 bytes), and the reason is to **minimize the cost of seeks**.

> If the seek time is about 10ms and trasfer rate is 100MB/s, to make the seek time 1% of the transfer time, the block size should be 100MB

**Benefits**

* A file can be larger than any single disk in the network
* Making the unit of abstraction a block rather than a file simplifies the storage subsystem
    + storage management
    + seperate metadata eliminating concerns
* Replication for providing fault tolerance and availibility

HDFS's `fsck` command (filesystem check) understands blocks

> $ hdfs fsck / -files -blocks

#### Namenodes and Datanodes

An HDFS cluster has two types of nodes operating in a **master-worker** pattern

* namenode: It maintains the **filesystem tree** and the **metadata for all the files and directories in the tree**. This information is stored persistently on the local disk in the form of two files: the **namespace image** and the **edit log**
* datanodes: They store and retrieve blocks when they are told to, and report back to the namenode **periodically** with lists of blocks that they are storing

**mechanisms for datanodes failure**

* Back up the files that make up the persistent state of the filesystem metadata
* Running secondary namenode: Its main role is to periodically merge the namespace image with the edit log to prevent the edit log from becoming too large. The secondary namenode usually **runs on a separate physical machine** because it requires plenty of **CPU** and **as much memory** as the namenode to perform the merge

#### Block Caching

For frequently accessed files the blocks may be explicitly **cached** in the datanode's memory, in an **off-heap block cache**. Job schedulers (for MapReduece, Spark etc.) cam take advantage of cached block. Users or applications instruct the namenode which files to cache (and for how long) by add a **cache directive to a cache pool**

---

**Difference between on-heap and off-heap**

> The on-heap store refers to objects that will be present in the Java heap (also subject to GC). On the other hand, the off-heap store refers to (serialized) objects that are managed by EHCache but stored outside the heap (not subject to GC). As the off-heap store continues to be managed in memory, it is slightly slower than on-heap store

---

#### HDFS Federation

HDFS Federation allows a cluster to scale by adding namenodes, each of which manages a portion of the filesystem namespace.

#### HDFS availibility

The new namenode is not able to serve requests until it has

* load its namespace image into memory
* replay its edit log
* receive enough block reports from the datanodes to leave safe mode

HDFS HA solve this **single point of failure** by a pair of namenodes in an active-standby configuration. The **architectual changes are**:

* The namenodes must be highly available shared storage to share the edit log
* Datanodes must send block reports to both namenodes because the block mapping are stored in a namenode's memory, and not on disk
* Clients must be configured to handle namenode failover
* The secondary namenode's role is subsumed by the standby, which takes periodic checkpoints of the active namenode's namespace

### The Command-Line Interface

Using `hdfs dfs`

Make a new dictionary for input

> hdfs dfs -mkdir /input

Copy from local

> hdfs dfs -copyFromLocal ~/hadoop-book/input/docs/quangle.txt /input

Check the copy

> hdfs dfs -ls /input

### The Java Interface

#### Hadoop URL

Read a file from a Hadoop filesystem by using a `java.net.URL` object to open a stream to read the data from.

1. Packaging the .jar from IDE **without main method**
2. In command line, `export HADOOP_CLASSPATH=ch03-hdfs.jar`
3. `hadoop URLCat hdfs://localhost/input/quangle.txt`
    + URLCat is one main method in ch03-hdfs.jar
    + hdfs://localhost/input/quangle.txt is the protocol of URL


    import java.io.InputStream;
    import java.net.URL;

    import org.apache.hadoop.fs.FsUrlStreamHandlerFactory;
    import org.apache.hadoop.io.IOUtils;

    public class URLCat {

        static {
            URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());
        }

        public static void main(String[] args) throws Exception {
            InputStream in = null;
            try {
                in = new URL(args[0].openStream());
                IOUtils.copyBytes(in, System.out, 4096, false);
            } finally {
                IOUtils.closeStream(in);
            }
        }
    }

Java is required to recognize **Hadoop's hdfs URL scheme**. This is achieved by calling the `setURLStreamHandlerFactory()` method on URL with an instance of `FsUrlStreamHandlerFactory`. This method can be **called only once per JVM**, so it is typically **executed in a static block**. This limiation means that if some other part of your program sets a `URLStreamHandlerFactory`, you won't be able to use this approach for reading data from Hadoop.

#### FileSystem API

A file in a Hadoop filesystem is represented by a Hadoop `Path` object.

Static factory methods for getting a `FileSystem` instance from HDFS:

    public static FileSystem get(Configuration conf) throws IOException
    public static FileSystem get(URL url, Configuration conf) throws IOException
    public static FileSystem get(URL url, Configuration conf, String user) throws IOException

Or

    public static LocalFileSystem getLocal(Configuration conf) throws IOException

With a `FileSystem` instance in hand, we invoke an `open()` method to get the input stream for a file

    public FSDataInputStream open(Path f) throws IOException
    public abstract FSDataInputStream open(Path f, int bufferSize) throws IOException

Hence,

    import java.io.InputStream;
    import java.net.URL;

    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.FileSystem;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.IOUtils;

    public class FileSystemCat {

        public static void main(String[] args) throws Exception {
            String url = args[0];
            Configuration conf = new Configuration();
            FileSystem fs = FileSystem.get(URL.create(url), conf);
            InputStream in = null;
            try {
                in = fs.open(new Path(url));
                IOUtils.copyBytes(in, System.out, 4096, false);
            } finally {
                IOUtils.closeStream(in);
            }
        }
    }

#### FSDataInputStream

The `open()` method on `FileSystem` actually returns an `FSDataInputStream` rather than a standard `java.io` class.

The `Seekable` interface permits seeking to a position in the file and provides a query method for the current offset from the start of the file

    public interface Seekable {
        void seek(long pos) throws IOException
        long getPos() throws IOException
    }

    ...
    in = fs.open(new Path(url))
    in.seek(0)
    ...

#### FSDataOutputStream

    public FSDataOutputStream create(Path f) throws IOException

---

**Warning**

The `create()` method create any parent directories of the file to be written that don't already exist. You should check for the existence of the parent directory first by calling the `exist()` method

    public boolean exists(Path f) throws IOException

---

However, unlike `FSDataInputStream`, `FSDataOutputStream` does not permit seeking

### Data Flow

#### Anatomy of a File Read

1. The **client** calls `open()` on the `FileSystem` object, which for HDFS is an instance of `DistributedFileSystem`
2. `DistributedFileSystem` calls the namenode, using **RPCs** (remote procedure calls), to determine the locations of the first few blocks in the file, and then it returns an `FSDataInputStream` to the client for it to read data from. **`FSDataInputStream` in turn wraps a `DFSInputStream`**, which manages the datanode and namenode I/O
3. The client calls `read()` on the stream `DFSInputStream`, and then connects to the first (closest) datanode for the first block in the file. (**only connect**)
4. Data is streamed from the datanode back to the client, which calls read() repeatedly on the stream.
5. When the end of the block is reached, `DFSInputStream` will close the connection to the datanode, then find the best datanode for the next block
6. When the client has finished reading, it calls `close()` on the `FSDataInputStream`

One important aspect of this design is that the **client contacts datanodes directly to retrieve data and is guided by the namenode to the best datanode for each block**. This leads to data traffic is **spread across all the datanodes in the cluster**. Meanwhile, the namenode merely has to service block location requests (which it stores in memory, making them very efficient)

---

**How to measure closity?**

Ideally, the bandwidth between two nodes is a measure of distance. In practice, the bandwidth is defined by:

* Processes on the same node
* Different nodes on the same rack
* Nodes on different racks in the same data center
* Nodes in different data centers

---

#### Anatomy of a File Write

1. The **client** calls `open()` on the `FileSystem` object, which for HDFS is an instance of `DistributedFileSystem`
2. `DistributedFileSystem` calls the namenode, using **RPCs** (remote procedure calls), to determine the locations of the first few blocks in the file, and then it returns an `FSDataInputStream` to the client for it to read data from. **`FSDataInputStream` in turn wraps a `DFSInputStream`**, which manages the datanode and namenode I/O
3. The client writes data, the `DFSOutputSStream` splits it into packets, which it writes to an internal queue called the **data queue**. The data queue is consumed by the `DataStreamer`, which is responsible for asking the namenode to allocate new blocks by picking a list of suitable datanodes to store the replicas. The list of datanodes forms a pipeline. The `DataStreamer` streams the packets to the first datanode in the pipeline, which stores each packet and forwards it to the second datanode in the pipeline.
4. The second datanode stores the packet and forwards it to the third datanode in the pipeline.
5. The `DFSOutputSStream` also maintains an internal queue of packets that are waiting to be acknowledged by datanodes, called the **ack queue**. A packet is removed from the ack queue only when it has been acknowledged by all the datanodes in the pipeline.
6. When the client has finished writing, it calls `close()` on the `FSDataOutputStream`
7. This action flushes all the remaining packets to the datanode pipeline and waits for acknowledgement before contacting the namenode to signal that the file is complete.

### Parallel Copying with `distcp`
