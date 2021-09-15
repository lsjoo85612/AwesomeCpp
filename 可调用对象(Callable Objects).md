# c++11中有一个重要的概念叫做可调用对象(Callable Objects)。

可调用对象用处广泛，比如在使用一些基于范围的模板函数时（如 sort()、all_of()、find_if() 等），常常需要我们传入一个可调用对象，以指明我们需要对范围中的每个元素进行怎样的处理。
又比如，在处理一些回调函数、触发函数时，也常常会使用可调用对象。

总的来说，可调用对象可以是以下几种情况：
- 普通函数
- 函数指针
- 仿函数，即重载了operator()运算符的类对象
- 匿名函数，即Lambda表达式
- std::function

----

## 普通函数
普通函数的定义简单，如：
```
bool cmp(const int &a, const int &b) {
	return a < b; // 从小到大排列
}
```

它的调用也很简单：
```
bool result= cmp(a, b);
```

在c++模板函数中也可简单进行使用
```
vector<int> v = {3, 6, 2, 7, 4, 9, 1};
sort(v.begin(), v.end(), cmp);//cmp为排序准则
```

## 函数指针
```
bool cmp(const int &a, const int &b) {
	return a < b; // 从小到大排列
}

bool (*p)(const int &a, const int &b);//创建一个函数指针
p = cmp;//与函数进行绑定
 ```

函数指针的使用与普通函数基本相同：
```
vector<int> v = {3, 6, 2, 7, 4, 9, 1};
sort(v.begin(), v.end(), p);
```

## 仿函数
仿函数其实就是重载了operator()运算符的类对象。如下：
```
#include <iostream>
using namespace std;

struct MyPlus{
    int operator()(const int &a , const int &b) const{
        return a + b;
    }
};

int main()
{
    MyPlus a;
    cout << MyPlus()(1,2) << endl;    //1、通过产生临时对象 调用重载运算符
    cout << a.operator()(1,2) << endl;//2、通过对象 显示调用重载运算符
    cout << a(1,2) << endl;           //3、通过对象 隐示地调用重载运算符
    return 0;
}
```

在c++模板函数中也可简单进行使用，如下例子，判断vector中各个值是否大于设定值base_：
```
class Bigger {
private:
	int base_;
public:
	Bigger(int base) {
		base_ = base;
	}
	void operator() (const int &num) {
		cout << (num > base_);
	}
};

void main() {
	vector<int> v = {1, 2, 3, 4, 5};
	int base = 3;
	Bigger bigger(base);      
	for_each(v.begin(), v.end(), bigger); //直接调用对象bigger即可
}
```

## 匿名函数(Lambda表达式)
**定义**

Lambda函数，又可以称为Lambda表达式或者匿名函数，在C++11中加入标准。定义形式如下：
```
[captures] (params) -> return_type { statments;} 
```
其中：

---
[captures]为捕获列表，用于捕获外层变量。[&]表示捕获当前范围内所有局部变量。
(params)为匿名函数参数列表
-> return_type指定匿名函数返回值类型
{ statments; }部分为函数体，包括一系列语句
注意：

当匿名函数没有参数时，可以省略(params)部分
当匿名函数体的返回值只有一个类型或者返回值为void时，可以省略->return_type部分

---

**例子**

一个简单的 lambda 表达式的例子：
```
auto f = [] { return "hello world"; }; 
cout << f() << endl; // 输出：hello world
```
另外一个例子：

判断一个数组中的元素与指定的 base 的大小关系。

```
#include <iostream>
#include <vector>
using namespace std;

// 类似函数传参，上面使用的是值捕获，也可以使用引用捕获，如
// [&sz] {sz = 1};

void larger( vector<int> v,int base = 5) {
    for_each(v.begin(), v.end(), [base](const int &num){cout << (num > base);});
}

int main() {
    
    vector<int> vec = {1,3,5,7,9};
    larger(vec,5);
    return  0;
}

```
 

**隐式捕获**

```
除了像上面那样明确指出我们需要捕获的变量列表外，我们还可以使用隐式捕获，即直接在函数体内使用我们需要使用的辅助变量而不用在捕获列表中声明，但需要指出捕获方式——值捕获 or 引用捕获。

void larger(int base, vector<int> v) {
    //隐式值捕获
	for_each(v.begin(), v.end(), [=](const int &num){cout << (num > base);});
    
    //隐式引用捕获
  //for_each(v.begin(), v.end(), [&](const int &num){cout << (num > base);});
} 
 ```

## std::function

**std::function 定义**

