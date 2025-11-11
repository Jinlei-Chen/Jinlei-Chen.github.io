---
layout: post
title: new和malloc使用和差异
date: 2025-11-11 08:00:00 +0800
---
# new和malloc使用和差异
## new和malloc基本介绍
### malloc 介绍
#### 功能
malloc函数用于动态分配内存，其原型为：
```c
void* malloc(size_t size);
```
其中，`size` 参数指定要分配的内存块的大小（以字节为单位）。

#### 返回值
如果分配成功，`malloc` 函数返回指向分配内存块的指针（类型为 `void*`）。如果分配失败，则返回 `NULL`。如果要使用，必须进行显式类型转换。

#### 包含头文件
C语言：#include <stdlib.h>
C++: #include <cstdlib>

#### 内存释放
使用 `malloc` 分配的内存块在使用完毕后，需要使用 `free` 函数释放，以避免内存泄漏。`free` 函数的原型为：

```c
void free(void* ptr);
```
其中，`ptr` 参数是要释放的内存块的指针。

#### 分配策略
`malloc` 函数使用系统的内存管理器来分配内存。内存管理器会根据系统的内存使用情况来决定如何分配内存。
- malloc是在堆上分配内存,堆上的内存需要程序员手动释放,否则会造成内存泄漏。
- 当malloc申请的内存块小于系统内存管理器的最小内存块大小，则系统会扩展内存块的大小。
- 当malloc申请的内存块大于系统内存管理器的最小内存块大小，则系统会直接分配一块新的内存块。
- 当malloc申请的内存较大时，会调用系统的mmap函数来分配内存，这个过程可能会比较慢。
- 当malloc申请的内存较小时，会调用系统的brk函数来分配内存，这个过程会比较快。

#### 注意事项
- 在使用 `malloc` 分配的内存块之前，必须进行类型转换，以获得正确的指针类型。
- 内存分配失败时，`malloc` 函数返回 `NULL`。
- 申请的内存是未初始化的，需要手动初始化。
- 在使用 `malloc` 释放内存块时，必须确保该内存块是正确的指针，并且没有被释放过。
- 在使用 `malloc` 分配的内存块之后，必须使用 `free` 函数释放内存，以避免内存泄漏。
- 释放内存时，不要重复释放同一个内存块，否则会导致程序崩溃。
- 释放后指针应设置为 `NULL`，避免悬空指针(野指针)。

#### 相关函数
- calloc：分配指定数量和大小的内存，并将内存初始化为0。原型：void* calloc(size_t num, size_t size);
- realloc：调整已分配内存块的大小。原型：void* realloc(void* ptr, size_t new_size);


####  示例

##### 常见用法
```c
#include <stdlib.h>

int main() {
    // 分配一个int大小的内存块
    int* p = (int*)malloc(sizeof(int)); 
    if (p == NULL) {
        // 分配失败
        return 1;
    }
    *p = 10; // 初始化内存块
    free(p); // 释放内存块
    p = NULL; // 避免悬空指针

    // 分配一个int数组，大小为n = 10
    int n = 10;
    int* arr = (int*)malloc(n* sizeof(int));
    if (arr == NULL) {
        // 分配失败
        return 1;
    }
    for (int i = 0; i < n; i++) {
        arr[i] = i;
    }
    free(arr); // 释放内存块
    arr = NULL; // 避免悬空指针

    // 分配结构体
    struct Person {
        char name[20];
        int age;
    };
    struct Person* p1 = (struct Person*)malloc(sizeof(struct Person));
    if (p1 == NULL) {
        // 分配失败
        return 1;
    }
    strcpy(p1->name, "Alice");
    p1->age = 20;
    free(p1); // 释放内存块
    p1 = NULL; // 避免悬空指针

    return 0;
}
```
##### 与realloc结合使用
```c
#include <stdlib.h>
#include <stdio.h>

int main() {
    int* arr = (int*)malloc(3 * sizeof(int));
    
    if (arr) {
        for (int i = 0; i < 3; i++) {
            arr[i] = i + 1;
        }
        
        // 扩展数组大小
        int* new_arr = (int*)realloc(arr, 6 * sizeof(int));
        if (new_arr) {
            arr = new_arr;
            // 初始化新分配的部分
            for (int i = 3; i < 6; i++) {
                arr[i] = (i + 1) * 10;
            }
            
            for (int i = 0; i < 6; i++) {
                printf("arr[%d] = %d\n", i, arr[i]);
            }
        }
        
        free(arr);
    }
    
    return 0;
}
```

