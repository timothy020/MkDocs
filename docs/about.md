[TOC]

# 16.1.1 函数模板

`tmplate`+**模板参数列表**

```c++ linenums="1" hl_lines="2 3"
template<typename T>
int compare(const T &v1, const T &v2){
    if(v1<v2) return -1;
    if(v2<v1) return 1;
    return 0;
}
```

> 模板参数列表不能为空

实例化：实现模板的某个特定版本

```c++
cout<<compare(0,1)<<endl; //T为int
cout<<compare(vector1, vector2); //T为vector<int>
```



**非类型模板参数**：表示一个值而非一个类型

值必须是常量表达式，模板在编译时被实例化

1. 如果是整型：必须是常量表达式
2. 如果是引用或指针：传入的实参必须有静态生存期static



例如,我们可以编写一个compare版本处理字符串字面常量。这种字面常量是constchar 的数组。由于不能拷贝一个数组，所以我们将自己的参数定义为数组的引用（参见6.2.4节，第195页)。由于我们希望能比较不同长度的字符串字面常量，因此为模板定义了两个非类型的参数。第一个模板参数表示第一个数组的长度，第二个参数表示第二个数组的长度:

```c++
template<unsigned N, unsigned M>
int compatre(const char (&p1)[N], const char (&p2)[M]){
    if(N<M) return -1;
    if(M<N) return 1;
    return 0;
}
```

```c++
compare("hi", "mom");
//编译器实例化出：
int compare(const char (&p1)[3], const char (&p2)[4]);
```



**模板的编译**

当编译器遇到一个模板定义时，它并不生成代码。

只有当我们**实例化**出模板的一个特定版本时，编译器才会**生成代码**。

> 模板则不同:为了生成一个实例化版本，编译器需要掌握函数模板或类模板成员函数的定义。因此，与非模板代码不同，模板的头文件通常既包括**声明也包括定义**。



**最佳实践：定义一个函数模板print，遍历打印任何类型任何大小的数组**

```c++
template<typename T, unsigned N>
void print(T (&array)[N]){
    for(T it : array){
        cout<<it<<" ";
    }
    cout<<endl;
}
```



## 16.1.2 类模板

与函数模板不同，类模板在实例化时要提供**额外类型信息**

### 定义类模板

```c++
template<typename T> class Blob{
public:
    typedef T vale_type;
    typedef typename vector<T>::size_type size_type;
    //构造函数
    Blob();
    Blob(std::initializer_list<T> il);
    //Blob中的元素数目
    size_type siez() const {return data->size(); }
    bool empty() const {return data->empty(); }
    //添加和删除元素
    void push_back(const T &t) {data->push_back(t);}
    void piop_back();
    //元素访问
    T& back();
    T& operator[](size_type i);
private:
    shared_ptr<vector<T>> data;
    //若data[i]无效，则抛出异常
    void check(size_type i, const string &msg) const;
};
```

### 实例化模板

```c++
Blob<int> ia; 				 //空Bolb<int>
Blob<int> ia2 = {0,1,2,3,4}; //有5个空元素的Blob<int>
```

当编译器从我们的Blob模板实例化出一个类时，它会重写Blob模板，将模板参数T的每个实例替换为给定的模板实参，在本例中是int。



### 在类外定义成员函数

在类外定义成员函数时，不仅要表明类的作用域，还要带上**模板实参**

```c++
template<typename T>
void Blob<T>::check(size_type i, const string &msg)const{
    if( i >= data->size() )
        throw std::out_of_range(msg);
}
```



### 类模板成员函数的实例化

默认情况下，一个类模板的成员函数只有当程序用到它时才进行实例化。例如，下面代码

```c++
//实例化Blob<int>和接受initializer_list<int>的构造函数
Blob<int> squares = {0, 1,2,3,4,5,6,7,8,9};
//实例化Blob<int>::size() const
for (size_t i= 0; i != squares.size(); ++i)
	squares[i] = i*i;//实例化Blob<int>::operator[](size_t)
```



