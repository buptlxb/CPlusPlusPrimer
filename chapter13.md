## 第13章 复制控制

* 复制控制包括： **复制构造函数** 、 **赋值操作符** 、 **析构函数** 。
* 使用复制构造函数的三种情况：
	1. 使用同类型的对象初始化一个新对象。
	2. 将该类型的对象传递给函数。
	3. 从函数返回该类型的对象。
* 即使用要将一个对象赋值给自身，赋值操作符也必须保证总是正确的。
* 编译器总为为类合成一个析构函数，称为 **合成析构函数**。
* 合成析构函数按对象创建时每个成员构造的顺序的 **逆序** 撤销每个非`static`成员。
* 即使为类自定义析构函数，合成析构函数仍然会执行。
* 三法则：如果需要为类定义构造函数，那么基本上也应该为类定义复制构造函数和赋值操作符。


在练习堆排序的过程中，发现对复制控制的有些问题都忘记了，写的时候对复制构造函数和赋值操作符有了新的理解。

代码如下：
``` c++
#include <iostream>
#include <sstream>
#include <cstdlib>
#include <ctime> 

class Number {
    private :
        int x;
    public :
        Number () 
        {
            x = rand() % 10;
        }
        Number (const Number &num)
        {
            this->x = num.x;
            std::cout << this->x << ": copy constructor invoked!" << std::endl;
        }
        std::string toString(void) const 
        {
            std::ostringstream istr;
            istr << "Number: " << x;
            return istr.str();
        }
        bool operator< (const Number &num) const
        {
            return this->x < num.x;
        }
        bool operator> (const Number &num) const
        {
            return this->x > num.x;
        }
        bool operator>= (const Number &num) const
        {
            return this->x >= num.x;
        }
        bool operator<= (const Number &num) const
        {
            return this->x <= num.x;
        }
        bool operator== (const Number &num) const
        {
            return this->x == num.x;
        }
        bool operator!= (const Number &num) const
        {
            return this->x != num.x;
        }

        Number & operator= (const Number &num)
        {
            this->x = num.x;
            std::cout << this->x << ": operator= invoked!" << std::endl;
        }
};

std::ostream & operator<< (std::ostream &os, const Number &num)
{
    return os << num.toString();
}

template <typename T>
void print_array(T *, int);

template <typename T>
void heap_fix_down(T *, int, int);

template <typename T>
void heap_sort(T *, int);

template <typename T>
void heap_build(T *, int);

int main(void)
{
    srand(time(NULL));
    //int array[] = {4, 3, 1, 2, 0};
    //int length = 4;
    //Number *array = new Number[length];
    Number array[3] = {Number(), Number()};
    int length = sizeof(array) / sizeof(array[0]);
    print_array(array, length);
    heap_build(array, length);
    heap_sort(array, length);
    print_array(array, length);
//    delete []array;
    return 0;
}

template <typename T>
void print_array(T *array, int length)
{
    for (int i = 0; i < length; i++) {
        std::cout << array[i] << " ";
    }
    std::cout << std::endl;
}

template <typename T>
void heap_fix_down(T *array, int length, int index)
{
    T tmp = array[index];
    int min_child = 2*index + 1;
    while (min_child < length) {
        if (min_child+1 < length && array[min_child] > array[min_child+1])
            min_child++;
        if (array[min_child] > tmp)
            break;
        array[index] = array[min_child];
        index = min_child;
        min_child = 2*index + 1;
    }
    array[index] = tmp;
}

template <typename T>
void heap_build(T *array, int length)
{
    for (int i = length/2 - 1; i >= 0; i--)
        heap_fix_down(array, length, i);
}

template <typename T>
void heap_sort(T *array, int length)
{
    while (length-- > 0) {
        T tmp = array[0];
        array[0] = array[length];
        array[length] = tmp;
        heap_fix_down(array, length, 0);
    }
}
```

输出如下：

``` 
Number: 6 Number: 5 Number: 9 
6: copy constructor invoked!
5: operator= invoked!
6: operator= invoked!
5: copy constructor invoked!
9: operator= invoked!
5: operator= invoked!
9: copy constructor invoked!
6: operator= invoked!
9: operator= invoked!
6: copy constructor invoked!
9: operator= invoked!
6: operator= invoked!
9: copy constructor invoked!
9: operator= invoked!
9: copy constructor invoked!
9: operator= invoked!
9: operator= invoked!
9: copy constructor invoked!
9: operator= invoked!
Number: 9 Number: 6 Number: 5 
```

通过gdb单步调试，发现执行代码`int tmp = array[0];`时，会调用复制构造函数，分析后不难理解：`array[0]`在这作为`tmp`的初始化值。而tmp在些处，为声明+初始化，所以调用复制构造函数。

同时，在执行代码`array[length] = tmp`时，调用了赋值操作符，即此处的`array[length]`已经声明初始化完毕，`tmp`为右操作数，对其进行赋值操作。