##### 动态数组
```c


#include <stdlib.h>
#include <stdio.h>

int main() {
    int rows = 3, cols = 4;
    
    // 分配行指针数组
    int** matrix = (int**)malloc(rows * sizeof(int*));
    if (!matrix) return 1;
    
    // 为每一行分配内存
    for (int i = 0; i < rows; i++) {
        matrix[i] = (int*)malloc(cols * sizeof(int));
        if (!matrix[i]) {
            // 错误处理：释放已分配的内存
            for (int j = 0; j < i; j++) {
                free(matrix[j]);
            }
            free(matrix);
            return 1;
        }
        
        // 初始化
        for (int j = 0; j < cols; j++) {
            matrix[i][j] = i * cols + j;
        }
    }
    
    // 使用矩阵...
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%2d ", matrix[i][j]);
        }
        printf("\n");
    }
    
    // 释放内存
    for (int i = 0; i < rows; i++) {
        free(matrix[i]);
    }
    free(matrix);
    
    return 0;
}
```
### new介绍

#### 功能
new 是C++中的运算符，用于动态内存分配。
```cpp

    // 分配单个对象
    Type* pointer = new Type;

    // 分配并初始化
    Type* pointer = new Type(value);

    // 分配数组
    Type* array = new Type[size];

    // 释放内存
    delete pointer;        // 单个对象
    delete[] array;        // 数组
```



#### 返回值
- new 函数的返回值是一个指针，指向分配的内存块。
- 如果分配失败，new 函数会抛出一个 `std::bad_alloc` 异常。
- new是类型安全的，它会根据类型的大小自动分配内存。

#### 包含头文件
new的使用不需要包含头文件。

#### 内存释放
- 使用 `delete` 运算符释放由 new 分配的内存。
- 释放内存时，会调用对象的析构函数。
- 如果释放的是数组，需要使用 `delete[]` 运算符。

#### 分配策略
new 的内存是从堆中分配的，而不是从栈中分配的。这意味着分配的内存不会自动释放，需要手动释放。
当new 分配内存失败时，会抛出一个 `std::bad_alloc` 异常。
当分配的内存较大时，new会调用mmap函数来分配内存，这个过程可能会比较慢。

#### 注意事项
- new是C++中的运算符，运算符是可以重载的，所以new可以用于任何类型。
- new 在使用时，会调用对象的构造函数，而malloc不会调用构造函数
#### 相关函数
- `new` 运算符用于动态内存分配，并调用对象的构造函数。
- `delete` 运算符用于释放由 `new` 分配的内存，并调用对象的析构函数。
- `new[]` 运算符用于分配数组，并调用对象的构造函数。
- `delete[]` 运算符用于释放由 `new[]` 分配的数组，并调用对象的析构函数。
#### 示例

##### 常见用法
```cpp
#include <iostream>

int main() {
    // 分配单个 int，未初始化
    int* p1 = new int;
    
    // 分配并初始化
    int* p2 = new int(42);
    double* p3 = new double(3.14);
    char* p4 = new char('A');
    
    std::cout << "*p2 = " << *p2 << std::endl;  // 输出: 42
    std::cout << "*p3 = " << *p3 << std::endl;  // 输出: 3.14
    std::cout << "*p4 = " << *p4 << std::endl;  // 输出: A
    
    delete p1;
    delete p2;
    delete p3;
    delete p4;


    // 数组
    int size = 5;
    
    // 分配整型数组
    int* arr = new int[size];
    
    // 初始化数组
    for (int i = 0; i < size; i++) {
        arr[i] = i * 10;
    }
    
    // 使用数组
    for (int i = 0; i < size; i++) {
        std::cout << "arr[" << i << "] = " << arr[i] << std::endl;
    }
    
    delete[] arr;  // 注意使用 delete[]
    
    return 0;
}

```
##### 分配类对象
```cpp
#include <iostream>
#include <string>

class Person {
private:
    std::string name;
    int age;
    
public:
    // 构造函数
    Person(const std::string& n, int a) : name(n), age(a) {
        std::cout << "构造函数: " << name << std::endl;
    }
    
    // 析构函数
    ~Person() {
        std::cout << "析构函数: " << name << std::endl;
    }
    
    void display() const {
        std::cout << "姓名: " << name << ", 年龄: " << age << std::endl;
    }
};

int main() {
    // 分配 Person 对象
    Person* person = new Person("张三", 25);
    person->display();
    
    delete person;  // 调用析构函数
    
    return 0;
}

```

