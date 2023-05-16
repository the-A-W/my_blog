---
title: 坦克大战（IKUN版）
date: 2023-04-20 22:19:13
tags: hey
categories: haha
---
# C 语言坦克大战（IKUN 版）

**前言：该坦克大战基于 90 经典坦克大战，对地图音乐等元素进行重新定制，实现成分十足的 IKUN 版坦克大战**

**设计思路：把地图和坦克在地图上的位置都存储在一个二维数组里，坦克的移动，地图的破环以及碰撞检测都是通过修改或是检查二维数组的数据实现，以下是设计的过程**

## 游戏窗口的实现

通过 easyx 库中所提供的函数实现

### 初始化一个 650* 650（单位：像素）大小的窗口

```C++
//初始化窗口
	initgraph(650, 650);
```

### 菜单功能

#### 菜单显示

```C++
//显示主菜单
IMAGE display_1_img;
loadimage(&display_1_img, _T("display_1.png"), 650, 650);
putimage(0, 0, &display_1_img);
//help菜单图片
IMAGE help_img;//help菜单图片
loadimage(&help_img, _T("help.bmp"), 650, 650);//加载图片
```

主菜单我采用了直接制作一张完整的图片然后覆盖整个窗口

可以采用 easyx 中自带的绘制图形或输出文字的函数，如果要改变菜单按钮等可以直接修改代码完成，如果用整图的话就需要重新制图。

主菜单的帮助界面我采用直接显示一张完整的图，所以先提前加载一下图片

#### 菜单按钮实现

```C++
MOUSEMSG mouse; //储存鼠标信息变量
```
首先定义一个变量存储鼠标信息，通过获取到的鼠标信息在进行选择操作

```C++
//int finish = 1;//用于结束主菜单循环

//主菜单按钮功能实现
while (1) 
{
	mouse = GetMouseMsg();

	switch (mouse.uMsg)
	{
	//help功能显示
	case WM_MOUSEMOVE:
		if (mouse.x > 245 && mouse.x <430 && mouse.y>320 && mouse.y < 385)
			putimage(0, 0, &help_img);
		else
			putimage(0, 0, &display_1_img);
		break;
	case WM_LBUTTONDOWN:
		if (mouse.x > 245 && mouse.x < 430 && mouse.y>155 && mouse.y < 225)
		{
			setbkcolor(BLACK);//设置背景色
			cleardevice();
			int stage = 1;
			while (1)
			{
				int result = game(stage);
				gameover(result);
				while (1)
				{
					system("pause");
					if (KEY_DOWN(Key_ENTER))
						break;
				}
				if (result)
					stage++;
			}
			
			finish = 0;
			break;
		}
		else if (mouse.x > 245 && mouse.x < 430 && mouse.y>235 && mouse.y < 300)
		{
			game();
			break;
		}
		else if (mouse.x > 245 && mouse.x < 430 && mouse.y>400 && mouse.y < 470)
		{
			exit(0);
			break;
		}
	}
```

这里先说明一下第一个 case 语句，**WM_MOUSEMOVE** 获取的是鼠标的移动信息，在鼠标移动的时候就进入该 case 语句，然后后面的 if 语句则进行判断，如果鼠标移动的区域在 help 框的范围那么就显示 help 图片，如果没有的话就显示主菜单的图片

然后再说明一下第二个 case 语句， **WM_LBUTTONDOWN** 可以获取鼠标的左键点击信息，如果按下左键则进入该语句，同样也是通过 if 语句判断鼠标点击的位置执行不同的操作，首先说最后一个，当我们点击 exit 框的时候就直接执行 exit（0）终止程序

然后说明一下第一个 if 语句，当我们点击 **new game** 时则直接进行游戏

如果点击 **continue** 则会加载游戏存档，继续上一次的游戏，继续游戏和新游戏公用一个函数，通过传递不同的参数值，来选择进入 **game()** 的时候是直接初始化，还是使用存档中的内容进行初始化操作

## 游戏模块

#### 地图的加载以及显示

