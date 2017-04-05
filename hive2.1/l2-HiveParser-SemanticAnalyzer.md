## l2-HiveParser-SemanticAnalyzer

Hive入口在(main方法)

```shell
find . -name '*.java' | xargs grep --color 'main(' | awk '{print $1}' | uniq | grep -v test
```

Hive执行流程

* 客户端使用`run`方法，生成Driver，Driver通过`compile`方法，编译命令（包括语法解析和语义分析）
* 语法解析: 

```java
ParseDriver pd = new ParseDriver();
ASTNode tree = pd.parse(command, ctx);
```

* `ParseDriver`会调用`HiveLexerX`做词法分析，词法分析文件在`HiveLexer.g`

```java
HiveLexerX lexer = new HiveLexerX(new ANTLRNoCaseStringStream(command));
TokenRewriteStream tokens = new TokenRewriteStream(lexer);
```

* 语法解析HiveParser，它是Antlr根据`HiveParser.g`生成的文件，利用`HiveParser.statement().getTree()`得到ASTNode. 这里会有对应的语法文件**`HiveParser.g`**很重要，是理解AST和之后优化的基础

---

**E.g: select 语句**

TOK_QUERY

* LEFT: TOK_FROM
  * RIGHT: TOK_SUBQUERY
* RIGHT: TOK_INSERT
  * LEFT: TOK_DESTINATION
    * LEFT: TOK_DIR
      * RIGHT: TOK_TMP_FILE (**这是Hive比Impala慢的原因，需要读写本地文件，或者说SELECT语句会写入这个文件，结束后展示这个文件. Impala没有这个机制，但当有节点失效时，并非所有数据都会被打印**)
  * RIGHT: TOK_SELECT (这里面有孩子还可以增加，orderBy, clusterBy, distributeBy, sortBy, limit)
    * LEFT: TOK_SELEXPR
    * LEFT: TOK_ALLCOLREF

---

* **语义解析Semantic Analyzer**: 输入AST树，输出Operator图. Semantic Analyzer有13183行代码，Hive优化的全部都在这里

---

**SQL执行顺序**

```SQL
SELECT (5) DISTINCT (6) <list>
FROM <table source> (1)
WHERE <condition> (2)
GROUP BY <group by list> (3)
HAVING <condition> (4)
ORDER BY <order by list> (7)
```

**对应的Operators**

* TableScanOperator -> (1) FROM
* ReduceSinkOperator (把数据发送给Reduce端做聚合操作) -> JOIN/GROUPBY
* JoinOperator -> JOIN
* SelectOperator -> (5) SELECT
* FileSinkOperator -> 结果发送给TOK_TMP_FILE
* FilterOperator -> (2) WHERE (4) HAVING
* GroupByOperator -> (3) GROUP BY (6) DISTINCT
* MapJoinOperator -> JOIN
* LimitOperator -> LIMIT
* UnionOperator -> UNION

---

* 生成QB: `SemanticAnalyzer.genResolvedParseTree()`
  * `SemanticAnalyzer.doPhase1()` (递归，深度优先搜索)
  * 经过一轮深度优先遍历，不带元数据的QB树就生成了
  * 然后通过`getMeta()`访问`MetaStore`，把`QB`树填充完成
* Logical Plan Generator -> **`genPlan`方法**，实现QB -> Operator，也是深度优先递归