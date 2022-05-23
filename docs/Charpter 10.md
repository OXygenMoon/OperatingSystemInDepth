<p align='center'>
<a href="https://oxygenpanda.github.io/" target="_blank"><img alt="Website" src="https://img.shields.io/badge/博客-劳振煜的知識倉儲-faf2f2.svg?style=flat-square&logo=Blogger"></a>
<a href="https://www.github.com/OXygenPanda" target="_blank"><img src="https://img.shields.io/badge/Github-@劳振煜-f3e1e1.svg?style=flat-square&logo=GitHub"></a>
<a href="https://i.loli.net/2020/11/11/SBZ2mFJGKLjUtTO.jpg" target="_blank"><img src="https://img.shields.io/badge/微信-@OXygen-f1d1d1.svg?style=flat-square&logo=WeChat"></a>


# 深入理解操作系统 第十章

>   第十章的主要内容是：信号量和管程

## 信号量

信号量的抽象数据类型

-   一个整形(sem),具有两个原子操作
-   P(): sem减一,如果sem<0,等待,否则继续
-   V(): sem加一,如果sem≤0,唤醒一个等待的P

信号量是整数

信号量是被保护的变量

-   初始化完成后,唯一改变一个信号量的值的办法是通过P()和V()
-   操作必须是原子

P()能够阻塞,V()不会阻塞

我们假定信号量是公平的

-   没有线程被阻塞在P()仍然堵塞如果V()被无限频繁调用(在同一个信号量)
-   在实践中,FIFO经常被使用

两个类型信号量

-   二进制信号量: 可以是0或1
-   计数信号量: 可以取任何非负数
-   两者相互表现(给定一个可以实现另一个)

信号量可以用在2个方面

-   互斥
-   条件同步(调度约束——一个线程等待另一个线程的事情发生)

## 信号量使用

1.  用二进制信号量实现的互斥

    ```cpp
    mutex = new Semaphore(1);
    
    mutex->P();
    ...
    mutex->V();
    ```

2.  用二进制信号量实现的调度约束

    ```cpp
    condition = new Semaphore(0);
    
    //Thread A
    ...
    condition->P(); //等待线程B某一些指令完成之后再继续运行,在此阻塞
    ...
    
    //Thread B
    ...
    condition->V(); //信号量增加唤醒线程A
    ...
    ```

3.  一个线程等待另一个线程处理事情

    比如生产东西或消费东西(生产者消费者模式),互斥(锁机制)是不够的

    有界缓冲区的生产者-消费者问题

    -   一个或者多个生产者产生数据将数据放在一个缓冲区里
    -   单个消费者每次从缓冲区取出数据
    -   在任何一个时间只有一个生产者或消费者可以访问该缓冲区

    正确性要求

    -   在任何一个时间只能有一个线程操作缓冲区(互斥)
    -   当缓冲区为空时,消费者必须等待生产者(调度,同步约束)
    -   当缓存区满,生产者必须等待消费者(调度,同步约束)

    每个约束用一个单独的信号量

    -   二进制信号量互斥
    -   一般信号量 fullBuffers
    -   一般信号了 emptyBuffers

    ```cpp
    class BoundedBuffer{
    		mutex = new Semaphore(1);
    		fullBuffers = new Semaphore(0);   //说明缓冲区初始为空
     		emptyBuffers = new Semaphore(n);  //同时可以有n个生产者来生产
    };
    
    BoundedBuffer::Deposit(c){
    		emptyBuffers->P();
    		mutex->P();
    		Add c to the buffer;
    		mutex->V();
    		fullBuffers->V();
    }
    
    BoundedBuffer::Remove(c){
    		fullBuffers->P();
    		mutex->P();
    		Remove c from buffer;
    		mutex->V();
    		emptyBuffers->V();
    }
    ```

## 信号量实现

使用硬件原语

-   禁用中断
-   原子指令

类似锁

-   禁用中断

