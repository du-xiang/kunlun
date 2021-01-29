# C语言实现迷宫的生成及寻路

   	这个博客主要记录一下自己写的代码与想法。**迷宫的主要功能包括自定义或者自动生成一个随机迷宫，并且找出迷宫的最短路径。**虽然这个代码略显粗糙，但却是我第一次尽心力完成的。所以我还是把它给放了上来。

## 要点：

### 1.数据存储

​	迷宫的不同结点的信息（行，列，状态，路径长度，前一结点的地址）存储在结构体中，并且自定义了一个**结构体指针型的二维数组**来存放对应结点的结构体地址。

​	由于地图大小是可以随机的，为了**防止申请过大的二维数组空间**,先申请一个三级指针型顺序表存放二级指针型顺序表的头节点，二级指针型顺序表用来存放结构体指针。这样就有效解决了不能用未知数申请二维数组的问题。

### 2.迷宫随机生成原理

​	随机迷宫的生成暂时没有好的想法，还比较粗糙。简单介绍就是从起点开始**向任一方向挖路，到终点结束**。这里需要先设定一个起点与终点，再随机生成一定范围的随机数决定挖掘方向，并适当提高终点那两个方向的比重。（起点（1，1），终点（10，10），生成1~6的随机数，1表示向上，2表示向左，3，4表示向下，5，6表示向右）

### 3.寻路算法原理

​	寻路主要利用迭代。按顺序判断一个结点的上下左右，如果符合条件就进入此方向的结点，将上一结点的地址放在此节点的结构体中的结构体型指针中，形成路径链接。并重复上面的判断过程，直到迭代结束。

**全部源码如下：**

