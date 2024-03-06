---
title: C++与python日常使用区别小记
date: 2024-01-31
---

### 数组（列表）使用

Python:
```python
nums = []  # 创建空列表
nums.append(1)  # 添加元素
nums.append(2)
len(nums)  # 获取列表长度
nums[0]  # 访问元素
value_to_find in nums  # 查找值
```

C++:
```cpp
#include <vector>

std::vector<int> nums;  // 创建空向量
nums.push_back(1);  // 添加元素
nums.push_back(2);
nums.size();  // 获取向量大小
nums[0];  // 访问元素
std::find(nums.begin(), nums.end(), value_to_find) != nums.end(); //查找值
```

### 字符串操作

Python:
```python
str = "hello"
len(str)  # 获取字符串长度
str[1]  # 访问字符
str.find("e")  # 查找子字符串
"world" in str  # 子字符串检查
```

C++:
```cpp
#include <string>

std::string str = "hello";
str.length();  // 获取字符串长度
str[1];  // 访问字符
str.find("e");  // 查找子字符串
str.find("world") != std::string::npos;  // 子字符串检查
```

我明白了，您需要的是在 Python 和 C++ 中集合操作的对比。让我为您展示这两种语言中集合的基本用法：

### 集合

Python:
```python
# 创建和操作集合
nums = {1, 2, 3, 4, 5}  # 创建集合
nums.add(6)  # 添加元素
nums.remove(2)  # 移除元素
3 in nums  # 检查元素是否存在
```

C++:
```cpp
#include <unordered_set>

// 创建和操作集合
std::unordered_set<int> nums = {1, 2, 3, 4, 5};  // 创建集合
nums.insert(6);  // 添加元素
nums.erase(2);  // 移除元素
dict.count(3); // 检查元素是否存在
```

在这两种语言中，集合都用于存储唯一元素。Python 使用 `{}` 语法创建集合，而 C++ 中则使用 `std::unordered_set`。两种语言都提供了添加和移除元素的方法，以及检查元素是否存在的方法。不过，需要注意的是，在 C++ 中检查元素是否存在是通过 `find` 方法完成的，它返回一个迭代器，如果找到元素，迭代器不会等于 `end()` 迭代器。

### 字典（映射）使用

Python:
```python
dict = {}  # 创建字典
dict["key"] = "value"  # 添加键值对
dict.get("key")  # 获取值
"key" in dict  # 检查键存在
```

C++:
```cpp
#include <map>

std::map<std::string, std::string> dict;  // 创建映射
dict["key"] = "value";  // 添加键值对
dict["key"];  // 获取值
dict.count("key");  // 检查键存在
```

### 排序

Python:
```python
nums = [3, 1, 4, 1, 5, 9, 2]
nums.sort()  # 原地排序
sorted_nums = sorted(nums)  # 返回新的排序列表
```

C++:
```cpp
#include <vector>
#include <algorithm>

std::vector<int> nums = {3, 1, 4, 1, 5, 9, 2};
std::sort(nums.begin(), nums.end());  // 原地排序
std::vector<int> sorted_nums(nums);  // 创建副本
std::sort(sorted_nums.begin(), sorted_nums.end());  // 排序副本
```

### 多线程

Python:
```python
import threading

def task():
    # 执行任务
    pass

thread = threading.Thread(target=task)
thread.start()
thread.join()  # 等待线程结束
```

C++:
```cpp
#include <thread>

void task() {
    // 执行任务
}

std::thread thread(task);
thread.join();  // 答应线程结束
```
