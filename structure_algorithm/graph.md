图
===

图的定义
-------------

图（Graph）是由顶点的又穷非空集合和顶点之间边的集合组成，通常表示为：G(V, E)，其中，G表示一个图，V是图G中顶点的集合，E是图G中边的集合。

* 数据元素的叫法，线性表叫元素，树叫结点，图叫顶点（Vertex）；
* 关于没有元素，线性表称为空表，树称为空树，图则不能为空；
* 元素间的关系，线性表元素之间是线性关系，树结点之间具有层次关系，图中元素之间没有固定关系，用边来表示顶点间的逻辑关系；

图的基本概念

* 无向边，若顶点vi到vj之间的边没有方向，则这条边称为无向边(Edge)，用无序偶对(vi,vj)表示；
* 无向图，如果图中任意两个顶点之间都是无向边，则称改图为无向图(Undirected graphs)；
* 有向边，若从顶点vi到vj的边有方向，则称这条边为有向边，也成为弧(Arc)，用有序偶对<vi,vj>表示，vi称为弧尾(Tail)，vj称为弧头(Head)；
* 有向图，如果图中任意两个顶点之间的边都是有向边，则称改图为有向图(Directed graphs)；
* 简单图，在图中，若不存在顶点到其自身的边，且同一条边不重复出现，则称为简单图；
* 在无向图中，如果意义两个顶点之间都存在边，则称为无向完全图；n个顶点的无向完全图边的个数为：n*(n-1)/2；
* 在有向图中，如果任意两个顶点之间都存在弧，则称为有向完全图；n个顶点的有向完全图的边的个数：n*(n-1)；
* 有很少边或弧的图称为稀疏图，反之称为稠密图；
* 与图的边或弧相关的数叫做权(Weight)；
* 带权的图称为网(Network)；
* 假设有两个图G=(V, {E})和G1=(V1,{E1})，其中V1是V的子集，E1是E的子集，则称G1为G的子图(Subgragh)；

图的顶点与边之间的关系

* 对于无向图G=(V,{E})，如果顶点vi和vj的有序对(vi,vj)属于E，则称vi和vj互为邻接点(Adjacent)，即vi和vj相邻接，边(vi,vj)依附(incident)于顶点vi和vj；或则称边(vi,vj)与顶点vi和vj相关联；
* 顶点v的度(Degree)是和v相关联的边的数目，记作TD(v)；
* 对于有向图G=(V,{E})，如果顶点vi和vj的有序对<vi,vj>属于E，则称vi邻接到vj，顶点vj邻接自顶点vi，弧<vi,vj>和顶点vi和vj相关联；
* 以顶点v为头的弧的数目称为v的入度(InDegree)，记作ID(v)，以v为尾的弧的数目称为v的出度(OutDegree)，记作OD(v)；顶点v的度为TD(v)=ID(v)+OD(v)；
* 路径(Path)，图G(V,{E})中，顶点vi到vj的路径是一个顶点序列，(vi,...vm,...vj)，序列中相邻两个顶点(vm,vm+1)或<vm,vm+1>属于E；
* 路径的长度是路径上的边或弧的数目；
* 第一个和最后一个顶点相同的路径称为回路或环(Cycle)；
* 路径顶点序列中，顶点不重复出现的称为简单路径；
* 除第一个和最后一个顶点外，其余顶点不重复的回路，称为简单回路或简单环；

连通图相关概念

* 无向图G中，如果顶点vi到顶点vj有路径，则称vi和vj是连通的；
* 如果无向图中任意两个顶点都是连通的，则称G是连通图(Connected Graph)；
* 无向图中的极大连通子图称为连通分量；
* 有向图G中，如果存在vi到vj的路径，并且存在vj到vi的路径，则称vi和vj是连通的；
* 如果有向图中任意两个顶点都是连通的，则称G是强连通图；
* 有向图中的极大强连通子图称为有向图的强连通分量；
* 连通图的生成树是一个极小的连通子图，它含有图中全部的n个顶点，但只有足已构成一颗树的n-1条边。

图的抽象数据类型
-----------------

<pre>
ADT 图(Graph)

Data
    顶点的又穷非空集合和边的集合

Operation
    CreateGraph(*G, V, VR): 按照顶点集V和边弧集VR的定义构造图
    DestroGraph(*G): 图G存在则销毁
    LocateVex(G, u): 若图中存在顶点u，则返回其在图中的位置
    GetVex(G, v): 返回图G中顶点v的值
    PutVex(G, v, value): 将图中顶点v赋值value
    FirstAdjVex(G, *v): 返回顶点v的一个邻接顶点，若顶点在G中无邻接点则返回false
    NextAdjVex(G, v, *w): 返回顶点v相对于顶点w的下一个临界顶点，若w是v的最后一个邻接点则返回false
    InsertVex(*G, v): 在图G中添加新顶点v；
    DeleteVex(*G, v): 删除图G中的顶点v及相关的边或弧
    InsertArc(*G, v, w): 在图中增加(v,w)或<v,w>
    DeleteArc(*G, v, w): 在图中删除(v,w)或<v,w>
    DFSTraverse(G): 对图G进行深度优先遍历，在遍历过程中调用每个顶点
    HFSTraverse(G): 对图G进行广度优先遍历，遍历过程中调用每个顶点