```C++
//首先在game（）函数实现的中定义一个全局二维数组以存储地图信息
int map[map_row][map_col];//map_row,map_col为地图的长和宽，也是地图数组行和列的大小（在gamesettings.h中定义）

//然后是地图资源的加载，需要从文件中加载地图资源并存储到map数组中
void loadmap(int stage)
{
	//地图文件流
	ifstream map_file;
	//保存地图文件名
	wstring temp;
	temp = map_file_path;
	temp += map_file_name;
	int text = 0;
	map_file.open(temp);
	if (!map_file.is_open())
	{
		text = 1;
		cout << "打开地图失败！" << endl;
		//exit(1);
	}

	//调整文件读取位置，到最大关卡数之后则从第一关开始
	stage -= 1;
	stage = stage % max_stage;
	map_file.seekg(stage * map_row * map_col * 2);
	int num_temp;
	for (int row = 0;row < map_row;row++)
		for (int col = 0;col < map_col;col++)
		{
			map_file >> dec >> num_temp;
			map[row][col] = num_temp;
		}
	map_file.close();

}

//地图显示函数
void Map(int stage) {
	int row, col;
	IMAGE img_home, img_wall_1, img_wall_2;

	loadimage(&img_home, _T("home_1.jpg"), 50, 50);// 老鹰
	loadimage(&img_wall_1, _T("wall_1.jpg"), 25, 25);//不可消除的墙
	loadimage(&img_wall_2, _T("wall_2.jpg"), 25, 25);//可消除的墙


	for (i = 0; i < 26; i++) {
		for (j = 0; j < 26; j++) {
			if (map[i][j] == 1) {
				putimage(25 * j, 25 * i, &img_wall_1);
			}
			else if (map[i][j] == 2) {
				putimage(25 * j, 25 * i, &img_wall_2);
			}
			else if (map[i][j] == 3) {
				putimage(25 * j, 25 * i, &img_home);

				map[i][j] = 4;
				map[i][j + 1] = 4;
				map[i + 1][j] = 4;
				map[i + 1][j + 1] = 4;
			}
		}
	}
}
```

game ()函数

