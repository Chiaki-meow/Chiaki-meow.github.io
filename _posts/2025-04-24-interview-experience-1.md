---
layout: post
title: 图形开发相关面经分享整理 1.0
date: 2025-04-24 20:30:00
description: 
tags: graphics
categories: coding
---

以下内容是我+chatgpt联手整理的！可靠性性不能完全保证，做为我自己留存学习用:)
欢迎大家一起学习分享和参考！


# 图形学：

## 图形渲染管线原理：

正常分为四个阶段 - 应用程序阶段、几何阶段、光栅化阶段和像素处理阶段。

https://github.com/ssloy/tinyrenderer

### Vertex Shader & Fragment Shader

在现代渲染管线（Graphics Pipeline）中，**Vertex Shader（顶点着色器）** 和 **Fragment Shader（片段着色器）** 扮演着不同的角色：

**Vertex Shader 负责：**

1️⃣ **顶点变换**（Model Space → Clip Space → Screen Space）

2️⃣ **法线变换**（用于光照计算）

3️⃣ **纹理坐标计算**（用于纹理映射）

4️⃣ **顶点颜色计算**（可用于渐变色、顶点动画）

**光栅化（Rasterization）**（固定管线，不属于 FS）

•由 GPU **将顶点数据转换为像素级 Fragment**

•计算插值数据（颜色、纹理坐标、法线等）

**Fragment Shader 负责：**

1️⃣ **颜色计算**（包括纹理采样、光照计算）

2️⃣ **透明度处理**（Alpha Blending、Alpha Test）

3️⃣ **阴影、法线贴图等高级效果**

4️⃣ **后处理（Post-processing）**（如 HDR、Bloom）

### 采样原理：

### PBR：

**PBR（Physically Based Rendering，基于物理的渲染）** 是一种 **基于物理真实的光照计算** 方法。它的目标是 **模拟现实世界中的光线反射、吸收和散射**，让材质看起来更加真实。

PBR 主要有两个核心部分：

**BRDF（双向反射分布函数，Bidirectional Reflectance Distribution Function）**

计算 **光线如何与表面相互作用**

结合 **漫反射（Diffuse）+ 镜面反射（Specular）**

**IBL（基于图像的光照，Image-Based Lighting）**

通过 **环境贴图** 模拟 **复杂光源对物体的影响**

让物体 **更真实地适应环境光照**

**PBR 主要公式：Cook-Torrance 反射模型**

PBR 采用 **Cook-Torrance BRDF** 来计算光照，公式如下：

$$
f(l, v) = \frac{D(h) \cdot F(v, h) \cdot G(l, v, h)}{4 (n \cdot v) (n \cdot l)}
$$

其中：

•**D(h)** - 法线分布函数（Normal Distribution Function，NDF）

•**F(v, h)** - 菲涅尔反射（Fresnel Effect）

•**G(l, v, h)** - 几何遮挡函数（Geometric Attenuation）

这些因素决定了 **材质的真实感**，比如金属、塑料、玻璃等材质的光照行为。

```cpp
// 计算 Cook-Torrance BRDF
// DistributionGGX - 计算法线分布
// FresnelSchlick - 计算菲涅尔效应（角度越大，反射越强）
float DistributionGGX(vec3 N, vec3 H, float roughness) {
    float a = roughness * roughness;
    float a2 = a * a;
    float NdotH = max(dot(N, H), 0.0);
    float NdotH2 = NdotH * NdotH;
    
    float num = a2;
    float denom = (NdotH2 * (a2 - 1.0) + 1.0);
    denom = 3.14159265 * denom * denom;
    
    return num / denom;
}

// 菲涅尔反射
vec3 FresnelSchlick(float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
```

### DX12的raytracing demo：

### gbuffer：

主要是把数据全都存在了gbuffer里，针对于多光源场景下，这种计算方法会让我们计算更少的面片数目。但是需要注意的是，gbuffer并不能支持渲染透明面片。

