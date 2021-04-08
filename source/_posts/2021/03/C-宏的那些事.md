---
title: C++宏定义的那些事
date: 2021-03-23 21:57:57
tags: 
- C++ 
categories:
- C++相关
---

# 宏定义
宏定义将一个标识符定义为一个字符串，源代码中的标识符将被指定的字符串替换，这个过程发生在预处理阶段。
发方
C++程序的完整编译过程包括预处理、编译、汇编、链接四个步骤。其中预处理阶段进行宏展开、条件编译等。

在C语言中，通过宏来定义简单函数，可以降低函数调用过程中的开销，而在c++中，可以用`inline`来声明内联函数来降低函数调用过程的开销。

也可以通过宏来定义一些常量，但由于宏替换发生在预处理阶段，因此如果在编译过程中产生错误，很难进行错误的定位和溯源。

# 简单的宏定义
```c++
//第一种
#define PI2 3.1415926
#define Add(x) x+1 //错误示范
#define Double(x) (x*2) //错误示范
// Add(x) * 2 ==> x + 1 * 2
#define Add(x) ((x)+1)
#define Double(x) ((x)*2)


//第二种
#ifndef SOURCE_H
#define SOURCE_H

#define TEST
#ifdef TEST
std::cout << "this is a TEST " << endl;
#ifndef TEST
std::cout << "there are not TEST" << endl;
#endif
```
宏的两种简单用法如上所示，第一种用法将`PI`全部替换为`3.1415926`，这里的替换是直接展开的，如错误示范中所示，因此我们在使用宏定义时，要利用括号保证其在展开时不会产生错误。

第二种用法通过宏定义来实现条件编译、避免头文件的重复引入。

# 宏的高阶用法
`#`、`##`、`#@`三种可以称之为宏的高阶用法，可以实现许多高级特性。

```c++
#define STRING(msg) #msg
char* str = "hello";
char* str = STRING(hello);  //二者等价


#define CHAR(ch) #ch
char ch = 'a';
char ch = CHAR(a); //二者等价

#define 
```
当宏中遇到`#`和`##`时，是不能够进行嵌套替换的，不会对`#`和`##`之后宏进行展开。

# 可变参数宏

```c++
#define VARGS_(_10, _9, _8, _7, _6, _5, _4, _3, _2, _1, N, ...) N
#define VARGS(...) VARGS_(__VA_ARGS__, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0)


#define CONCAT_(a, b) a ## b
#define CONCAT(a, b) CONCAT_(a, b) // 由于##会阻止宏的展开，因此两次宏替换解决宏的嵌套问题

#define FUNC_3(prefix, name, n) func(prefix, name, n)
#define FUNC_2(prefix, name) FUNC_3(prefix, name, 1)

#define FUNC(...) CONCAT("FUNC_", VARGS(__VA_ARGS__))(__VA_ARGS__)


// 从而实现如下变参宏
FUNC(prefix, name, 4)
FUNC(prefix, name)
```

# 工厂宏
```c++
class Any {
public:
    Any() : _var_ptr(NULL) {}
    template<typename T>
    Any(const T& value) : _var_ptr(new Type<T>(value)) {}
    Any(const Any& other) : _var_ptr(other._var_ptr ? other._var_ptr->clone() : NULL) {}
    ~Any() {
        delete _var_ptr;
    }
    template<typename T>
    T* any_cast() {
        return _var_ptr ? &static_cast<Type <T>*>(_var_ptr)->_var : NULL;
    }

private:
    class Typeless {
    public:
        virtual ~Typeless() {}
        virtual Typeless* clone() const = 0;
    };
    /// @brief Type calss template to hold a specific type
    template<typename T>
    class Type : public Typeless {
    public:
        explicit Type(const T& value) : _var(value) {}
        virtual Typeless* clone() const {
            return new Type(_var);
        }
        T _var;             ///< The real variable of a specific type
    };
    Typeless* _var_ptr;     ///< Typeless variable pointer
};

struct ConcreteFactory {
    typedef Any (* FactoryIntf)();  /// a Function Pointer 

    FactoryIntf get_instance;      ///< Function pointer to get instance
    FactoryIntf get_singleton;     ///< Function pointer to get singleton
};

typedef std::map<std::string, ConcreteFactory> FactoryMap;
typedef std::map<std::string, FactoryMap> BaseClassMap;

BaseClassMap& g_factory_map() {
    static BaseClassMap base_class_map;    
    return base_class_map;
}

#define REGISTER_FACTORY(base_class) \
    class base_class ## Factory { \
    public: \
        static base_class *get_instance(const ::std::string &name) { \
            FactoryMap &map = g_factory_map()[#base_class]; \
            FactoryMap::iterator iter = map.find(name); \
            if (iter == map.end()) { \
                return NULL; \
            } \
            Any object = iter->second.get_instance(); \
            return *(object.any_cast<base_class*>()); \
        } \
        static base_class* get_singleton(const ::std::string& name) { \
            FactoryMap& map = g_factory_map()[#base_class]; \
            FactoryMap::iterator iter = map.find(name); \
            if (iter == map.end()) { \
                return NULL; \
            }\
            Any object = iter->second.get_singleton(); \
            return *(object.any_cast<base_class*>()); \
        } \
        static const ::std::string get_uniq_instance_name() { \
            FactoryMap &map = g_factory_map()[#base_class]; \
            if (map.empty() || map.size() != 1) { \
                return ""; \
            } \
            return map.begin()->first; \
        } \
        static base_class *get_uniq_instance() { \
            FactoryMap &map = g_factory_map()[#base_class]; \
            if (map.empty() || map.size() != 1) { \
                return NULL; \
            } \
            Any object = map.begin()->second.get_instance(); \
            return *(object.any_cast<base_class*>()); \
        } \
        static bool is_valid(const ::std::string &name) { \
            FactoryMap &map = g_factory_map()[#base_class]; \
            return map.find(name) != map.end(); \
        } \
        static std::vector<std::string> list_class() {\
            std::vector<std::string> ret; \
            auto &m = g_factory_map()[#base_class]; \
            for (auto& iter : m) { \
              ret.emplace_back(iter.first);\
            }\
            return ret;\
        }\
    }; \


#define REGISTER_CLASS(base_class, sub_class) \
    namespace { \
    Any sub_class##get_instance() { \
        return Any(new sub_class()); \
    } \
    Any sub_class##get_singleton() { \
        return Any(Singleton<sub_class>::get()); \
    } \
    __attribute__((constructor)) void register_factory_##sub_class() { \
        FactoryMap &map = g_factory_map()[#base_class]; \
        if (map.find(#sub_class) == map.end()) { \
            ConcreteFactory factory = {&sub_class##get_instance, \
                                                 &sub_class##get_singleton}; \
            map[#sub_class] = factory; \
        } \
    } \
    }


```

```c++
typedef char (*func)(int); //函数指针
```