```C++
int game(int stage)
{
	setbkcolor(BLACK);//设置背景色
	cleardevice();
	//背景音乐
	mciSendString(_T("play cxk_1.mp3 repeat"), NULL, 0, NULL);
	//初始化随机数种子
	srand((unsigned int)time(NULL));
	

	//我方坦克
	Tank my_tank;
	//我方坦克发射的子弹
	Bullet my_bullet;

	//敌方坦克
	Tank enemy_tank[enemy_num];
	//敌方坦克发射的子弹
	Bullet enemy_bullet[enemy_num];


	//加载地图
	loadmap(stage);
	Map(stage);

	//记录当前的程序的休眠次数，每次10ms
	int times = 0;
	//敌方坦克存在数量
	int enemy_total = 0;
	//子弹目前不存在
	my_bullet.status = 0;
	
	//测试
	

	//加载坦克图片
	IMAGE img_mytank[DIRECTIONCOUNT];
	IMAGE img_enemytank[DIRECTIONCOUNT];
	loadimage(&img_mytank[UP], mytank_filename[UP], 50, 50);
	loadimage(&img_mytank[DOWN], mytank_filename[DOWN], 50, 50);
	loadimage(&img_mytank[LEFT],mytank_filename[LEFT], 50, 50);
	loadimage(&img_mytank[RIGHT], mytank_filename[RIGHT], 50, 50);
	loadimage(&img_enemytank[UP], enemytank_filename[UP], 50, 50);
	loadimage(&img_enemytank[DOWN], enemytank_filename[DOWN], 50, 50);
	loadimage(&img_enemytank[LEFT], enemytank_filename[LEFT], 50, 50);
	loadimage(&img_enemytank[RIGHT], enemytank_filename[RIGHT], 50, 50);
		
	//初始化坦克
	my_tank.x = 8;
	my_tank.y = 24;
	my_tank.direction = UP;
	my_tank.live = 1;

	set_map(my_tank.x, my_tank.y, 200);

	//设置敌方坦克出场位置
	for (int i = 0;i < enemy_num;i++)
	{
		if (i % 3 == 0)
		{
			enemy_tank[i].x = 0;
		}
		else if (i % 3 == 1)
		{
			enemy_tank[i].x = 12;
		}
		else if (i % 3 == 2)
		{
			enemy_tank[i].x = 24;
		}
		enemy_tank[i].y = 0;
		enemy_tank[i].live = 1;
		enemy_tank[i].direction = DOWN;
		enemy_tank[i].Is_Mycamp = 0;
		//set_map(enemy_tank[i].x, enemy_tank[i].y, 100 + i);
		enemy_bullet[i].status = 0;
	}

enemy_total = 3;
	//enemy_tank[0].x = 4;
	//enemy_tank[0].y = 0;
	//enemy_tank[0].live = 0;
	//敌方坦克登场
	for (int i = 0;i < enemy_total;i++)
	{
		if (enemy_tank[i].live)
		{
			set_map(enemy_tank[i].x, enemy_tank[i].y, 100 + i);
			do_tank_walk(&enemy_tank[i], enemy_tank[i].direction, &img_enemytank[enemy_tank[i].direction], 0);
		}

	}

	//显示坦克		
	putimage(my_tank.x * 25, my_tank.y * 25, &img_mytank[my_tank.direction]);
	
	
	while (1)
	{
		//防止坦克出生重叠
		if (map[enemy_tank[enemy_total].x][enemy_tank[enemy_total].y] == 0)
		{
			if (times > 0 && times % 1000 == 0 && enemy_total < enemy_num)
			{
				set_map(enemy_tank[enemy_total].x, enemy_tank[enemy_total].y, 100 + enemy_total);
				enemy_total++;
			}
		}
		if (times>0 && times % 200 == 0)
		{
			for (int i = 0;i < enemy_total;i++)
			{
				if (!enemy_tank[i].live)
					continue;
				//攻击我方老巢
				if (i % 2 == 0)
				{
					DIRECTION d = enemy_direction(&enemy_tank[i], 12, 24);
					tank_walk(&enemy_tank[i], d, &img_enemytank[d]);
				}
				else
				{
					DIRECTION d = enemy_direction(&enemy_tank[i], my_tank.x, my_tank.y);
					tank_walk(&enemy_tank[i], d, &img_enemytank[d]);
				}
				tank_fire(&enemy_tank[i], &enemy_bullet[i]);
			}
			
		}
		
		//0.5s移动一次敌方坦克
		else if (times % 50 == 0)
		{
			for (int i = 0;i < enemy_total;i++)
			{
				if (enemy_tank[i].live)
				{
					tank_walk(&enemy_tank[i], enemy_tank[i].direction, &img_enemytank[enemy_tank[i].direction]);
				}
			}
		}
		if (times % 10 == 0)
		{
			if (KEY_DOWN(Key_LEFT))
			{
				tank_walk(&my_tank, LEFT, &img_mytank[LEFT]);
			}
			if (KEY_DOWN(Key_UP))
			{
				if (my_tank.direction == UP && my_tank.y - 1 >= 0 && map[my_tank.y - 1][my_tank.x] == 0 && map[my_tank.y - 1][my_tank.x + 1] == 0)
					{
						do_tank_walk(&my_tank, UP, &img_mytank[my_tank.direction], 1);
					}
					else if (my_tank.direction != UP)
					{
						my_tank.direction = UP;
						do_tank_walk(&my_tank, UP, &img_mytank[my_tank.direction], 0);
					}
			}
			if (KEY_DOWN(Key_DOWN))
			{
				if (my_tank.direction == DOWN && my_tank.y + 2 <= 25 && map[my_tank.y + 2][my_tank.x] == 0 && map[my_tank.y + 2][my_tank.x + 1] == 0)
				{
					do_tank_walk(&my_tank, DOWN, &img_mytank[my_tank.direction], 1);
				}
				else if (my_tank.direction != DOWN)
				{
					my_tank.direction = DOWN;
					do_tank_walk(&my_tank, DOWN, &img_mytank[my_tank.direction], 0);
				}
			}
			if (KEY_DOWN(Key_RIGHT))
			{
				if (my_tank.direction == RIGHT && my_tank.x + 2 <= 25 && map[my_tank.y][my_tank.x + 2] == 0 && map[my_tank.y + 1][my_tank.x + 2] == 0)
				{
					do_tank_walk(&my_tank, RIGHT, &img_mytank[my_tank.direction], 1);
				}
				else if (my_tank.direction != RIGHT)
				{
					my_tank.direction = RIGHT;
					do_tank_walk(&my_tank, RIGHT, &img_mytank[my_tank.direction], 0);
				}
			}
			//开火
			if (KEY_DOWN(Key_J_Bullut))
			{

				PlaySound(_T("paoji.wav"), NULL, SND_FILENAME | SND_ASYNC);
				tank_fire(&my_tank, &my_bullet);
			}
			//暂停
			if (KEY_DOWN(Key_P_Stop))
			{
				while (1)
				{
					system("pause");
					if (KEY_DOWN(Key_P_Stop))
						break;
				}
			}
			if (KEY_DOWN(Key_ESC))
			{
				
				if (1)
				{
					save(enemy_tank, &my_tank, enemy_total, times,map);
				}
				display_1();
				//break;
			}
		}
		
		
		if (my_bullet.status)
		{
			if (bullet_action(&my_bullet, enemy_tank, my_tank.Is_Mycamp))
			{
				return 0;
			}
		}
		for (int i = 0;i < enemy_num;i++)
		{
			if (enemy_bullet[i].status)
			{
				if (bullet_action(&enemy_bullet[i], enemy_tank,0))
				{
					return 0;
				}
			}
		}
		//判断敌方坦克是否全部消灭
		int isWin = 0;
		for (int i = 0;i < enemy_num;i++)
		{
			if (enemy_tank[i].live == 0)
				isWin++;
			if (isWin == 10)
				return 1;
		}
		Sleep(10);
		times++;
	}
}
```