```cpp
class Semaphore{
		int sem;
		WaitQueue q;
};

Semaphore::P(){
		--sem;
		if(sem < 0){
				Add this thread t to q;
				block(p);
		}
};

Semaphore::V(){
		++sem;
		if(sem <= 0){
				Remove a thread t from q;
				wakeup(t);
		}
}
```

信号量的双用途

-   互斥和条件同步
-   但等待条件是独立的互斥

读,开发代码比较困难

-   程序员必须非常精通信号量

容易出错

-   使用的信号量已经被另一个线程占用
-   忘记释放信号量

不能够处理死锁问题

## 管程

目的: 分离互斥和条件同步的关注

什么是管程

-   一个锁: 指定临界区
-   0或者多个条件变量: 等待,通知信号量用于管程并发访问共享数据

一般方法

-   收集在对象,模块中的相关共享数据
-   定义方法来访问共享数据

Lock

-   Lock::Acquire() 等待直到锁可用,然后抢占锁
-   Lock::Release() 释放锁,唤醒等待者如果有

Condition Variable

-   允许等待状态进入临界区
    -   允许处于等待(睡眠)的线程进入临界区
    -   某个时刻原子释放锁进入睡眠
-   Wait() operation
    -   释放锁,睡眠,重新获得锁放回
-   Signal() operation(or broadcast() operation)
    -   唤醒等待者(或者所有等待者),如果有

实现

-   需要维持每个条件队列
-   线程等待的条件等待signal()

```cpp
class Condition{
		int numWaiting = 0;
		WaitQueue q;
};

Condition::Wait(lock){
		numWaiting++;
		Add this thread t to q;
		release(lock);
		schedule(); //need mutex
		require(lock);
}

Condition::Signal(){
		if(numWaiting > 0){
				Remove a thread t from q;
				wakeup(t); //need mutex
				numWaiting--;
		}
}
```

管程解决生产者-消费者问题

```cpp
class BoundedBuffer{
		Lock lock;
		int count = 0;  //buffer 为空
		Condition notFull, notEmpty;
};

BoundedBuffer::Deposit(c){
		lock->Acquire();    //管程的定义:只有一个线程能够进入管程
		while(count == n)
				notFull.Wait(&lock); //释放前面的锁
		Add c to the buffer;
		count++;
		notEmpty.Signal();
		lock->Release();
}

BoundedBuffer::Remove(c){
		lock->Acquire();
		while(count == 0)
				notEmpty.Wait(&lock);
		Remove c from buffer;
		count--;
		notFull.Signal();
		lock->Release();
}
```

开发,调试并行程序很难

-   非确定性的交叉指令

同步结构

-   锁: 互斥
-   条件变量: 有条件的同步
-   其他原语: 信号量

怎么样有效地使用这些结构

-   制定并遵循严格的程序设计风格,策略

## 经典同步问题

