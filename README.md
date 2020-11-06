# leetcode

 记录刷题过程中的一些值得注意的知识点和经典题

***

## 2020/11/06	根据数字二进制下1的数目排序

给你一个整数数组 arr 。请你将数组中的元素按照其二进制表示中数字 1 的数目升序排序。如果存在多个数字二进制中 1 的数目相同，则必须将它们按照数值大小升序排列。请你返回排序后的数组。

```c++
示例1：

输入：arr = [0,1,2,3,4,5,6,7,8]
输出：[0,1,2,4,8,3,5,6,7]
解释：[0] 是唯一一个有 0 个 1 的数。
[1,2,4,8] 都有 1 个 1 。
[3,5,6] 有 2 个 1 。
[7] 有 3 个 1 。
按照 1 的个数排序得到的结果数组为 [0,1,2,4,8,3,5,6,7]

示例2：

输入：arr = [1024,512,256,128,64,32,16,8,4,2,1]
输出：[1,2,4,8,16,32,64,128,256,512,1024]
解释：数组中所有整数二进制下都只有 1 个 1 ，所以你需要按照数值大小将它们排序。
```

***

代码实现：

```c++
class Solution {
public:
    static int cal(int a) {
        int count = 0;
        while (a) {
            count += (a % 2);
            a /= 2;
        }
        return count;
    }
    
    static bool compare(int a, int b) {
        if (cal(a) > cal(b)) {
            return false;
        }
        else if (cal(a) == cal(b)) {
            return a > b ? false : true;
        }
        else {
            return true;
        }
    }
    
    vector<int> sortBybits(vector<int>& arr) {
 		sort(arr.begin(), arr.end(), conmpare);
        return arr;
    }
}
```

***

这道题本身很简单，只需要改写sort函数的比较函数重新排序就好。但是在写的过程中遇到了报错的问题`invalid use of non-static member function`

经过查询可知，在类中写比较函数时，compare需要被定义成static类型。原因在于其实我们写参数compare时，是把函数名作为实参传递给了sort函数，而sort函数内部是用一个函数指针去调用这个函数的，我们知道class普通类成员函数需要通过`对象名.compare()`来调用，而sort()函数早就定义好了，那个时候哪知道你定义的是什么对象，所以内部是直接compare()`的，那你不加static时，去让sort()直接用``compare()`当然会报错。

**顺便复习一下几个基础知识点：**

### compare和static的用法

- `compare(int a, int b)`是一个布尔类型的函数，当`a > b`，返回true时，表示数据降序排列。当`a < b`，返回true时，表示数据升序排列

- `static`的用法
  - 在全局变量前加上关键字static，全局变量就定义成一个全局静态变量。存储在静态存储区中。全局静态变量在声明他的文件之外是不可见的，准确地说是从定义之处开始，到文件结尾。
  - 在局部变量之前加上关键字static，局部变量就成为一个局部静态变量。存储在静态存储区中。表示当局部静态变量离开作用域后，并没有销毁，而是仍然驻留在内存当中，只不过我们不能再对它进行访问，直到该函数再次被调用，并且值不变；
  - 在函数返回类型前加static，函数就定义为静态函数。函数的定义和声明在默认情况下都是extern的，但静态函数只是在声明他的文件当中可见，不能被其他文件所用。
  - 在类中，静态成员可以实现多个对象之间的数据共享，并且使用静态数据成员还不会破坏隐藏的原则，即保证了安全性。因此，静态成员是类的所有对象中共享的成员，而不是某个对象的成员。对多个对象来说，静态数据成员只存储一处，供所有对象共用。
  - 静态成员函数和静态数据成员一样，它们都属于类的静态成员，它们都不是对象成员。因此，对静态成员的引用不需要用对象名。在静态成员函数的实现中不能直接引用类中说明的非静态成员，可以引用类中说明的静态成员（这点非常重要）。如果静态成员函数中要引用非静态成员时，可通过对象来引用。从中可看出，调用静态成员函数使用如下格式：<类名>::<静态成员函数名>(<参数表>);