#### 坦克移动功能的实现

```C++
int do_tank_walk(Tank* tank,DIRECTION dir,IMAGE *img,int step)
{
	int new_x = tank->x;
	int new_y = tank->y;
	int old_map = map[tank->y][tank->x];
	if (step) {
		if (dir == UP)
		{
			new_y--;
		}
		else if (dir == DOWN)
		{
			new_y++;
		}
		else if (dir == LEFT)
		{
			new_x--;
		}
		else if (dir == RIGHT)
		{
			new_x++;
		}
		else
		{
			return 0;
		}
		//清除地图数组上原坦克位置
		set_map(tank->x, tank->y, 0);
	}
	//擦除原位置的坦克图像
	setfillcolor(BLACK);
	solidrectangle(tank->x * 25, tank->y * 25, tank->x * 25 + 50, tank->y * 25 + 50);
	if (step)
	{
		//更新地图数组坦克位置

		set_map(new_x, new_y, old_map);

		//更新坦克坐标
		tank->x = new_x;
		tank->y = new_y;
	}
	//调整方向应在外部实现，否则会传入键入前的坦克方向
	//调整方向
	//tank->direction = dir;
	putimage(tank->x * 25, tank->y * 25, img);
	return 1;
}
```

坦克移动的实现还需要设置地图数据的函数，这个函数主要是简化代码，因为每个坦克都会占据 4 个地图单位所以每次移动都需要把重新设置地图数组上四个数据

```C++
void set_map(int x,int y,int val)
{
	map[y][x] = val;
	map[y][x+1] = val;
	map[y+1][x] = val;
	map[y+1][x+1] = val;
}
```

#### 坦克炮击功能


