---
title: 算法
keywords: [algorithm] 
---

# 递归

# 图

## 最短路径问题

- 主要有两中算法解决最短路径问题
  - 最短路径即在图中点与点之间的最短路径

### 迪杰斯特拉算法（Dijkstra）

- 属于贪心算法

- 动态维护一个数组，记录到各个点的最短路径

  ```mermaid
  graph TD;
      1--10-->2; 
      2--50-->3;
      1--100-->5;
      1--30-->4;
      3--10-->5;
      4--20-->3;
      4--60-->5;
  ```

- 迭代过程

  | 迭代 | S               | u    | dist[2] | dist[3] | dist[4] | dist[5] |
  | ---- | --------------- | ---- | ------- | ------- | ------- | ------- |
  | 初始 | {1}             | -    | 10      | +∞      | 30      | 100     |
  | 1    | {1，2}          | 2    | 10      | 60      | 30      | 100     |
  | 2    | {1，2，4}       | 4    | 10      | 50      | 30      | 90      |
  | 3    | {1，2，4，3}    | 3    | 10      | 50      | 30      | 60      |
  | 4    | {1，2，4，3，5} | 5    | 10      | 50      | 30      | 60      |

  - 将s看成一个整体，记录到达其他点的最短距离

- 实现

  ```java
  public static int[] dijkastra(int[][] graph, int startPointIndex) {
      //集合s表示，已经求出最短路径的点的集合
      boolean[] s = new boolean[graph.length];
      //初始化s集合，只有起始点为true
      for (int i = 0; i < graph.length; i++) {
          s[i] = (i == startPointIndex);
      }
      //最短路径集合
      int[] dis = new int[graph.length];
      //初始化，起始点到其他点的距离
      for (int i = 0; i < graph.length; i++) {
          dis[i] = graph[startPointIndex][i];
      }
      //求到每个点的最短路径
      for (int i = 0; i < graph.length; i++) {
          int tmpdis = max;
          int tmpindex = 0;
          for (int j = 0; j < graph.length; j++) {
              //不在s集合(未求出该点的最短路径)
              if (!s[j] && dis[j] < tmpdis) {
                  tmpdis = dis[j];
                  tmpindex = j;
              }
          }
          //dis中距离最短的点加入到s集合中
          s[tmpindex] = true;
  	    //更新起点到其他点距离
          for (int j = 0; j < graph.length; j++) {
              if (dis[j] > dis[tmpindex] + graph[tmpindex][j]) {
                  dis[j] = dis[tmpindex] + graph[tmpindex][j];
              }
          }
      }
      return dis;//返回起点到每个点的最短路径长度
  }
  ```

### 弗洛伊德算法（Floyd）

- 属于动态规划
- 直接算出距离
- 路径负权值也可以计算，但是要求图中不能有负环（）

### Bellman-Ford 算法