实例化了 Blob<int>类和它的三个成员函数:operator[]、size和接受initializer_list<int>的构造函数。
如果一个成员函数没有被使用，则它不会被实例化。成员函数只有在被用到时才进行实例化，这一特性使得即使某种类型不能完全符合模板操作的要求（参见9.2节，第294页)，我们仍然能用该类型实例化类。

> 默认情况下,对于一个实例化了的类模板,其**成员只有在使用时才被实例化**



### 类模板和友元

1. 一对一友好关系:两模板类的同类型实例化互为友元

```c++
//前置声明，在Blob中声明友元所需要的
template <typename> class BlobPtr;
template <typename> class Blob;  //运算符==中的参数所需要的
template <typenaem T>
bool operator==(const Blob<T> &, const Blob<T>&);

template <typename T> class Blob{
    //每个Blob实例将访问权限授予用相同类型实例化的BlobPtr和相等运算符
    friend class BlobPtr<T>;
    friend bool operator==<T>
        (const Blob<T>&, const Blob<T>&);
    //其他成员定义不变
};
```

友元的声明**用Blob的模板形参作为它们自己的模板实参**。因此，友好关系被限定在用**相同类型**实例化的Blob 与 BlobPtr相等运算符之间;

```c++
Blob<char> ca; //BlobPtr<char>和operator==<char>都是本对象的友元
Blob<int> ia;  //BlobPtr<int>和operator==<int>都是本对象的友元
```

2. 通用和特定的模板友好关系：一个模板类将另一个模板类的类型的实例都声明为自己的友元

```c++
//前置声明，在模板类声明时需要用到
template <typename T> class Pal;
class C{//C是一个普通的非模板类
    friend class Pal<C>; //用类C实例化的Pal是C的一个友元
    //Pal2的所有实例化都是C的友元，这种情况无需前置声明
    template <typename T> friend class Pal2;
};
template <typename T> class C2{//C2本身是一个类模板
	//C2的每个实例都将相同的实例化的Pal声明为友元
    friend class Pal<T>; //Pal模板什么必须在作用域之内
    //Pal2的所有实例都是C2的所有实例的友元，不需要前置声明
    template <typename X> friend class Pal2;
    //Pal3是一个非模板类，它是C2所有实例的友元
    friend class Pal3; //不需要Pal3的前置声明
};
```

> 为了让另一模板类的所有实例成为友元，友元声明中必须使用与类模板本身不同的模板参数。

3. 令模板自己类型成为友元

```c++
8template <typename Type> class Bar{
friend Type; //将访问权限授予用来实例化的Bar类型
    //...
};
```

### 模板类型别名

由于模板不是一种类型，所以我们不能用`typedef`而要用`using`

```c++
template<typename T> using twin = pair<T,T>;
twin<string> authors; //authors是一个pair<string, string>
```

一个模板类型是一族类的别名

```c++
twin<int> win_loss; //win_loss是一个pair<int,int>
twin<double> area;  //area是一个pair<double,double>
```

当我们使用模板类型别名时，可以固定**一个或多个参数**

```c++
template <typename T> using partNo = pair<T, unsigned>;
partNo<string> books; //books是一个pair<string, unsigned>
partNo<Vehicle> cars; //cars是一个pair<Vehicle, unsigned>
partNo<Student> kisd; //kids是一个pair<Student, unsigned>
```



### 类模板参数的static成员

每种类型的实例化对象间共享，不同类型的实例化对象间不共享



# 16.1.3 模板参数

## 使用类的类型成员

回忆一下，我们用作用域运算符(::)来访问static成员和类型成员。在普通（非模板）代码中，编译器掌握类的定义。因此，它知道通过作用域运算符访问的名字是类型还是 static成员。例如，如果我们写下`string::size_type`，编译器有string 的定义，从而知道size_type是一个类型。