```C++
void tank_fire(Tank* tank, Bullet* bullet)
{
	if (!bullet->status)
	{
		//PlaySound(_T("paoji.wav"), NULL, SND_FILENAME | SND_ASYNC);
		if (tank->direction == UP)
		{
			//子弹绘图坐标修正
			bullet->pos_x = tank->x * 25 + 23;
			bullet->pos_y = tank->y * 25 - 3;
		}
		else if (tank->direction == LEFT)
		{
			//子弹绘图坐标修正
			bullet->pos_x = tank->x * 25 - 3;
			bullet->pos_y = tank->y * 25 + 23;
		}
		else if (tank->direction == DOWN)
		{
			//子弹绘图坐标修正
			bullet->pos_x = tank->x * 25 + 23;
			bullet->pos_y = tank->y * 25 + 50;
		}
		else if (tank->direction == RIGHT)
		{
			//子弹绘图坐标修正
			bullet->pos_x = tank->x * 25 + 50;
			bullet->pos_y = tank->y * 25 + 23;
		}
		//获取子弹方向
		bullet->direction = tank->direction;
		//改变子弹状态
		bullet->status = !bullet->status;
	}
}

int bullet_action(Bullet* bullet,Tank* enemy_tank,int Is_Mytank)
{
	//子弹在地图中二维数组的坐标
	int x, y,x1,y1;
	x = bullet->pos_x / 25;
	y = bullet->pos_y / 25;

	//擦除上一次绘制的子弹
	setfillcolor(BLACK);
	solidrectangle(bullet->pos_x, bullet->pos_y, bullet->pos_x+3, bullet->pos_y+3);

	//根据方向计算子弹的坐标
	if (bullet->direction == UP)
	{
		bullet->pos_y -= 2;
		x1 = x + 1;
		y1 = y;
	}
	else if (bullet->direction == DOWN)
	{
		bullet->pos_y += 2;
		x1 = x + 1;
		y1 = y;
	}
	else if (bullet->direction == LEFT)
	{
		bullet->pos_x -= 2;
		x1 = x;
		y1 = y + 1;
	}
	else if (bullet->direction == RIGHT)
	{
		bullet->pos_x += 2;
		x1 = x;
		y1 = y + 1;
	}
	else
	{
		return 0;
	}
	if (bullet->pos_x < 0 || bullet->pos_x>650 || bullet->pos_y < 0 || bullet->pos_y>650)
	{
		bullet->status = 0;
		return 0;
	}
	
	//碰撞检查
	if (map[y][x] == 4 || map[y1][y1] == 4)
	{
		bullet->status = 0;
		PlaySound(_T("boom.wav"), NULL, SND_FILENAME | SND_ASYNC);
		return 1;
	}

	//击中我方坦克
	if (map[y][x] == 200 || map[y1][x1] == 200)
	{
		//判断是否是敌方坦克发射的子弹
		if(Is_Mytank==0)
		return 1;
	}

	//击中敌方坦克
	if ((map[y][x] >= 100 && map[y][x] <= 109) || (map[y1][x1] >= 100 && map[y1][x1] <= 109))
	{
		//判断是否是我方发射的子弹
		if (Is_Mytank == 1)
		{
			Tank* tank = NULL;
			bullet->status = 0;
			if (map[y][x] > 100 && map[y][x] <= 109)
				tank = enemy_tank + (map[y][x] - 100);
			else
				tank = enemy_tank + (map[y1][x1] - 100);

			tank->live = 0;
			set_map(tank->x, tank->y, 0);
			setfillcolor(BLACK);
			solidrectangle(tank->x * 25, tank->y * 25, tank->x * 25 + 50, tank->y * 25 + 50);
		}
	}


	//子弹击中可消除的墙
	if (map[y][x] == 1)
	{
		map[y][x] = 0;
		bullet->status = 0;
		setfillcolor(BLACK);
		solidrectangle(x * 25, y * 25, x * 25 + 25, y * 25 + 25);
	}
	else if(map[y][x]==2)//击中不可消除的墙
	{
		bullet->status = 0;
	}
	//子弹击中可消除的墙
	if (map[y1][x1] == 1)
	{
		map[y1][x1] = 0;
		bullet->status = 0;
		setfillcolor(BLACK);
		solidrectangle(x1 * 25, y1 * 25, x1 * 25 + 25, y1 * 25 + 25);
	}
	else if (map[y1][x1] == 2)//击中不可消除的墙
	{
		bullet->status = 0;
	}


	//重新绘制子弹
	if (bullet->status)
	{
		setfillcolor(WHITE);
		solidrectangle(bullet->pos_x, bullet->pos_y, bullet->pos_x + 3, bullet->pos_y + 3);
	}
	return 0;
}
```

