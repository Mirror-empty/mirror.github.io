# BFS和DFS

### BFS

模板：

```java
  public static int shortestPathBinaryMatrix(int[][] grid) {

        // 顺时针从最左边开始 {{0,-1},{-1,-1},{-1,0},{-1,1},{0,1},{1,1},{1,0},{1,-1}}
        if (grid[0][0]!=0){
            return -1;
        }

        // 难点在于 这个dir的设计去作为方向的选择
        int[][] dir = {{0,-1},{-1,-1},{-1,0},{-1,1},{0,1},{1,1},{1,0},{1,-1}};
        int m = grid.length;
        int n = grid[0].length;
        // 1.一个队列存储起始节点，一个去判断是否走过
        Queue<int[]> queue = new LinkedList<>();
        boolean[][] visted =new boolean[grid.length][grid[0].length];
        // 2、将最初的起始节点放入队列，状态更新
        queue.offer(new int[]{0,0});
        visted[0][0]=true;
        // 记录步数
        int path=1;
        // 3、遍历队列中的所有可能作为起始的节点
        while (!queue.isEmpty()){
            int sz=queue.size();
            // 4、当前队列中的所有节点为同一扩散层级
            for (int i = 0; i < sz; i++) {
                // 5、每次读取队列顶端节点，进行相应的判断，并从队列中弹出
                int[] cur=queue.poll();
                int x=cur[0];
                int y=cur[1];
                // 6、判断当前该节点是否为最终目标，满足条件return
                if (x==m-1&&y==n-1){
                    return path;
                }
                // 7、当前节点满足条件，将其子节点放入队列中存储，在下一次循环（扩散）时进行读取
                // 根据题目的条件去判断需要遍历的节点
                for (int [] d:dir) {
                    int x1=x+d[0];
                    int y1=y+d[1];
                    // 8、边界判断，其子节点是否越界、已被访问以及其他限制条件
                    if (x1>m-1||x1<0||y1>n-1||y1<0||grid[x1][y1]!=0||visted[x1][y1]){
                        continue;
                    }
                    queue.offer(new int[]{x1,y1});
                    visted[x1][y1]=true;

                }
            }
            path++;

        }
        return -1;
```

上述为单向BFS算法模板，可以提炼为7个步骤：
1. 定义两个队列q和visited，分别用来存储可作为起始的节点和已被访问过的节点
2. 将最开始的起始节点放入队列q，并在visited中更新其状态
3. 遍历能作为起始节点的队列q（while循环队列）
4. 对当前同一层节点进行遍历(对应for循环)
5. 每次读取队列顶端节点，进行相应的判断，并从队列中弹出
6. 判断当前节点是否为最终目标，满足条件return
7. 当前节点满足条件，但不是最终目标。将其子节点放入队列q，在下一次循环（扩散）时进行读取
8. 在将子节点放入队列是需要判断子节点是否合法，如果合法放入队列，在visited中进行状态更新


### DFS

模板：

```java

    int [][] dir={{0,-1},{-1,0},{0,1},{1,0}};
    public int numIslands(char[][] grid) {
        Stack<char[]> stack=new Stack<>();
        boolean [][] visted= new boolean[grid.length][grid[0].length];
        stack.push(new char[]{grid[0][0]});
        int x=0,y = 0;

        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[0].length; j++) {

            if (grid[i][j]=='1'){
                dfs(grid,i,j);
                x++;
            }
            }
        }
        return x;

    }

    // 递归，表示深度的变量
    void dfs(char[][] grid, int x, int y){
        //递归出口
        if (grid[x][y]=='2'){
            //可能需要的操作
            return;
        }
        int sum=0;
        for (int[] d:dir
             ) {
            int x1=x+d[0];
            int y1=y+d[1];
            if (x1>grid.length-1||x1<0||y1> grid[0].length-1||y1<0||grid[x1][y1]=='0'){
                continue;
            }
            grid[x][y]='2';
            //枚举下一种情况
            dfs(grid,x1,y1);

        }

    }

```

还沉浸在BFS中，不需要dir这些选择方向，直接dfs去选择就行，优化了的

```java
    void dfs2(char[][] grid,int x1,int y1){
        if (x1>grid.length-1||x1<0||y1> grid[0].length-1||y1<0||grid[x1][y1]=='0'){
            return;
        }
        grid[x1][y1]='0';
        dfs2(grid,x1,y1-1);
        dfs2(grid,x1-1,y1);
        dfs2(grid,x1,y1+1);
        dfs2(grid,x1+1,y1);
    }
```