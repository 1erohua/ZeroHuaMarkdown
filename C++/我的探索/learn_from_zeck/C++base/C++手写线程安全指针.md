
```cpp
#include <iostream>
#include <atomic>
#include <mutex>
#include <thread>


template<typename T>
class ThreadSafeSharedPtr{
private:
    T* ptr; // 对象指针
    std::atomic<int>* counter; // 计数器指针

    mutable std::mutex mux; // 本对象的实例锁
    void release(){
        // 解除本指针的引用，并且如果计数为0,同时释放引用
        // 对counter的操作，会自动进行锁操作。
        if(counter){
            // 自减1
            counter->fetch_sub(1,std::memory_order_acq_rel);
            std::cout << "已取消本指针对对象引用，当前引用该对象的数量为" << counter->load(std::memory_order_acquire) << std::endl;
            if(counter->load(std::memory_order_acquire) == 0){
                delete ptr;
                delete counter;
                // std::cout << "对象已被销毁" << std::endl;
            }
        }
        ptr = nullptr;
        counter = nullptr;
    }

public:
    // 默认构造函数
    ThreadSafeSharedPtr():ptr(nullptr),counter(nullptr){}
    // 参数构造函数
    ThreadSafeSharedPtr(T* ptr):ptr(ptr){
        if(ptr){
            counter = new std::atomic<int>;
            counter->store(1,std::memory_order_release);
            std::cout << "对象已被参数构造" << std::endl;
        }else{
            counter = nullptr;
            // 要考虑到参数构造的各种情况
        }
    }
    // 析构函数
    ~ThreadSafeSharedPtr(){
        release();
    }

    // 拷贝构造函数
    ThreadSafeSharedPtr(const ThreadSafeSharedPtr& TSSP){
        // 拷贝构造，本质是构造，因而还没有被构造出来
        std::lock_guard<std::mutex> lock(TSSP.mux);

        // 共享指针
        ptr = TSSP.ptr; // 共享指向数据的指针
        counter = TSSP.counter; // 共享指向计数器的指针

        if(counter){
            counter->fetch_add(1,std::memory_order_release);
            std::cout << "拷贝构造成功" <<std::endl;
        }
    }
    // 拷贝赋值
    ThreadSafeSharedPtr& operator=(const ThreadSafeSharedPtr& tssp){
        // 由scoped进行多锁，避免死锁
        if(this !=  &tssp){
            std::scoped_lock<std::mutex> lock(tssp.mux,this->mux);
            // if(this->ptr != nullptr){
            //     delete  ptr;
            // }
            // if(this->counter != nullptr){
            //     delete counter;
            // } // 干嘛呢这是
            release(); // 上面是犯了错误做法，需要进行赋值时，一定要用release来清除自己的资源
            // 否则原来的shared指针指向的对象怎么办

            ptr = tssp.ptr;
            counter = tssp.counter;

            if(counter){
                counter->fetch_add(1,std::memory_order_release);
                std::cout << "拷贝赋值成功" << std::endl;
            }
        }
        return *this; // 不管什么if情况都要返回
    }

    // 移动构造
    ThreadSafeSharedPtr(ThreadSafeSharedPtr && tssp)noexcept{
        // 加锁
        std::lock_guard<std::mutex> lock(tssp.mux);
        // 转移赋值
        counter = tssp.counter;
        ptr = tssp.ptr;
        // 转移
        tssp.counter = nullptr;
        tssp.ptr = nullptr;

        std::cout << "移动构造成功" << std::endl;
    }
    // 移动赋值
    ThreadSafeSharedPtr& operator=(ThreadSafeSharedPtr && tssp)noexcept{
        // 加锁
        std::scoped_lock<std::mutex> lock(tssp.mux, this->mux);
        if(this != &tssp){
            // if(this->ptr != nullptr){
            //     delete ptr;
            // }
            // if(this->counter != nullptr){
            //     delete counter;
            // }
            release();  // 犯病时刻
        }
        // 转移
        ptr = tssp.ptr;
        counter = tssp.counter;
        // 滞空
        tssp.ptr = nullptr;
        tssp.counter = nullptr;

        std::cout << "移动赋值成功" <<std::endl;
        
        // 很好又忘了return了
        return *this;

    }

    // 获取资源
    // 把当前对象当成指针来用，才需要重构这些运算符
    T& operator*()const{ // 重构解引用
        std::lock_guard<std::mutex> lock(mux);
        return *(this->ptr);
    }

    T* operator->()const{ // 重构指向符
        std::lock_guard<std::mutex> lock(mux);
        return this->ptr;
    }

    // 获取引用计数
    int use_count()const{
        std::lock_guard<std::mutex> lock(mux);
        return counter ? counter->load(std::memory_order_acquire) : 0;
    }

    // 获取裸指针
    T* get()const{
        std::lock_guard<std::mutex> lock(mux);
        return this->ptr;
    }

    // 重置指向与计数器
    void reset(T* ptr = nullptr){
        std::lock_guard<std::mutex> lock(mux);
        // 这里重新传的是指针，因而可以肯定，我们是第一个获取到该指针的人，否则它直接使用赋值运算符就行了
        // 不管怎么样，我需要先释放当前指向
        release();  
        this->ptr = ptr;
        
        if(ptr){
            this->counter = new std::atomic<int>(1);
        }else{
            this->counter = nullptr;
        }
    }
};
```

