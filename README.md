# Maze
这是一个迷宫游戏，使用文件操作，深度优先算法和深度优先搜索进行生成和寻找出路。
#include <iostream>
using namespace std;
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<stdbool.h>
#include<cstdlib>
#include <ctime>    // 对于 time(NULL)
#define edgemax 25
#define number 10
#define mazewidth 21 // 生成迷宫宽度
#define mazelength 21 // 生成迷宫高度
int timeee;
int timeeee[number];
// 方向数组：上、下、左、右
int directions[4][2] = {{-2, 0},
                        {2, 0},
                        {0, -2},
                        {0, 2}};
// 迷宫最终找到的x,y坐标
int findx = -1;
int findy = -1;
// 已经存在的迷宫的所有数据
int existmapnumber;
int length[number];
int width[number];
int start_x[number],start_y[number];
int end_x[number],end_y[number];
int maze[number][edgemax][edgemax];
int Maze[mazewidth][mazelength];// 自己生成的迷宫
int ReamMazeData(FILE *file);
bool load_maze(FILE *file,int existlength,int existwidth,int existstart_x,int existstart_y,int existend_x,int existend_y, int existmaze[mazewidth][mazelength]);
void print_maze(int nownumber);
void print_maze1(int ma[mazelength][mazewidth]);
int is_valid(int x,int y);
void generate_maze(int maze[mazewidth][mazelength], int x, int y);
void find_maze(int Maze[mazelength][mazewidth], int x, int y);
int is_output(int Maze[mazelength][mazewidth],int x,int y);
int main() {
    string filename;
    cout << "Please enter the file : (example C:\\Users\\LENOVO\\CLionProjects\\test\\test.txt):";
    getline(cin, filename);
    int option = 1;
    cout<< "Please choose the option: (1) load the map and print the answer road  \n";
    cout<< "Please choose the option: (2) print a new map and print the answer \n";
    cout<< "Enter your choose: ";
    cin>>option;
    while(option!=1&&option!=2){
        cout<<"wrong input";
        cout<< "Please choose the option: (1) load the map and print the answer road  \n";
        cout<< "Please choose the option: (2) print a new map and print the answer \n";
        cout<< "Enter your choose: ";
        cin>>option;
    }
    FILE *file = fopen(filename.c_str(),"r");  // 只读
    existmapnumber = ReamMazeData(file);  // 将读入的地图数量参数导入到existmaonumber中
    FILE *file1 = fopen(filename.c_str(),"a"); // 只写
    // 文件打不开
    if (file == NULL) {
        printf("can not open the data file \n");
        return 1;
    }
    if(option == 1){
        cout<<"(1) load sussessfully and will find all the maze output\n";
        cout<<"choose a map you want to find , total member: "<<existmapnumber<<endl;
        cout<<"the map number: ";
        int i;
        cin>>i;
        if(i>existmapnumber){
            cout<<"choose again the map you want to find , total member: "<<existmapnumber<<endl;
            cin>>i;
        }
        i = i-1;
        print_maze(i);
        printf("_____________________________________________________\n");
        for(int j=0;j<length[i];j++){
            for(int k=0;k<width[i];k++){
                Maze[j][k] = maze[i][j][k];
            }
        }
        find_maze(Maze, 1, 0);
        cout<<"the only road of the maze:\n";
        print_maze1(Maze);
        printf("_____________________________________________________\n");
    }else{
        // 初始化迷宫
        for(int i = 0 ;i<mazelength;i++){
            for (int j = 0; j < mazewidth; j++) {
                Maze[i][j] = 1;
            }
        }
        // 设置迷宫出口和入口
        Maze[1][0]=0;
        Maze[mazelength-2][mazewidth-1]=0;
        // 从入口(1, 0)开始生成迷宫
        generate_maze(Maze, 1, 1);
        // 设置出口
        Maze[mazelength-2][mazewidth-1]=9;
        print_maze1(Maze);
        printf("_____________________________________________________\n");
        bool state = load_maze(file1,mazelength,mazewidth,1,1,mazelength-2,mazewidth-1,Maze);
        if(!state){
            printf("Wrong load\n");
        }
        // 寻找迷宫唯一路径
        find_maze(Maze,1,0);
        cout<<"the only road of the maze:\n";
        print_maze1(Maze);
        printf("_____________________________________________________\n");
    }
    return 0;
}
// 多文件读入文件信息函数,nownumber指定读入多少张地图
int ReamMazeData(FILE *file){
    static int existnumber = 0 ;
    // 读取长 宽 初始坐标 结尾坐标
    if (fscanf(file, "%d %d %d %d %d %d %d", &length[existnumber], &width[existnumber], &start_x[existnumber], &start_y[existnumber], &end_x[existnumber], &end_y[existnumber],&timeeee[existnumber]) != 7) {
        //printf("Read failed\n");
        return 0;  // 返回
    }
    if(length[existnumber]<end_x[existnumber]||width[existnumber]<end_y[existnumber]){
        //printf("Wrong data\n");
        return 0;  // 返回
    }
    // 读取迷宫地图数据（0 或 1）
    for (int i = 0; i < length[existnumber]; i++) {
        for (int j = 0; j < width[existnumber]; j++) {
            if (fscanf(file, "%1d", &maze[existnumber][i][j]) != 1) {
                //printf("Read failed\n");
                return 0;  // 返回
            }
        }
    }
    existnumber++;
    ReamMazeData(file);
    return existnumber;  // 成功
}
// 迷宫输入文件
bool load_maze(FILE *file,int existlength,int existwidth,int existstart_x,int existstart_y,int existend_x,int existend_y, int existmaze[mazelength][mazewidth]){
    for(int i =0;i<existmapnumber;i++){
        // 检查随机戳是否相同 如果相同则直接返回
        if(timeeee[i]==timeee) return false;
    }
    // 输入长 宽 初始坐标 结尾坐标
    fprintf(file, "%d %d %d %d %d %d %d\n",existlength,existwidth,existstart_x,existstart_y,existend_x,existend_y,timeee);
    for(int i = 0;i<existlength;i++){
        for(int j = 0;j<existwidth;j++){
            fprintf(file,"%d ",existmaze[i][j]);
        }
        fprintf(file, "\n");  // 每行迷宫结束后换行
    }
    //fprintf(file, "\n");
    return true;
}
// 指定地图打印迷宫地图（0和1）
void print_maze(int nownumber){
    printf("The map of the maze:\n");
    for (int i = 0; i < length[nownumber]; i++) {
        for (int j = 0; j < width[nownumber]; j++) {
            int data = maze[nownumber][i][j];
            if(data==1){
                printf("1 ");
                //printf("▇ ");
            }
            else {
                if(data==0) {
                    printf("0 ");
                    //printf("□ ");
                }
                else {
                    printf("8 ");
                    //printf("※ ");
                }
            }
        }
        printf("\n");
    }
}
// 生成迷宫打印
void print_maze1(int ma[mazelength][mazewidth]) {
    for (int i = 0; i < mazelength; i++) {
        for (int j = 0; j < mazelength; j++) {
            int data = ma[i][j];
            if(data==1){
                printf("1 ");
                //printf("▇ ");
            }
            else {
                if(data==0) {
                    printf("0 ");
                    //printf("□ ");
                }
                else {
                    printf("8 ");
                    //printf("※ ");
                }
            }
        }
        printf("\n");
    }
}
// 检查某个数是否在迷宫内部
int is_valid(int x,int y){
    return x>=0&&y>=0&&x<mazelength&&y<mazewidth;
}
// 深度优先搜索生成迷宫
void generate_maze(int Maze[mazelength][mazewidth], int x, int y){
    srand(time(nullptr));
    // 以当前时间为时间戳从而随机打乱方向
    timeee = rand();
    for (int i = 0; i < 4; i++) {
        int j = rand()%4;
        int temp[2] = {directions[i][0], directions[i][1]};
        directions[i][0] = directions[j][0];
        directions[i][1] = directions[j][1];
        directions[j][0] = temp[0];
        directions[j][1] = temp[1];
    }
    Maze[x][y] = 0;;// 标记当前位置为路径
    // 尝试四个方向
    for(int i = 0;i<4;i++){
        int nx = x + directions[i][0];
        int ny = y + directions[i][1];
        // 此时这个方向存在迷宫中且仍然是墙
        if(is_valid(nx,ny)&&Maze[nx][ny] == 1){
            Maze[x + directions[i][0] / 2][y + directions[i][1] / 2] = 0;
            // 发现有效的墙壁位置后，打破它以创建路径。
            // 这通过将 (x, y) 周围相邻的一半墙壁（/ 2）标记为路径实现。
            generate_maze(Maze, nx, ny);
        }
    }
}
// 出口判断
int is_output(int Maze[mazelength][mazewidth],int x,int y){
    return (Maze[x][y]==9);
}
// 深度优先搜索寻找路径
void find_maze(int Maze[mazelength][mazewidth], int x, int y){
    for(int i=0;i<4;i++){
        int nx = x + directions[i][0]/2;
        int ny = y + directions[i][1]/2;
        if(is_output(Maze,nx,ny)){
            findx = nx;
            findy = ny;
            return ;
        }
        if(is_valid(nx,ny)&&Maze[nx][ny] == 0){
            // 此时有路 用8标记走过
            Maze[nx][ny]=8;
            find_maze(Maze,nx,ny);
            // 如果找到出口，退出递归
            if (findx != -1 && findy != -1) {
                return;
            }
            // 如果没有找到出口，回溯，恢复当前位置
            Maze[nx][ny]=0;
        }
    }
}