##### 定位new
```cpp

#include <iostream>
#include <new>  // 需要包含这个头文件

class MyClass {
public:
    MyClass(int val) : value(val) {
        std::cout << "构造函数, value = " << value << std::endl;
    }
    
    ~MyClass() {
        std::cout << "析构函数, value = " << value << std::endl;
    }
    
    void show() {
        std::cout << "value = " << value << std::endl;
    }
    
private:
    int value;
};

int main() {
    // 预分配内存缓冲区,大小为MyClass对象的大小
    char buffer[sizeof(MyClass)];
    
    std::cout << "缓冲区地址: " << (void*)buffer << std::endl;
    
    //在缓冲区内创建了对象
    MyClass* obj = new (buffer) MyClass(100);
    
    std::cout << "对象地址: " << obj << std::endl;
    obj->show();
    
    // 手动调用析构函数
    obj->~MyClass();
    
    return 0;
}
```

##### nothrow new
```cpp 
#include <iostream>
#include <new>
int main()
{
    try{
        auto* ptr = new int[1000000000000LL];// 分配一个超大的数组
        delete[] ptr;
     }catch(const std::bad_alloc& e){
         std::cout << "内存分配失败: " << e.what() << std::endl;
     }


     // 使用nothrow new
     int* ptr_nothrow = new(std::nothrow) int[1000000000000LL];
     if(ptr_nothrow == nullptr){
         std::cout << "内存分配失败" << std::endl;
     }else{
         delete[] ptr_nothrow;
     }

     return 0;
}
```

##### 重载new/delete
```cpp

#include <iostream>
#include <new>

class MyClass {

    private:
        int32_t data;
public:
     MyClass(int val) : data(val) {
        std::cout << "构造函数, value = " << data << std::endl;
     }
     ~MyClass() {
        std::cout << "析构函数, value = " << data << std::endl;
     }
    
    static void* operator new(size_t size)  {
        std::cout << "自定义 new"<< size << std::endl;
        void* ptr = ::operator new(size); // 调用全局的 new 操作符
        return ptr;
    }
    static void operator delete(void* ptr)
    {
        std::cout << "自定义 delete" << std::endl;

        ::operator delete(ptr); // 调用全局的 delete 操作符
    }
    static void* operator new[](size_t size)  {
        std::cout << "请求分配的字节数: " << size << std::endl;
        void* ptr = ::operator new[](size); // 调用全局的 new[] 操作符
        return ptr;
    }
    static void  operator delete[](void* ptr)
    {
        std::cout << "自定义数组删除" << std::endl;
        ::operator delete[](ptr); // 调用全局的 delete[] 操作符

    }
};

int main() {
    // 使用自定义的 operator new/delete
    MyClass* obj = new MyClass(42);
    delete obj;
    
    MyClass* arr = new MyClass[3]{1, 2, 3};
    delete[] arr;
    
    return 0;
}

```
PS:请求分配的字节数除以myClass对象大小，并不等于设置的数组元素个数
是因为 当使用 new[] 创建对象数组时，编译器会在实际数组元素之前分配额外的内存来存储数组信息;
编译器需要在 delete[] 时知道数组的大小，以便正确调用每个对象的析构函数。这个信息就存储在额外分配的内存中。
不同的编译器可能采用不同的方式存储元数据：
有些存储元素个数
有些存储数组的总大小
有些可能使用其他管理信息


#### 使用建议