这是它的测试代码：
请一定要阅读本文件夹下的[[难绷，传递函数参数也会触发构造函数]]
```cpp
#include <iostream>
#include <thread>
#include <vector>
#include "ThreadSafeSharedPtr.h" // 假设将上述代码保存为该头文件

// 测试类
class Test {
public:
    Test(int val) : value(val) {
        std::cout << "Test Constructor: " << value << std::endl;
    }
    ~Test() {
        std::cout << "Test Destructor: " << value << std::endl;
    }
    void show() const {
        std::cout << "Value: " << value << std::endl;
    }

private:
    int value;
};

void thread_func_copy(const ThreadSafeSharedPtr<Test>&sptr, int thread_id) {
    std::cout << "Thread " << thread_id << " is copying shared_ptr." << std::endl;
    ThreadSafeSharedPtr<Test> local_sptr = sptr;
    std::cout << "Thread " << thread_id << " copied shared_ptr, use_count = " << local_sptr.use_count() << std::endl;
    local_sptr->show();
}

void thread_func_reset(ThreadSafeSharedPtr<Test>& sptr, int new_val, int thread_id) {
    std::cout << "Thread " << thread_id << " is resetting shared_ptr." << std::endl;
    sptr.reset(new Test(new_val));
    std::cout << "Thread " << thread_id << " reset shared_ptr, use_count = " << sptr.use_count() << std::endl;
    sptr->show();
}

int main() {
    std::cout << "Creating ThreadSafeSharedPtr with Test(100)." << std::endl;
    ThreadSafeSharedPtr<Test> sptr(new Test(100));
    std::cout << "Initial use_count: " << sptr.use_count() << std::endl;

    // 创建多个线程进行拷贝操作
    const int num_threads = 5;
    std::vector<std::thread> threads_copy;

    for(int i = 0; i < num_threads; ++i) {
	    // 这里使用
	    // threads_copy.emplace_back(thread_func_copy, &sptr, i);
	    // 这里也有问题的，&只有在定义与声明时才是进行引用
	    // 因此在这里 &实参 进行传递，本质是传递地址，因为这样是取地址符了
        threads_copy.emplace_back(thread_func_copy, std::ref(sptr), i);
    }

    for(auto& t : threads_copy) {
        t.join();
    }

    std::cout << "After copy threads, use_count: " << sptr.use_count() << std::endl;

    // // 创建多个线程进行 reset 操作
    // std::vector<std::thread> threads_reset;

    // for(int i = 0; i < num_threads; ++i) {
    //     threads_reset.emplace_back(thread_func_reset, std::ref(sptr), 200 + i, i);
    // }

    // for(auto& t : threads_reset) {
    //     t.join();
    // }

    // std::cout << "After reset threads, final use_count: " << sptr.use_count() << std::endl;

    // std::cout << "Exiting main." << std::endl;
    return 0;
}

```

**注：以下解释是在threads_copy.emplace_back(thread_func_copy, std::ref(sptr), i);中将std::ref(sptr)改为sptr,即原来直接传参的情况。上述代码已更改，只是非常有必要将这件事记录下来**

在这段代码中，调用 `ThreadSafeSharedPtr<Test>& sptr` 的拷贝构造函数的原因涉及到 C++ 中的函数参数传递机制，特别是当你将 `ThreadSafeSharedPtr<Test>` 传递给线程函数时。详细解释如下：

### 1. **传递方式分析**

- **传值传递（pass-by-value）**：当你将 `sptr` 以值传递给线程函数时，会调用 `ThreadSafeSharedPtr<Test>` 的拷贝构造函数，创建一个新的 `ThreadSafeSharedPtr` 副本。你注释掉的代码 `ThreadSafeSharedPtr<Test> local_sptr = sptr;` 就是这种情况。
- **引用传递（pass-by-reference）**：当你使用引用（例如，`ThreadSafeSharedPtr<Test>& sptr`）将 `sptr` 传递给线程时，不会调用拷贝构造函数，只会创建一个引用。

### 2. **为何拷贝构造函数会被调用？**

在你的 `main()` 函数中，虽然传递给 `thread_func_copy` 函数的参数是一个引用 `const ThreadSafeSharedPtr<Test>& sptr`，但是在 C++ 中，当你将一个对象传递给另一个线程时，**C++ 会对这个对象进行“复制”**。因为 `std::thread` 的构造函数要求传入的参数是可拷贝的，而不支持直接传递引用（尤其是非 `const` 引用），因此会发生拷贝构造。

具体来说，`std::thread` 会对传递给它的每一个参数进行拷贝（除非显式使用 `std::ref` 或 `std::cref`）。因此，即使你在 `thread_func_copy` 中使用了引用 `const ThreadSafeSharedPtr<Test>& sptr`，C++ 在创建线程时依然会调用 `ThreadSafeSharedPtr` 的拷贝构造函数。

你可以通过以下修改避免拷贝构造的调用：

```cpp
threads_copy.emplace_back(thread_func_copy, std::cref(sptr), i);
```

这样，`std::cref(sptr)` 会创建一个 `const` 引用包装器，保证 `sptr` 被按引用传递给每个线程，而不是被拷贝。

### 3. **线程函数中对 `ThreadSafeSharedPtr` 的引用**

在函数 `thread_func_copy` 内部，`sptr` 是按引用传递的（`const ThreadSafeSharedPtr<Test>& sptr`）。这样你就不会在函数内部进行拷贝，而是操作传入的共享指针的引用。

总结一下，拷贝构造函数在以下情况下会被调用：

- 线程的构造函数需要通过拷贝来传递参数，无法直接传递引用。
- 需要显式使用 `std::ref` 或 `std::cref` 来确保按引用传递。

### 结论：

如果你希望避免拷贝构造函数的调用，并希望 `sptr` 在各线程间以引用方式传递，你可以使用 `std::cref` 或者 `std::ref` 来确保 `ThreadSafeSharedPtr` 作为引用传递给每个线程。