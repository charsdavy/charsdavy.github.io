---
layout: post
title:  "均匀分布随机函数的实现"
date:   2016-05-18 18:20:58 +0800
categories: jekyll update
tags: [算法]
---

#前言
随机函数就是产生数的函数，C语言里使用rand(),srand()等随机函数实现随机数生成。
 
#函数简介
```int rand( void );```
返回的是一个界于0～32767（0x7FFF）之间的伪随机数，包括0和32767。
C预先生成一组随机数，每次调用随机函数时从指针所指向的位置开始取值，因此使用rand()重复运行程序产生的随机数都是相同的，可以通过srand()函数来改变指针位置。
srand()会设置供rand()使用的随机数种子。如果在第一次使用rand()之前没有调用srand()，那么系统会自动调用srand()。而使用同种子相同的数调用 rand()会导致相同的随机数序列被生成。
 
```void srand( unsigned int seed );```
改变随机数表的指针位置（用seed变量控制）。
使用系统定时/计数器的值作为随机种子。每个种子对应一组根据算法预先生成的随机数，所以，在相同的平台环境下，不同时间产生的随机数会是不同的，相应的，若将srand（unsigned）time(NULL)改为srand(TP)（TP为任一常量），则无论何时运行、运行多少次得到的“随机数”都会是一组固定的序列，因此srand生成的随机数是伪随机数。
一般配合time(NULL)使用，因为时间每时每刻都在改变，产生的seed值都不同。
 
#场景
使用rand函数生成的随机数严格满足正态分布。而在很多时候，我们希望随机数的生成不要满足正态分布，特别是在处理网络通信报文的时候。
例如，我们需要在交换机处理到海量报文时，能够使远端的从设备尽可能的分段同时向局端回应报文，以减轻局部报文处理压力。
 
#均匀分布随机函数实现
##开发环境
![](http://upload-images.jianshu.io/upload_images/1492739-b2f082073e33ca94.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##实现步骤
1）打开Qt Creater，创建GUI工程
![](http://upload-images.jianshu.io/upload_images/1492739-1c06b87e7e219e93.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
![](http://upload-images.jianshu.io/upload_images/1492739-4b288fe958b3e962.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2）在mainwindow.h中添加函数声明
```void paintEvent(QPaintEvent *);```

 
3）在mainwindow.cpp中添加函数实现
导入头文件
```#include <QPainter>```

 
实现void paintEvent(QPaintEvent *)函数
```
/*
 *Qt中函数paintEvent(QPaintEvent*)是被系统自动调用。
 *paintEvent(QPaintEvent *)函数是QWidget类中的虚函数，用于ui的绘制，会在多种情况下被其他函数自动调用。
*/
void MainWindow::paintEvent(QPaintEvent *)
{
    QPainter painter(this);
    QPen pen; //画笔
    pen.setColor(QColor(255,0,0)); //设置画笔颜色
    painter.setPen(pen); //添加画笔

    long int r[kSum] = {0};
    int i = 0;
    int j = 0;

    do{
        r[i] = Uniform(0, 300);
        i++;
    }while(i < kSum);

    while((j + 30) < (kSum + 30)){
        painter.drawPoint(j, r[j]);
        j++;
    }
}
```

 
4）添加随机函数实现代码
```
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define kSum 1000
//算法一
/*
 *均匀分布随机函数均匀化
*/
double _uniform(double min, double max, long int *seed) {
    double t = 0;
    *seed = 2045 * (*seed) + 1;
    *seed = *seed - (*seed / 1048576) * 1048576;
    t = (*seed) / 1048576.0;
    t = min + (max - min) * t;
    return t;
}

/*
 *均匀分布随机函数产生随机数
*/
long int Uniform(double min, double max) {
    long int s = 0;
    double r = 0;

    //srand((unsigned int)time(NULL)); /*同一个时间种子可能会从产生相同的随机数列*/
    s = rand();
    r = _uniform(min, max, &s);

    return ((long int)r);
}

//算法二
double AverageRandom(double min, double max) {
    int minInteger = (int)(min * 10000);
    int maxInteger = (int)(max * 10000);
    int randInteger = rand() * rand();
    int diffInteger = maxInteger - minInteger;
    int resultInteger = randInteger % diffInteger + minInteger;

    return (resultInteger/10000.0);
}
```

 
#实现效果
![](http://upload-images.jianshu.io/upload_images/1492739-af95ccbacb9b3231.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
#小结
从图中可以看出，使用上述函数生成的随机数符合均匀分布。
本案例主要使用了Qt的绘图功能，用来直观展示生成随机数的效果。检验随机函数生成随机数的效果。
 
#附录
最后附上该算法实现的全部代码：
```
//mainwindow.h
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>

namespace Ui {
class MainWindow;
}

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = 0);
    ~MainWindow();
    void paintEvent(QPaintEvent *);

private:
    Ui::MainWindow *ui;
};

#endif // MAINWINDOW_H
```

 
```
//mainwindow.cpp
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <QPainter>

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define kSum 1000

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

MainWindow::~MainWindow()
{
    delete ui;
}

/*
 *均匀分布随机函数均匀化
*/
double _uniform(double min, double max, long int *seed) {
    double t = 0;
    *seed = 2045 * (*seed) + 1;
    *seed = *seed - (*seed / 1048576) * 1048576;
    t = (*seed) / 1048576.0;
    t = min + (max - min) * t;
    return t;
}

/*
 *均匀分布随机函数产生随机数
*/
long int Uniform(double min, double max) {
    long int s = 0;
    double r = 0;

    //srand((unsigned int)time(NULL)); /*同一个时间种子可能会从产生相同的随机数列*/
    s = rand();
    r = _uniform(min, max, &s);

    return ((long int)r);
}

/*
 *Qt中函数paintEvent(QPaintEvent*)是被系统自动调用。
 *paintEvent(QPaintEvent *)函数是QWidget类中的虚函数，用于ui的绘制，会在多种情况下被其他函数自动调用。
*/
void MainWindow::paintEvent(QPaintEvent *)
{
    QPainter painter(this);
    QPen pen; //画笔
    pen.setColor(QColor(255,0,0)); //设置画笔颜色
    painter.setPen(pen); //添加画笔

    long int r[kSum] = {0};
    int i = 0;
    int j = 0;

    do{
        r[i] = Uniform(0, 300);
        i++;
    }while(i < kSum);

    while((j + 30) < (kSum + 30)){
        painter.drawPoint(j, r[j]);
        j++;
    }
}
```

 
```
//main.cpp
#include "mainwindow.h"
#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    MainWindow w;
    w.show();

    return a.exec();
}
```