std::function在C++11后加入标准，可以用它来描述C++中所有可调用实体，它是是可调用对象的包装器，声明如下：

```
#include <functional>

// 声明一个返回值为int，参数为两个int的可调用对象类型
std::function<int(int, int)> Func;
 ```

**其他函数实体转化为std::function**

std::function强大的地方在于，它能够兼容所有具有相同参数类型的函数实体。

相比较于函数指针，std::function能兼容带捕获的lambda函数，而且对类成员函数提供支持。

 ```
#include <iostream>
#include <functional>

// std::function
std::function<int(int, int)> SumFunction;

// 普通函数
int func_sum(int a, int b)
{
    return a + b;
}

class Calcu
{
public:
    int base = 20;
    // 类的成员方法，参数包含this指针
    int class_func_sum(const int a, const int b) const { return this->base + a + b; };
    // 类的静态成员方法，不包含this指针
    static int class_static_func_sum(const int a, const int b) { return a + b; };
};

// 仿函数
class ImitateAdd
{
public:
    int operator()(const int a, const int b) const { return a + b; };
};

// lambda函数
auto lambda_func_sum = [](int a, int b) -> int { return a + b; };

// 函数指针
int (*func_pointer)(int, int);

int main(void) 
{
    int x = 2; 
    int y = 5;

    // 普通函数
    SumFunction = func_sum;
    int sum = SumFunction(x, y);
    std::cout << "func_sum：" << sum << std::endl;

    // 类成员函数
    Calcu obj;
    SumFunction = std::bind(&Calcu::class_func_sum, obj, 
        std::placeholders::_1, std::placeholders::_2); // 绑定this对象
    sum = SumFunction(x, y);
    std::cout << "Calcu::class_func_sum：" << sum << std::endl;

    // 类静态函数
    SumFunction = Calcu::class_static_func_sum;
    sum = SumFunction(x, y);
    std::cout << "Calcu::class_static_func_sum：" << sum << std::endl;

    // lambda函数
    SumFunction = lambda_func_sum;
    sum = SumFunction(x, y);
    std::cout << "lambda_func_sum：" << sum << std::endl;

    // 带捕获的lambda函数
    int base = 10;
    auto lambda_func_with_capture_sum = [&base](int x, int y)->int { return x + y + base; };
    SumFunction = lambda_func_with_capture_sum;
    sum = SumFunction(x, y);
    std::cout << "lambda_func_with_capture_sum：" << sum << std::endl;

    // 仿函数
    ImitateAdd imitate;
    SumFunction = imitate;
    sum = SumFunction(x, y);
    std::cout << "imitate func：" << sum << std::endl;

    // 函数指针
    func_pointer = func_sum;
    SumFunction = func_pointer;
    sum = SumFunction(x, y);
    std::cout << "function pointer：" << sum << std::endl;

    getchar();
    return 0;
}
  ```

注意其中的类成员函数，使用了std::bind。
因为类成员函数包含this指针参数，所以单独使用std::function是不够的，还需要结合使用std::bind函数绑定this指针以及参数列表。


## std::bind参数绑定规则
在使用std::bind绑定类成员函数的时候需要注意绑定参数顺序：

```
// 承接上面的例子
SumFunction = std::bind(&Calcu::class_func_sum, obj, 
        std::placeholders::_1, std::placeholders::_2);
SumFunction(x, y);
```

第一个参数为类成员函数名的引用（推荐使用引用）
第二个参数为this指针上下文，即特定的对象实例
之后的参数分别制定类成员函数的第1,2,3依次的参数值
使用std::placeholders::_1表示使用调用过程的第1个参数作为成员函数参数
std::placeholders::_n表示调用时的第n个参数
看下面的例子：

// 绑定成员函数第一个参数为4，第二个参数为6
SumFunction = std::bind(&Calcu::class_func_sum, obj, 4, 6);
SumFunction(); // 值为 10

// 绑定成员函数第一个参数为调用时的第一个参数，第二个参数为10
SumFunction = std::bind(&Calcu::class_func_sum, obj, std::placeholders::_1, 10);
SumFunction(5); // 值为 15

// 绑定成员函数第一个参数为调用时的第二个参数，第一个参数为调用时的第二个参数
SumFunction = std::bind(&Calcu::class_func_sum, obj, std::placeholders::_2, std::placeholders::_1);
SumFunction(5, 10); // 值为 15

**参考文章**

https://www.cnblogs.com/youyoui/p/8933006.html
https://blog.csdn.net/zh_94/article/details/88532482
