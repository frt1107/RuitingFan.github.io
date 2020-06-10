---
layout: post
title:  "Java多线程入门"
date:   2020-06-10 12:28:01 +0800
categories: Java多线程
tag: Java多线程
---


* content
{:toc}


# Java多线程入门

**功能描述**：

四个线程t1,t2,t3,t4,向4个文件中写入数据，t1只能写入1，t2只能写入2，t3只能写入3，t4只能写入4，对4个文件A，B，C，D写入如下内容:

 * A:123412341234.....
 * B:234123412341....
 * C:341234123412....
 * D:412341234123....

怎么实现同步可以让线程并行工作？

**代码实现思路：**

***悲观锁***适用于实现此类写操作线程同步问题，因此使用`ReentrantLock`实现：写操作前加锁，写完后释放锁。

4个`Thread_ti`线程类均继承`Thread`类，通过重写`run()`方法实现写文件操作。

`File_X`线程类通过实现`Runnable`接口，并实现其中的`run()`方法实现按题述要求分别写入A、B、C、D四个文件的操作。即：

对A文件：写入次序为1234

对B文件：写入次序为2341

...

通过线程的`start()`方法运行即可。

**拓展说明：**

- 悲观锁：适合写操作多的场景，先加锁可以保证写操作时数据正确。如sychronized、ReentrantLock。
- 乐观锁：适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅提升。如AtomicInteger。

```java
package threads;

import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 四个线程t1,t2,t3,t4,向4个文件中写入数据，t1只能写入1，t2只能写入2，t3只能写入3，t4只能写入4，对4个文件A，B，C，D写入如下内容:
 * A:123412341234.....
 * B:234123412341....
 * C:341234123412....
 * D:412341234123....
 * 怎么实现同步可以让线程并行工作？
 *
 */
public class Hello {
    static boolean b = false;
    static ReentrantLock lock = new ReentrantLock();
    static ReentrantLock lock_2 = new ReentrantLock();
    static Condition condition_for = lock.newCondition();
    
    // 线程Threa_t1 ~ Thread_t4分别只写入1~4
    class Thread_t1 extends Thread{
        String path;
        Thread_t1(String path){
            this.path = path;
        }
        public void run(){
            print(path, 1);
        }
    }

    class Thread_t2 extends Thread{
        String path;
        Thread_t2(String path){
            this.path = path;
        }
        public void run(){
            print(path, 2);
        }
    }

    class Thread_t3 extends Thread{
        String path;
        Thread_t3(String path){
            this.path = path;
        }
        public void run(){
            print(path,3);
        }
    }

    class Thread_t4 extends Thread{
        String path;
        Thread_t4(String path){
            this.path = path;
        }
        public void run(){
            print(path, 4);
        }
    }

    // 写入方法
    public void print(String path, int i) {
        lock_2.lock();
        PrintWriter pw = null;
        try {
            pw = new PrintWriter(new FileWriter("D:/git-project/multi-thread/result/" + path + ".txt", true), true);
            pw.print(i);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(pw != null) {
                pw.close();
            }
            lock_2.unlock();
        }
    }

    class File_A implements Runnable{
        int one, two, three, four;

        File_A(int one, int two, int three, int four){
            this.one = one;
            this.two = two;
            this.three = three;
            this.four = four;
        }

        @Override
        public void run() {
            for(int x = 0; x < 100; x++){
                select("A", one, two, three, four);
            }
        }
    }

    class File_B implements Runnable{
        int one, two, three, four;

        File_B(int one, int two, int three, int four){
            this.one = one;
            this.two = two;
            this.three = three;
            this.four = four;
        }

        @Override
        public void run() {
            for(int x = 0; x < 100; x++){
                select("B", one, two, three, four);
            }
        }
    }

    class File_C implements Runnable{
        int one, two, three, four;

        File_C(int one, int two, int three, int four){
            this.one = one;
            this.two = two;
            this.three = three;
            this.four = four;
        }

        @Override
        public void run() {
            for(int x = 0; x < 100; x++){
                select("C", one, two, three, four);
            }
        }
    }

    class File_D implements Runnable{
        int one, two, three, four;

        File_D(int one, int two, int three, int four){
            this.one = one;
            this.two = two;
            this.three = three;
            this.four = four;
        }

        @Override
        public void run() {
            for(int x = 0; x < 100; x++){
                select("D", one, two, three, four);
            }
        }
    }

    public void select(String path, int one, int two, int three, int four) {
        lock.lock();
        try {
            while (b){
                condition_for.await();
            }
            b = true;

            int[] order = {one, two, three, four};
            for(int i : order) {
                switch (i){
                    case 1:
                        Thread_t1 t1 = new Thread_t1(path);
                        t1.start();
                        t1.join();  //t1.join()是等线程t1执行完后再执行当前线程
                        break;
                    case 2:
                        Thread_t2 t2 = new Thread_t2(path);
                        t2.start();
                        t2.join();
                        break;
                    case 3:
                        Thread_t3 t3 = new Thread_t3(path);
                        t3.start();
                        t3.join();
                        break;
                    case 4:
                        Thread_t4 t4 = new Thread_t4(path);
                        t4.start();
                        t4.join();
                        break;
                }
            }
            b = false;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args){
        Hello hello = new Hello();
        new Thread(hello.new File_A(1, 2, 3, 4)).start();
        new Thread(hello.new File_B(2, 3, 4, 1)).start();
        new Thread(hello.new File_C(3, 4, 1, 2)).start();
        new Thread(hello.new File_D(4, 1, 2, 3)).start();
    }
}
```

