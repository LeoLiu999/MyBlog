---
layout : post
title :  数据结构：线性表
date : 2019-08-01
categories: 数据结构 线性表
permalink : DataStructure/LinearTable.html
#description: 数据结构的基本概念

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
            
            	int j;
            	if (i<1 || i> L-length) {
                
                Error("Position error");
                
              }
            
            	for(j=i;i<=L-length-1;j++){
                	
                L->Data[j-1] = L->Data[j];
                
              }
            	
            	L->length--;
           
          }
          ```
          
       
       ##### 顺序表中插入与删除操作都是通过移动结点实现的,插入操作时从最后开始后移一位，删除时从元素后一个开始前移一位。


 2. **链表** 

     1. **单链表**
     
        逻辑特征：用任意的存储单元存放，结点的逻辑次序和物理次序不一定相同。
     
        单链表的C语言描述
     
        ```c
        typedef char DataType;
        typedef struct node{
          
          DataType data;
          struct node *next;

        }ListNode;
        typedef ListNode *LinkList;
        ListNode *p;
        LinkList head;
        
        ```
     
        注意：区分指针变量和结点变量这两个概念。
     
        动态变量是通过标准函数生成的，即：
     
        ```
        p = (ListNode *)malloc(sizeof(ListNode));
        ```
     
        又可通过free(p)来释放p所指结点空间。
     
        ***单链表的几种运算***
     
        （1）建立单链表
     
        ​		头插法建表
     
        ```c
        LinkList CreateListF(void){
          	
          	char ch;
          	LinkList head;
          	ListNode *s;
          	
          	head = NULL;
          	
          	
          	while( (ch=getchar() ) != '\n'){
              
              s = (ListNode *)malloc(sizeof(ListNode));
              s->data = ch;
              s->next = head;
              head = s;
              
            }
          	return head;
        }
        ```
     
        ​		尾插法建表
     
        ```c
        LinkList CreateListR(void){
          
          char ch;
          LinkList head;
          ListNode *s, *r;
          
          head = NULL; r = NULL;
          
          while( (ch = getchar()) != '\n' ) {
            
            s = (ListNode *)malloc(sizeof(ListNode));
            s->data = ch;
            if(head == NULL){
              head = s;//头结点进list
            }else{
              r->next = s;//上一个尾结点链上新生成的结点
            }
            r = s;//新生成的结点成为尾结点  
          }
          if( r != NULL ) {
            r->next =NULL;//尾结点赋值
          }
          return head;
        }
        ```
     
        ​		带头结点的尾插法建表（为了简化边界条件，一般都附加一个头结点，此时上述算法可简化为：）
     
        ```c
        LinkList CreateListRHaveHead(void){
          
          char ch;
          LinkList head = (LinkList)malloc(sizeof(ListNode));
          ListNode *s, *r;
          r = head;
          while ( (ch=getchar()) != '\n' ){
            s = (ListNode *)malloc(sizeof(ListNode));
            s->data = ch;
            r->next = s;//上一个尾结点链接上新生成的结点 
            r = s;//新生成的结点成为尾结点
          }
        	r->next = null;
          return head;
        }
        ```
     
        ​	(2) 查找运算
     
        ​		按序号查找
     
        ```c
        ListNode *GetNode(LinkList head, int i){
          int j;
          ListNode *p;
          p=head; j=0;
          while(p->next&&j<i){
            p=p->next;
            j++;
          }
          
          if(i==j){
            return p;
          }else{
            return NULL;
          }
          
        }
        ```
     
        ​		按值查找
     
        ```c
        ListNode *LocateNode(LinkList head, DataType key){
          
          ListNode *p = head->next;
          while(p&&p->data != key){
            p=p->next;
          }
          return p;
        }
        ```
     
        ​	(3) 插入操作
     
        ```c
        void InsertList(LinkList head, DataType x, int i){
          
          ListNode *p;
          p = GetNode(head, i-1);//取上一个位置的结点出来操作
          if (p==NULL){
            Error('position error');
          }
          
          s = (ListNode *)malloc(sizeof(ListNode));
          s->data = x;
          s->next = p->next;
          p->next = s;
        
        }
        ```
     
        ​	(4) 删除运算
     
        ```c
        void DeleteList(LinkList head, int i){
          
          ListNode *p, *r;
          p = GetNode(head, i-1); //取上一个位置的结点出来操作
          if(p==NULL || p->next == NULL){ //上一个位置和当前位置不能为空 
            Error('position error');
          }
          
          r = p->next;
          p->next = r->next;
          free(r);
          
        }
        ```
     
        ***链表上实现的插入和删除运算，无需移动结点，仅需修改指针***
       
    2. **单向循环链表**
    	
       概念：一种首尾相连的链表，形成一个回路。即尾指针指向头指针（初始化时将尾指针指向头指针）。
            设尾指针rear，开始结点a<sub>1</sub>的位置为rear->next->next，终端结点a<sub>n</sub>的位置为rear。
       
       例 
       
       链接两个单链表，算法如下:
       
       ```c
       LinkList Connect(LinkList A, LinkList B){
         
         LinkList p = A->next;
         A->next = B->next->next;
         free(B->next);
         B->next = p;
         return B;
         
       }
       ```
       
    3. **双链表** 
      
        概念：一种有前趋结点和后继结点的链表。前趋指向前一个结点，后继指向后一个结点。
        
        ```
        typedef struct dlistnode{
        	
        	DataType data;
          struct dlistnode *prior, *next;
                          
        }DListNode;
        
        typedef dlistnode *DlinkList;
        DlinkList head;
        ```
        
        双链表的运算
        
        (1) 前插操作
        
        ```c
        void DInsertBefore(DListNode *p, DataType x){
        	DListNode *s = malloc(sizeof(DListNode));
        	s->data = x;
        	s->prior = p->prior;
        	s->next = p;
        	
        	p->prior->next = s;
        	p->prior = s;
        
        }
        ```
        
        (2) 删除
        
        ```c
        void DDeleteNode(DListNode *p){
          
          p->prior->next = p->next;
          p->next->prior = p->prior;
          free(p);
          
        }
        ```
        
        两种算法的时间复杂度均为O（1）。
    
    #####顺序表和链表的比较
    
    1. **基于空间的考虑**
    
       ​	存储密度:是指结点数据本身所占的存储量和整个结点结构所占的存储量之比。
    
       ​	存储密度越大，存储空间的利用率就越高。
    
       ​	**当线性表的长度变化较大，难以估计其存储规模时，以采用动态链表作为存储结构为好。**
    
       ​	**当线性表的长度变化不大，容易事先确定其大小时，为了节约存储空间，宜采用顺序表作为存储结构。**
    
    2. **基于时间的考虑**
    
       ​	若线性表的操作主要是进行进行查找，采用顺序表做存储结构为宜。
    
       ​	对于频繁进行插入和删除的线性表，采用链表作为存储结构。
    
    3. **例子**
    
       1. 从顺序表中删除具有最小值的元素并由函数返回，空出的位置由最后一个元素填补，若线性表为空，则显示错误信息并退出。
    
          ```c
          DataType DMValue(SeqList *L)
          {
            DataType x, y;
            int i, k;
            
            if ( ListEmpty(L) ) {
              printf("list is empty! \n");
              return;
            }
            
            x = L->data[0];
            k = 0;
            
            for ( i = 1; i < L->length; i++ ) {
              y	=	L->data[i];
              if (y	<	x) {
                x	=	y; 
                k = i;
              }
              
            }
            L->data[k] = L->data[L->length-1];
            L->length--;
            return x;
          }
          ```
    
       2. 从顺序表里删除具有给定值x的所有元素
    
          ```c
          void delelte(SeqList *L, DataType x)
          {
            
            int i=0,j;
            while(i < L->length ){
              
              if( L->data[i]  == x) {
                
                
                	for(j=i+1;j<L->length;j++){
                    
                    L->data[j-1] = L->data[j]; 
                  }
                	
                	L->length--;
              }
              
              i++;
              
            }
            
          }
          ```
    
       3. 一个单链表（数据域可能相同），其头指针为head，编写一个函数计算数据域为x的结点个数。
    
          ```c
          int count(head, x) 
            linklist head;
            Datatype x;
          {
            
            listnode *p;
            
            
            
            
          }
          
          
          ```
    
          
    
    
    
    
    
    ​    
    
    ​    
    
    ​    