在硬件层面上，在vertex shader前会有一些背面剔除的操作。但是物体间相互遮挡的面片仍然会经过vertex shader到fragment shader里。这种情况如果不剔除的话，就会重复计算很多相互遮挡的面片。加上非常多的光源，就会导致我们计算很多实际上看不到的光照数据。

因此，有人提出了gbuffer来解决这个问题：延迟渲染会分为geometry pass和lighting pass的部分，geometry pass会在vertex shader里给gbuffer赋值，其中可以包含法线、深度、漫反射强度、镜面反射强度等等；lighting pass会在fragment shader里进行执行。lighting pass会读取gbuffer所存储的信息，在屏幕的每一个像素点上计算光照的贡献度。

[https://learnopengl-cn.readthedocs.io/zh/latest/05 Advanced Lighting/08 Deferred Shading/](https://learnopengl-cn.readthedocs.io/zh/latest/05%20Advanced%20Lighting/08%20Deferred%20Shading/)

### WEBGL是怎么渲染在页面上的：

### 蒙皮动画系统：

# 游戏引擎：

### **Entity Component System (ECS)：**

### **RAII（Resource Acquisition Is Initialization）**

RAII是 C++ 中的一种 **资源管理** 方式，核心思想是**将资源（如内存、文件句柄、GPU 资源）绑定到对象的生命周期上**，从而实现资源的自动释放，避免手动管理带来的内存泄漏问题。

```cpp
class Texture {
public:
    Texture(const std::string& filePath) {
        glGenTextures(1, &textureID);
        glBindTexture(GL_TEXTURE_2D, textureID);
        // 加载纹理数据（伪代码）
        loadTextureFromFile(filePath);
    }

    ~Texture() {  
        glDeleteTextures(1, &textureID);  // 自动释放 GPU 资源
    }

    void bind() const {
        glBindTexture(GL_TEXTURE_2D, textureID);
    }

private:
    GLuint textureID;  // OpenGL 纹理ID
};
```

# 操作系统:

### Malloc一个16g内存，操作系统会发生什么：

### 如何Debug一个大型的应用，判断他的瓶颈在哪里：

# C++：

### C++特性整理内容：

**版本**	**主要特性**

| **版本** | **主要特性** |
| --- | --- |
| **C++11** | auto✅、nullptr✅、右值引用✅、std::thread、std::unique_ptr |
| **C++14** | 泛型 lambda、std::make_unique、二进制字面量 |
| **C++17** | 结构化绑定、std::optional、std::variant、if constexpr |
| **C++20** | concepts、std::ranges、std::span、协程 |
| **C++23** | std::expected、if consteval |

### 引用和指针的区别：

https://csguide.cn/cpp/memory/difference_of_pointers_and_ref.html#%E5%8C%BA%E5%88%AB

| 指针 | 引用 |
| --- | --- |
| 指针是一个变量，保存了另一个变量的内存地址 | 引用是另一个变量的别名，与原变量共享内存地址 |
| 指针可以被重新赋值，指向不同的变量 | 引用在**初始化之后不能更改**，始终指向同一个变量 |
| 指针可以为nullptr，不指向任何一个变量 | 引用必须绑定到一个变量，不能为nullptr |
| 指针需要解引用以 获取/修改 其指向变量的值 | 引用可以直接使用，不需要符号解引用 |

### 静态（static）

全局变量：限制作用于，仅在当前文件可见

局部变量：改变存储周期，使得变量在整个程序生命周期内存在

类的成员变量：所有对象共享，属于类而不是属于对象

类的成员函数：不能访问非static成员、不依赖对象

### 类里的static/静态成员函数可以被继承吗：

可以（如果你觉得不override也是继承的话）：

1. 静态成员函数属于类，而不属于对象实例。
2. 静态成员函数需要通过类名进行直接调用，而不需要通过对象来访问。比如可以直接调用MyClass::staticFunction()。
3. 静态成员函数不能访问类的非静态成员（成员变量或成员函数），因为静态函数没有this指针

静态函数不会像普通的函数一样被继承和重写（毕竟没有this指针），但是派生类可以**直接调用**基类的静态函数。同时，如果派生类声明了一个和基类同名的静态函数，那么派生类将会**隐藏**基类的同名静态函数。这里是隐藏！而不是override

### 类里的static/静态成员函数可以是虚函数吗：

不可以：

1. 静态成员函数没有this指针：
    1. 静态函数属于类本身，而非对象实例。调用的时候不通过对象调用，因此不存在this指针；
    2. 虚函数的多态依靠对象的动态类型，其调用需要通过对象的虚函数表（vtable）查找正确的实现，而vtable的访问依赖于this指针
2. 静态函数与对象实例无关：
    1. 静态函数的调用不依赖于任何的对象实例，他们只能访问静态成员变量或其他静态函数；
    2. 虚函数的调用依赖于对象的实例，this指针指向了每个实例的不同属性
3. 静态函数可以通过对象的引用来调用虚函数

    ```cpp
    #include <iostream>
    
    class Base {
    public:
        virtual void display() { std::cout << "Base class display" << std::endl; }
    
        static void staticDisplay(Base& obj) {
            // 通过传入的对象来调用虚函数
            obj.display();  // 正确：可以通过对象来调用虚函数
        }
    };
    
    class Derived : public Base {
    public:
        void display() override { std::cout << "Derived class display" << std::endl; }
    };
    
    int main() {
        Derived d;
        Base::staticDisplay(d);  // 正确：静态成员函数通过对象来调用虚函数
        return 0;
    }
    ```


### 动态链接库和静态链接库的区别：

动态链接库已经编译好了，但是静态链接库需要跟着程序一起编译之后才能融入到系统的可执行文件里。因此如果我们已经有一个打包好的程序，我们可以直接通过更换动态链接库来实现对于库内部的函数的更新，而不需要去重新编译整个项目来解决这个问题。

### **volatile**的作用：

最主要的功能：阻止编译器对于代码的优化，确保每次访问时都重新从内存中读取该变量的值

1. 硬件寄存器：如果不加volatile，代码可能会觉得相同的两次读取硬件寄存器的数值不会改变，不再从硬件中读取相应的数值。
2. 多线程处理：

    ```cpp
    volatile bool flag = false;
    
    // 线程1
    void thread1() {
        flag = true;
    }
    
    // 线程2
    void thread2() {
        while (!flag) {
            // 等待 flag 被线程1设置为 true
        }
        // 执行一些操作
    }
    ```

   如果没有volatile，编译器可能会假设flag在thread2不会改变，导致thread2进入死循环

3. 信号处理

### 类里默认会实现的函数是什么？

默认构造函数：如果显示的定义一个含参构造函数，就不会定义默认构造函数了

拷贝构造函数：默认是浅拷贝；如果有需要动态分配资源的内容，需要额外写深拷贝的构造函数

拷贝构造运算符：ClassName& operator=(const ClassName& other); 和拷贝构造函数一样，也是浅拷贝

析构函数：不会释放动态分配的内存

移动构造函数：ClassName(ClassName&& other); 默认的移动构造函数是浅移动

移动构造运算符：ClassName& operator=(ClassName&& other); 默认的移动构造运算符也是浅移动

### 实现一个String：

https://www.cnblogs.com/downey-blog/p/10470912.html

今天才看完这个链接，感觉总结出来有用的就三个点：

- C++需要支持“+”等等的运算符，所以要进行运算符重载
- 需要自己实现一个迭代器
- 支持字符串自动扩容的相关内容

### 实现一个读写锁：

写在CppPractices里了，除了实现RWLock本身，还需要注意也实现了

### 实现一个shared_ptr：

写在CppPractices里了，重点是存T* ptr和int* count，以及重新写移动与拷贝构造/运算符的逻辑。在只对指针进行操作的话会让这个过程变得区别更大。

**std::enable_shared_from_this<T>**

如果类的成员函数内部需要获得shared_ptr<T>，如果直接使用return(this) 会让两个shared_ptr相互独立，并不是共用同一个shared_ptr的引用内容

通过std::enable_shared_from_this<T>可以让T内部安全的获取指向自己的shared_ptr<t>。

### 虚函数相关:

https://cloud.tencent.com/developer/article/1510207

虚表vtable保存在静态数据区/全局数据区，每一个对象会有一个指向vtable的指针（vptr，虚指针）

虚表：每一个有虚函数的类都有一张vtable，保存了该类所有虚函数的地址

虚指针：每一个具体对象都包含一个虚指针，

## C++特性

### auto

`auto` 可以在声明变量时自动推导变量类型，并初始化为默认值。

```cpp
auto x = 42;       // x 被推导为 int
auto y = 3.14;     // y 被推导为 double
auto str = "Hello"; // str 被推导为 const char*

auto a; // ❌ 错误，auto 需要初始化值
```

但注意： `auto` 必须有初始化值来推导类型，否则会报错

---

`auto` 的特性：

指针：`auto` 会保持变量的指针或者引用类型：`auto` 默认去掉引用，如果需要保持需使用`auto&`

```cpp
int a = 10;
int& ref = a;
auto b = ref;  // b 被推导为 int，而不是 int&
b = 20;        // 修改 b 不会影响 a

auto& c = ref; // c 被推导为 int&
c = 30;        // 修改 c 会影响 a
```

---

### noexcept：

https://blog.csdn.net/qiuguolu1108/article/details/114796903

### 左值和右值的区别：

左值：可以取地址，因为对象有一个明确的内存地址；可以修改对应的数值。

右值：不能取地址；通常是临时变量，并且生命周期非常短。

左值引用：T&

右值引用：T&&

```cpp
// 左值引用
int a = 5;
int &ref = a;

// 右值引用
int && ref = 10+20;
```

右值引用用来和**移动语义**配合。

### std::move()和std::forward()

**std::move**：将变量转换为右值引用（T&&）

**使用场景**：用于**避免拷贝，提高性能**，特别适用于 **大对象转移（如 std::vector, std::string, std::unique_ptr 等）**。其实主要还是进行转移数据，而不是要复制一遍。

**std::forward<T>(arg):（完美转发）保持函数模板参数的左值/右值属性**，让参数在转发时不会失去其原始类型，常用于**模板函数**，防止参数不必要的拷贝或移动。

**使用场景**：主要在 **泛型代码（如构造函数转发、工厂模式）** 中，避免左值误用 std::move 导致资源被错误地移动。

```cpp
template<typename T, typename... Args>
std::unique_ptr<T> createObject(Args&&... args) {
    return std::make_unique<T>(std::forward<Args>(args)...);
}
```

## C++ STL小技巧

### Vector reserve和resize

reserve()的主要作用是预先分配内存空间，主要更改的是capacity()（容量，看后面会不会需要扩容），而不是直接更改vector的size()

- 如果在reverse之后再进行push_back()，不会有内存的重新分配的过程！（说明已经预先找到了一个足够大的空间了）

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec;

    // 不调用 reserve，直接添加元素
    for (int i = 0; i < 10; ++i) {
        vec.push_back(i);
    }

    std::cout << "Size: " << vec.size() << ", Capacity: " << vec.capacity() << std::endl;
		// Size: 10, Capacity: 16
    return 0;
}
```

如果没有提前reserve的话，会发现这里的capacity会随着翻倍逐渐变成16！

reserve只会增加容量。如果希望减少的话可以考虑用shrink_to_fit()

resize()的主要作用是直接去更改vector的size()，这会根据resize的参数去决定是扩大数组还是缩小数组。

- 如果 n 小于当前大小，元素会被删除。
- 如果 n 大于当前大小，vector 会新增元素，元素会被默认初始化（或者通过提供的值初始化）。

因此，resize其实会调用类型的构造函数或者是析构函数！

### 对象池

## C++ Coding技巧

### split string

```cpp
// 存在一个以空格为隔断的string s, 如s = "cat dog cat"
istringstream str(s);
string out;
vector<string> split_s;

while(str>>out){
	split_s.push_back(out);
}
```