但对于模板代码就存在困难。例如，假定T是一个模板类型参数，当编译器遇到类似`T::mem`这样的代码时，它不会知道mem是一个类型成员还是一个static数据成员，直至实例化时才会知道。

但是，为了处理模板，编译器必须知道名字是否表示一个类型。例如，假定T是一个类型参数的名字，当编译器遇到如下形式的语句时:

```c++
T::size_type * p;
```

它需要知道我们是正在定义一个名为p的变量还是将一个名为size_type的static数据成员与名为p的变量相乘。
默认情况下，C++语言假定通过作用域运算符访问的名字不是类型。因此，如果我们希望使用一个模板类型参数的类型成员，就必须显式告诉编译器该名字是一个类型。我们通过使用关键字`typename`来实现这一点:

```c++
template <typename T>
typename T::value_type top(const T& c)
{
    if ( !c.empty())
		return c.back();
    else
		return typename T::value_type ();
}
```



> 当我们希望通知编译器一个名字表示类型时，必须使用关键字typename，而不能使用class。



## 默认模板实参

### 函数模板的默认实参

就像我们能为函数参数提供默认实参一样，我们也可以提供**默认模板实参(default template argument)**。在新标准中，我们可以为**函数和类**模板提供默认实参。而更早的C++标准只允许为类模板提供默认实参。

例如，我们重写compare，默认使用标准库的less函数对象模板

```c++
/// l compare有一个默认模板实参less<T>和一个默认函数实参F()
template <typename T, typename F = less<T>>
int compare (const T &v1, const T &v2，F f = F()){
	if ( f(v1, v2) ) return -1;
	if (f(v2, v1) ) return 1;
    return 0;
}
```

在这段代码中，我们为模板添加了第二个类型参数，名为F，表示**可调用对象**的类型;并定义了一个新的函数参数f，绑定到一个可调用对象上。

我们为此模板参数提供了默认实参，并为其对应的函数参数也提供了默认实参。默认模板实参指出compare将使用标准库的 less函数对象类，它是使用与 compare一样的类型参数实例化的。默认函数实参指出f将是类型F的一个默认初始化的对象。
当用户调用这个版本的compare时，可以提供自己的比较操作，但这并不是必需的:

```c++
bool i = compare(0,42);//使用less; i为-1
//结果依赖于item1和item2中的isbn
Sales_data item1(cin), item2(cin) ;
bool j = compare (item1,item2,compareIsbn) ;
```

第一个调用使用默认函数实参，即，类型less<T>的一个默认初始化对象。在此调用中，T为int，因此可调用对象的类型为less<int>。compare 的这个实例化版本将使用less<int>进行比较操作。

在第二个调用中，我们传递给compare三个实参: compareIsbn(参见11.2.2节)和两个sales_data类型的对象。当传递给 compare三个实参时，第三个实参的类型必须是一个可调用对象，该可调用对象的返回类型必须能转换为bool值，且接受的实参类型必须与compare的前两个实参的类型兼容。与往常一样，模板参数的类型从它们对应的函数实参推断而来。在此调用中，T的类型被推断为sales_data，F被推断为compareIsbn的类型。

与函数默认实参一样,对于一个模板参数，只有当它**右侧的所有参数**都有默认实参时，它才可以有默认实参。



### 类模板的默认实参

无论何时使用一个类模板，我们都必须在模板名之后接上尖括号。尖括号指出类必须从一个模板实例化而来。特别是，如果一个类模板为其所有模板参数都提供了默认实参，且我们希望使用这些默认实参，就必须在模板名之后跟一个空尖括号对:

```c++
template <class T = int> class Numbers { // T默认为int
public:
	Numbers(T v = 0): val(v) { }
	//对数值的各种操作
private:
	Tval;
};
Numbers<long double> lots_of_precision;
Numbers<> average_precision; //空<>表示我们希望使用默认类型
```

此例中我们实例化了两个 Numbers版本: average_precision是用int 代替T实例化得到的; lots_of _precision是用long double代替T实例化而得到的。
