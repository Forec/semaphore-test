# 信号量算法测试

> 简陋的用于测试信号量算法是否正确的工具。

## 使用说明

* 代码包括一个基类 `WorkStation`（包含在 `base.hpp` 中） 和两个可以用来做参考的测试文件。其中 `bridge1.cpp` 用来验证 [信号量编程（上）](http://blog.forec.cn/2017/01/06/os-concepts-14/) 中习题 7.16， `bridge2.cpp` 用来验证 [互斥读者-读者问题](http://blog.forec.cn/2017/01/08/os-concepts-16/) 中我针对修改后问题给出的自己的解答。你可以仿照这两个文件，继承 `WorkStation` 类来测试自己的信号量程序。
* 测试举例：以老虎与猪问题为例，定义一个类 `Coop` 继承 `WorkStation`。老虎与猪问题具体可参考 [信号量编程（下）](http://blog.forec.cn/2017/01/08/os-concepts-15/) 中的描述。
 * 将所需变量添加到子类中，并实现虚方法 `init()`，该方法对你需要用到的信号量/变量做初始化。注意 `init()` 函数中需要设置 `thread_type`，这个变量表示有多少不同类型的进程，此问题有 4 种（猎人 A、B，厨师、饲养员），故 `thread_type` 应当为 4。
 * 定义自己的参数结构体，对于上面老虎与猪问题而言需要将所用到的所有变量/信号量的地址放置到一个结构体中，对于此问题为：

```c
struct Param{
    int *pig_count;
    HANDLE *coop, *tiger, *pig;
    HANDLE *pig_space, *pig_count_mutex;
```
 * 实现虚函数 `void * getParam(int index)`，此函数格式比较固定，对于此问题应当为：

```c
void * getParam(int index){
    Param *p = new Param;
    p->pig_count = &pig_count;
    p->coop = &coop;
    p->tiger = &tiger;
    p->pig = &pig;
    p->pig_space = &pig_space;
    p->pig_count_mutex = &pig_count_mutex;
}
```
 * 实现四个进程对应的函数，只需要将已经写好的代码复制到你的代码中，格式参考 `bridge1.cpp` 和 `bridge2.cpp`。例如上面的猎人 A，应当写为：

```c
DWORD WINAPI hunter_A(LPVOID param){
    Param p = *(Param*) param;
    // 抓到一只老虎
    WaitForSingleObject(*(p.coop), INFINITE);
    // 将老虎放入笼子
    ReleaseSemaphore(*(p.coop), 1, NULL);
}
```
 * 假设你实现的代表四个进程的四个函数分别名为 `hunter_A`、`hunter_B`、`cook` 和 `feeder`。则向 `init()` 函数添加下面的代码更新驱动表：

```c
_threads[0] = hunter_A;
_threads[1] = hunter_B;
_threads[2] = cook;
_threads[3] = feeder;
```
 * 修改 `main()` 函数以测试。将 `main()` 函数中 `simulate()` 方法的第一个参数修改为 1，此参数表示每个进程创建多少个实例，此问题四种进程各一个，因此为 1。后一个参数表示程序运行多久退出，通常默认 60s 足够测试完成。

# 许可证
此仓库中的全部代码均受仓库中 [License](https://github.com/Forec/semaphore-test/blob/master/LICENSE) 声明的许可证保护。