```c
/*
北京联合大学2020-2021学年数据结构实训作业
作者：杜祥，文康，尹海龙
时间：2020年12月30日
*/

#include<stdio.h>
#include<stdlib.h>
#include<time.h>

typedef struct Node
{	//记录迷宫中的结点数据
	int line;	//结点行坐标
	int col;	//结点列坐标
	int staue;	//记录迷宫结点状态（1.障碍。0.通路）
	int len;	//记录结点目前最短距离
	struct Node* front;	//指向前一个Node的地址
}Node, * LNode;

void Surround(int i, int j);   //函数声明

void InitPathTree(LNode** Maze,const int a,const int b)
{	//初始化二维结构体数组
	LNode node;
	int i = 1, j = 1;
	for (i = 1; i <= a; i++)
	{
		for (j = 1; j <= b; j++)
		{//为结构体指针型二维数组赋予初值
			node = (LNode)malloc(sizeof(Node));
			node->front = NULL;
			node->line = i;
			node->col = j;
			node->len = 0;
			Maze[i - 1][j - 1] = node;
			
		}
	}
}

void Distance(LNode** Maze,const int a,const int b ,int i, int j, int k)
{	//判断结点的最短距离,形参k表示上一结点的方位（1，上，2，下，3，左，4，右）
	LNode last = NULL;
	switch (k)		//用于判断上一结点位置
	{
	case 1:
		last = Maze[i - 2][j - 1];
		break;
	case 2:
		last = Maze[i][j - 1];
		break;
	case 3:
		last = Maze[i - 1][j - 2];
		break;
	case 4:
		last = Maze[i - 1][j];
		break;
	default:
		break;
	}
	if (!Maze[i - 1][j - 1]->len || Maze[i - 1][j - 1]->len > last->len + 1)
	{	//此节点的到原点的最短距离为零或此结点显示距离大于上一结点距离加一
		//令此结点指向上一个结点
		Maze[i - 1][j - 1]->front = last;
		Maze[i - 1][j - 1]->len = last->len + 1;
		Surround(Maze, a, b, i, j);
	}
}

void Surround(LNode** Maze,const int a,const int b,int i, int j)
{	//判断四周是否为通路
	if (Maze[i - 2][j - 1]->staue == 0 && i != 1)	//此节点不为障碍物，且未超出坐标范围
		Distance(Maze, a, b, i - 1, j, 2);
	if (Maze[i][j - 1]->staue == 0 && i != b)	//此节点不为障碍物，且未超出坐标范围
		Distance(Maze, a, b, i + 1, j, 1);
	if (Maze[i - 1][j - 2]->staue == 0 && i != 1)	//此节点不为障碍物，且未超出坐标范围
		Distance(Maze, a, b, i, j - 1, 4);
	if (Maze[i - 1][j]->staue == 0 && i != a)	//此节点不为障碍物，且未超出坐标范围
		Distance(Maze, a, b, i, j + 1, 3);
}

void ShortestPath(LNode** Maze,const int a,const int b)
{	//求最短路径
	int i = 2, j = 2;	//规定起点坐标
	Maze[i - 1][j - 1]->len = 1;		//规定起点的初始距离为1
	Surround(Maze,a,b,i, j);
}

void MakePath(LNode** Maze,const int a,const int b)
{	//输出路径
	LNode P;
	int i = a, j = b;		//定义终点坐标
	P = Maze[i - 2][j - 2];
	while (P->front != NULL)
	{
		Maze[P->line - 1][P->col - 1]->staue = 2;
		printf("(%d,%d) -> ", P->col, P->line);
		P = P->front;
	}
	Maze[P->line - 1][P->col - 1]->staue = 2;
	printf("(%d,%d)", P->line, P->col);
	printf("\n");
}

void OutPut(LNode** Maze,const int a,const int b)
{	//实现迷宫的可视化
	int i = 0, j = 0;	//先假定迷宫为9*10的大小

	printf("   ");
	for (i = 1; i <= a; i++)	//输出列坐标
		printf("%2d", i);
	printf("\n");

	for (i = 0; i < a; i++)
	{
		printf("%2d ", i + 1);		//输出行坐标
		for (j = 0; j < b; j++)	//图形输出迷宫
			switch (Maze[i][j]->staue)
			{
			case 0:
				printf("  ");
				break;
			case 1:
				printf("%c%c", 0xa8, 0x80);
				break;
			case 2:
				printf("tq");
			}
		printf("\n");
	}
}

void PathLen(LNode** Maze,const int a,const int b)
{	//绘制各结点路径长度图
	int i = 0, j = 0;
	for (i = 0; i < a; i++)
	{
		for (j = 0; j < b; j++)
		{
			if (Maze[i][j]->staue)
				printf("%2d ", 0);
			else
				printf("%2d ", Maze[i][j]->len);
		}
		printf("\n");
	}
}

void DigTrunk(int* row, int* col,const int i,const int j)//row是当前行坐标，col是列坐标，i是总行数，j是总列数
{	//判断挖掘方向
	switch ((rand() % 6) / 2)	//0，挖上。1，挖左。2，3，挖下。4，5，挖右
	{	//返回挖掘方向，1.上。2.下。3.左。4.右。
	case 0:
		if (rand() % 6 == 0 && (*row) != 2)		//保证挖掘方向未超出围墙
			(*row)--;
		else if ((*col) != 2)
			(*col)--;
		break;
	case 1:
		if ((*row) != i - 1)
			(*row)++;
		break;
	case 2:
		if ((*col) != j - 1)
			(*col)++;
		break;
	}
}

void DigBranch(LNode** Maze,int row,int col,const int i,const int j)
{	//挖掘树枝通路
	if (!(rand() % 5))	//设置挖掘分支的概率为1/5
	{
		switch ((rand() % 6) / 2)	//0，挖下。1，挖右。2，3，挖上。4，5，挖左
		{	//返回挖掘方向，1.上。2.下。3.左。4.右。
		case 0:
			if (rand() % 6 == 0 && row != i-1)		//保证挖掘方向未超出围墙
				row--;
			else if(col != j-1)
				col--;
			break;
		case 1:
			if (row != 2)
				row++;
			break;
		case 2:
			if (col != 2)
				col++;
			break;
		}
	}
	Maze[row - 1][col - 1]->staue = 0;
	DigBranch(Maze, row, col, i, j);
}

void MakeMap(LNode** Maze,const int i,const int j)
{
	int row = 0, col = 0;	//row为行向量,col为列向量
	//修砌围墙
	for (row = 0; row < i; row++)
	{
		Maze[row][0]->staue = 1;
		Maze[row][j - 1]->staue = 1;
	}
	for (col = 0; col < j; col++)
	{
		Maze[0][col]->staue = 1;
		Maze[i - 1][col]->staue = 1;
	}

	row = 2; col = 2;	//初始化起点为（2，2）
	Maze[row - 1][col - 1]->staue = 0;		//起始坐标上数值为0，表示为通路
	while ((row != i - 1) || (col != j - 1))	//循环到（i-1,j-1)结束
	{
		//DigBranch(Maze,row, col, i, j);	//先挖掘支路，支路不改变当前结点坐标
		DigTrunk(&row, &col, i, j);	//挖掘主要通路
		Maze[row - 1][col - 1]->staue = 0;
	}
	for (row = 0; row < i; row++)	//填充围墙
		for (col = 0; col < j; col++)
		{
			if (Maze[row][col]->staue != 0 && Maze[row][col]->staue != 1)
				Maze[row][col]->staue = 1;
		}
}

void UserMake(LNode** Maze, const int i, const int j)
{	//用户自定义迷宫地图
	int t = 0, h = 0, num = 0;
	for (t = 0; t < i; t++)
	{
		printf("输入%d行(0.通路。1.障碍):",t+1);
		for (h = 0; h < j; h++)
		{
			scanf_s("%d",&num);
			Maze[t][h]->staue = num;
		}
	}
	system("cls");
}

int main()
{
	int chiose = 0;
	int random = 0;
	int i = 0, j = 0, t = 0;
	LNode** Maze;
	srand((unsigned)time(NULL));	//生成随机数种子

	while (1)
	{
		//用户交互操作
		printf("#####################\n#\t\t    #\n#输入地图的行数:___ #\b\b\b\b"); scanf_s("%d", &i);
		printf("#####################\n#\t\t    #\n#输入地图的列数:___ #\b\b\b\b"); scanf_s("%d", &j);

		//动态申请空间 Maze[i][j]
		Maze = (LNode**)malloc(sizeof(Node*) * i);
		for (t = 0; t < i; t++)
			Maze[t] = (LNode*)malloc(sizeof(LNode) * j);

		//用户交互操作
		system("cls");
		printf("################\n#1.自定义地图  #\n################\n#2.随机生成地图#\n################\n\n请输入选择的序号:___\b\b");
		scanf_s("%d", &random);
		system("cls");
		InitPathTree(Maze, i, j);	//初始化二维地址数组
		if (random == 1)
			UserMake(Maze, i, j);	//用户自定义迷宫地图
		else
			MakeMap(Maze, i, j);	//随机生成迷宫地图
		printf("----------迷宫图为--------------------------------------------------\n\n");
		OutPut(Maze, i, j);	//迷宫可视化
		 
		ShortestPath(Maze, i, j);	//规划最短路径
		printf("\n\n----------迷宫图通路中各结点到原点的最短距离为---------------------\n\n");
		PathLen(Maze, i, j);	//画出各结点的路径长度结点图
		printf("\n\n----------迷宫坐标结点路径为-------------------------------------\n\n");
		MakePath(Maze, i, j);		//输出路径图
		printf("\n\n----------带路径迷宫图为-----------------------------------------\n\n");
		OutPut(Maze, i, j);		//输出带最短路径的图

		system("pause"); system("cls");
		printf("0，退出。1，继续\n是否退出？___\b\b");
		scanf_s("%d", &chiose);
		if (chiose == 0)
			break;
	}
	return 0;
}
```