1. 优先使用现代 C++ 特性--如智能指针和容器
```cpp
#include <memory>
#include <vector>

// 推荐：使用智能指针
void modern_way() {
    auto ptr = std::make_unique<int>(42);
    auto shared_ptr = std::make_shared<double>(3.14);
    
    // 自动管理内存，无需手动 delete
}

// 推荐：使用容器
void container_way() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    // 自动管理内存
}
```
2. RAII 模式
```cpp
class ResourceManager {
private:
    int* resource;
    
public:
    // 构造函数获取资源
    ResourceManager(size_t size) : resource(new int[size]) {}
    
    // 析构函数释放资源
    ~ResourceManager() {
        delete[] resource;
        std::cout << "资源已自动释放" << std::endl;
    }
    
    // 禁用拷贝（或实现移动语义）
    ResourceManager(const ResourceManager&) = delete;
    ResourceManager& operator=(const ResourceManager&) = delete;
    
    int* get() { return resource; }
};

void use_resource() {
    ResourceManager manager(100);  // 自动管理内存
    // 使用 manager.get()...
    // 函数结束时自动调用析构函数释放内存
}
```

## new和malloc的区别


### 语法差异

```cpp
class MyClass {
public:
    MyClass() { cout << "Constructor" << endl; }
    ~MyClass() { cout << "Destructor" << endl; }
}
// new 的使用
int* ptr1 = new int(10);           // 分配并初始化
int* arr1 = new int[5];            // 分配数组
MyClass* obj = new MyClass();      // 创建对象

// malloc 的使用
int* ptr2 = (int*)malloc(sizeof(int));  // 分配内存
int* arr2 = (int*)malloc(5 * sizeof(int)); // 分配数组
```

### 主要区别

| 特性 | new | malloc |
|------|-----|---------|
| **语言** | C++运算符 | C库函数 |
| **类型安全** | 是 | 否 |
| **构造函数调用** | 自动调用 | 不调用 |
| **返回类型** | 正确类型指针 | void*，需强制转换 |
| **失败处理** | 抛出std::bad_alloc异常 | 返回NULL |
| **内存位置** | 自由存储区(free store) | 堆(heap) |
| **重载可能性** | 可以重载 | 不能重载 |
| **释放方式** | delete / delete[] | free() |

### 初始化差异

```cpp
// new 会自动初始化
int* p1 = new int(5);        // 值为5
int* p2 = new int();         // 值为0（默认初始化）

// malloc 不会初始化
int* p3 = (int*)malloc(sizeof(int));  // 包含随机值
int* p4 = (int*)calloc(1, sizeof(int)); // 值为0
```

### 对象创建示例

```cpp
class MyClass {
private:
    int value;
public:
    MyClass() : value(0) { cout << "Constructor called\n"; }
    ~MyClass() { cout << "Destructor called\n"; }
    void setValue(int v) { value = v; }
};

// 使用 new
MyClass* obj1 = new MyClass();  // 调用构造函数
delete obj1;                    // 调用析构函数

// 使用 malloc
MyClass* obj2 = (MyClass*)malloc(sizeof(MyClass));  // 不调用构造函数
free(obj2);                                         // 不调用析构函数
```

### 错误处理

```cpp
// new 的错误处理
try {
    int* ptr = new int[1000000000];
} catch (std::bad_alloc& e) {
    cout << "Allocation failed: " << e.what() << endl;
}

// malloc 的错误处理
int* ptr = (int*)malloc(1000000000 * sizeof(int));
if (ptr == NULL) {
    cout << "Allocation failed" << endl;
}
```

###  数组分配

```cpp
// new 方式
int* arr1 = new int[10];
delete[] arr1;  // 使用 delete[] 释放

// malloc 方式
int* arr2 = (int*)malloc(10 * sizeof(int));
free(arr2);     // 使用 free 释放
```

### 混合使用的问题

```cpp
// 错误的做法 - 混合使用会导致未定义行为
int* ptr1 = new int(10);
free(ptr1);  // 错误！应该使用 delete

int* ptr2 = (int*)malloc(sizeof(int));
delete ptr2;  // 错误！应该使用 free
```
## 总结

总的来说，new 是 C++ 的标准运算符，用于分配和初始化对象，而 malloc 是 C 语言的库函数，用于分配内存。new 的主要优点是会自动调用构造函数，而 malloc 则不会。但是，malloc 的主要优点是它允许我们分配任意大小的内存，而 new 只允许分配指定类型的内存。
