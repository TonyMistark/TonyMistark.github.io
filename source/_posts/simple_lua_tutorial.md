---
layout: layout
title: Lua 简单入门教程
date: 2025-02-13 17:49:57
tags:
---

Lua 是一种轻量级、高效、可嵌入的脚本语言，广泛应用于游戏开发（如《魔兽世界》、《愤怒的小鸟》）、嵌入式系统和其他需要脚本扩展的场景。以下是一个 Lua 的入门教程，帮助你快速掌握 Lua 的基础知识。

## **1. Lua 环境搭建**
### **安装 Lua**
- **Windows**:
  1. 下载 Lua 安装包：访问 [Lua 官方网站](https://www.lua.org/download.html)。
  2. 解压并配置环境变量。
- **Linux/macOS**:
  使用包管理器安装：
  ```bash
  sudo apt-get install lua5.3  # Ubuntu/Debian
  brew install lua             # macOS
  ```

### **运行 Lua 代码**
- 在终端中输入 `lua` 进入交互式解释器。
- 也可以将代码保存为 `.lua` 文件，然后运行：
  ```bash
  lua filename.lua
  ```

---

## **2. Lua 基础语法**
### **注释**
- 单行注释：
  ```lua
  -- 这是单行注释
  ```
- 多行注释：
  ```lua
  --[[
    这是多行注释
    可以写多行内容
  ]]
  ```

### **变量**
- Lua 是动态类型语言，变量无需声明类型。
- 变量默认是全局的，除非用 `local` 声明为局部变量。
  ```lua
  a = 10          -- 全局变量
  local b = 20    -- 局部变量
  print(a, b)     -- 输出: 10 20
  ```

### **数据类型**
Lua 支持以下基本数据类型：
- **nil**: 表示空值或未定义。
- **boolean**: `true` 或 `false`。
- **number**: 整数或浮点数。
- **string**: 字符串，用单引号或双引号包裹。
- **table**: 表（类似数组或字典）。
- **function**: 函数。
- **userdata**: 用户自定义数据。
- **thread**: 协程。

示例：
```lua
print(type("Hello"))  -- 输出: string
print(type(3.14))     -- 输出: number
print(type(true))     -- 输出: boolean
print(type(nil))      -- 输出: nil
```

---

## **3. 控制结构**
### **条件语句**
- `if` 语句：
  ```lua
  local age = 18
  if age >= 18 then
      print("You are an adult.")
  else
      print("You are a minor.")
  end
  ```

- `elseif`：
  ```lua
  local score = 85
  if score >= 90 then
      print("A")
  elseif score >= 80 then
      print("B")
  else
      print("C")
  end
  ```

### **循环**
- `while` 循环：
  ```lua
  local i = 1
  while i <= 5 do
      print(i)
      i = i + 1
  end
  ```

- `for` 循环：
  ```lua
  for i = 1, 5 do
      print(i)
  end
  ```

- `repeat...until` 循环：
  ```lua
  local i = 1
  repeat
      print(i)
      i = i + 1
  until i > 5
  ```

---

## **4. 函数**
- 定义函数：
  ```lua
  function add(a, b)
      return a + b
  end
  print(add(3, 5))  -- 输出: 8
  ```

- 匿名函数：
  ```lua
  local mul = function(a, b)
      return a * b
  end
  print(mul(3, 5))  -- 输出: 15
  ```

---

## **5. 表（Table）**
表是 Lua 中唯一的数据结构，可以用作数组、字典或对象。
- 数组：
  ```lua
  local arr = {1, 2, 3, 4, 5}
  print(arr[1])  -- 输出: 1（Lua 索引从 1 开始）
  ```

- 字典：
  ```lua
  local dict = {name = "Lua", version = "5.3"}
  print(dict["name"])  -- 输出: Lua
  print(dict.version)  -- 输出: 5.3
  ```

---

## **6. 模块与包**
- 定义模块：
  ```lua
  -- mymodule.lua
  local M = {}
  function M.add(a, b)
      return a + b
  end
  return M
  ```

- 使用模块：
  ```lua
  local mymodule = require("mymodule")
  print(mymodule.add(3, 5))  -- 输出: 8
  ```

---

## **7. 文件操作**
- 读取文件：
  ```lua
  local file = io.open("test.txt", "r")
  local content = file:read("*a")
  print(content)
  file:close()
  ```

- 写入文件：
  ```lua
  local file = io.open("test.txt", "w")
  file:write("Hello, Lua!")
  file:close()
  ```

---

## **8. 错误处理**
- 使用 `pcall` 捕获错误：
  ```lua
  local status, err = pcall(function()
      error("Something went wrong!")
  end)
  if not status then
      print("Error:", err)
  end
  ```

---

## **9. 协程**
Lua 支持协程（coroutine），用于实现协作式多任务。
- 示例：
  ```lua
  local co = coroutine.create(function()
      print("Hello")
      coroutine.yield()
      print("World")
  end)

  coroutine.resume(co)  -- 输出: Hello
  coroutine.resume(co)  -- 输出: World
  ```

---

## **10. 实战示例**
### **计算斐波那契数列**
```lua
function fibonacci(n)
    if n <= 1 then
        return n
    else
        return fibonacci(n - 1) + fibonacci(n - 2)
    end
end

for i = 0, 10 do
    print(fibonacci(i))
end
```

---

## **11. 学习资源**
- [Lua 官方文档](https://www.lua.org/docs.html)
- 《Programming in Lua》（Lua 编程指南）
- [Lua Tutorial on TutorialsPoint](https://www.tutorialspoint.com/lua/index.htm)

---
