前提：我们需要知道，单纯的一个指针，是没办法遍历的，因为你在遍历的时候是不知道一个指针遍历到多远的。因而std::begin()没有 int* 类型的重载。
```c++
int main(){

    int a[3][5] = {};
    for(auto i : a)
    {
	    // for(auto j:i)   会产生报错，for范围遍历需要被遍历对象有std::begin()
	    // 然而经过打印我们才知道，i在这里只是一个整型指针，占用8字节，它不是编译器认可的数组
	    // 在这里你也可以理解为数组的拷贝，但是并不是原来的数组
        std::cout << "拷贝值的大小" << sizeof(i) << std::endl;
        break;
    }

    for(auto &i : a)
    {
	    // 这里使用了引用，就是原来的数组
        std::cout << "引用值的大小" << sizeof(i) << std::endl;
        break;
    }

}
```
我们把光标移动到连个i下面，会发现，第一个i的描述为![[Pasted image 20250112142855.png]]
第二个i的描述为![[Pasted image 20250112143006.png]]
第二个i其实等同于
```cpp
int b[5] = {};
```

因而，指针和数组本身是不同的东西, **数组名可以被转换为指针**，但本身并不是指针(这里其实不能说是转化，应该是称为退化)
![[Pasted image 20250112143729.png]]
![[Pasted image 20250112143814.png]]













