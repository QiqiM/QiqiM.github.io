---
layout: _post
title: C++实现双向链表
date: 2019-6-25 10:43:13
tags: [algorithm, C++,LinkList]
---

　　双向链表(双链表)是链表的一种。和单链表一样，双链表也是由节点组成，它的每个数据结点中都有两个指针，分别指向直接后继和直接前驱。所以，从双向链表中的任意一个结点开始，都可以很方便地访问它的前驱结点和后继结点。一般我们都构造双向循环链表。
<!--more-->

### C++实现双向链表

双向链表头文件(LinkList.h)
```C++
#pragma once
#ifndef _LINK_LIST_H
#define _LINK_LIST_H

#include <iostream>
using namespace std;

template<class T>
struct DNode
{
	public:
		T value;
		DNode *prev;
		DNode *next;

	public:
		DNode() {}
		DNode(T t, DNode *prev, DNode *next) {
			this->value = t;
			this->prev = prev;
			this->next = next;
		}
};

template<class T>
class DoubleLink {
	public:
		DoubleLink();
		~DoubleLink();

		int size();
		int isEmpty();

		T get(int index);
		T getFirst();
		T getLast();

		int insert(int index, T t);
		int insertFirst(T t);
		int appendLast(T t);

		int del(int index);
		int delFirst();
		int delLast();
	private:
		int count;
		DNode<T> *pHead;
	private:
		DNode<T> *getNode(int index);
};

template<class T>
DoubleLink<T>::DoubleLink() : count(0)
{
	// 创建表头。注意：表头没有存储数据
	pHead = new DNode<T>();
	pHead->next = NULL;
}

// 析构函数
template<class T>
DoubleLink<T>::~DoubleLink()
{
	// 删除所有结点
	DNode<T>* pTemp;
	DNode<T>* pNode = pHead->next;
	while (pNode != pHead) 
	{
		pTemp = pNode;
		pNode = pNode->next;
		delete pTemp;
	}

	// 删除表头
	delete pHead;
	pHead = NULL;

}

// 返回结点数目
template<class T>
int DoubleLink<T>::isEmpty()
{
	return count == 0;
}

// 返回节点数目
template < class T>
int DoubleLink<T>::size()
{   
	return count;
}

// 获取index位置的结点
template<class T>
DNode<T>* DoubleLink<T>::getNode(int index)
{
	// 判断参数有效性
	if (index < 0 || index >= count)
	{
		cout << " getNode failed! the index is out of round" << endl;
		return NULL;
	}

	//  正向查找， 查找优化，减少查找次数
	if (index <= count / 2) 
	{
		int i = 0;
		DNode<T>* pIndex = pHead->next;
		while (i++ < index)
		{
			pIndex = pIndex->next;
		}

		return pIndex;
	}
	 
	// 反向查找
	int j = 0;
	int rIndex = count - index - 1;
	DNode<T>* pRindex = pHead->prev;
	while (j++ < rIndex)
	{
		pRindex = pRindex->prev;
	}

	return pRindex;
}

// 获取第index位置的结点的值
template<class T>
T DoubleLink<T>::get(int index)
{
	return getNode(index)->value;
}

// 获取第一个结点的值
template<class T>
T DoubleLink<T>::getLast()
{
	return getNode(count-1)->value;
}

// 获取最后一个结点的值
template<class T>
T DoubleLink<T>::getFirst()
{
	return getNode(0)->value;
}

// 将结点插入到第index位置之前
template<class T>
int DoubleLink<T>::insert(int index, T t)
{
	if (index == 0)
		return insertFirst(t);

	DNode<T>* pIndex = getNode(index);
	DNode<T>* pNode = new DNode<T>(t, pIndex->prev, pIndex);
	pIndex->prev->next = pNode;
	pIndex->prev = pNode;
	count++;

	return 0;
}

// 将结点插入到第一个结点处
template<class T>
int DoubleLink<T>::insertFirst(T t)
{
	DNode<T>* pNode = new DNode<T>(t, pHead, pHead->next);   // 构造函数时就已经指定结点的前驱和后继结点了

	// 这里第一个结点的时候需要判断空指针
	if (pHead->next == NULL)
		pHead->prev = pNode;
	else
		pHead->next->prev = pNode;
	
	pHead->next = pNode;
	count++;

	return 0;
}

// 将结点追加到链表的末尾
template<class T>
int DoubleLink<T>::appendLast(T t)
{
	DNode<T>* pNode = new DNode<T>(t, pHead->prev, pHead);
	pHead->prev->next = pNode;
	pHead->prev = pNode;
	count++;
	return 0;
}

// 删除index位置的结点
template<class T>
int DoubleLink<T>::del(int index)
{
	DNode<T>* pIndex = getNode(index);
	pIndex->next->prev = pIndex->prev;
	pIndex->prev->next = pIndex->next;
	delete pIndex;
	count--;
	
	return 0;
}

// 删除第一个结点
template<class T>
int DoubleLink<T>::delFirst()
{
	return del(0);
}

// 删除最后一个结点
template<class T>
int DoubleLink<T>::delLast()
{
	return del(count - 1);
}
 
#endif // !_LINK_LIST_H
```
#### 双向链表测试文件(LinkList.cpp)
```C++
#include "pch.h"
#include <iostream>
#include <string>
#include "LinkList.h"

using namespace std;

// 双向链表操作int数据
void intTest(){
	int intArr[4] = { 10,20,30,40 };

	cout << "\n -------------intTest------------" << endl;

	// 创建双向链表
	DoubleLink<int>* pDlinkList = new DoubleLink<int>();

	pDlinkList->insert(0, 20);
	pDlinkList->appendLast(10);
	pDlinkList->insertFirst(30);
	pDlinkList->insert(1, 40);
	pDlinkList->delFirst();

	// 双向链表是否为空
	cout << "isEmpty= " << pDlinkList->isEmpty() << endl;

	// 双向链表的长度
	cout << "size=" << pDlinkList->size() << endl;

	// 打印双向链表的全部数据
	int length = pDlinkList->size();
	for (int i = 0; i < length; i++)
	{
		cout << "pDlinkList(" << i << ")=" << pDlinkList->get(i) << endl;
	}
}

void stringTest()
{
	string sArr[4] = { "ten","tewnty","thirty","forty" };

	cout << "\n------------stringTest-----------" << endl;

	DoubleLink<string>* pDlinkList = new DoubleLink<string>();

	pDlinkList->insert(0, sArr[1]);
	pDlinkList->appendLast(sArr[0]);
	pDlinkList->insertFirst(sArr[2]);


	// 双向链表是否为空
	cout << "isEmpty()=" << pDlinkList->isEmpty() << endl;

	// 双向链表的长度
	cout << "size=" << pDlinkList->size() << endl;

	// 打印双向链表的全部数据
	int length = pDlinkList->size();
	for (int i = 0; i < length; i++)
	{
		cout << "pDlinkList(" << i << ")=" << pDlinkList->get(i) << endl;
	}
}

struct stu
{
	int id;
	char name[20];
};

static stu stuArr[] = 
{ 
	{10,"one"},
	{20,"two"},
	{30,"three"},
	{40,"four"}

};

#define ARR_STU_SIZE ((sizeof(stuArr)) / (sizeof(stuArr[0])))

void objectTest()
{
	cout << "\n------------stringTest-----------" << endl;

	DoubleLink<stu>* pDlinkList = new DoubleLink<stu>();

	pDlinkList->insert(0, stuArr[1]);
	pDlinkList->appendLast(stuArr[0]);
	pDlinkList->insertFirst(stuArr[2]);

	// 双向链表是否为空
	cout << "isEmpty()=" << pDlinkList->isEmpty() << endl;

	// 双向链表的长度
	cout << "size=" << pDlinkList->size() << endl;

	// 打印双向链表的全部数据
	struct stu p;
	int length = pDlinkList->size();
	for (int i = 0; i < length; i++)
	{
		p = pDlinkList->get(i);
		cout << "pDlinkList(" << i << ")=[" << p.id <<" ," << p.name<<"]" << endl;
	}
}


int main()
{	
	intTest();
	stringTest();
	objectTest();
	return 0;
}

```

　　此文实现[参考](https://www.cnblogs.com/skywang12345/p/3561803.html),侵权必删，大佬的有一些错误，经过测试已经修改。