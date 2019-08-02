---
layout : post
title :  数据结构（二）：线性表
date : 2019-08-01
categories: 数据结构 线性表
permalink : DataStructure/LinearTable.html
#description: 数据结构的基本概念啊

---

线性表的概念：由n(n>=0)个数据元素（结点）a<sub>1</sub>,a<sub>2</sub>,a<sub>3</sub>....a<sub>n</sub>组成的有限序列，记作(a<sub>1</sub>,a<sub>2</sub>,a<sub>3</sub>....a<sub>n</sub>)。


线性表的逻辑特征：内部结点有且仅有一个直接前趋和一个直接后继。

线性表的基本运算：
  + InitList(L) 表的初始化
  + ListLength(L) 表的长度
  + GetNode(L, i) 求表L中第i个结点
  + LocateNode(L, x) 查找值为x的结点
  + InsertList(L, x, i) 插入值为x的结点i
  + DeleteList(L, i) 删除结点i

线性表按存储结构分为顺序存储结构（顺序表）和链式存储结构（链表）
	
 1. **顺序表**

    1. 顺序表的定义

       ​	把结点按逻辑次序依次存放在一组地址连续的存储单元里。    

       ​	表中结点a<sub>i</sub>的地址为：  LOC(a<sub>i</sub>) = LOC(a<sub>1</sub>) + (i-1) * c  1<=i<=n。c为单结点所占的存储单元。
    
    2. 顺序表的描述
    
       ​	顺序表的C语言描述为：
    
       ```c
       #define ListSize 100
       typedef int DataType;
       typedef struct{
         
         DataType  data[ListSize];
         int length;
         
       } Seqlist;
       ```
    
    3. 顺序表的基本运算
    
       1. 插入
    
          在表的第i个位置，插入一个新结点x，表长度+1。
    
          ```c
          void InsertList(SeqList *L, DataType x, int i){
            
            int j;
            if (i < 1 || i > L->length+1) {
              Error("Position error");
            }
            
            if ( L->length >= ListSize ) {
              Error("overflow");
            }
            
            for(j = L->length-1; j>=i-1; j--){
              
              L->data[j+1] = L->data[j];//最后一个元素开始往后移动一位，直至移动到i-1
              
            }
      
            L->data[i-1] = x;
         L->length++;
            
          }
          ```
          
          
       
       2. 删除
       
          将表的第i个结点删除，使长度-1。
       
          ```c
          void DeleteList(seqList *L, int i){
            
            	
            
            
            
            
            
            
          }
          ```
       
          


 2. **链表** 

    



