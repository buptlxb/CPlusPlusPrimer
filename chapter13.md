## 第13章 复制控制

* 复制控制包括： **复制构造函数** 、 **赋值操作符** 、 **析构函数** 。
* 编译器总为为类合成一个析构函数，称为 **合成析构函数**。
* 合成析构函数按对象创建时每个成员构造的顺序的 **逆序** 撤销每个非`static`成员。
* 即使为类自定义析构函数，合成析构函数仍然会执行。
* 即使用要将一个对象赋值给自身，赋值操作符也必须保证总是正确的。