endADT
</pre>

图的存储结构
--------------

邻接矩阵(Adjacency Matrix)存储方式：用两个数组表示图，一个一维数组存储图中顶点信息；一个二维数组(Adjacency Matrix)存储图中的边或弧的信息；
* 无向图的邻接矩阵是一个对称矩阵
* arc[i][j]的值意味着vi和vj之间的邻接关系；
* vi的度就是arc中第i行或第i列不为空或0的元素个数
* 可以通过arc[i][j]和arc[j][i]设置不同的值，即非对称矩阵来存储有向图的边集
* 可以通过设置arc[i][j]为0,n,无穷大等不同的值来存储有权图即网的权值；

邻接矩阵(Adjacency Matrix)的实现
<pre>
// 邻接矩阵的数据结构
typedef char VertexType; // 定义顶点类型
typedef int EdgeType; // 边上的权值类型
#define MAXVEX 100 // 最大顶点数
#define INFINITY 65535 // 定义无穷大
typedef struct
{
  VertexType vexs[MAXVEX]; // 顶点数
  EdgeType arc[MAXVEX][MAXVEX]; // 邻接矩阵，边表
  int numVertexes, numEdges; // 图中当前的顶点数和变数
} MGraph;

// 基于邻接矩阵结构生成一个无向网图
void CreateMGraph(MGraph *G)
{
  int i, j, k, w;
  printf("输入顶点数和变数：\n");
  scanf("%d, %d", &G->numVertexes, &G->numEdges); // 输入顶点数和边数
  for (i = 0; i < G->numVertexes; i++) { // 读入顶点信息，建立顶点表
    scanf(&G->vexs[i]);
  }

  for (i = 0; i < G->numVertexed; i++) {
    for (j = 0; j < G->numVertexes; j++) {
      G->arc[i][j] = INFINITY; // 邻接矩阵初始化
    }
  }

  for (k = 0; k < G->numEdges; k++) { // 读入numEdges条边，建立邻接矩阵
    printf("输入边(vi, vj)上的下标i，下标j和权w:\n");
    scanf("%d, %d, %d", &i, &j, &w); // 输入边(vi,vj)上的权w
    G->arc[i][j] = w;
    G->arc[j][i] = G->arc[i][j]; // 无向图，矩阵对称
  }
}
// n个顶点e条边的无向图的创建，时间复杂度O(n+n2+e)，其中邻接矩阵的初始化为O(n2)
</pre>

邻接表(Adjacency List): 数组与链表相结合的存储方法称为邻接表。

邻接表(Adjacency List)的代码实现
<pre>
typedef char VertexType; // 定义顶点类型
typedef int EdgeType; // 定义边的权值类型

typedef struct EdgeNode // 边表结点
{
  int adjvex; // 邻接点域，存储该顶点对应的下标
  EdgeType weight; // 用于存储权值，
  struct EdgeNode *next; // 链域，指向洗衣歌邻接点
} EdgeNode;

typedef struct VertexNode // 定点表结点
{
  VertexType data; // 定点域，存储顶点信息
  EdgeNode *firstedge; // 边表头指针
} VertexNode, AdjList[MAXVEX];

typedef struct
{
  AdjList adjList;
  int numVertexex, numEdges; // 图中顶点数和边数
} GraphAdjList;

// 基于邻接表结构创建图
void CreateALGraph(GraphAdjList *G)
{
  int i, j, k;
  EdgeNode *e;
  printf("输入顶点数和边数：\n");
  scanf("%d, %d", &G->numVertexed, &G->numEdges); // 输入顶点数和边数
  for (i = 0; i < G->numVertexed; i++) { // 读入顶点信息，建立顶点表
    scanf(&G->adjList[i].data); // 输入顶点信息
    G->adjList[i].firstedge = null; // 边表置空
  }

  for (k = 0; k < G->numEdges; k++) { // 建立边表
    printf("输入边(vi,vj)上的顶点序号:\n");
    scanf("%d, %d", &i, &j); // 输入边(vi,vj)上的顶点序号
    e = (EdgeNode *) malloc(sizeof(EdgeNode)); // 生成边表结点
    e->adjvex = j; // 邻接序号为j
    e->next = G->adjList[i].firstedge; // 将e指针指向当前顶点指向的结点
    G->adjList[i].firstedge = e; // 将当前顶点的指针指向e

    e = (EdgeNode *) malloc(sizeof(EdgeNode)); 
    e->adjvex = i; // 邻接序号为i
    e->next = G->adjList[j].firstedge; 
    G->adjList[j].firstedge = e;
  }
}
// 针对n个结点e条边的图，时间复杂度O(n+e);
</pre>



图的遍历
-----------

最小生成树
-----------

最短路径
----------

拓扑排序
----------

关键路径
----------

总结
-----