1.  读者-写者问题

    动机: 共享数据的访问

    两种类型的使用者: 读者(不修改数据) 写者(读取和修改数据)

    问题的约束:

    -   允许同一时间有多个读者,但在任何时候只有一个写者
    -   当没有写者时,读者才能访问数据
    -   当没有读者和写者时,写者才能访问数据
    -   在任何时候只能有一个线程可以操作共享变量

    多个并发进程的数据集共享

    -   读者: 只读数据集;他们不执行任何更新
    -   写者: 可以读取和写入

    共享数据

    -   数据集
    -   信号量CountMutex初始化为1
    -   信号量WriteMutex初始化为1
    -   整数Rcount初始化为0(当前读者个数)

    读者优先设计

    只要有一个读者处于活动状态, 后来的读者都会被接纳.如果读者源源不断的出现,那么写者使用处于阻塞状态.

    ```cpp
    //信号量实现
    //writer
    sem_wait(WriteMutex);
    write;
    sem_post(WriteMutex);
    
    //reader
    sem_wait(CountMutex);
    if(Rcount == 0)
    		sem_wait(WriteMutex); //确保后续不会有写者进入
    ++Rcount;
    read;
    --Rcount;
    if(Rcount == 0)
    		sem_post(WriteMutex); //全部读者全部离开才能唤醒写者
    sem_post(CountMutex);
    ```

    写者优先设计

    一旦写者就绪,那么写者会尽可能的执行写操作.如果写者源源不断的出现的话,那么读者就始终处于阻塞状态.

    ```cpp
    //writer
    Database::Write(){
    		Wait until readers/writers;
    		write database;
    		check out - wake up waiting readers/writers;
    }
    //reader
    Database::Read(){
    		Wait until no writers;
    		read database;
    		check out - wake up waiting writers;
    }
    
    //管程实现
    AR = 0; // # of active readers
    AW = 0; // # of active writers
    WR = 0; // # of waiting readers
    WW = 0; // # of waiting writers
    Condition okToRead;
    Condition okToWrite;
    Lock lock;
    //writer
    Public Database::Write(){
    		//Wait until no readers/writers;
    		StartWrite();
    		write database;
    		//check out - wake up waiting readers/writers;
    		DoneWrite();
    }
    
    Private Database::StartWrite(){
    		lock.Acquire();
    		while((AW + AR) > 0){
    				WW++;
    				okToWrite.wait(&lock);
    				WW--;		
    		}
    		AW++;
    		lock.Release();
    }
    
    Private Database::DoneWrite(){
    		lock.Acquire();
    		AW--;
    		if(WW > 0){
    				okToWrite.signal();
    		}
    		else if(WR > 0){
    				okToRead.broadcast(); //唤醒所有reader 
    		}
    		lock.Release();
    }
    
    //reader
    Public Database::Read(){
    		//Wait until no writers;
    		StartRead();
    		read database;
    		//check out - wake up waiting writers;
    		DoneRead();
    }
    
    Private Database::StartRead(){
    		lock.Acquire();
    		while(AW + WW > 0){    //关注等待的writer,体现出写者优先
    				WR++;
    				okToRead.wait(&lock);
    				WR--;
    		}
    		AR++;
    		lock.Release();
    }
    
    private Database::DoneRead(){
    		lock.Acquire();
    		AR--;
    		if(AR == 0 && WW > 0){  //只有读者全部没有了,才需要唤醒
    				okToWrite.signal();
    		}
    		lock.Release();
    }
    ```

2.  哲学家就餐问题(学习自 [github.com/cyc2018](http://github.com/cyc2018))

    共享数据:

    -   Bowl of rice(data set)
    -   Semaphone fork [5] initialized to 1

    ```cpp
    #define N 5
    #define LEFT (i + N - 1) % N // 左邻居
    #define RIGHT (i + 1) % N    // 右邻居
    #define THINKING 0
    #define HUNGRY   1
    #define EATING   2
    typedef int semaphore;
    int state[N];                // 跟踪每个哲学家的状态
    semaphore mutex = 1;         // 临界区的互斥，临界区是 state 数组，对其修改需要互斥
    semaphore s[N];              // 每个哲学家一个信号量
    
    void philosopher(int i) {
        while(TRUE) {
            think(i);
            take_two(i);
            eat(i);
            put_two(i);
        }
    }
    
    void take_two(int i) {
        down(&mutex);
        state[i] = HUNGRY;
        check(i);
        up(&mutex);
        down(&s[i]); // 只有收到通知之后才可以开始吃，否则会一直等下去
    }
    
    void put_two(i) {
        down(&mutex);
        state[i] = THINKING;
        check(LEFT); // 尝试通知左右邻居，自己吃完了，你们可以开始吃了
        check(RIGHT);
        up(&mutex);
    }
    
    void eat(int i) {
        down(&mutex);
        state[i] = EATING;
        up(&mutex);
    }
    
    // 检查两个邻居是否都没有用餐，如果是的话，就 up(&s[i])，使得 down(&s[i]) 能够得到通知并继续执行
    void check(i) {         
        if(state[i] == HUNGRY && state[LEFT] != EATING && state[RIGHT] !=EATING) {
            state[i] = EATING;
            up(&s[i]);
        }
    }
    ```