#### 敌方坦克的实现


```C++
//设置敌方坦克出场位置
for (int i = 0;i < enemy_num;i++)
{
	if (i % 3 == 0)
	{
		enemy_tank[i].x = 0;
	}
	else if (i % 3 == 1)
	{
		enemy_tank[i].x = 12;
	}
	else if (i % 3 == 2)
	{
		enemy_tank[i].x = 24;
	}
	enemy_tank[i].y = 0;
	enemy_tank[i].live = 1;
	enemy_tank[i].direction = DOWN;
	enemy_tank[i].Is_Mycamp = 0;
	//set_map(enemy_tank[i].x, enemy_tank[i].y, 100 + i);
	enemy_bullet[i].status = 0;
}

enemy_total = 3;
//enemy_tank[0].x = 4;
//enemy_tank[0].y = 0;
//enemy_tank[0].live = 0;
//敌方坦克登场
for (int i = 0;i < enemy_total;i++)
{
	if (enemy_tank[i].live)
	{
		set_map(enemy_tank[i].x, enemy_tank[i].y, 100 + i);
		do_tank_walk(&enemy_tank[i], enemy_tank[i].direction, &img_enemytank[enemy_tank[i].direction], 0);
	}

```

#### 敌方坦克的路由控制

首先我们设置敌方坦克会不断移动，那么我们只需要控制他的方向，让他的移动方向面向目标，如果有障碍物，可以通过炮击清除障碍物

```c++
		//防止坦克出生重叠
		if (map[enemy_tank[enemy_total].x][enemy_tank[enemy_total].y] == 0)
		{
			if (times > 0 && times % 1000 == 0 && enemy_total < enemy_num)
			{
				set_map(enemy_tank[enemy_total].x, enemy_tank[enemy_total].y, 100 + enemy_total);
				enemy_total++;
			}
		}
		if (times>0 && times % 200 == 0)
		{
			for (int i = 0;i < enemy_total;i++)
			{
				if (!enemy_tank[i].live)
					continue;
				//攻击我方老巢
				if (i % 2 == 0)
				{
					DIRECTION d = enemy_direction(&enemy_tank[i], 12, 24);
					tank_walk(&enemy_tank[i], d, &img_enemytank[d]);
				}
				else
				{
					DIRECTION d = enemy_direction(&enemy_tank[i], my_tank.x, my_tank.y);
					tank_walk(&enemy_tank[i], d, &img_enemytank[d]);
				}
				tank_fire(&enemy_tank[i], &enemy_bullet[i]);
			}
			
		}
		
		//0.5s移动一次敌方坦克
		else if (times % 50 == 0)
		{
			for (int i = 0;i < enemy_total;i++)
			{
				if (enemy_tank[i].live)
				{
					tank_walk(&enemy_tank[i], enemy_tank[i].direction, &img_enemytank[enemy_tank[i].direction]);
				}
			}
		}
```

#### 游戏结束场景的实现

炮击函数中会返回一个游戏状态值，如果该状态值为真（1），则 game 函数就返回 0，如果敌方坦克全部歼灭，则返回 1，将该数值传入 gameover（）中，如果为 1 则显示胜利图像，如果为 0 则显示失败的图像，然后在 display 的循环中，如果为 1 则 stage+1，将该值传递给 game（）就可以进行下一关游戏

```C++

```