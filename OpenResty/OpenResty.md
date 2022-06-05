Lua笔记

### 安装Lua

```bash
wget http://luajit.org/download/LuaJIT-2.1.0-beta3.tar.gz
tar -xvf LuaJIT-2.1.0-beta3.tar.gz
cd LuaJIT-2.1.0-beta3
make 
sudo make install
```

由于LuaJIT 2.1 目前还是beta版本，所以在make install后，并没有进行luajit的符号连接，可 

以执行下面的指令将luajit-2.1.0-beta1和luajit进行软连接，从而可以直接使用luajit命令

```bash
sudo ln -sf luajit-2.1.0-beta3 /usr/local/bin/luajit
```

验证 **LuaJIT** 是否安装成功 

```bash
luajit -v
```

### 第一个Hello World

首先编写一个lua文件

```bash
cat hello.lua
```

```lua
print("hello world")
```

然后使用LuaJIT运行

```bash
luajit hello.lua
```

### Lua基础数据类型

#### type

返回一个值或一个变量所属的类型

```lua
print(type("hello world")) -->output:string 
print(type(print)) -->output:function 
print(type(true)) -->output:boolean 
print(type(360.0)) -->output:number 
print(type(nil)) -->output:nil
```

#### nil

nil是一种类型

一个变量在第一次赋值前的默认值是 nil

**将 nil 赋给一个全局变量就等同于删除它**

```lua
local num 
print(num) -->output:nil 
num = 100 
print(num) -->output:100
```

**OpenResty 的 Lua 接口还提供了一种特殊的空值，即 ngx.null**

- 用来表示 不同于 nil 的“空值”。

#### boolean

布尔类型：true/false

**Lua中nil和false为假**，其他所有为真

- 0和空字符串就是真

```lua
local a = true 
local b = 0 
local c = nil 
if a then print("a") -->output:a 
else print("not a") --这个没有执行 
end if b then print("b") -->output:b elseprint("not b") --这个没有执行 end if c then print("c") --这个没有执行 elseprint("not c") -->output:not c end
```

#### number（实数）

number类型用于表示**实数**，和C/C++里面double类型很类似

- 可以用数学函数，math.floor（向下取整）和math.ceil（向上取整）
- **使用函数与四舍五入无关**

```lua
local order = 3.99 
local score = 98.01 
print(math.floor(order)) -->output:3 
print(math.ceil(score)) -->output:99
```

Lua的number类型是用**双精度浮点数**来实现的

LuaJIT 支持所 谓的“dual-number”（双数）模式，

- **即 LuaJIT 会根据上下文用整型来存储整数，而用双精度浮 点数来存放浮点数。**

LuaJIT支持longlong类型（64位整数）

```lua
print(9223372036854775807LL - 1) -->output:9223372036854775806LL
```

#### string

三种方式表示字符串

- 使用一对匹配的单引号，例如：'hello'
- 使用一对匹配的双引号，例如：“abcd”
- 长括号（里面不会进行转义），例如：``[[]]``
  - 我们把两个正的方括号（即[[）间插入 n 个等号定义为第 n 级正长括号
  - 0 级正的长括号写作 [[
  - 一级正的长括号写作 [=[
  - 4 级反的长括号写作 ]====]

**Lua的字符串是不可改变的值，不能通过下标进行访问，如果要修改字符只能创建新字符串**

```lua
local str1 = 'hello world' 
local str2 = "hello lua" 
local str3 = [["add\name",'hello']] 
local str4 = [=[string have a [[]].]=] 
print(str1) -->output:hello world 
print(str2) -->output:hello lua 
print(str3) -->output:"add\name",'hello' 
print(str4) -->output:string have a [[]].
```

**Lua字符串一般会经历一个“内化”（intern）的过程**

- 两个完全一样的Lua字符串在Lua虚拟机中只会存储一份
- **每一个Lua字符串在创建时都会插入到Lua虚拟机内部的一个全局的哈希表中**
  - Lua字符串之间进行相等性比较的时候是O（1）而不是O（n）
  - 创建相同的Lua字符串不会进行新的内存分配，但是仍有全局哈希表查询的开销

#### table（表）

table类型实现了一种抽象的“关联数组”。——具有特殊索引方式的数组

- 索引通常是字符串（string）或者number类型
- **也可以是除``nil``以外的任意类型的值**

**下标从1开始，0为nil，可以越界（值为nil）（应该是哈希表的原因，查不到就返回nil）**

**在内部实现上，table 通常实现为一个哈希表、一个数组、或者两者的混合。**具体的实现为何种形式，动态依赖于具体的 table 的键分布特点。 

#### function（函数）

**函数是一种数据类型，可以存储在变量中，可以通过参数传递给其他函数，也可以作为其他函数的返回值**

```lua
local function foo() 
	print("in the function") 
    --dosomething() 
    local x = 10 
    local y = 20 
    return x + y 
end 
local a = foo 
--把函数赋给变量 
print(a()) --output: in the function 30
```

有名函数的定义，本质上是匿名函数对变量的赋值

```lua
function foo()
end
-- 等价于
foo = function()
end
```

类似的

```lua
local function foo()
end
-- 等价于
local foo = function()
end
```

### 表达式

#### 算术运算符

```lua
print(1 + 2) -->打印 3 
print(5 / 10) -->打印 0.5。 这是Lua不同于c语言的 
print(5.0 / 10) -->打印 0.5。 浮点数相除的结果是浮点数 
print(10 / 0) -->打印inf  注意除数不能为0，计算的结果会出错 
print(2 ^ 10) -->打印 1024。 求2的10次方 

local num = 1357 print(num % 2) -->打印 1 
print((num % 2) == 1) -->打印 true。 判断num是否为奇数 
print((num % 5) == 0) -->打印 false。判断num是否能被5整数
```

#### 关系运算符

不等于：``~=``

```lua
print(1 < 2) -->打印 true 
print(1 == 2) -->打印 false 
print(1 ~= 2) -->打印 true 
local a, b = true, false 
print(a == b) -->打印 false
```

Lua在使用``==``做等于判断时，要注意对于 table, userdate 和函数， Lua 是作引用比较的

- **只有当两个变量引用同一个对象时，才认为他们相等**

```lua
local a = { x = 1, y = 0} 
local b = { x = 1, y = 0} 
if a == b then print("a==b") 
else print("a~=b") end 
---output: a~=b
```

#### 逻辑运算符

``a and b`` 如果 a 为 nil，则返回 a，否则返回 b

``a or b`` 如果 a 为 nil，则返回 b，否则返回 a

**所有逻辑操作符将false和nil视为假，其他任何值视为真**

对于and和or，短路求值，对于not，永远只返回true和false

```lua
local c = nil 
local d = 0 
local e = 100 
print(c and d) -->打印 nil 
print(c and e) -->打印 nil 
print(d and e) -->打印 100 
print(c or d) -->打印 0 
print(c or e) -->打印 100 
print(not c) -->打印 true 
print(not d) -->打印 false
```

### 字符串连接

连接两个字符串，可以使用操作符``“..”``（两个点）。

如果任意一个操作数为数字的话，Lua会将这个数字转换成字符串

**连接操作符只会创建一个新字符串，而不会改变原操作数**

- 也可以使用string库函数，``string.format``连接字符串

```lua
print("Hello " .. "World") -->打印 Hello World 
print(0 .. 1) -->打印 01 
str1 = string.format("%s-%s","hello","world") 
print(str1) -->打印 hello-world 
str2 = string.format("%d-%s-%.2f",123,"world",1.21) 
print(str2) -->打印 123-world-1.21
```

**不建议在循环的时候使用 .. 来拼接，因为性能损耗会非常大**

在这种情况下，推荐使用table和``table.concat()``来进行很多字符串的拼接

```lua
local pieces = {} 
for i, elem in ipairs(my_list) do 
	pieces[i] = my_process(elem) 
end 
local res = table.concat(pieces)
```

上面的例子还可以使用 LuaJIT 独有的 table.new 来恰当地初始化 pieces 表的空间，避免该表的动态生长

### 优先级

```lua
local a, b = 1, 2 
local x, y = 3, 4 
local i = 10 
local res = 0 
res = a + i < b/2 + 1 -->等价于res = (a + i) < ((b/2) + 1) 
res = 5 + x^2*8 -->等价于res = 5 + ((x^2) * 8) 
res = a < y and y <=x -->等价于res = (a < y) and (y <= x)
```

**最好使用括号来指定运算顺序，可以提高代码的可读性**

### 控制结构

#### 控制结构 if-else

**单个if分支型**

```lua
x = 10
if x > 0 then 
    print("x is a positive number")
end
```

**两个分支 if-else型**

```lua
x = 10
if x > 0 then
    print("x is a positive number")
else
    print("x is a non-positive number")
end
```

**多个分支if-elseif-else型**

```lua
score = 90
if score == 100 then
    print("Very good!Your score is 100")
elseif score >= 60 then
    print("Congratulations, you have passed it,your score greater or equal to 60")
-- 此处可以添加多个elseif
else
    print("Sorry, you do not pass the exam! ") 
end
```

**若将 else 与 if 写成 "else if" 则相当于在else里嵌套另一个if语句**

```lua
score = 90
if score == 100 then
    print("Very good!Your score is 100")
elseif score >= 60 then
    print("Congratulations, you have passed it,your score greater or equal to 60")
-- 此处可以添加多个elseif
else
    if score < 60 then 
    	print("...........") 
end
```

#### while型控制结构

```lua
while 表达式 do
    -- body
end
```

```lua
x = 1 
sum = 0 
while x <= 5 do 
    sum = sum + x 
    x = x + 1 end 
print(sum) -->output 15
```

Lua中没有``continue``，只有``break``跳出循环，例如遍历``table``，查找值为11的数组下标索引

```lua
local t = {1, 3, 5, 8, 11, 18, 21} 
local i 
for i, v in ipairs(t) do 
    if 11 == v then 
        print("index[" .. i .. "] have right value[11]")
        break 
    end 
end
```

#### repeat控制结构

类似于do-while，但是控制方式是刚好相反的

**执行repeat循环体后，直到until的条件为真时才结束**

以下代码将会形成死循环

```lua
x = 10 
repeat
    print(x) 
until false
```

#### for控制结构

**for数字型**

for 语句有两种形式：数字 for（numeric for）和范型 for（generic for）

数字型for的语法，step默认为1

```lua
for var = begin, finish, step do 
	--body 
end
```

示例

```lua
for i = 1, 5 do 
	print(i) 
end 
-- output: 
1
2
3
4
5
```

如果不想给循环设置上限的话，可以使用常量 math.huge

```lua
for i = 1, math.huge do 
	if (0.3*i^3 - 20*i^2 - 500 >=0) then 
        print(i) 
        break 
    end 
end
```

#### for 泛型

泛型for循环通过一个迭代器（iterator）函数来遍历所有值

```lua
-- 打印数组a的所有值 
local a = {"a", "b", "c", "d"} 
	for i, v in ipairs(a) do 
		print("index:", i, " value:", v) 
end 
-- output: 
index: 1 value: a 
index: 2 value: b 
index: 3 value: c 
index: 4 value: d
```

**Lua的基础库提供了``iparis``，这个一个用于遍历数组的迭代器函数**

```lua
-- 打印table t中所有的key 
for k in pairs(t) do 
    print(k) 
end
```

标准库提供了几种迭代器

- 迭代文件中每行（io.lines）
- 迭代table元素的（pairs）
- 迭代数组元素的（ipairs）
- 迭代字符串中单词的（string.gmatch）等

逆向table示例

```lua
local days = { "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday","Sunday" }
local revDays = {} 
for k, v in pairs(days) do 
    revDays[v] = k 
end 
-- print value 
for k,v in pairs(revDays) do 
    print("k:", k, " v:", v) 
end 
-- output: 
k: Tuesday v: 2 
k: Monday v: 1 
k: Sunday v: 7 
k: Thursday v: 4
k: Friday v: 5 
k: Wednesday v: 3 
k: Saturday v: 6
```

**在LuaJIT2.1中，``ipairs()``内建函数是可以被JIT编译的，而``pairs()``只能被解释执行**

- 在性能比较敏感的场景，合理使用数据结构，避免对哈希表进行遍历

### break、return、和goto

#### break

用来终止while、repeat、for三种循环的执行，并跳出当前循环体

```lua
-- 计算最小的x,使从1到x的所有数相加和大于100 
sum = 0 
i = 1 
while true do 
    sum = sum + i 
    if sum > 100 then 
        break 
    end 
    i = i + 1 
end 
print("The result is " .. i) -->output:The result is 14
```

#### return

主要从函数中返回结果，或者用于简单的结束一个函数的执行

**return后面不能写代码，否则会报错，除非是在if里面使用**

```lua
local function add(x, y) 
    return x + y 
    --print("add: I will return the result " .. (x + y)) 
    --因为前面有个return，若不注释该语句，则会报错
end 

local function is_positive(x) 
    if x > 0 then
        return x .. " is positive" 
    else
        return x .. " is non-positive" 
    end --由于return只出现在前面显式的语句块，所以此语句不注释也不会报错 
    --，但是不会被执行，此处不会产生输出 print("function end!") 
end 

local sum = add(10, 20) 
print("The sum is " .. sum) -->output:The sum is 30 
local answer = is_positive(-10) 
print(answer) -->output:-10 is non-positive
```

**如果想提前控制return，可以放在do...end中**

```lua
local function foo() 
    print("before") 
    do return end 
    print("after") -- 这一行语句永远不会执行到 
end
```

#### goto

LuaJIT 一开始对标的是 Lua 5.1，但渐渐地也开始加入部分 Lua 5.2 甚至 Lua 5.3 的有用特性

有了goto就可以实现continue的功能

```lua
for i=1, 3 do 
    if i <= 2 then 
        print(i, "yes continue") 
        goto continue 
    end 
    print(i, " no continue") 
    ::continue:: 
    print([[i'm end]]) 
end
```

**goto的另一个用途就是简化错误处理的流程，直接goto到函数末尾统一的错误处理过程，是更为清晰的写法**

```lua
local function process(input) 
    print("the input is", input) 
    if input < 2 then 
        goto failed 
    end 
    -- 更多处理流程和 goto err 
    print("processing...")
    do return end 
    ::failed:: 
    print("handle error with input", input) 
end 
process(1) 
process(3)
```

http://luajit.org/extensions.html#lua52

### Lua函数

定义了一个全局函数

```lua
function function_name (arc) -- arc 表示参数列表，函数的参数列表可以为空 
    -- body 
end
```

等价于

```lua
function_name = function(arc)
    -- body
end
```

**全局变量一般会污染全局名字空间，同时也有性能损耗（即查询全局环境表的开销）**

**尽量使用局部函数**

```lua
local function function_name(arc)
    -- body
end
```

由于函数定义本质上就是变量赋值，而变量的定义总是应放置在变量使用之前，所以函数的定义也需要放置在函数调用之前。 

由于函数定义等价于变量赋值，我们也可以把函数名替换为某个 Lua 表的某个字段

```lua
function foo.bar(a,b,c)
    -- body ...
end
```

等价于下面的函数，对于这种形式的函数定义，不能再使用local修饰符，**因为不存在定义新的局部变量了**

```lua
foo.bar = function(a,b,c)
    print(a,b,c)
end
```

### 函数的参数

#### 按值传递

Lua 函数的参数大部分是按值传递的

```lua
local function swap(a, b) --定义函数swap,函数内部进行交换两个变量的值 
    local temp = a 
    a = b 
    b = temp 
    print(a, b) 
end 

local x = "hello" 
local y = 20 
print(x, y) 
swap(x, y) --调用swap函数 
print(x, y) --调用swap函数后，x和y的值并没有交换 -->output hello 20 20 hello hello 20
```

在调用函数的时候，若形参个数和实参个数不同时，Lua 会自动调整实参个数

调整规则： 

- 若实参个数大于形参个数，从左向右，多余的实参被忽略；
- 若实参个数小于形参个数，从左向右，没有被实参初始化的形参会被初始化为 nil。 

```lua
local function fun1(a, b) --两个形参，多余的实参被忽略掉 
    print(a, b) 
end 

local function fun2(a, b, c, d) --四个形参，没有被实参初始化的形参，用nil初始化 
    print(a, b, c, d) 
end 

local x = 1 
local y = 2 
local z = 3 
fun1(x, y, z) -- z被函数fun1忽略掉了，参数变成 x, y 
fun2(x, y, z) -- 后面自动加上一个nil，参数变成 x, y, z, nil 
-->output 
1 2 
1 2 3 nil
```

#### 变长参数

上面函数的参数都是固定的，其实 Lua 还支持变长参数。若形参为 ... , 表示该函数可以接收不同长度的参数。访问参数的时候也要使用 ... 

```lua
local function func( ... ) -- 形参为 ... ,表示函数采用变长参数 
    local temp = {...} -- 访问的时候也要使用 ... 
    local ans = table.concat(temp, " ") -- 使用 table.concat 库函数对数 
    -- 组内容使用 " " 拼接成字符串。
    print(ans) 
end 

func(1, 2) -- 传递了两个参数 
func(1, 2, 3, 4) -- 传递了四个参数 
-->output 
1 2 
1 2 3 4
```

**LuaJIT2尚不能JIT编译这种变长参数的用法，只能解释执行。所以对性能敏感的代码，应当避免使用此种形式**

#### 具名参数

Lua 还支持通过名称来指定实参，这时候要把所有的实参组织到一个 table 中，并将这个table作为唯一的实参传给函数

```lua
local function change(arg) -- change 函数，改变长方形的长和宽，使其各增长一倍 
    arg.width = arg.width * 2 
    arg.height = arg.height * 2 
    return arg 
end 

local rectangle = { width = 20, height = 15 } 
print("before change:", "width =", rectangle.width, 
    "height =", rectangle.height) 
rectangle = change(rectangle) 
print("after change:", "width =", rectangle.width, 
    "height =", rectangle.height) 
-->output 
before change: width = 20 height = 15 
after change: width = 40 height = 30
```

#### 按引用传递

**当函数参数是 table 类型时，传递进来的是实际参数的引用**

**在常用基本类型中，除了 table 是按址传递类型外，其它的都是按值传递参数**

### 函数返回值

Lua 具有一项与众不同的特性，允许函数返回多个值。

示例代码：使用库函数 string.find ，在源字符串中查找目标字符串，若查找成功，则返回目标字符串在源字符串中的起始位置和结束位置的下标。 

```lua
local s, e = string.find("hello world", "llo")
print(s, e)
```

返回多个值时，值之间用逗号隔开

```lua
local function swap(a, b) -- 定义函数 swap，实现两个变量交换值 
    return b, a -- 按相反顺序返回变量的值 
end 
local x = 1 
local y = 20 
x, y = swap(x, y) -- 调用 swap 函数 
print(x, y) --> output 20 1
```

当函数返回值的个数和接收返回值的变量的个数不一致时，Lua 也会自动调整参数个数。 

调整规则： 若返回值个数大于接收变量的个数，多余的返回值会被忽略掉； 若返回值个数小于参数个数，从左向右，没有被返回值初始化的变量会被初始化为 nil。

```lua
function init() --init 函数 返回两个值 1 和 "lua" \
    return 1, "lua" 
end 
x = init() 
print(x) 
x, y, z = init() 
print(x, y, z) 
--output 
1
1 lua nil
```

当一个函数有一个以上返回值，且函数调用不是一个列表表达式的最后一个元素，那么函数调用只会产生一个返回值, 也就是第一个返回值。 

```lua
local function init()
    return 1, "lua"
end

local x, y, z = init(), 2 -- init 函数的位置不在最后，此时只返回 1 
print(x, y, z) -->output 1 2 nil 
local a, b, c = 2, init() -- init 函数的位置在最后，此时返回 1 和 "lua" 
print(a, b, c) -->output 2 1 lua
```

函数调用的实参列表也是一个列表表达式。考虑下面的例子： 

```lua
local function init() 
    return 1, "lua" 
end 
print(init(), 2) -->output 1 2 
print(2, init()) -->output 2 1 lua
```

如果你确保只取函数返回值的第一个值，可以使用括号运算符，例如 

```lua
local function init() 
    return 1, "lua" 
end 
print((init()), 2) -->output 1 2 
print(2, (init())) -->output 2 1
```

如果实参列表中某个函数会返回多个值，同时调用者又没有显式地使用括号运算符来筛选和过滤，则这样的表达式是不能被 **LuaJIT 2** 所 **JIT** 编译的，而只能被解释执行

### 全动态函数调用

调用回调函数，并把一个数组参数作为回调函数的参数

```lua
local args = {...} or {}
method_name(unpack(args, 1, table.maxn(args)))
```

**如果你的实参table中确定没有nil空洞，可以简化为**

```lua
method_name(unpack(args))
```

- 你要调用的函数参数是未知的
- 函数的实际参数的类型和数目也都是未知的

**伪代码**

```lua
add_task(end_time, callback, params) 

if os.time() >= endTime then 
    callback(unpack(params, 1, table.maxn(params))) 
end
```

``unpack``内建函数还不能为LuaJIT所JIT编译，因此这种用法总是会被解释执行。**对性能敏感的代码路径应避免这种用法**

```lua
local function run(x, y) 
    print('run', x, y) 
end 

local function attack(targetId) 
    print('targetId', targetId) 
end 

local function do_action(method, ...) 
    local args = {...} or {} 
    method(unpack(args, 1, table.maxn(args))) 
end
do_action(run, 1, 2) -- output: run 1 2 
do_action(attack, 1111) -- output: targetId 1111
```

### 模块

从 Lua 5.1语言添加了对模块和包的支持。一个 Lua模块的数据结构是用一个Lua值（通常是一个Lua表或者Lua函数）

**一个代码模块就是一个程序库，可以通过``require()``来加载**。模块加载后的结果通过是一个Lua table，这个表就像是一个命名空间，其内容就是模块中导出的所有东西，比如函数和变量

#### require函数

require函数用来加载模块，要加载一个模块，只需要简单地调用``require``“file”就可以了，file指模块所在的文件名

这个调用会返回一个由模块函数组成的table，并且还会定义一个包含该table的全局变量

**在Lua中创建一个模块最简单的方法是，创建一个table，并将所有需要导出的函数放入其中，最后返回这个table就可以了**

- 相当于将导出的函数作为table的一个字段，在Lua中函数是第一类值，提供了天然的优势

**my.lua**

```lua
local _M = {}
local function get_name() 
    return "Lucy" 
end 

function _M.greeting() 
    print("hello " .. get_name()) 
end
return _M
```

**main.lua**

```lua
local my_module = require("my") 
my_module.greeting() -->output: hello Lucy
```

运行结果

```bash
$ luajit main.lua 
helloLucy
```

**在LuaJIT中，require函数内不能进行上下文切换，所以不能够在模块的顶级上下文中调用cosocket一类的API，否则会报``attempt to yield across C-call boundary``错误**

### String库

#### string.byte(s[,i[,j]])

返回字符 s[i]、s[i + 1]、s[i + 2]、······、s[j] 所对应的 ASCII 码。 i 的默认值为 1，即第一个字节, j 的默认值为 i 

示例代码

```lua
print(string.byte("abc", 1, 3))
print(string.byte("abc", 3)) -- 缺少第三个参数，第三个参数默认与第二个相同，此时为 3 
print(string.byte("abc")) -- 缺少第二个和第三个参数，此时这两个参数都默认为 1
-->output 
97 98 99 
99
97
```

``string.byte``只返回整数，并不像``string.sub``等函数那样创建新的Lua字符串

**使用``string.byte``来进行字符串相关的扫描和分析是最为高效的，尤其是在被LuaJIT2所JIT编译之后**

#### string.char(...)

接收 0 个或更多的整数（整数范围：0~255），返回这些整数所对应的 ASCII 码字符组成的字符串。当参数为空时，默认是一个 0。

示例代码

```lua
print(string.char(96, 97, 98))
print(string.char()) -- 参数为空，默认是一个0， 
-- 你可以用string.byte(string.char())测试一下 
print(string.char(65, 66)) 
--> output
`ab 
AB
```

**此函数特别适合从具体的字节构造出二进制字符串**

这经常比使用 table.concat 函数和 .. 连接运算符更加高效

### string.upper(s)

接收一个字符串 s，返回一个把所有小写字母变成大写字母的字符串。 

```lua
print(string.upper("Hello Lua")) -->output HELLO LUA
```

#### string.lower(s)

接收一个字符串 s，返回一个把所有大写字母变成小写字母的字符串。 

```lua
print(string.lower("Hello Lua")) -->output hello lua
```

#### string.len(s)

接收一个字符串，返回它的长度。 

**使用此函数是不推荐的。应当总是使用 # 运算符来获取 Lua 字符串的长度。**

由于 Lua 字符串的长度是专门存放的，因此获取字符串长度的操作总是 O(1) 的时间复杂度。

```lua
print(string.len("hello lua")) -->output 9
```

#### string.find(s, p [, init [, plain]])

在s字符串中第一次匹配p字符串

- 如果匹配成功，则返回p字符串在s字符串中出现的开始位置和结束位置
- 如果匹配失败，则返回nil

-  init 为负数时，表示从 s 字符串的 string.len(s) + init + 1 索引处开始向后匹配字符串 p

- 第四个参数默认为 false，当其为 true 时，只会把 p 看成一个字符串对待。

```lua
local find = string.find 
print(find("abc cba", "ab"))
print(find("abc cba", "ab", 2)) -- 从索引为2的位置开始匹配字符串：ab
print(find("abc cba", "ba", -1)) -- 从索引为7的位置开始匹配字符串：ba
print(find("abc cba", "ba", -3)) -- 从索引为5的位置开始匹配字符串：ba 
print(find("abc cba", "(%a+)", 1)) -- 从索引为1处匹配最长连续且只含字母的字符串 
print(find("abc cba", "(%a+)", 1, true)) --从索引为1的位置开始匹配字符串：(%a+) 
-->output 
1 2 
nil
nil
6 7
1 3 abc
nil
```

#### string.format(formatstring, ...)

```lua
print(string.format("%.4f", 3.1415926)) -- 保留4位小数 
print(string.format("%d %x %o", 31, 31, 31))-- 十进制数31转换成不同进制
d = 29; m = 7; y = 2015 -- 一行包含几个语句，用；分开 
print(string.format("%s %02d/%02d/%d", "today is:", d, m, y)) 
-->output 
3.1416 
31 1f 37 
today is: 29/07/2015
```

#### **string.match(s, p [, init])** 

```lua
print(string.match("hello lua", "lua"))
print(string.match("lua lua", "lua", 2)) --匹配后面那个lua
print(string.match("lua lua", "hello"))
print(string.match("today is 27/7/2015", "%d+/%d+/%d+")) 
-->output
lua 
lua 
nil 
27/7/2015
```

string.match 目前并不能被 JIT 编译，应 尽量 使用 ngx_lua 模块提供的 ngx.re.match 等接口

#### string.gmatch(s, p)

返回一个迭代器函数，通过这个迭代器函数可以遍历到在字符串 s 中出现模式串 p 的所有地方

```lua
s = "hello world from Lua"
for w in string.gmatch(s, "%a+") do --匹配最长连续且只含字母的字符串
    print(w) 
end

-->output
hello 
world
from
Lua 

t = {} 
s = "from=world, to=Lua" 
for k, v in string.gmatch(s, "(%a+)=(%a+)") do --匹配两个最长连续且只含字母的
    t[k] = v --字符串，它们之间用等号连接 
end 
for k, v in pairs(t) do 
    print (k,v) 
end 
-->output 
to Lua
from world
```

此函数目前并不能被 LuaJIT 所 JIT 编译，而只能被解释执行。应 尽量 使用 ngx_lua 模块提供的ngx.re.gmatch等接口

#### **string.rep(s, n)** 

返回字符串s的n次拷贝

```lua
print(string.rep("abc", 3)) --拷贝3次"abc" 
-->output abcabcabc
```

#### **string.sub(s, i [, j])** 

返回字符串 s 中，索引 i 到索引 j 之间的子字符串。

```lua
print(string.sub("Hello Lua", 4, 7)) 
print(string.sub("Hello Lua", 2)) 
print(string.sub("Hello Lua", 2, 1)) --看到返回什么了吗 
print(string.sub("Hello Lua", -3, -1)) 
-->output 
lo L 
ello Lua 

Lua
```

如果你只是想对字符串中的单个字节进行检查，使用 string.char 函数通常会更为高效。 

#### **string.gsub(s, p, r [, n])** 

将目标字符串 s 中所有的子串 p 替换成字符串 r。可选参数 n，表示限制替换次数。返回值有两个，第一个是被替换后的字符串，第二个是替换了多少次。

```lua
print(string.gsub("Lua Lua Lua", "Lua", "hello")) 
print(string.gsub("Lua Lua Lua", "Lua", "hello", 2)) --指明第四个参数 
-->output 
hello hello hello 3 
hello hello Lua 2
```

此函数不能为 LuaJIT 所 JIT 编译，而只能被解释执行。一般我们推荐使用 ngx_lua 模块提供的 ngx.re.gsub 函数。

#### **string.reverse (s)** 

接收一个字符串 s，返回这个字符串的反转。

```lua
print(string.reverse("Hello Lua")) --> output: auL olleH
```

### table库

#### 下标从1开始

Lua 最初设计是一种类似 XML 的数据描述语言，所以索引（index）反应的是数据在里面的位置，而不是偏移量。

当我们把table当作栈或者队列使用的时候，容易犯错，追加到table的末尾使用的是``s[#s+1] = something``而不是``s[#s]=something``

- 如果这个something是一个nil的话，会导致这一次压栈（或者入队列）没有存入任何东西，#s的值没有变

- 如果 s = { 1, 2, 3, 4, 5, 6 } ，你令 s[4] = nil ，#s会令你“匪夷所思”地变成 3。

#### table.getn获取长度

对于常规的数组，里面从 1 到 n 放着一些非空的值的时候，它的长度就精确的为 n，即最后一个值的下标。如果数组有一个“空洞”（就是说，nil 值被夹在非空值之间），那么 #t 可能是指向任何一个是 nil 值的前一个位置的下标（就是说，任何一个 nil 值都有可能被当成数组的 结束）。这也就说明对于有“空洞”的情况，table 的长度存在一定的 不可确定性。

```lua
local tblTest1 = { 1, a = 2, 3 } 
print("Test1 " .. table.getn(tblTest1)) 

local tblTest2 = { 1, nil } 
print("Test2 " .. table.getn(tblTest2)) 

local tblTest3 = { 1, nil, 2 } 
print("Test3 " .. table.getn(tblTest3)) 

local tblTest4 = { 1, nil, 2, nil } 
print("Test4 " .. table.getn(tblTest4)) 

local tblTest5 = { 1, nil, 2, nil, 3, nil } 
print("Test5 " .. table.getn(tblTest5)) 

local tblTest6 = { 1, nil, 2, nil, 3, nil, 4, nil } 
print("Test6 " .. table.getn(tblTest6))
```

Lua 5.1 和 LuaJIT 2.1 分别执行这个用例，结果如下

```bash
# lua test.lua 
Test1 2
Test2 1 
Test3 3 
Test4 1 
Test5 3 
Test6 1 

# luajit test.lua 
Test1 2
Test2 1 
Test3 1 
Test4 1 
Test5 1 
Test6 1
```

**不要在Lua的table中使用nil值，如果一个元素要删除，直接remove，不要使用nil代替**

#### table.concat (table [, sep [, i [, j ] ] ])

对于元素是 string 或者 number 类型的表 table，返回 table[i]..sep..table[i+1] ···sep..table[j] 连接成的字符串

填充字符串 sep 默认为空白字符串

起始索引位置 i 默认为1，结束索引未知j默认是table的长度

如果i大于j，返回一个空字符串

```lua
local a = {1, 3, 5, "hello" }
print(table.concat(a)) -- output: 135hello
print(table.concat(a, "|")) -- output: 1|3|5|hello 
print(table.concat(a, " ", 4, 2)) -- output: 
print(table.concat(a, " ", 2, 4)) -- output: 3 5 hello
```

#### table.insert (table, [pos ,] value)

在（数组型）表 table 的 pos 索引位置插入 value，其它元素向后移动到空的地方。pos 的默认值是表的长度加一，即默认是插在表的最后。

示例代码

```lua
local a = {1, 8} --a[1] = 1,a[2] = 8 
table.insert(a, 1, 3) --在表索引为1处插入3 
print(a[1], a[2], a[3]) 
table.insert(a, 10) --在表的最后插入10 
print(a[1], a[2], a[3], a[4]) 
-->output 
3 1 8 
3 1 8 10
```

#### **table.maxn (table)** 

返回（数组型）表 table 的最大索引编号；如果此表没有正的索引编号，返回 0

**当长度省略时，此函数通常需要 O(n) 的时间复杂度来计算 table 的末尾。**

- 因此用这个函数省略索引位置的调用形式来作table元素的末尾追加，是高代价操作

```lua
local a = {} 
a[-1] = 10 
print(table.maxn(a)) 
a[5] = 10 
print(table.maxn(a)) 
-->output 
0
5
```

此函数的行为不同于 # 运算符，因为 # 可以返回数组中任意一个 nil 空洞或最后一个 nil 之前的元素索引。当然，该函数的开销相比 # 运算符也会更大一些。

#### table.remove (table [, pos])

在表 table 中删除索引为 pos（pos 只能是 number 型）的元素，并返回这个被删除的元素，它后面所有元素的索引值都会减一。pos 的默认值是表的长度，即默认是删除表的最后一个元素。

```lua
local a = { 1, 2, 3, 4} 
print(table.remove(a, 1)) --删除速索引为1的元素 
print(a[1], a[2], a[3], a[4]) 

print(table.remove(a)) --删除最后一个元素 
print(a[1], a[2], a[3], a[4]) 

-->output 
1
2 3 4 nil 
4
2 3 nil nil
```

#### **table.sort (table [, comp])** 

按照给定的比较函数 comp 给表 table 排序，也就是从 table[1] 到 table[n]，这里 n 表示 table的长度。 比较函数有两个参数，如果希望第一个参数排在第二个的前面，就应该返回 true， 否则返回 false。 如果比较函数 comp 没有给出，默认从小到大排序。

```lua
local function compare(x, y) --从大到小排序 
    return x > y --如果第一个参数大于第二个就返回true，否则返回false 
end 

local a = { 1, 7, 3, 4, 25} 
table.sort(a) --默认从小到大排序 
print(a[1], a[2], a[3], a[4], a[5]) 
table.sort(a, compare) --使用比较函数进行排序 
print(a[1], a[2], a[3], a[4], a[5]) 

-->output 
1 3 4 7 25 
25 7 4 3 1
```

#### **table** 其他非常有用的函数

LuaJIT 2.1 新增加的 table.new 和 table.clear 函数是非常有用的。

- 前者主要用来预分配Lua table空间，后者主要用来高效的释放table空间
- 它们都是可以被JIT编译的

### 日期时间函数

在 Lua 中，函数 time、date 和 difftime 提供了所有的日期和时间功能。

**在 OpenResty 的世界里，不推荐使用这里的标准时间函数，因为这些函数通常会引发不止一个昂贵的系统调用，同时无法为 LuaJIT JIT 编译，对性能造成较大影响**

推荐使用 ngx_lua模块提供的带缓存的时间接口，如 ngx.today , ngx.time , ngx.utctime , ngx.localtime ,ngx.now , ngx.http_time ，以及 ngx.cookie_time 等。

### 数学库

```lua
print(math.pi) -->output 3.1415926535898 
print(math.rad(180)) -->output 3.1415926535898 
print(math.deg(math.pi)) -->output 180

print(math.sin(1)) -->output 0.8414709848079 
print(math.cos(math.pi)) -->output -1 
print(math.tan(math.pi / 4)) -->output 1 

print(math.atan(1)) -->output 0.78539816339745
print(math.asin(0)) -->output 0 

print(math.max(-1, 2, 0, 3.6, 9.1)) -->output 9.1 
print(math.min(-1, 2, 0, 3.6, 9.1)) -->output -1 

print(math.fmod(10.1, 3)) -->output 1.1 
print(math.sqrt(360)) -->output 18.97366596101 

print(math.exp(1)) -->output 2.718281828459 
print(math.log(10)) -->output 2.302585092994 
print(math.log10(10)) -->output 1 

print(math.floor(3.1415)) -->output 3
print(math.ceil(7.998)) -->output 8
```

**使用math.random()函数获得伪随机数时，如果不使用math.randomseed()设置伪随机数生成种子或者设置相同的伪随机数生成种子，那么得到的伪随机数序列是一样的**

```lua
math.randomseed (os.time()) --把100换成os.time() 
print(math.random()) -->output 0.88369396038697 
print(math.random(100)) -->output 66 
print(math.random(100, 360)) -->output 228
```

### 文件操作

Lua I/O库提供两种不同的方式处理文件

- 隐式文件描述
- 显式文件描述

这些文件 I/O 操作，在 OpenResty 的上下文中对事件循环是会产生阻塞效应

**OpenResty 比较擅长的是高并发网络处理，在这个环境中，任何文件的操作，都将阻塞其他并行执行的请求。**

实际中的应用，在 OpenResty 项目中应尽可能让网络处理部分、文件 I/0 操作部分相互独立，不要揉和在一起。

#### 隐式文件描述

设置一个默认的输入或输出文件，然后在这个文件上进行所有的输入或输出操作。所有的操作函数由 io 表提供。 

```lua
file = io.input("test1.txt") -- 使用 io.input() 函数打开文件 
repeat
    line = io.read() -- 逐行读取内容，文件结束时返回nil 
    if nil == line then 
        break 
    end 
    print(line) 
until (false) 
io.close(file) -- 关闭文件 

--> output 
my test file 
hello 
lua
```

```lua
file = io.open("test1.txt", "a+") -- 使用 io.open() 函数，以添加模式打开文件
io.output(file) -- 使用 io.output() 函数，设置默认输出文件
io.write("\nhello world") -- 使用 io.write() 函数，把内容写到文件 
io.close(file)
```

#### 显式文件描述

使用 file:XXX() 函数方式进行操作, 其中 file 为 io.open() 返回的文件句柄。

```lua
file = io.open("test2.txt", "r") -- 使用 io.open() 函数，以只读模式打开文件
for line in file:lines() do -- 使用 file:lines() 函数逐行读取文件 
    print(line) 
end 
file:close() 
-->output 
my test2 
hello lua
```

```lua
file = io.open("test2.txt", "a") -- 使用 io.open() 函数，以添加模式打开文件 
file:write("\nhello world") -- 使用 file:write() 函数，在文件末尾追加内容
file:close()
```

#### 文件操作函数（略）

### 元表

元表 *(metatable)* 的表现行为类似于 C++ 语言中的操作符重载

Lua 提供了两个十分重要的用来处理元表的方法

- setmetatable(table, metatable)：此方法用于为一个表设置元表。 
- getmetatable(table)：此方法用于获取表的元表对象。

设置元表

```lua
local mytable = {}
local mymetatable = {}
setmetatable(mytable, mymetatable)
```

简写成如下的代码

```lua
local mytable = setmetatable({}, {})
```

#### 修改表的操作符行为

**重载_add元方法**

```lua
local set1 = {10, 20, 30} -- 集合 
local set2 = {20, 40, 50} -- 集合

-- 将用于重载__add的函数，注意第一个参数是self 
local union = function (self, another) 
    local set = {} 
    local result = {} 
    
    -- 利用数组来确保集合的互异性 
    for i, j in pairs(self) do set[j] = true end 
    for i, j in pairs(another) do set[j] = true end 
    
    -- 加入结果集合 
    for i, j in pairs(set) do table.insert(result, i) end 
    return result 
end setmetatable(set1, {__add = union}) -- 重载 set1 表的 __add 元方法 

local set3 = set1 + set2 
for _, j in pairs(set3) do 
    io.write(j.." ") -->output：30 50 20 40 10 
end
```

#### __index元方法

```lua
mytable = setmetatable({key1 = "value1"}, --原始表 
    {__index = function(self, key) --重载函数 
            if key == "key2" then
                return "metatablevalue" 
            end 
        end 
    })
print(mytable.key1,mytable.key2) --> output：value1 metatablevalue
```

关于`` __index`` 元方法，有很多比较高阶的技巧，例如：__index 的元方法不需要非是一个函数，他也可以是一个表

```lua
t = setmetatable({[1] = "hello"}, {__index = {[2] = "world"}}) 
print(t[1], t[2]) -->hello world
```

- 先是把 ``{__index = {}}`` 作为元表，但 __index 接受一个表，而 不是函数，这个表中包含 [2] = "world" 这个键值对。 
-  t[2] 去在自身的表中找不到时， 在 __index 的表中去寻找，然后找到了 [2] = "world" 这个键值对。

#### **__tostring** 元方法

实现自定义的字符串转换

```lua
arr = {1, 2, 3, 4} 
arr = setmetatable(arr, {__tostring = function (self) 
            local result = '{' 
            local sep = '' 
            for _, i in pairs(self) do 
                result = result ..sep .. i 
                sep = ', ' 
            end
            result = result .. '}' 
            return result
        end}) 
print(arr) --> {1, 2, 3, 4}
```

#### __call元方法

__call 元方法的功能类似于 C++ 中的仿函数，使得普通的表也可以被调用

```lua
functor = {} 
function func1(self, arg) 
    print ("called from", arg)
end 

setmetatable(functor, {__call = func1}) 

functor("functor") --> called from functor 
print(functor) --> output：0x00076fc8 （后面这串数字可能不一样）
```

#### __metatable元方法

假如我们想保护我们的对象使其使用者既看不到也不能修改 metatables。我们可以对metatable 设置了 __metatable 的值，getmetatable 将返回这个域的值，而调用 setmetatable 将会出错： 

```lua
Object = setmetatable({}, {__metatable = "You cannot access here"}) print(getmetatable(Object)) --> You cannot access here 
setmetatable(Object, {}) --> 引发编译器报错
```

### Lua面向对象编程

#### 类

在 Lua 中，我们可以使用表和函数实现面向对象。**将函数和相关的数据放置于同一个表中就形成了一个对象**

**``account.lua``**

```lua
local _M = {} 
local mt = { __index = _M } 
function _M.deposit (self, v) 
    self.balance = self.balance + v
end
function _M.withdraw (self, v) 
    if self.balance > v then 
        self.balance = self.balance - v 
    else
        error("insufficient funds") 
    end 
end 
function _M.new (self, balance)
    balance = balance or 0
    return setmetatable({balance = balance}, mt) 
end 
return _M
```

示例

```lua
local account = require("account") 

local a = account:new() 
a:deposit(100) 

local b = account:new() 
b:deposit(50) 

print(a.balance) --> output: 100 
print(b.balance) --> output: 50
```

#### 继承

继承可以用元表实现，它提供了在父类中查找存在的方法和变量的机制

**在 Lua 中是不推荐使用继承方式完成构造的，这样做引入的问题可能比解决的问题要多**

#### 成员私有性

在动态语言中引入成员私有性并没有太大的必要，反而会显著增加运行时的开销，毕竟这种检查无法像许多静态语言那样在编译期完成。

**在 Lua 中，成员的私有性，使用类似于函数闭包的形式来实现**

在我们之前的银行账户的例子中，我们使用一个工厂方法来创建新的账户实例，通过工厂方法对外提供的闭包来暴露对外接口。而不想暴露在外的例如 balance 成员变量，则被很好的隐藏起来。

```lua
function newAccount (initialBalance) 
    local self = {balance = initialBalance} 
    local withdraw = function (v) 
        self.balance = self.balance - v 
    end
    local deposit = function (v) 
        self.balance = self.balance + v
    end 
    local getBalance = function () return self.balance end
    return { 
        withdraw = withdraw, 
        deposit = deposit, 
        getBalance = getBalance 
    } 
end 
a = newAccount(100) 
a.deposit(100)
print(a.getBalance()) --> 200
print(a.balance) --> nil
```

### 局部变量

在一个 block 中的变量，如果之前没有定义过，那么认为它是一个全局变量，而不是这个 block 的局部变量。这一点和别的语言不同。容易造成不小心覆盖了全局同名变量的错误

#### 定义

Lua 中的局部变量要用 local 关键字来显式定义，不使用 local 显式定义的变量就是全局变量：

```lua
g_var = 1 -- global var 
local l_var = 2 -- local var
```

#### 使用局部变量的好处

- 局部变量可以避免因为命名问题污染了全局环境
- local 变量的访问比全局变量更快 
- 由于局部变量出了作用域之后生命周期结束，这样可以被垃圾回收器及时释放

#### 检查模块的函数使用全局变量

Lua 上下文中应当严格避免使用自己定义的全局变量。可以使用一个 lj-releng 工具来扫描 

Lua 代码，定位使用 Lua 全局变量的地方。lj-releng 的相关链接：https://github.com/openresty/openresty-devel-utils/blob/master/lj-releng

**Linux安装``lj-releng``**

```bash
curl -L https://raw.githubusercontent.com/openresty/openresty-devel-utils/master/lj-re leng > /usr/local/bin/lj-releng
chmod +x /usr/local/bin/lj-releng
```

### 判断数组大小

table.getn(t) 等价于 #t 但计算的是数组元素，不包括 hash 键值。而且数组是以第一个 nil 元素来判断数组结束。

\# 只计算 array 的元素个数，它实际上调用了对象的 metatable 的 ``__len`` 函数。对于有 __len 方法的函数返回函数返回值，不然就返回数组成员数目。

Lua数组中允许nil值的存储，但是数组默认结束标志却是nil

**一定不要使用 # 操作符或 table.getn 来计算包含 nil 的数组长度，这是一个未定义的操作，不一定报错，但不能保证结果如你所想**

### 非空判断

在很多情况下我们在访问一些 table 型变量时，需要先判断该变量是否为 nil

```lua
local person = {name = "Bob", sex = "M"} 

-- do something 
person = nil 
-- do something 
if person ~= nil and person.name ~= nil then
    print(person.name) 
else
    -- do something 
end
```

对于简单类型的变量，我们可以用 *if (var == nil) then* 这样的简单句子来判断。但是对于 table型的 Lua 对象，就不能这么简单判断它是否为空了。一个 table 型变量的值可能是 {} ，这时它不等于 nil。我们来看下面这段代码： 

```lua
local next = next 
local a = {} 
local b = {name = "Bob", sex = "Male"} 
local c = {"Male", "Female"} 
local d = nil

print(#a) 
print(#b) 
print(#c) 
--print(#d) -- error 

if a == nil then print("a == nil") end 
if b == nil then print("b == nil") end 
if c == nil then print("c == nil") end 
if d== nil then print("d == nil") end 
if next(a) == nil then print("next(a) == nil") end 
if next(b) == nil then print("next(b) == nil") end 
if next(c) == nil then print("next(c) == nil") end
```

返回的结果

```lua
0
0
2
d == nil 
next(a) == nil
```

**我们要判断一个table是否为{}，不能采用``#table == 0``的方式来判断，可以用下面的方式来判断**

```lua
function isTableEmpty(t) 
    return t == nil or next(t) == nil
end
```

注意： next 指令是不能被 LuaJIT 的 JIT 编译优化，并且 LuaJIT 貌似没有明确计划支持这个指令优化，在不是必须的情况下，尽量少用。

### 正则表达式

在OpenResty中，同时存在两套正则表达式规范：Lua语言的规范和`ngx.re.*`的规范

**不建议使用Lua中的正则表达式**

- 性能不如``ngx.re.*``
- 不符合POSIX规范

#### Lua正则简单汇总

*Lua* 中正则表达式语法上最大的区别，*Lua* 使用 *%* 来进行转义，而其他语言的正则表达式使用 \ 符号来进行转义

其次，*Lua* 中并不使用 ? 来表示非贪婪匹配，而是定义了不同的字符来表示是否是贪婪匹配。

### 虚变量

当一个方法返回多个值时，有些返回值有时候用不到，要是声明很多变量来一一接收。Lua 提供了一个虚变量(dummy variable)的概念， 按照惯例以一个下划线（“_”）来命名，用它来表示丢弃不需要的数值，仅仅起到占位的作用。

### 抵制使用module()定义模块

module("filename", package.seeall) 这种写法是不提倡的，官方给出的原因

- package.seeall 这种方式破坏了模块的高内聚，原本引入 "filename" 模块只想调用它的 *foobar()* 函数，但是它却可以读写全局属性，例如 "filename.os" 
- module 函数压栈操作引发的副作用，污染了全局环境变量。例如 module("filename") 会创建一个 filename 的 table ，并将这个 table 注入全局环境变量中，这样使得没有引用它的文件也能调用 filename 模块的方法。

**比较推荐的模块定义方法**

```lua
-- square.lua 长方形模块 
local _M = {} -- 局部的变量
_M._VERSION = '1.0' -- 模块版本

local mt = { __index = _M } 

function _M.new(self, width, height) 
    return setmetatable({ width=width, height=height }, mt)
end 

function _M.get_square(self)
    return self.width * self.height 
end

function _M.get_circumference(self) 
    return (self.width + self.height) * 2 
end 

return _M
```

示例代码

```lua
local square = require "square" 

local s1 = square:new(1, 2) 
print(s1:get_square()) --output: 2 
print(s1:get_circumference()) --output: 6
```

另一个跟 Lua 的 module 模块相关需要注意的点是，当 lua_code_cache on 开启时，require加载的模块是会被缓存下来的，这样我们的模块就会以最高效的方式运行，直到被显式地调用如下语句（这里有点像模块卸载）：

```lua
package.loaded["square"] = nil
```

可以利用这个特性代码来做一些高阶玩法，比如代码热更新

### 点号和冒号操作符的区别

```lua
local str = "abcde"
print("case 1:", str:sub(1, 2)) 
print("case 2:", str.sub(str, 1, 2))
```

执行结果

```bash
case 1: ab 
case 2: ab
```

冒号操作会带入一个 self 参数，用来代表自己 。而点号操作，只是 内容 的展开。

在函数定义时，**使用冒号将默认接收一个 self 参数，而使用点号则需要显式传入 self 参数。**

```lua
obj = { x = 20 } 
function obj:fun1() 
    print(self.x) 
end
```

等价于

```lua
obj = { x = 20 } 
function obj.fun1(self) 
    print(self.x) 
end
```

**冒号的操作，只有当变量是类对象时才需要。**

### module是邪恶的

由于 lua_code_cache off 情况下，缓存的代码会伴随请求完结而释放

**module 的最大好处缓存这时候是无法发挥的，**所以本章的内容都是基于 lua_code_cache on 的情况下。

哪里有正确的 module 所有格式？

- OpenResty 默认绑定的各种 lua-resty-* 代码开始熟悉吧

### FFI

FFI 库，是 LuaJIT 中最重要的一个扩展库。它允许从纯 Lua 代码调用外部 C 函数，使用 C 数据结构。

对于那些能够被 Lua 调用的 C 函数来说，它的接口必须遵循 Lua 要求的形式

- 就是 typedef int (*lua_CFunction)(lua_State* L) ，这个函数包含的参数是 lua_State 类型的指针 L 。

可以通过这个指针进一步获取通过 Lua 代码传入的参数。这个函数的返回值类型是一个整型，表示返回值的数量。

**用C编写的函数无法把返回值返回给Lua代码，而是通过虚拟栈来传递Lua和C之间的调用参数和返回值**

- 不仅在编程上开发效率变低，而且性能上比不上FFI库调用C函数

#### **ffi.\* API** 

在 lua 文件中使用 ffi 库的时候，必须要有下面的一行。

```lua
local ffi = require "ffi"
```

#### ffi.cdef

语法：``ffi.cdef(def)``

创建一个``myffi.c``

```c
int add(int x, int y)
{
    return x + y;
}
```

接下来在Linux下生成动态链接库

```bash
gcc -g -o libmyffi.so -fpic -shared myffi.c
```

为了方便我们测试，我们在 LD_LIBRARY_PATH 这个环境变量中加入了刚刚库所在的路径，因为编译器在查找动态库所在的路径的时候其中一个环节就是在 LD_LIBRARY_PATH 这个环境变量中的所有路径进行查找

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:your_lib_path
```

完整示例

```lua
local ffi = require "ffi" 
ffi.load('myffi',true) 
ffi.cdef[[
int add(int x, int y); /* don't forget to declare */
]]
local res = ffi.C.add(1, 2)
print(res) -- output: 3 Note: please use luajit to run this script
```

#### ffi.typeof

语法： *ctype = ffi.typeof(ct)* 

功能： 创建一个 *ctype* 对象，会解析一个抽象的 *C* 类型定义。

```lua
local uintptr_t = ffi.typeof("uintptr_t") 
local c_str_t = ffi.typeof("const char*") 
local int_t = ffi.typeof("int")
local int_array_t = ffi.typeof("int[?]")
```

#### ffi.new

语法： *cdata = ffi.new(ct [,nelem] [,init...])* 

功能： 开辟空间，第一个参数为 *ctype* 对象，*ctype* 对象最好通过 *ctype = ffi.typeof(ct)* 构建。

**ffi.new 和 ffi.C.malloc 有什么区别呢？**

- 如果使用 ffi.new 分配的 cdata 对象指向的内存块是由垃圾回收器 LuaJIT GC 自动管理的，所以不需要用户去释放内存。 
- 如果使用 ffi.C.malloc 分配的空间便不再使用 LuaJIT 自己的分配器了，所以不是由 LuaJIT GC 来管理的
  - ffi.C.malloc 返回的指针本身所对应的 cdata 对象还是由 LuaJIT GC 来管理的，也就是这个指针的 cdata 对象指向的是用 ffi.C.malloc 分配的内存空间。
  - 你应该通过 ffi.gc() 函数在这个 C 指针的 cdata 对象上面注册自己的析构函数，这个析构函数里面你可以再调用 ffi.C.free ，这样的话当 C 指针所对应的cdata 对象被 Luajit GC 管理器垃圾回收时候，也会自动调用你注册的那个析构函数来执行 C 级别的内存释放。 

#### ffi.fill

语法： *ffi.fill(dst, len [,c])* 

功能： 填充数据，此函数和 *memset(dst, c, len)* 类似，注意参数的顺序。

```lua
ffi.fill(self.bucket_v, ffi_sizeof(int_t, bucket_sz), 0) 
ffi.fill(q, ffi_sizeof(queue_type, size + 1), 0)
```

#### ffil.cast

语法： *cdata = ffi.cast(ct, init)* 

功能： 创建一个 *scalar cdata* 对象。

```lua
local c_str_t = ffi.typeof("const char*") 
local c_str = ffi.cast(c_str_t, str) -- 转换为指针地址 
local uintptr_t = ffi.typeof("uintptr_t") 
tonumber(ffi.cast(uintptr_t, c_str)) -- 转换为数字
```

#### cdata对象的垃圾回收

所有由显式的 ffi.new(), ffi.cast() etc. 或者隐式的 accessors 所创建的 cdata 对象都是能被垃圾回收的，当他们被使用的时候，你需要确保有在 Lua stack ， upvalue ，或者Lua table 上保留有对 cdata 对象的有效引用，一旦最后一个 cdata 对象的有效引用失效了，那么垃圾回收器将自动释放内存（在下一个 GC 周期结束时候）

如果你要分配一个 cdata 数组给一个指针的话，你必须保持这个持有这个数据的 cdata 对象活跃

```lua
ffi.cdef[[ 
typedef struct { int *a; } foo_t; 
]]

local s = ffi.new("foo_t", ffi.new("int[10]")) -- WRONG! 

local a = ffi.new("int[10]") -- OK 
local s = ffi.new("foo_t", a) 
-- Now do something with 's', but keep 'a' alive until you're done.
```

#### 调用C函数

```lua
local ffi = require("ffi") 
ffi.cdef[[ 
int printf(const char *fmt, ...); 
]]
ffi.C.printf("Hello %s!", "world")
```

- 使用标准 C 库的命名空间 ffi.C
- 通过符号名printf 索引这个命名空间，自动绑定标准 C 库
- 索引结果是一个特殊类型的对象，当被调用时，执行 printf 函数
- 传递给这个函数的参数，从 Lua 对象自动转换为相应的 C 类型

### 什么是JIT？

自从 OpenResty 1.5.8.1 版本之后，默认捆绑的 Lua 解释器就被替换成了 LuaJIT，而不再是标准 Lua

LuaJIT 的运行时环境包括一个用手写汇编实现的 Lua 解释器和一个可以直接生成机器代码的 JIT 编译器。

**Lua 代码在被执行之前总是会先被 lfn 成 LuaJIT 自己定义的字节码（Byte Code）**

如果当前 Lua 代码路径上的所有的操作都可以被 JIT 编译器顺利编译，则这条编译过的代码路径便被称为一个“trace”，在物理上对应一个 trace 类型的 GC 对象（即参与 Lua GC 的对象）。

你可以通过 ngx-lj-gc-objs 工具看到指定的 Nginx worker 进程里所有 trace 对象的一些基本的统计信息，见 https://github.com/openresty/stapxx#ngx-lj-gc-objs 

比如下面这一行 ngx-lj-gc-objs 工具的输出

```bash
102 trace objects: max=928, avg=337, min=160, sum=34468 (in bytes)
```

JIT 编译器不支持的原语被称为 NYI（Not Yet Implemented）原语。比较完整的 NYI 列表在 

这篇文档里面：

```bash
http://wiki.luajit.org/NYI
```

我们如何才能知道具体是哪一行 Lua 代码上的哪一个 NYI 原语终止了 trace 编译呢？答案很简单。就是使用 LuaJIT 安装自带的 jit.v 和 jit.dump 这两个 Lua 模块。 

**这两个 Lua 模块会打印出 JIT 编译器工作的细节过程。**

在 Nginx 的上下文中，我们可以在 nginx.conf 文件中的 http {} 配置块中添加下面这一段：

```nginx
init_by_lua_block { 
    local verbose = false
    if verbose then
        local dump = require "jit.dump" 
        dump.on(nil, "/tmp/jit.log") 
    else
        local v = require "jit.v" 
        v.on("/tmp/jit.log")
    end 
    require "resty.core" 
}
```

那一行 require "resty.core" 倒并不是必需的，放在那里的主要目的是为了尽量避免使用ngx_lua 模块自己的基于 lua_CFunction 的 Lua API，减少 NYI 原语。

# Nginx

Nginx使用基于事件驱动的架构，能够并发处理百万级别的TCP连接

## Nginx新手起步

### 为什么选择Nginx

1. 处理响应请求很快
2. 高并发连接
   - Nginx支持的并发连接上限取决于内存，10万远未封顶
3. 低的内存消耗
   - 一万个非活跃的HTTP Keep-Alive连接在Nginx中仅消耗2.5MB的内存，这也是Nginx支持高并发连接的基础
4. 具有很高的可靠性
5. 高扩展性
   - 由多个不同功能、不同层次、不同类型且耦合度极地的模块组成
6. 热部署
   - master管理进场与worker工作进程的分离设计，使得Nginx具有热部署的功能
7. 自由的BSD许可协议

### 如何使用Nginx

1. 获取 Nginx，在 http://nginx.org/en/download.html 上可以获取当前最新的版本。 

2. 解压缩 nginx-xx.tar.gz 包。 

3. 进入解压缩目录，执行 ./configure 

4. make & make install

Nginx配置示例

```bash
ubuntu: /opt/nginx-1.7.7/conf$ tree |grep -v default 
.
├── fastcgi.conf 
├── fastcgi_params 
├── koi-utf 
├── koi-win 
├── mime.types
├── nginx.conf 
├── scgi_params
├── uwsgi_params
└── win-utf
```

除了nginx.conf，其余配置文件，一般只需要使用默认提供

nginx.conf是主配置文件

在``/etc/nginx``目录中

```bash
worker_process # 表示工作进程的数量，一般设置为cpu的核数 
worker_connections # 表示每个工作进程的最大连接数 
server{} # 块定义了虚拟主机 
	listen # 监听端口 
	server_name # 监听域名 
	location {} # 是用来为匹配的 URI 进行配置，URI 即语法中的“/uri/” 
	location /{} # 匹配任何查询，因为所有请求都以 / 开头 
		root # 指定对应uri的资源查找路径，这里html为相对路径，完整路径为 # /opt/nginx-1.7.7/html/ 
		index # 指定首页index文件的名称，可以配置多个，以空格分开。如有多 # 个，按配置顺序查找。
```

### location匹配规则

#### 语法规则

前缀匹配时，Nginx 不对 url 做编码，因此请求为 /static/20%/aa ，可以被规则 ^~ /static/ /aa 匹配到（注意是空格）

多个 location 配置的情况下匹配顺序为

- 首先精确匹配 = 

- 其次前缀匹配 ^~ 

- 其次是按文件中顺序的正则匹配 

- 然后匹配不带任何修饰的前缀匹配。 

- 最后是交给 / 通用匹配 

- 当有匹配成功时候，停止匹配，按当前匹配规则处理请求

匹配规则

```nginx
location = / { 
    echo "规则A"; 
}
location = /login { 
    echo "规则B"; 
}
location ^~ /static/ { 
    echo "规则C"; 
}
location ^~ /static/files { 
    echo "规则X"; 
}
location ~ \.(gif|jpg|png|js|css)$ { 
    echo "规则D"; 
}
location ~* \.png$ { 
    echo "规则E"; 
}
location /img { 
    echo "规则Y"; 
}
location / { 
    echo "规则F"; 
}
```

产生的效果

- 访问根目录 / ，比如 http://localhost/ 将匹配 规则A 

- 访问 http://localhost/login 将匹配 规则B ， http://localhost/register 则匹配 规则F 

- 访问 http://localhost/static/a.html 将匹配 规则C 

- 访问 http://localhost/static/files/a.exe 将匹配 规则X ，虽然 规则C 也能匹配到，但因为最大匹配原则，最终选中了规则X 。你可以测试下，去掉规则 X ，则当前 URL 会匹配上规则C 。 

- 访问 http://localhost/a.gif , http://localhost/b.jpg 将匹配 规则D 和 规则 E ，但是规则 D 顺序优先， 规则 E 不起作用，而 http://localhost/static/c.png 则优先匹配到规则 C 

- 访问 http://localhost/a.PNG 则匹配 规则 E ，而不会匹配规则 D ，因为 规则E不区分大小写。 

- 访问 http://localhost/img/a.gif 会匹配上 规则D ,虽然 规则Y 也可以匹配上，但是因为正则匹配优先，而忽略了规则Y 。 

- 访问 http://localhost/img/a.tiff 会匹配上 规则Y 。

访问 http://localhost/category/id/1111 则最终匹配到规则 F ，因为以上规则都不匹配，这个时候应该是 Nginx 转发请求给后端应用服务器，比如 FastCGI（php），tomcat（jsp），Nginx 作为反向代理服务器存在。

**所以实际使用中，笔者觉得至少有三个匹配规则定义**

```nginx
# 直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。 
# 这里是直接转发给后端应用服务器了，也可以是一个静态首页 
# 第一个必选规则 
location = / { 
    proxy_pass http://tomcat:8080/index 
}
# 第二个必选规则是处理静态文件请求，这是 nginx 作为 http 服务器的强项 
# 有两种配置模式，目录匹配或后缀匹配，任选其一或搭配使用 
location ^~ /static/ { 
    root /webroot/static/; 
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ { 
    root /webroot/res/; 
}
# 第三个规则就是通用规则，用来转发动态请求到后端应用服务器 
# 非静态文件请求就默认是动态请求，自己根据实际把握 
# 毕竟目前的一些框架的流行，带.php、.jsp后缀的情况很少了 
location / { 
    proxy_pass http://tomcat:8080/ 
}
```

### rewrite语法

- last – 基本上都用这个 flag 

- break – 中止 rewrite，不再继续匹配 

- redirect – 返回临时重定向的 HTTP 状态 302 

- permanent – 返回永久重定向的 HTTP 状态 301

1. 下面是可以用来判断的表达式

   ```
   -f 和 !-f 用来判断是否存在文件
   -d 和 !-d 用来判断是否存在目录
   -e 和 !-e 用来判断是否存在文件或目录
   -x 和 !-x 用来判断文件是否可执行
   ```

2. 可以用作判断的全局变量

   ```
   例：http://localhost:88/test1/test2/test.php?k=v 
   $host：localhost 
   $server_port：88 
   $request_uri：/test1/test2/test.php?k=v 
   $document_uri：/test1/test2/test.php 
   $document_root：D:\nginx/html 
   $request_filename：D:\nginx/html/test1/test2/test.php
   ```

### redirect语法

```nginx
server { 
    listen 80; 
    server_name start.igrow.cn; 
    index index.html index.php;
    root html; 
    if ($http_host !~ "^star\.igrow\.cn$") { 
        rewrite ^(.*) http://star.igrow.cn$1 redirect; 
    }
}
```

### 防盗链

```nginx
location ~* \.(gif|jpg|swf)$ { 
    valid_referers none blocked start.igrow.cn sta.igrow.cn; 
    if ($invalid_referer) { 
        rewrite ^/ http://$host/logo.png; 
    } 
}
```

### 根据文件类型设置过期时间

```nginx
location ~* \.(js|css|jpg|jpeg|gif|png|swf)$ {
    if (-f $request_filename) { 
        expires 1h;
        break; 
    } 
}
```

## Nginx静态文件服务

```nginx
http {
    # 这个将为打开文件指定缓存，默认是没有启用的，max 指定缓存数量， 
    # 建议和打开文件数一致，inactive 是指经过多长时间文件没被请求后删除缓存。 
    open_file_cache max=204800 inactive=20s; 
    
    # open_file_cache 指令中的inactive 参数时间内文件的最少使用次数， 
    # 如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个 
    # 文件在inactive 时间内一次没被使用，它将被移除。 
    open_file_cache_min_uses 1; 
    
    # 这个是指多长时间检查一次缓存的有效信息 
    open_file_cache_valid 30s; 
    
    # 默认情况下，Nginx的gzip压缩是关闭的， gzip压缩功能就是可以让你节省不 
    # 少带宽，但是会增加服务器CPU的开销哦，Nginx默认只对text/html进行压缩 ， 
    # 如果要对html之外的内容进行压缩传输，我们需要手动来设置。 
    gzip on; 
    gzip_min_length 1k; 
    gzip_buffers 4 16k; 
    gzip_http_version 1.0; 
    gzip_comp_level 2; 
    gzip_types text/plain application/x-javascript text/css application/xml; 
    server {
        listen 80; 
        server_name www.test.com; 
        charset utf-8;
        root /data/www.test.com; 
        index index.html index.htm; 
    } 
}
```

## 文件缓存漫谈

在浏览器和应用服务器之间，存在多种潜在缓存，如：客户端浏览器缓存、中间缓存、内容分发网络（CDN）和服务器上的负载平衡和反向代理。

缓存内容结果，可以更高效的使用应用服务器，因为不需要每次都去做重复的页面生成工作

**Web 缓存还可以用来提高网站可靠性。当服务器宕机或者繁忙时，比起返回错误信息给用户，不如通过配置 Nginx 将已经缓存下来的内容发送给用户。这意味着，网站在应用服务器或者数据库故障的情况下，可以保持部分甚至全部的功能运转。**

### 如何安装和配置基础缓存

只需要两个命令就可以启用基础缓存：**proxy_cache_path**和**proxy_cache**

- proxy_cache_path用来设置缓存的路径和配置
- proxy_cache用来启用缓存

```nginx
proxy_cache_path/path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive =60m 
use_temp_path=off; 
server {
    ...
    location / { 
        proxy_cache my_cache; 
        proxy_pass http://my_upstream;
    }
}
```

proxy_cache_path 命令中的参数及对应配置说明如下：

- 用于缓存的本地磁盘目录是 /path/to/cache/
- levels 在 /path/to/cache/ 设置了一个两级层次结构的目录
  - 将大量的文件放置在单个目录中会导致文件访问缓慢，所以针对大多数部署，我们推荐使用两级目录层次结构
  - 如果 levels 参数没有配置，则 Nginx 会将所有的文件放到同一个目录中。 

- keys_zone 设置一个共享内存区，该内存区用于存储缓存键和元数据，有些类似计时器的用途。
  - 将键的拷贝放入内存可以使 Nginx 在不检索磁盘的情况下快速决定一个请求是HIT 还是 MISS ，这样大大提高了检索速度。
  - 一个 1MB 的内存空间可以存储大约 8000 个 key，那么上面配置的 10MB 内存空间可以存储差不多 80000 个 key。

- max_size 设置了缓存的上限（在上面的例子中是 10G）
  - 这是一个可选项；如果不指定具体值，那就是允许缓存不断增长，占用所有可用的磁盘空间。
  - 当缓存达到这个上限， 处理器便调用 cache manager 来移除最近最少被使用的文件，这样把缓存的空间降低至这个限制之下。

- inactive 指定了项目在不被访问的情况下能够在内存中保持的时间。
  - 在上面的例子中，如果一个文件在 60 分钟之内没有被请求，则缓存管理将会自动将其在内存中删除，不管该文件是否过期。该参数默认值为 10 分钟（10m）。注意，非活动内容有别于过期内容。 
  - Nginx 不会自动删除由缓存控制头部指定的过期内容（本例中 Cache-Control:max-age=120）。过期内容只有在 inactive 指定时间内没有被访问的情况下才会被删除。
  - 如果过期内容被访问了，那么 Nginx 就会将其从原服务器上刷新，并更新对应的 inactive 计时器。 

- Nginx 最初会将注定写入缓存的文件先放入一个临时存储区域，use_temp_path=off 命令指示 Nginx 将在缓存这些文件时将它们写入同一个目录下。我们强烈建议你将参数设置 为 off 来避免在文件系统中不必要的数据拷贝。

### 陈旧总比没有强

Nginx 内容缓存的一个非常强大的特性是：当无法从原始服务器获取最新的内容时，Nginx 可以分发缓存中的陈旧（stale，编者注：即过期内容）内容。

```nginx
location / {
    ...
    proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
}
```

按照上面例子中的配置，当 Nginx 收到服务器返回的 error，timeout 或者其他指定的 5xx 错误，并且在其缓存中有请求文件的陈旧版本，则会将这些陈旧版本的文件而不是错误信息发送给客户端。

### 缓存微调

```nginx
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactiv e=60m 
use_temp_path=off;
server { 
    ... 
    location / { 
        proxy_cache my_cache;
        proxy_cache_revalidate on;
        proxy_cache_min_uses 3; 
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_5 04; 
        proxy_cache_lock on;
        proxy_pass http://my_upstream; 
    }
}
```

配置的行为

proxy_cache_revalidate

- 指示 Nginx 在刷新来自服务器的内容时使用 GET 请求

proxy_cache_min_uses

- 该指令设置同一链接请求达到几次即被缓存，默认值为 1

proxy_cache_use_stale

- updating 参数告知 Nginx 在客户端请求的项目的更新正在 原服务器中下载时发送旧内容，而不是向服务器转发重复的请求

proxy_cache_lock

- 当启用时，当多个客户端请求一个缓存中不存在的文件（或称之为一个 MISS），只有这些请求中的第一个被允许发送至服务器

## 日志

Nginx日志主要有两种：access_log（访问日志）和error_log（错误日志）

### access_log访问日志

access_log 主要记录客户端访问 Nginx 的每一个请求，格式可以自定义。

- 通过 access_log 你可以得到用户地域来源、跳转来源、使用终端、某个 URL 访问量等相关信息。

log_format 指令用于定义日志的格式，语法:  ``log_format name string;``

默认的无需设置的组合日志格式

```nginx
log_format combined '$remote_addr - $remote_user [$time_local] ' 
	' "$request" $status $body_bytes_sent '
	' "$http_referer" "$http_user_agent" ';
```

access_log 指令用来指定访问日志文件的存放路径（包含日志文件名）、格式和缓存大小，语法：`` access_log path [format_name [buffer=size | off]];`` 其中 path 表示访问日志存放路径，format_name 表示访问日志格式名称，buffer 表示缓存大小，off 表示关闭访问日志。

log_format 使用示例：在 access.log 中记录客户端 IP 地址、请求状态和请求时间

```nginx
log_format myformat '$remote_addr $status $time_local'; 
access_log logs/access.log myformat;
```

**需要注意的是：log_format 配置必须放在 http 内，否则会出现警告。Nginx 进程设置的用户和组必须对日志路径有创建文件的权限，否则，会报错。 **

### error_log错误日志

error_log 主要记录客户端访问 Nginx 出错时的日志，格式不支持自定义

- 通过查看错误日 志，你可以得到系统某个服务或 server 的性能瓶颈等。

error_log 指令用来指定错误日志，语法: ``error_log path [level] ;`` 

- 其中 path 表示错误日志存放路径，
- level 表示错误日志等级，日志等级包括 debug、info、notice、warn、error、crit、 alert、emerg，从左至右，日志详细程度逐级递减，即 debug 最详细，emerg 最少，默认为 error。 

**注意： error_log off 并不能关闭错误日志记录，此时日志信息会被写入到文件名为 off 的文件当中。**

关闭错误日志记录，使用如下配置

Linux系统把存储位置设置为空设备

```nginx
error_log /dev/null; 
http {
    # ... 
}
```

## 反向代理

### 什么是反向代理

反向代理（Reverse Proxy）方式是指用代理服务器来接受 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。 

### 反向代理应用场景

反向代理的典型用途是将防火墙后面的服务器提供给 Internet 用户访问，加强安全防护。

反向代理还可以为后端的多台服务器提供负载均衡，或为后端较慢的服务器提供 缓冲 服务

**场景描述：**

访问本地服务器上的 README.md 文件 http://localhost/README.md，本地服务器进行反向代理，从 https://github.com/moonbingbing/openresty-best-practices/blob/master/README.md 获取页面内容。

**``nginx.conf``**配置

```nginx
worker_processes 1;

pid logs/nginx.pid; 
error_log logs/error.log warn; 

events { 
    worker_connections 3000; 
}

http {
    include mime.types; 
    server_tokens off; 
    
    ## 下面配置反向代理的参数 
    server {
        listen 8866; 
        
        ## 1. 用户访问 http://ip:port，则反向代理到 https://github.com 
        location / { 
            proxy_pass https://github.com; 
            proxy_redirect off; 
            proxy_set_header Host $host; 
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        }
        
        ## 2.用户访问 http://ip:port/README.md，则反向代理到 
        ## https://github.com/.../README.md 
        location /README.md { 
            proxy_set_header X-Real-IP $remote_addr; 
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
            proxy_pass https://github.com/moonbingbing/openresty-best-practices/blob/m aster/README.md; 
        }
    }
}
```

成功启动 Nginx 后，我们打开浏览器，验证下反向代理的效果。在浏览器地址栏中输入 localhost/README.md ，返回的结果是我们 GitHub 源代码的 README 页面

location

- location 项对请求 URI 进行匹配，location 后面配置了匹配规则

proxy_pass

- proxy_pass 后面跟着一个 URL，用来将请求反向代理到 URL 参数指定的服务器上

proxy_set_header

- 默认情况下，反向代理不会转发原始请求中的 Host 头部，如果需要转发，就需要加上这句：`proxy_set_header Host $host;`
- 其他配置用法[Module ngx_http_proxy_module (nginx.org)](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)

### 正向代理

正向代理就像一个跳板，例如一个用户访问不了某网站（例如 `www.google.com`），但是他能访问一个代理服务器，这个代理服务器能访问 `www.google.com`，于是用户可以先连上代理服务器，告诉它需要访问的内容，代理服务器去取回来返回给用户

[负载均衡 · OpenResty最佳实践 (gitbooks.io)](https://moonbingbing.gitbooks.io/openresty-best-practices/content/ngx/balancer.html)

# 安装openresty

## docker安装

拉取镜像

```bash
docker pull openresty/openresty
```

创建两个文件夹，conf和logs

然后在conf中创建nginx.conf

```bash
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;
        location / {
            default_type text/html;
            content_by_lua '
                ngx.say("<p>hello, world</p>")
            ';
        }
    }
}
```

启动docker

```bash
docker run -d --name="openresty" -p 180:80 -v $PWD/config:/usr/local/openresty/nginx/conf:ro -v $PWD/logs:/usr/local/openresty/nginx/logs openresty/openresty
```



```bash
docker run -d --name openresty -p 8989:80 -v $PWD/config:/usr/local/openresty/nginx/conf -v $PWD/logs:/usr/local/openresty/nginx/logs  -d openresty/openresty
```

hehopenresty

```bash
docker run -d --name="heheopenresty" -p 180:80 -e SRC="docker://10.0.2.15:5000/" -v $PWD/config:/usr/local/openresty/nginx/conf:ro -v $PWD/logs:/usr/local/openresty/nginx/logs openresty/openresty
```



安装vim

```bash
apt update
apt install -y vim
```



不挂载了

```bash
docker run -d --name="openresty" -p 180:80
```



启动失败删除镜像

```bash
docker rm $(docker ps -a -q) 
```



**docker部署**

启动OpenResty

```bash
docker run -id --name openresty -p 80:80 openresty/openresty
```

创建挂载目录

```bash
mkdir -p /opt/docker/openresty
```

将容器内的初始配置拷贝

```bash
docker cp openresty:/usr/local/openresty /opt/docker/
```

删除OpenResty容器

```bash
docker rm -f openresty
```

挂载方式启动

```bash
docker run -id -p 80:80 \
--name openresty --restart always --privileged=true \
-v /opt/docker/openresty/nginx/conf:/usr/local/openresty/nginx/conf \
-v /opt/docker/openresty/nginx/logs:/usr/local/openresty/nginx/logs \
-v /var/www/download:/var/www/download \
-v /etc/localtime:/etc/localtime \
openresty/openresty
```

Master02-openresty_base

```bash
docker run -id -p 80:80 \
--name openresty --restart always --privileged=true \
-v /opt/docker/openresty/nginx/conf:/usr/local/openresty/nginx/conf \
-v /opt/docker/openresty/nginx/logs:/usr/local/openresty/nginx/logs \
-v /etc/localtime:/etc/localtime \
louisyuan/openresty_base
```

youyou

```bash
docker run -id -p 9090:8080 --name youyou louisyuan/youyou
```



hehe

```bash
docker run -id -p 80:80 \
--name heheopenresty --restart always --privileged=true \
-v /opt/docker/openresty/nginx/conf:/usr/local/openresty/nginx/conf \
-v /opt/docker/openresty/nginx/logs:/usr/local/openresty/nginx/logs \
-v /etc/localtime:/etc/localtime \
louisyuan/openresty:1.17.8.2-ubuntu20.04
```

合合openresty

```bash
docker run -id -p 880:80 -e SRC="192.168.1.103:6000" -e DST="192.168.1.103:6000" -e PREFIX="studio,bulk"  \
--name openresty --restart always --privileged=true \
-v /opt/docker/openresty/nginx/conf:/usr/local/openresty/nginx/conf \
-v /opt/docker/openresty/nginx/logs:/usr/local/openresty/nginx/logs \
-v /etc/localtime:/etc/localtime \
openresty/openresty
```

本地

```bash
docker run -id -p 880:80 -e SRC="192.168.1.103:6000" -e DST="192.168.1.103:6000" -e PREFIX="studio,bulk"  \
--name openresty --restart always --privileged=true \
-v /Users/louis/MyDir/Company/transfer_image/conf:/usr/local/openresty/nginx/conf \
-v /Users/louis/MyDir/Company/transfer_image/logs:/openresty/nginx/logs \
openresty/openresty
```

本地合合

```bash
docker run -id -p 80:80  \
--name heheopenresty --restart always --privileged=true \
-v /Users/louis/MyDir/Company/StudyProject/TestTee:/usr/local/openresty/nginx/conf \
harbor.intsig.net/base/openresty:1.17.8.2-ubuntu20.04
```

oidc master01

```
docker run -id -p 80:80 \
--name openresty_oidc --restart always --privileged=true \
-v /root/oidc/openresty/nginx/conf:/usr/local/openresty/nginx/conf \
-v /root/oidc/openresty/nginx/logs:/usr/local/openresty/nginx/logs \
-v /etc/localtime:/etc/localtime \
louisyuan/openresty_oidc
```



openresty

```bash
docker run -id -p 180:80 \
--name openresty --restart always --privileged=true \
-v /Users/louis/MyDir/Company/tmp/tee/nginx:/usr/local/openresty/nginx/conf \
-v /Users/louis/MyDir/logs:/openresty/nginx/logs \
openresty/openresty
```

**本地测试Openresty**

```bash
docker run -id -p 80:80 \
--name openresty --restart always --privileged=true \
-v /Users/louis/MyDir/Company/StudyProject/openresty-oidc/conf:/usr/local/openresty/nginx/conf \
-v /Users/louis/MyDir/Company/StudyProject/openresty-oidc/logs:/openresty/nginx/logs \
louisyuan/openresty_oidc:latest
```

run

```
docker run -id -p 880:80 -e SRC="192.168.1.103:6000" -e DST="192.168.1.103:6000" -e PREFIX="studio,build-" --name transfer registry.intsig.net/jun_yuan/transfer_images
```

bendi

```bash
docker login -u jun_yuan -p @#Yuanjun0903 registry.intsig.net/jun_yuan    
docker build . -t transfer:latest
docker tag transfer:latest registry.intsig.net/jun_yuan/transfer_images:latest
docker push registry.intsig.net/jun_yuan/transfer_images:latest
```

服务器

```
docker run --rm --name kubectl -v /home/jun_yuan:/.kube/config cnych/kubectl get pods --kubeconfig=/.kube/config/config
```





查看日志

```bash
docker logs -f 镜像ID
```



## Centos安装OpenResty

### 安装nginx

```bash
yum install gcc-c++
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
yum install wget 
wget -c https://nginx.org/download/nginx-1.20.2.tar.gz
tar -zxvf nginx-1.20.2.tar.gz
cd nginx-1.20.2
./configure
make
make install
```

启动，停止nginx

```bash
cd /usr/local/nginx/sbin/
./nginx 
./nginx -s stop
./nginx -s quit
./nginx -s reload
```

### 安装OpenResty

```bash
# add the yum repo:
wget https://openresty.org/package/centos/openresty.repo
sudo mv openresty.repo /etc/yum.repos.d/

# update the yum index:
sudo yum check-update

sudo yum install openresty

sudo yum install openresty-resty

sudo yum --disablerepo="*" --enablerepo="openresty" list available
```

# OpenResty

创建工作目录

```bash
mkdir geektime
cd geektime
mkdir logs/ conf/
```

创建最简化的nginx.conf

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 8080;
        location / {
            content_by_lua '
                ngx.say("hello, world")
            ';
        }
    }
}
```

启动OpenResty服务

```bash
killall -9 openresty
openresty -p `pwd` -c conf/nginx.conf
```

```bash
$ curl -i 127.0.0.1:8080
HTTP/1.1 200 OK
Server: openresty/1.13.6.2
Content-Type: text/plain
Transfer-Encoding: chunked
Connection: keep-alive

hello, world
```

创建lua目录，放lua代码

```bash
$ mkdir lua
$ cat lua/hello.lua
ngx.say("hello, world")
```

修改nginx配置

```nginx
pid logs/nginx.pid;
events {
  worker_connections 1024;
}

http {
  server {
    listen 8080;
    location / {
      content_by_lua_file lua/hello.lua;
      }
    }
  }
```

重启OpenResty

```bash
sudo kill -HUP `cat logs/nginx.pid`
```

首次启动/重启nginx使用：

killall -9 nginx //杀死进程

``/usr/local/openresty/nginx/sbin/nginx -c /root/workplace/conf/nginx.conf -p /usr/local/openresty/nginx/sbin/nginx -s reload `` 

后续修改配置文件后，重启可使用：
``/usr/local/openresty/nginx/sbin/nginx -s reload``

杀死进程，OpenResty

ps -A | grep openresty 

kill -9 pid1

### NGINX C模块

OpenResty 的项目命名都是有规范的，以 *-nginx-module命名的就是 NGINX 的 C 模块

最核心的是lua-nginx-module和stream-lua-nginx-module

- 前者用来处理七层流量，后者用来处理四层流量

OpenResty后面不会开发更多的NGINX C库，**专注于cosocket的Lua库**

### OPM

OpenResty Package Manager，是OpenResty自带的包管理器

**应该使用cosocket的lua-resty-*库解决问题**

找发送HTTP请求的库 ``opm search http``

### 元表的操作

__index重载

把 patch 从 version 这个 table 中去掉

```bash
$ resty -e ' local version = {
  major = 1,
  minor = 1
  }
version = setmetatable(version, {
     __index = function(t, key)
         if key == "patch" then
             return 2
         end
     end,
     __tostring = function(t)
      return string.format("%d.%d.%d", t.major, t.minor, t.patch)
    end
  })
  print(tostring(version))
'
```

__index 不仅可以是一个函数，也可以是一个 table。你试着运行下面这段代码，就会看到，它们实现的效果是一样的。

```bash
$ resty -e ' local version = {
  major = 1,
  minor = 1
  }
version = setmetatable(version, {
     __index = {patch = 2},
     __tostring = function(t)
      return string.format("%d.%d.%d", t.major, t.minor, t.patch)
    end
  })
  print(tostring(version))
'
```

__call，类似于仿函数，可以让table被调用

```bash
$ resty -e '
local version = {
  major = 1,
  minor = 1,
  patch = 1
  }

local function print_version(t)
     print(string.format("%d.%d.%d", t.major, t.minor, t.patch))
end

version = setmetatable(version,
     {__call = print_version})

  version()
'
```

 getmetatable 是和 setmetatable 配对的操作，可以获取到已经设置的元表，比如下面这段代码：

```bash
$ resty -e ' local version = {
  major = 1,
  minor = 1
  }
version = setmetatable(version, {
     __index = {patch = 2},
     __tostring = function(t)
      return string.format("%d.%d.%d", t.major, t.minor, t.patch)
    end
  })
  print(getmetatable(version).__index.patch)
'
```

### 面向对象

Lua 并不是一个面向对象（Object Orientation）的语言，但我们可以使用 metatable 来实现 OO。

```bash
$ resty -e 'local mysql = require "resty.mysql" -- 先引用 lua-resty 库
local db, err = mysql:new() -- 新建一个类的实例
db:set_timeout(1000) -- 调用类的方法'
```

在调用类方法的时候，为什么是冒号而不是点号呢？

- 在这里冒号和点号都是可以的，db:set_timeout(1000) 和 db.set_timeout(db, 1000) 是完全等价的。冒号是 Lua 中的一个语法糖，可以省略掉函数的第一个参数 self。

### json编码时无法区分array和dict

json 编码时无法区分 array 和 dict。由于 Lua 中只有 table 这一个数据结构，所以在 json 对空 table 编码的时候，自然就无法确定编码为数组还是字典：

```bash
resty -e 'local cjson = require "cjson"
local t = {}
print(cjson.encode(t))
'
```

比如上面这段代码，它的输出是 {}，由此可见， OpenResty 的 cjson 库，默认把空 table 当做字典来编码。当然，我们可以通过 encode_empty_table_as_object 这个函数，来修改这个全局的默认值：

```bash
resty -e 'local cjson = require "cjson"
cjson.encode_empty_table_as_object(false)
local t = {}
print(cjson.encode(t))
'
```

这次，空table就被编码成数组：[]

不过，全局这种设置的影响面比较大，那能不能指定某个 table 的编码规则呢？答案自然是可以的，我们有两种方法可以做到。

第一种方法，把 cjson.empty_array 这个 userdata 赋值给指定 table。这样，在 json 编码的时候，它就会被当做空数组来处理：

```bash
$ resty -e 'local cjson = require "cjson"
local t = cjson.empty_array
print(cjson.encode(t))
'
```

不过，有时候我们并不确定，这个指定的 table 是否一直为空。我们希望当它为空的时候编码为数组，那么就要用到 cjson.empty_array_mt 这个函数，也就是我们的第二个方法。它会标记好指定的 table，当 table 为空时编码为数组。从cjson.empty_array_mt 这个命名你也可以看出，它是通过 metatable 的方式进行设置的，比如下面这段代码操作：

```bash
$ resty -e 'local cjson = require "cjson"
local t = {}
setmetatable(t, cjson.empty_array_mt)
print(cjson.encode(t))
t = {123}
print(cjson.encode(t))
'
```

### 变量个数有限制

变量个数可以修改，不过最大也只能设置为 250

不要过多地使用局部变量和 upvalue，而是要尽可能地使用 do .. end 做一层封装，来减少局部变量和 upvalue 的个数。

例如

```lua
local re_find = ngx.re.find
  function foo() ... end
function bar() ... end
function fn() ... end
```

如果只有函数 foo 使用到了 re_find， 那么我们可以这样改造下：

```lua
do
     local re_find = ngx.re.find
     function foo() ... end
end
function bar() ... end
function fn() ... end
```

## 基于FFI实现的lua-resty-lrucache

lrucache 仓库中包含了两种实现方案，一种是使用 Lua table 来实现缓存，另外一种则是使用 hash 表来实现。前者更适合命中率高的情况，后者适合命中率低的情况。两个方案没有哪个更好，要看你的线上环境更适合哪一个。

**在你面对一个陌生的开源项目时，文档和测试案例永远是最好的上手方式。而你后期如果要阅读源码，也不要先去抠细节，而是应该先去看主要的数据结构，围绕重点逐层深入。**

https://github.com/iresty/geektime-slides

### Lua规则和NGINX配置文件产生冲突怎么办

当 OpenResty 中的 Lua 规则和 NGINX 配置文件产生冲突时，比如 NGINX 配置了 rewrite 规则，又同时引用了 rewrite_by_lua_file，那么这两条规则的优先级是什么？

- 其实，这个具体要看 NGINX 配置的 rewrite 规则是怎么写的了，是 break 还是 last。这一点，在 OpenResty 的官方文档中有注明，并且配了一个示例代码：

- ```nginx
   location /foo {
       rewrite ^ /bar;
       rewrite_by_lua 'ngx.exit(503)';
   }
   location /bar {
       ...
   }
  ```

在示例代码的这个配置中，ngx.exit(503) 是不会被执行的。

但是，如果你改成下面这样的写法，ngx.exit(503) 就可以被执行

```nginx
rewrite ^ /bar break；
```

为了避免这种歧义，我还是建议都使用 OpenResty 来处理 rewrite，而不是 NGINX 的配置。说实话，NGINX 的很多配置是比较晦涩的，需要你反复查阅文档才能读懂。

**关于空值的困惑**

```lua
local res, err = red:get("dog")
```

如果返回值 res 是 nil，就说明函调用失败了；如果 res 是 ngx.null ，就说明 redis 中不存在 dog 这个 key。这是因为， Lua 的 nil 无法作为 table 的 value，所以 OpenResty 引入了 ngx.null，作为 table 中的空值。

**文中一直说的 API 网关是指什么？和 NGINX、Tomcat、Apache 这种 Web 服务器，又有什么区别呢？**

- API 网关其实是用来统一管理服务的网关。举个例子，像是支付、用户登录等，都是 API 形式对外提供的服务，它们都需要一个网关来做统一的安全和身份认证。
- API 网关可以替代传统的 NGINX、Apache 来处理南北向流量，也可以在微服务环境下处理东西向的流量，是更加贴近业务的一种中间件，而非底层的 Web 服务器。

## OpenResty和别的开发平台有什么不同？

OpenResty 的 master 和 worker 进程中，都包含一个 LuaJIT VM。在同一个进程内的所有协程，都会共享这个 VM，并在这个 VM 中运行 Lua 代码。

而在同一个时间点上，每个 worker 进程只能处理一个用户的请求，也就是只有一个协程在运行。看到这里，你可能会有一个疑问：NGINX 既然能够支持 C10K （上万并发），不是需要同时处理一万个请求吗？

当然不是，NGINX 实际上是通过 epoll 的事件驱动，来减少等待和空转，才尽可能地让 CPU 资源都用于处理用户的请求。毕竟，只有单个的请求被足够快地处理完，整体才能达到高性能的目的。如果采用的是多线程模式，让一个请求对应一个线程，那么在 C10K 的情况下，资源很容易就会被耗尽的。

在 OpenResty 层面，Lua 的协程会与 NGINX 的事件机制相互配合。如果 Lua 代码中出现类似查询 MySQL 数据库这样的 I/O 操作，就会先调用 Lua 协程的 yield 把自己挂起，然后在 NGINX 中注册回调；在 I/O 操作完成（也可能是超时或者出错）后，再由 NGINX 回调 resume 来唤醒 Lua 协程。这样就完成了 Lua 协程和 NGINX 事件驱动的配合，避免在 Lua 代码中写回调。

我们可以来看下面这张图，描述了这整个流程。其中，lua_yield 和 lua_resume 都属于 Lua 提供的 lua_CFunction。

![img](https://static001.geekbang.org/resource/image/fa/34/fae1008edb43c7476cf2f20da9928234.png)

 sleep 的主函数，主要的代码

```lua
static int ngx_http_lua_ngx_sleep(lua_State *L)
{
    coctx->sleep.handler = ngx_http_lua_sleep_handler;
    ngx_add_timer(&coctx->sleep, (ngx_msec_t) delay);
    return lua_yield(L, 0);
}
```

- 这里先增加了 ngx_http_lua_sleep_handler 这个回调函数；
- 然后调用 ngx_add_timer 这个 NGINX 提供的接口，向 NGINX 的事件循环中增加一个定时器；
- 最后使用 lua_yield 把 Lua 协程挂起，把控制权交给 NGINX 的事件循环

当 sleep 操作完成后， ngx_http_lua_sleep_handler 这个回调函数就被触发了。它里面调用了 ngx_http_lua_sleep_resume, 并最终使用 lua_resume 唤醒了 Lua 协程

**基本概念**

OpenResty 中阶段和非阻塞这两个重要的概念。

- set_by_lua，用于设置变量；
- rewrite_by_lua，用于转发、重定向等；
- access_by_lua，用于准入、权限等；
- content_by_lua，用于生成返回内容；
- header_filter_by_lua，用于应答头过滤处理；
- body_filter_by_lua，用于应答体过滤处理；
- log_by_lua，用于日志记录。

当然，如果你的代码逻辑并不复杂，都放在 rewrite 或者 content 阶段执行，也是可以的。

在使用 API 之前，一定记得要先查阅文档，确定其能否在代码的上下文中使用

回顾下非阻塞。首先明确一点，**由 OpenResty 提供的所有 API，都是非阻塞的。**

### 变量和生命周期

通常来说，试图用全局变量来解决的问题，其实更应该用模块的变量来解决，而且还会更加清晰。下面是一个模块中变量的示例：

```lua
local _M = {}

_M.color = {
      red = 1,
      blue = 2,
      green = 3
  }

  return _M
```

在一个名为 hello.lua 的文件中定义了一个模块，模块包含了 color 这个 table。然后，我又在 nginx.conf 中增加了对应的配置：

```lua
location / {
    content_by_lua_block {
        local hello = require "hello"
        ngx.say(hello.color.green)
     }
}
```

这段配置会在 content 阶段中 require 这个模块，并把 green 的值作为 http 请求返回体打印出来。

实际上，在同一 worker 进程中，模块只会被加载一次；之后这个 worker 处理的所有请求，就可以共享模块中的数据了。我们说“全局”的数据很适合封装在模块内，是因为 OpenResty 的 worker 之间完全隔离，所以每个 worker 都会独立地对模块进行加载，而模块的数据也不能跨越 worker。

**访问模块变量的时候，你最好保持只读，而不要尝试去修改，不然在高并发的情况下会出现 race**

OpenResty 提供了 ngx.ctx，跨越阶段的、可以读写的变量。它是一个 Lua table，可以用来存储基于请求的 Lua 数据，且生存周期与当前请求相同。我们来看下官方文档中的这个示例：

```nginx
location /test {
      rewrite_by_lua_block {
          ngx.ctx.foo = 76
      }
      access_by_lua_block {
          ngx.ctx.foo = ngx.ctx.foo + 3
      }
      content_by_lua_block {
          ngx.say(ngx.ctx.foo)
      }
  }
```

你可以看到，我们定义了一个变量 foo，存放在 ngx.ctx 中。这个变量跨越了 rewrite、access 和 content 三个阶段，最终在 content 阶段打印出了值，并且是我们预期的 79。

ngx.ctx的局限性

- 比如说，使用 ngx.location.capture 创建的子请求，会有自己独立的 ngx.ctx 数据，和父请求的 ngx.ctx 互不影响；
- 再如，使用 ngx.exec 创建的内部重定向，会销毁原始请求的 ngx.ctx，重新生成空白的 ngx.ctx。
- https://github.com/openresty/lua-nginx-module#ngxctx

## 文档和测试案例

### shdict get API

shared dict（共享字典）是基于 NGINX 共享内存区的 Lua 字典对象，它可以跨多个 worker 来存取数据，一般用来存放限流、限速、缓存等数据。shared dict 相关的 API 有 20 多个，是 OpenResty 中最常用也是最重要的一组 API。

https://github.com/openresty/lua-nginx-module/#ngxshareddictget

最简化代码

```nginx
  http {
      lua_shared_dict dogs 10m;
      server {
          location /demo {
              content_by_lua_block {
                  local dogs = ngx.shared.dogs
         dogs:set("Jim", 8)
         local v = dogs:get("Jim")
                  ngx.say(v)
              }
          }
      }
  }
```

在 Lua 代码中使用 shared dict 之前，我们需要在 nginx.conf 中用 lua_shared_dict 指令增加一块内存空间，它的名字是 dogs，大小为 10M。修改完 nginx.conf 后，你还需要重启进程，用浏览器或者 curl 访问才能看到结果

**使用 resty CLI 的这种方式，和在 nginx.conf 中嵌入代码的效果是一致的。**

```bash
$ resty --shdict 'dogs 10m' -e 'local dogs = ngx.shared.dogs
 dogs:set("Jim", 8)
 local v = dogs:get("Jim")
 ngx.say(v)
 '
```

### 哪些阶段不能使用共享内存相关的 API ？

文档中专门有一个 context （即上下文部分），里面列出了在什么环境下可以使用这个 API：

```bash
context: set_by_lua*, rewrite_by_lua*, access_by_lua*, content_by_lua*, header_filter_by_lua*, body_filter_by_lua*, log_by_lua*, ngx.timer.*, balancer_by_lua*, ssl_certificate_by_lua*, ssl_session_fetch_by_lua*, ssl_session_store_by_lua*
```

init 和 init_worker 两个阶段不在其中，也就是说，共享内存的 get API 不能在这两个阶段使用。需要注意的是，每个共享内存的 API 可以使用的阶段并不完全相同，比如 set API 就可以在 init 阶段使用。

OpenResty 的测试案例都放在 /t 目录下，并且命名也是有规律的，即**自增数字-功能名.t。搜索shdict**，你可以找到 043-shdict.t，而这就是共享内存的测试案例集了，它里面有接近 100 个测试案例，包含各种正常和异常情况的测试。

把 content 阶段改为 init 阶段，并精简掉无关代码，看看 get 接口能否运行。

```lua
=== TEST 1: string key, int value
 --- http_config
     lua_shared_dict dogs 1m;
 --- config
     location = /test {
         init_by_lua '
             local dogs = ngx.shared.dogs
             local val = dogs:get("foo")
             ngx.say(val)
         ';
     }
 --- request
 GET /test
 --- response_body
 32
 --- no_error_log
 [error]
 --- ONLY
```

在测试案例的最后，加了 --ONLY 标记，这表示忽略其他所有测试案例，只运行这一个测试案例，以提高运行速度

用 prove 命令，就可以运行这个测试案例

```bash
$ prove t/043-shdict.t
```

得到一个报错，这也就印证了文档中描述的阶段限制。

```bash
nginx: [emerg] "init_by_lua" directive is not allowed here
```

### get 函数何时会有多个返回值？

```lua
value, flags = ngx.shared.DICT:get(key)
```

- 第一个参数value 返回的是字典中 key 对应的值；但当 key 不存在或者过期时，value 的值为 nil。
- 第二个参数 flags 就稍微复杂一些了，如果 set 接口设置了 flags，就返回，否则不返回。

一旦 API 调用出错，value 返回 nil，flags 返回具体的错误信息。

``local v = dogs:get("Jim") ``这种只有一个接收参数的写法并不完善，因为它只覆盖了普通的使用场景，没有接收第二个参数，也没有做异常处理。我们可以把它修改为下面这样

```lua
local data, err = dogs:get("Jim")
if data == nil and err then
    ngx.say("get not ok: ", err)
    return
end
```

到测试案例集里搜索一下，印证下我们对文档的理解

```lua
=== TEST 65: get nil key
 --- http_config
     lua_shared_dict dogs 1m;
 --- config
     location = /test {
         content_by_lua '
             local dogs = ngx.shared.dogs
             local ok, err = dogs:get(nil)
             if not ok then
                 ngx.say("not ok: ", err)
                 return
             end
             ngx.say("ok")
         ';
     }
 --- request
 GET /test
 --- response_body
 not ok: nil key
 --- no_error_log
 [error]
```

在这个测试案例中，get 接口的入参为 nil，返回的 err 信息是 nil key。这一方面验证了我们对文档的分析是正确的，另一方面，也为第三个问题提供了部分答案——起码，get 的入参不能是 nil

**get API 接受的 key 类型为字符串和数字**

**key 的最大长度正是 65535**

**在 OpenResty 的 API 中，凡是返回值中带有错误信息的，都必须有变量来接收并做错误处理**

```
驱动测试案例：
1. 安装相关模块
sudo yum install cpan -y
sudo cpan YAML
sudo cpan Test::Nginx

2. test 测试案例
git clone git@github.com:openresty/lua-nginx-module.git
cd lua-nginx-module/
prove t/043-shdict.t

请问OpenResty的测试环境要怎样搭建？
最简单的方式是通过openresty的RPM源来安装
yum -y install perl-Test-Nginx

可以试下cpanm来安装
yum -y install -y install perl-App-cpanminus.noarch
cpanm --mirror http://mirrors.163.com/cpan --mirror-only YAML
cpanm --mirror http://mirrors.163.com/cpan --mirror-only Test::Nginx

得安装 test::nginx
https://github.com/openresty/test-nginx#installation
```

## 动态处理请求和响应

### API分类

- 处理请求和响应；
- SSL 相关；
- shared dict；
- cosocket；
- 处理四层流量；
- process 和 worker；
- 获取 NGINX 变量和配置；
- 字符串、时间、编解码等通用功能。

https://github.com/openresty/lua-nginx-module/#nginx-api-for-lua

### 请求行

HTTP 的请求行中包含请求方法、URI 和 HTTP 协议版本。

在 NGINX 中，你可以通过内置变量的方式，来获取其中的值；而在 OpenResty 中对应的则是 ngx.var.* 这个 API。

- $scheme 这个内置变量，在 NGINX 中代表协议的名字，是 “http” 或者 “https”；而在 OpenResty 中，你可以通过 ngx.var.scheme 来返回同样的值。
- $request_method 代表的是请求的方法，“GET”、“POST” 等；而在 OpenResty 中，你可以通过 ngx.var. request_method 来返回同样的值。

既然可以通过ngx.var.* 这种返回变量值的方法，来得到请求行中的数据，为什么 OpenResty 还要单独提供针对请求行的 API 呢？

- 首先是对性能的考虑。ngx.var 的效率不高，不建议反复读取；
- 也有对程序友好的考虑，ngx.var 返回的是字符串，而非 Lua 对象，遇到获取 args 这种可能返回多个值的情况，就不好处理了；
- 另外是对灵活性的考虑，绝大部分的 ngx.var 是只读的，只有很少数的变量是可写的，比如 $args 和 limit_rate，可很多时候，我们会有修改 method、URI 和 args 的需求。

**OpenResty 提供了多个专门操作请求行的 API，它们可以对请求行进行改写，以便后续的重定向等操作。**

如何通过 API 来获取 HTTP 协议版本号

- OpenResty 的 API ngx.req.http_version 和 NGINX 的 $server_protocol 变量的作用一样，都是返回 HTTP 协议的版本号。不过这个 API 的返回值是数字格式，而非字符串，可能的值是 2.0、1.0、1.1 和 0.9，如果结果不在这几个值的范围内，就会返回 nil

获取请求行中的请求方法

- ngx.req.get_method 和 NGINX 的 $request_method 变量的作用、返回值一样，都是字符串格式的方法名。

改写当前 HTTP 请求方法的 API

-  ngx.req.set_method，它接受的参数格式却并非字符串，而是内置的数字常量

```nginx
ngx.req.set_method(ngx.HTTP_POST)
```

这个 NGINX 配置：

```bash
rewrite ^ /foo?a=3? break;
```

如何用等价的 Lua API 来解决呢？

```lua
ngx.req.set_uri_args("a=3")
ngx.req.set_uri("/foo")
```

### 请求头

HTTP 的请求头是 key : value 格式的，比如

```html
Accept: text/css,*/*;q=0.1
Accept-Encoding: gzip, deflate, br
```

在 OpenResty 中，你可以使用 ngx.req.get_headers 来解析和获取请求头，返回值的类型则是 table：

```lua
local h, err = ngx.req.get_headers()
 
  if err == "truncated" then
      -- one can choose to ignore or reject the current request here
  end
 
  for k, v in pairs(h) do
      ...
  end
```

这里默认返回前 100 个 header，如果请求头超过了 100 个，就会返回 truncated 的错误信息，由开发者自己决定如何处理。

OpenResty 并没有提供获取某一个指定请求头的 API，也就是没有 ngx.req.header['host'] 这种形式。如果你有这样的需求，那就需要借助 NGINX 的变量 $http_xxx 来实现了，那么在 OpenResty 中，就是 ngx.var.http_xxx 这样的获取方式。

改写和删除请求头

```lua
ngx.req.set_header("Content-Type", "text/css")
ngx.req.clear_header("Content-Type")
```

### 请求体

出于性能考虑，OpenResty 不会主动读取请求体的内容，除非你在 nginx.conf 中强制开启了lua_need_request_body 指令，对于比较大的请求体，OpenResty 会把内容保存在磁盘的临时文件中，所以读取请求体的完整流程是下面这样的：

```lua
ngx.req.read_body()
local data = ngx.req.get_body_data()
if not data then
    local tmp_file = ngx.req.get_body_file()
     -- io.open(tmp_file)
     -- ...
 end
```

这段代码中有读取磁盘文件的 IO 阻塞操作。你应该根据实际情况来调整 client_body_buffer_size 配置的大小（64 位系统下默认是 16 KB），尽量减少阻塞的操作；你也可以把 client_body_buffer_size 和 client_max_body_size 配置成一样的，完全在内存中来处理，当然，这取决于你内存的大小和处理的并发请求数。

### 状态行

在默认情况下，返回的 HTTP 状态码是 200，也就是 OpenResty 中内置的常量 ngx.HTTP_OK

如果你检测了请求报文，发现这是一个恶意的请求，那么你需要终止请求:

```lua
ngx.exit(ngx.HTTP_BAD_REQUEST)
```

OpenResty 的 HTTP 状态码中，有一个特别的常量：ngx.OK。当 ngx.exit(ngx.OK) 时，请求会退出当前处理阶段，进入下一个阶段，而不是直接返回给客户端。

### 响应头

```lua
ngx.header.content_type = 'text/plain'
ngx.header["X-My-Header"] = 'blah blah'
ngx.header["X-My-Header"] = nil -- 删除
```

这里的 ngx.header 保存了响应头的信息，可以读取、修改和删除。

第二种设置响应头的方法是 ngx_resp.add_header ，来自 lua-resty-core 仓库，它可以增加一个头信息，用下面的方法来调用：

```lua
local ngx_resp = require "ngx.resp"
ngx_resp.add_header("Foo", "bar")
```

与第一种方法的不同之处在于，add header 不会覆盖已经存在的同名字段。

### 响应体

在 OpenResty 中，你可以使用 ngx.say 和 ngx.print 来输出响应体：

```lua
ngx.say('hello, world')
```

这两个 API 的功能是一致的，唯一的不同在于， ngx.say 会在最后多一个换行符

为了避免字符串拼接的低效，ngx.say / ngx.print 不仅支持字符串作为参数，也支持数组格式：

```lua
$ resty -e 'ngx.say({"hello", ", ", "world"})'
 hello, world
```

## work间的通信法宝：最重要的数据结构之shared dict

共享内存字典 shared dict，是你在 OpenResty 编程中最为重要的数据结构。它不仅支持数据的存放和读取，还支持原子计数和队列操作。

基于 shared dict，你可以实现多个 worker 之间的缓存和通信，以及限流限速、流量统计等功能。**你可以把 shared dict 当作简单的 Redis 来使用，只不过 shared dict 中的数据不能持久化，所以你存放在其中的数据，一定要考虑到丢失的情况。**

### 数据共享的几种方式

第一种是 Nginx 中的变量

```nginx
location /foo {
     set $my_var ''; # this line is required to create $my_var at config time
     content_by_lua_block {
         ngx.var.my_var = 123;
         ...
     }
 }
```

第二种是ngx.ctx，可以在同一个请求的不同阶段之间共享数据。它其实就是一个普通的 Lua 的 table，所以速度很快，还可以存储各种 Lua 的对象。**它的生命周期是请求级别的，当一个请求结束的时候，ngx.ctx 也会跟着被销毁掉。**

```nginx
location /test {
     rewrite_by_lua_block {
         ngx.ctx.host = ngx.var.host
     }
     access_by_lua_block {
        if (ngx.ctx.host == 'openresty.org') then
            ngx.ctx.host = 'test.com'
        end
     }
     content_by_lua_block {
         ngx.say(ngx.ctx.host)
     }
 }
```

第三种方法是使用模块级别的变量，在同一个 worker 内的所有请求之间共享数据。

先来看个例子，弄明白什么是 模块级别的变量：

```lua
-- mydata.lua
 local _M = {}

 local data = {
     dog = 3,
     cat = 4,
     pig = 5,
 }

 function _M.get_age(name)
     return data[name]
 end

 return _M
```

在 nginx.conf 的配置如下：

```nginx
location /lua {
     content_by_lua_block {
         local mydata = require "mydata"
         ngx.say(mydata.get_age("dog"))
     }
 }
```

mydata 就是一个模块，它只会被 worker 进程加载一次，之后，这个 worker 处理的所有请求，都会共享 mydata 模块的代码和数据。

mydata 模块中的 data 这个变量，就是 模块级别的变量，它位于模块的 top level，也就是模块最开始的位置，所有函数都可以访问到它。

除非你很确定这中间没有 yield 操作，不会把控制权交给 Nginx 事件循环，否则，我建议你还是保持对模块级别变量的只读。

**第四种，也是最后一种方法，用 shared dict 来共享数据，这些数据可以在多个 worker 之间共享。**

- 这种方法是基于红黑树实现的，性能很好，但也有自己的局限性——你必须事先在 Nginx 的配置文件中，声明共享内存的大小，并且这不能在运行期更改：

```lua
lua_shared_dict dogs 10m;
```

shared dict 同样只能缓存字符串类型的数据，不支持复杂的 Lua 数据类型。这也就意味着，当我需要存放 table 等复杂的数据类型时，我将不得不使用 json 或者其他的方法，来序列化和反序列化，这自然会带来不小的性能损耗。

### 共享字典

它对外提供了 20 多个 Lua API，不过所有的这些 API 都是原子操作，你不用担心多个 worker 和高并发的情况下的竞争问题。

https://github.com/openresty/lua-nginx-module#ngxshareddict

**字典读写类**

只有字典读写类的 API，它们也是共享字典最常用的功能。下面是一个最简单的示例：

```lua
$ resty --shdict='dogs 1m' -e 'local dict = ngx.shared.dogs
                               dict:set("Tom", 56)
                               print(dict:get("Tom"))'
```

除了 set 外，OpenResty 还提供了 safe_set、add、safe_add、replace 这四种写入的方法。这里safe 前缀的含义是，在内存占满的情况下，不根据 LRU 淘汰旧的数据，而是写入失败并返回 no memory 的错误信息。

除了 get 外，OpenResty 还提供了 get_stale 的读取数据的方法，相比 get 方法，它多了一个过期数据的返回值：

```lua
value, flags, stale = ngx.shared.DICT:get_stale(key)
```

你还可以调用 delete 方法来删除指定的 key，它和 set(key, nil) 是等价的。

**队列操作类**

再来看队列操作，它是 OpenResty 后续新增的功能，提供了和 Redis 类似的接口。队列中的每一个元素，都用 ngx_http_lua_shdict_list_node_t 来描述

```c
typedef struct { 
    ngx_queue_t queue; 
    uint32_t value_len; 
    uint8_t value_type; 
    u_char data[1]; 
} ngx_http_lua_shdict_list_node_t;
```

https://github.com/openresty/lua-nginx-module/pull/586/files

队列API

- lpush/rpush，表示在队列两端增加元素；
- lpop/rpop，表示在队列两端弹出元素；
- llen，表示返回队列的元素数量。

队列相关的测试，正是在 145-shdict-list.t 这个文件中

```nginx
=== TEST 1: lpush & lpop
--- http_config
    lua_shared_dict dogs 1m;
--- config
    location = /test {
        content_by_lua_block {
            local dogs = ngx.shared.dogs

            local len, err = dogs:lpush("foo", "bar")
            if len then
                ngx.say("push success")
            else
                ngx.say("push err: ", err)
            end

            local val, err = dogs:llen("foo")
            ngx.say(val, " ", err)

            local val, err = dogs:lpop("foo")
            ngx.say(val, " ", err)

            local val, err = dogs:llen("foo")
            ngx.say(val, " ", err)

            local val, err = dogs:lpop("foo")
            ngx.say(val, " ", err)
        }
    }
--- request
GET /test
--- response_body
push success
1 nil
bar nil
0 nil
nil nil
--- no_error_log
[error]
```

**管理类**

首先是 get_keys(max_count?)，它默认也只返回前 1024 个 key；如果你把 max_count 设置为 0，那就返回所有 key。

然后是 capacity 和 free_space，这两个 API 都属于 lua-resty-core 仓库，所以需要你 require 后才能使用：

```lua
require "resty.core.shdict"

 local cats = ngx.shared.cats
 local capacity_bytes = cats:capacity()
 local free_page_bytes = cats:free_space()
```

它们分别返回的，是共享内存的大小（也就是 lua_shared_dict 中配置的大小）和空闲页的字节数。因为 shared dict 是按照页来分配的，即使 free_space 返回为 0，在已经分配的页面中也可能存在空间，所以它的返回值并不能代表共享内存实际被占用的情况。

## OpenResty的核心和精髓：cosocket

**cosocket 是各种 lua-resty-* 非阻塞库的基础，没有 cosocket，开发者就无法用 Lua 来快速连接各种外部的网络服务。**

### 什么是cosocket？

cosocket 是 OpenResty 中的专有名词，是把协程和网络套接字的英文拼在一起形成的，即 cosocket = coroutine + socket。所以，你可以把 cosocket 翻译为“协程套接字”。

cosocket 不仅需要 Lua 协程特性的支持，也需要 Nginx 中非常重要的事件机制的支持，这两者结合在一起，最终实现了非阻塞网络 I/O。另外，cosocket 支持 TCP、UDP 和 Unix Domain Socket。

![img](https://static001.geekbang.org/resource/image/80/06/80d16e11d2750d6e4127445c126c9f06.png)

用户的 Lua 脚本每触发一个网络操作，都会有协程的 yield 以及 resume。遇到网络 I/O 时，它会交出控制权（yield），把网络事件注册到 Nginx 监听列表中，并把权限交给 Nginx；当有 Nginx 事件达到触发条件时，便唤醒对应的协程继续处理（resume）。

### cosocket API和指令简介

TCP 相关的 cosocket API 可以分为下面这几类。

- 创建对象：ngx.socket.tcp。
- 设置超时：tcpsock:settimeout 和 tcpsock:settimeouts。
- 建立连接：tcpsock:connect。
- 发送数据：tcpsock:send。
- 接受数据：tcpsock:receive、tcpsock:receiveany 和 tcpsock:receiveuntil。
- 连接池：tcpsock:setkeepalive。
- 关闭连接：tcpsock:close。

这些 API 可以使用的上下文：

```
rewrite_by_lua*, access_by_lua*, content_by_lua*, ngx.timer.*, ssl_certificate_by_lua*, ssl_session_fetch_by_lua*_
```

归咎于 Nginx 内核的各种限制，cosocket API 在 set_by_lua*， log_by_lua*， header_filter_by_lua* 和 body_filter_by_lua* 中是无法使用的。

而在 init_by_lua* 和 init_worker_by_lua* 中暂时也不能用，不过 Nginx 内核对这两个阶段并没有限制，后面可以增加对这它们的支持。

与这些 API 相关的，还有 8 个 lua_socket_ 开头的 Nginx 指令，我们简单来看一下。

- lua_socket_connect_timeout：连接超时，默认 60 秒。
- lua_socket_send_timeout：发送超时，默认 60 秒。
- lua_socket_send_lowat：发送阈值（low water），默认为 0。
- lua_socket_read_timeout： 读取超时，默认 60 秒。
- lua_socket_buffer_size：读取数据的缓存区大小，默认 4k/8k。
- lua_socket_pool_size：连接池大小，默认 30。
- lua_socket_keepalive_timeout：连接池 cosocket 对象的空闲时间，默认 60 秒。
- lua_socket_log_errors：cosocket 发生错误时，是否记录日志，默认为 on。

这里你也可以看到，有些指令和 API 的功能一样的，比如设置超时时间和连接池大小等。不过，如果两者有冲突的话，API 的优先级高于指令，会覆盖指令设置的值。所以，一般来说，我们都推荐使用 API 来做设置，这样也会更加灵活。

**发送 TCP 请求到一个网站，并把返回的内容打印出来**

```lua
$ resty -e 'local sock = ngx.socket.tcp()
        sock:settimeout(1000)  -- one second timeout
        local ok, err = sock:connect("www.baidu.com", 80)
        if not ok then
            ngx.say("failed to connect: ", err)
            return
        end

        local req_data = "GET / HTTP/1.1\r\nHost: www.baidu.com\r\n\r\n"
        local bytes, err = sock:send(req_data)
        if err then
            ngx.say("failed to send: ", err)
            return
        end

        local data, err, partial = sock:receive()
        if err then
            ngx.say("failed to receive: ", err)
            return
        end

        sock:close()
        ngx.say("response is: ", data)'
```

分析代码

- 首先，通过 ngx.socket.tcp() ，创建 TCP 的 cosocket 对象，名字是 sock。
- 然后，使用 settimeout() ，把超时时间设置为 1 秒。注意这里的超时没有区分 connect、receive，是统一的设置。
- 接着，使用 connect() 去连接指定网站的 80 端口，如果失败就直接退出。
- 连接成功的话，就使用 send() 来发送构造好的数据，如果发送失败就退出。
- 发送数据成功的话，就使用 receive() 来接收网站返回的数据。这里 receive() 的默认参数值是 *l，也就是只返回第一行的数据；如果参数设置为了*a，就是持续接收数据，直到连接关闭；
- 最后，调用 close() ，主动关闭 socket 连接。

**第一个动作，对 socket 连接、发送和读取这三个动作，分别设置超时时间**

我们刚刚用的settimeout() ，作用是把超时时间统一设置为一个值。如果要想分开设置，就需要使用 settimeouts() 函数，比如下面这样的写法：

```lua
sock:settimeouts(1000, 2000, 3000) 
```

- 表示连接超时为 1 秒，发送超时为 2 秒，读取超时为 3 秒。在 OpenResty 和 lua-resty 库中，大部分和时间相关的 API 的参数，都以毫秒为单位

**第二个动作，receive接收指定大小的内容**

刚刚说了，receive() 接口可以接收一行数据，也可以持续接收数据。不过，如果你只想接收 10K 大小的数据，应该怎么设置呢？这时，receiveany() 闪亮登场。它就是专为满足这种需求而设计的，一起来看下面这行代码：

```lua
local data, err, partial = sock:receiveany(10240)
```

关于 receive，还有另一个很常见的用户需求，那就是一直获取数据，直到遇到指定字符串才停止。

receiveuntil() 专门用来解决这类问题，它不会像 receive() 和 receiveany() 一样返回字符串，而会返回一个迭代器。这样，你就可以在循环中调用它来分段读取匹配到的数据，当读取完毕时，就会返回 nil。

```lua
 local reader = sock:receiveuntil("\r\n")

 while true do
     local data, err, partial = reader(4)
     if not data then
         if err then
             ngx.say("failed to read the data stream: ", err)
             break
         end

         ngx.say("read done")
         break
     end
     ngx.say("read chunk: [", data, "]")
 end
```

这段代码中的 receiveuntil 会返回 \r\n 之前的数据，并通过迭代器每次读取其中的 4 个字节，也就实现了我们想要的功能。

**第三个动作，不直接关闭socket，而是放入连接池中**

在你使用完一个 cosocket 后，可以调用 setkeepalive() 放到连接池中，比如下面这样的写法：

```lua
local ok, err = sock:setkeepalive(2 * 1000, 100)
if not ok then
    ngx.say("failed to set reusable: ", err)
end
```

这段代码设置了连接的空闲时间为 2 秒，连接池的大小为 100。这样，在调用 connect() 函数时，就会优先从连接池中获取 cosocket 对象。

连接池的使用

- 第一，不能把发生错误的连接放入连接池，否则下次使用时，就会导致收发数据失败。这也是为什么我们需要判断每一个 API 调用是否成功的一个原因。
- 第二，要搞清楚连接的数量。连接池是 worker 级别的，每个 worker 都有自己的连接池。所以，如果你有 10 个 worker，连接池大小设置为 30，那么对于后端的服务来讲，就等于有 300 个连接。

## 超越Web服务器：特权进程和定时任务

### 定时任务

ngx.timer ，看作是 OpenResty 模拟的客户端请求，用以触发对应的回调函数。

OpenResty的定时任务可以分为

- ngx.timer.at，用来执行一次性的定时任务；
- ngx.time.every，用来执行固定周期的定时任务

如何突破 init_worker_by_lua 中不能使用 cosocket 的限制，这个答案其实就是 ngx.timer

启动了一个延时为 0 的定时任务。它启动了回调函数 handler，并在这个函数中，用 cosocket 去访问一个网站：

```lua
init_worker_by_lua_block {
        local function handler()
            local sock = ngx.socket.tcp()
            local ok, err = sock:connect(“www.baidu.com", 80)
        end

        local ok, err = ngx.timer.at(0, handler)
    }
```

**这样，我们就绕过了 cosocket 在这个阶段不能使用的限制。**

ngx.time.every API更加接近 crontab 的解决方案

但美中不足的是，在启动了一个 timer 之后，你就再也没有机会来取消这个定时任务了，毕竟ngx.timer.cancel 还是一个 todo 的功能。

**面临一个问题：定时任务是在后台运行的，并且无法取消；如果定时任务的数量很多，就很容易耗尽系统资源。**

OpenResty 提供了 lua_max_pending_timers 和 lua_max_running_timers 这两个指令，来对其进行限制。前者代表等待执行的定时任务的最大值，后者代表当前正在运行的定时任务的最大值。

### 特权进程

Nginx 主要分为 master 进程和 worker 进程，其中，真正处理用户请求的是 worker 进程。我们可以通过 lua-resty-core 中提供的 process.type API ，获取到进程的类型。

```lua
$ resty -e 'local process = require "ngx.process"
ngx.say("process type:", process.type())'
```

你会看到，它返回的结果不是 worker， 而是 single。这意味 resty 启动的 Nginx 只有 worker 进程，没有 master 进程。

OpenResty 在 Nginx 的基础上进行了扩展，增加了特权进程：privileged agent

- 它不监听任何端口，这就意味着不会对外提供任何服务；
- 它拥有和 master 进程一样的权限，一般来说是 root 用户的权限，这就让它可以做很多 worker 进程不可能完成的任务；
- 特权进程只能在 init_by_lua 上下文中开启；
- 另外，特权进程只有运行在 init_worker_by_lua 上下文中才有意义，因为没有请求触发，也就不会走到content、access 等上下文去。

开启特权进程的示例

```lua
init_by_lua_block {
    local process = require "ngx.process"

    local ok, err = process.enable_privileged_agent()
    if not ok then
        ngx.log(ngx.ERR, "enables privileged agent failed error:", err)
    end
}
```

使用ngx.timer ，来周期性地触发

```lua
init_worker_by_lua_block {
    local process = require "ngx.process"

    local function reload(premature)
        local f, err = io.open(ngx.config.prefix() .. "/logs/nginx.pid", "r")
        if not f then
            return
        end
        local pid = f:read()
        f:close()
        os.execute("kill -HUP " .. pid)
    end

    if process.type() == "privileged agent" then
         local ok, err = ngx.timer.every(5, reload)
        if not ok then
            ngx.log(ngx.ERR, err)
        end
    end
}
```

上面这段代码，实现了每 5 秒给 master 进程发送 HUP 信号量的功能。自然，你也可以在此基础上实现更多有趣的功能，比如轮询数据库，看是否有特权进程的任务并执行。因为特权进程是 root 权限，这显然就有点儿“后门”程序的意味了。

### 非阻塞的ngx.pipe

lua-resty-shell 库应运而生，使用它来调用命令行就是非阻塞的：

```lua
$ resty -e 'local shell = require "resty.shell"
local ok, stdout, stderr, reason, status =
    shell.run([[echo "hello, world"]])
    ngx.say(stdout)
```

这段代码可以算是 hello world 的另外一种写法了，它调用系统的 echo 命令来完成输出。类似的，你可以用 resty.shell ，来替代 Lua 中的 os.execute 调用。

lua-resty-shell 的底层实现，依赖了 lua-resty-core 中的 [ngx.pipe https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/pipe.md] API，所以，这个使用 lua-resty-shell 打印出 hello wrold 的示例，改用 ngx.pipe ，可以写成下面这样：

```lua
$ resty -e 'local ngx_pipe = require "ngx.pipe"
local proc = ngx_pipe.spawn({"echo", "hello world"})
local data, err = proc:stdout_read_line()
ngx.say(data)'
```

## 时间、正则表达式API

在 OpenResty 中，我们应该一直使用 ngx.re.* 提供的一系列 API，来处理和正则表达式相关的逻辑，而不是用 Lua 自带的模式匹配

### ngx.re.split

ngx.re.split 这个 API 并不在 lua-nginx-module 中，而是在 lua-resty-core 里面；并且它也不在 lua-resty-core 首页的文档中，而是在 lua-resty-core/lib/ngx/re.md 这个第三级目录的文档中出现的

怎么来最快地解决这种问题呢？

- 除了阅读 lua-resty-core 首页文档外，你还需要把 lua-resty-core/lib/ngx/ 这个目录下的 .md 格式的文档也通读一遍才行。

### lua_regex_match_limit

如果我使用的正则引擎是基于回溯的 NFA 来实现的，那么就有可能出现灾难性回溯（Catastrophic Backtracking），即正则在匹配的时候回溯过多，造成 CPU 100%，正常服务被阻塞。

如何在 OpenResty 中简单有效地规避，也就是使用下面这行代码：

```lua
lua_regex_match_limit 100000;
```

lua_regex_match_limit ，就是用来限制 PCRE 正则引擎的回溯次数的。这样，即使出现了灾难性回溯，后果也会被限制在一个范围内，不会导致你的 CPU 满载。

### 时间API

返回当前时间的 API，如果没有非阻塞网络 IO 操作来触发，便会一直返回缓存的值，而不是像我们想的那样，能够返回当前的实时时间

举个例子，比如你有一段正在做密集运算的代码，需要花费比较多的时间，那么在这段时间内，这段代码对应的请求就会一直占用着 worker 和 CPU 资源，导致其他请求需要排队，无法得到及时的响应。这时，我们就可以在其中穿插 ngx.sleep(0)，使这段代码让出控制权，让其他请求也可以得到处理。

### worker和进程API

OpenResty 提供了 ngx.worker.* 和 ngx.process.* 这些 API， 来获取 worker 和进程相关的信息。其中，前者和 Nginx worker 进程有关，后者则是泛指所有的 Nginx 进程，不仅有 worker 进程，还有 master 进程和特权进程等等。

ngx.worker.* 由 lua-nginx-module 提供，而ngx.process.* 则是由 lua-resty-core 提供。还记得上节课我们留的作业题吗，如何保证在多 worker 的情况下，只启动一个 timer？其实，这就需要用到 ngx.worker.id 这个 API 了。你可以在启动 timer 之前，先做一个简单的判断：

```lua
if ngx.worker.id == 0 then
    start_timer()
end
```

这样，我们就能实现只启动一个 timer 的目的了。这里注意，worker id 是从 0 开始返回的，这和 Lua 中数组下标从 1 开始并不相同，千万不要混淆了。

### 真值和空值

Lua 中真值的定义：**除了 nil 和 false 之外，都是真值**

真值也就包括了：0、空字符串、空表等等。

**cdata:NULL**

当你通过 LuaJIT FFI 接口去调用 C 函数，而这个函数返回一个 NULL 指针，那么你就会遇到另外一种空值，即cdata:NULL

```lua
$ resty -e 'local ffi = require "ffi"
local cdata_null = ffi.new("void*", nil)
if cdata_null then
    ngx.say("true")
end'
```

和 ngx.null 一样，cdata:NULL 也是真值。但更让人匪夷所思的是，下面这段代码，会打印出 true，也就是说cdata:NULL 是和 nil 相等的：

```lua
$ resty -e 'local ffi = require "ffi"
local cdata_null = ffi.new("void*", nil)
ngx.say(cdata_null == nil)'
```

**cjson.null**

cjson 库会把 json 中的 NULL，解码为 Lua 的 lightuserdata，并用 cjson.null 来表示：

```lua
$ resty -e 'local cjson = require "cjson"
local data = cjson.encode(nil)
local decode_null = cjson.decode(data)
ngx.say(decode_null == cjson.null)'
```

Lua 中的 nil，被 json encode 和 decode 一圈儿之后，就变成了 cjson.null。你可以想得到，它引入的原因和 ngx.null 是一样的，因为 nil 无法在 table 中作为 value。

## lua-resty-requests

https://github.com/tokers/lua-resty-requests

### 处理四层流量，实现Memcached Server

### 原始需求和技术方案

为什么要实现一个 memcached server 呢？直接安装一个原版的 memcached 或者 redis 不就行了吗？

- HTTPS 流量逐渐成为主流，但一些比较老的浏览器并不支持 session ticket，那么我们就需要在服务端把 session ID 存下来。如果本地存储空间不够，就需要一个集群进行存放，而这个数据又是可以丢弃的，所以选用 memcached 就比较合适。

这时候，直接引入 memcached ，应该是最简单直接的方案。但出于以下几个方面的考虑，我还是选择使用 OpenResty 来造一个轮子：

- 第一，直接引入会多引入一个进程，增加部署和维护成本；
- 第二，这个需求足够简单，只需要 get 和 set 操作，并且支持过期即可；
- 第三，OpenResty 有 stream 模块，可以很快地实现这个需求。

memcached 的协议可以支持 TCP 和 UDP，这里我选择 TCP，下面是 get 和 set 命令的具体协议：

```bash
Get
根据 key 获取 value
Telnet command: get <key>*\r\n

示例：
get key
VALUE key 0 4 data END
```

```bash
Set
存储键值对到 memcached 中
Telnet command：set <key> <flags> <exptime> <bytes> [noreply]\r\n<value>\r\n

示例：
set key 0 900 4 data
STORED
```

除了 get 和 set 外，我们还需要知道 memcached 的协议的“错误处理”是怎么样做的。“错误处理”对于服务端的程序是非常重要的，我们在编写程序时，除了要处理正常的请求，也要考虑到各种异常。比如下面这样的场景：

- memcached 发送了一个 get、set 之外的请求，我要怎么处理呢？
- 服务端出错，我要给 memcached 的客户端一个什么样的反馈呢？

下面这张图出自 memcached 的文档，描述了出错的时候，应该返回什么内容和具体的格式，你可以用做参考：

![img](https://static001.geekbang.org/resource/image/37/b0/3767ed0047e34aabaa7bf7d568438ab0.png)



**OpenResty 的 shared dict 可以跨各个 worker 来使用，把数据放在 shared dict 里面，和放在 memcached 里面非常类似——它们都支持 get 和 set 操作，并且在进程重启后数据就丢失了。**

### 测试驱动开发

基于测试驱动开发的思想，在写具体的代码之前，让我们先来构造一个最简单的测试案例。

```lua
$ resty -e 'local memcached = require "resty.memcached"
    local memc, err = memcached:new()

    memc:set_timeout(1000) -- 1 sec
    local ok, err = memc:connect("127.0.0.1", 11212)
    local ok, err = memc:set("dog", 32)
    if not ok then
        ngx.say("failed to set dog: ", err)
        return
    end

    local res, flags, err = memc:get("dog")
    ngx.say("dog: ", res)'
```

**使用 stream 模块来接收和发送数据，同时使用 shared dict 来存储数据。**

### 搭建框架

是先搭建一个最小的可以运行的代码框架，然后再逐步地去填充代码。这样的好处是，在编码过程中，你可以给自己设置很多小目标；而且在完成一个小目标后，测试案例也会给你正反馈。

先来设置好 Nginx 的配置文件，因为 stream 和 shared dict 要在其中预设。下面是我设置的配置文件：

```nginx
stream {
    lua_shared_dict memcached 100m;
    lua_package_path 'lib/?.lua;;';
    server {
        listen 11212;
        content_by_lua_block {
            local m = require("resty.memcached.server")
            m.run()
        }
    }
}
```

- 代码运行在 Nginx 的 stream 上下文中，而非 HTTP 上下文中，并且监听了 11212 端口；
- shared dict 的名字为 memcached，大小是 100M，这些在运行期是不可以修改的；
- 代码所在目录为 lib/resty/memcached, 文件名为 server.lua, 入口函数为 run()，这些信息你都可以从lua_package_path 和 content_by_lua_block 中找到。

```lua
local new_tab = require "table.new"
local str_sub = string.sub
local re_find = ngx.re.find
local mc_shdict = ngx.shared.memcached

local _M = { _VERSION = '0.01' }

local function parse_args(s, start)
end

function _M.get(tcpsock, keys)
end

function _M.set(tcpsock, res)
end

function _M.run()
    local tcpsock = assert(ngx.req.socket(true))

    while true do
        tcpsock:settimeout(60000) -- 60 seconds
        local data, err = tcpsock:receive("*l")

        local command, args
        if data then
            local from, to, err = re_find(data, [[(\S+)]], "jo")
            if from then
                command = str_sub(data, from, to)
                args = parse_args(data, to + 1)
            end
        end

        if args then
            local args_len = #args
            if command == 'get' and args_len > 0 then
                _M.get(tcpsock, args)
            elseif command == "set" and args_len == 4 then
                _M.set(tcpsock, args)
            end
        end
    end
end

return _M
```

这段代码，便实现了入口函数 run() 的主要逻辑。虽然我还没有做异常处理，依赖的 parse_args、get 和 set 也都是空函数，但这个框架已经完整表达了 memcached server 的逻辑

### 填充代码

memcached协议文档：[memcached/protocol.txt at master · memcached/memcached (github.com)](https://github.com/memcached/memcached/blob/master/doc/protocol.txt)

```lua
local function parse_args(s, start)
    local arr = {}

    while true do
        local from, to = re_find(s, [[\S+]], "jo", {pos = start})
        if not from then
            break
        end

        table.insert(arr, str_sub(s, from, to))

        start = to + 1
    end

    return arr
end
```

实现下 get 函数。它可以一次查询多个键，所以下面代码中我用了一个 for 循环：

```lua
function _M.get(tcpsock, keys)
    local reply = ""

    for i = 1, #keys do
        local key = keys[i]
        local value, flags = mc_shdict:get(key)
        if value then
            local flags  = flags or 0
            reply = reply .. "VALUE" .. key .. " " .. flags .. " " .. #value .. "\r\n" .. value .. "\r\n"
        end
    end
    reply = reply ..  "END\r\n"

    tcpsock:settimeout(1000)  -- one second timeout
    local bytes, err = tcpsock:send(reply)
end
```

这里最核心的代码只有一行：``local value, flags = mc_shdict:get(key)``，也就是从 shared dict 中查询到数据；至于其余的代码，都在按照 memcached 的协议拼接字符串，并最终 send 到客户端。

set 函数。它将接收到的参数转换为 shared dict API 的格式，把数据储存了起来；并在出错的时候，按照 memcached 的协议做出处理：

```lua
function _M.set(tcpsock, res)
    local reply =  ""

    local key = res[1]
    local flags = res[2]
    local exptime = res[3]
    local bytes = res[4]

    local value, err = tcpsock:receive(tonumber(bytes) + 2)

    if str_sub(value, -2, -1) == "\r\n" then
        local succ, err, forcible = mc_shdict:set(key, str_sub(value, 1, bytes), exptime, flags)
        if succ then
            reply = reply .. “STORED\r\n"
        else
            reply = reply .. "SERVER_ERROR " .. err .. “\r\n”
        end
    else
        reply = reply .. "ERROR\r\n"
    end

    tcpsock:settimeout(1000)  -- one second timeout
    local bytes, err = tcpsock:send(reply)
end
```

OpenResty 中并没有断点调试的工具，所以我们都是使用 ngx.say 和 ngx.log 来调试的

## test::nginx





## 练习

1. 返回任意的东西，暴露HTTP接口，实现RESTful接口

2. 通过OpenResty操作数据库，增删改查

3. 把第二步打包成容器跑起来

### 安装MySQL

创建文件``mysql-rc.yaml``

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: mysql-rc
  labels:
    name: mysql-rc
spec:
  replicas: 1
  selector:
    name: mysql-pod
  template:
    metadata:
      labels:
        name: mysql-pod
    spec:
      containers:
      - name: mysql
        image: mysql
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
```

创建文件``mysql-svc.yaml``

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-svc
  labels:
    name: mysql-svc
spec:
  type: NodePort
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
    name: http
    nodePort: 32306
  selector:
    name: mysql-pod
```

启动pod和service

```bash
kubectl create -f mysql-rc.yaml
kubectl create -f mysql-svc.yaml
```

查看登录的IP端口

```bash
kubectl get pod -o wide
```

### lua 中链接云服务器报错

 报错 failed to connect: no resolver defined to resolve

```nginx
resolver 8.8.8.8; 
```

### 服务器重启之后，报错

**报错**

The connection to the server 124.223.83.116:6443 was refused - did you specify the right host or port?

重启docker ``systemctl start docker.service``



### 安装gitlab runner

https://docs.gitlab.com/runner/install/linux-repository.html

Ubuntu安装

```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
```

```bash
sudo apt-get install gitlab-runner
```

注册runner

```bash
gitlab-runner register

# 然后提示输入服务器的地址
https://gitlab.intsig.net
# 输入token，打开Setting，CI/CD，有个runner，里面复制token
1gSN--14zYzSfHca2RYP
# 输入描述
openresty_test
# tag
openresty
# note
shell,docker

```

gitlab-ci.yml默认配置

```yaml
stages:
- build
- test
- deploy
build:
    stage: build
    tags:
    - openresty
    script:
    - echo "代码编译..."
test:
    stage: test
    tags:
    - openresty
    script:
    - echo "测试代码"...
deploy:
    stage: deploy
    tags:
    - openresty
    script:
    - echo "部署项目..."
    - scp conf /usr/local/openresty/nginx/conf
    - scp lua /usr/local/openresty/nginx/lua
```

push

```yaml
stages:
  - push

push:
  image: docker
  services:
    - docker:dind
  stage: push
  before_script:
    - docker login -u $CI_HARBOR_USER -p $CI_HARBOR_PASSWORD registry.intsig.net/jun_yuan
  script:
    - docker build -f $CI_PROJECT_DIR/Dockerfile -t stress:v1 $CI_PROJECT_DIR
    - docker tag stress:v1 $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
```

使用docker安装

```bash
docker run -d --name gitlab-runner --restart always \
      -v /srv/gitlab-runner/config:/etc/gitlab-runner \
      -v /var/run/docker.sock:/var/run/docker.sock \
      gitlab/gitlab-runner:latest
```

register

```bash
docker run --rm -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register \
  --non-interactive \
  --executor "docker" \
  --docker-image ubuntu:latest \
  --url "https://gitlab.intsig.net/" \
  --registration-token "1gSN--14zYzSfHca2RYP" \
  --description "first-register-runner" \
  --tag-list "openresty" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

修改``/etc/gitlab-runner.toml``中的配置

```yaml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "first-register-runner"
  url = "https://gitlab.intsig.net/"
  token = "xR-hx3sx9sW2JotGtyTr"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = true  # 此处修改为true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
```

报错Cannot connect to the Docker daemon at tcp://localhost:2375. Is the docker daemon running?

```bash
1. 使用sudo + 命令 (每次多写sudo，还要输密码，密码错了还要重来 :P )
sudo docker ps

2.(一次性的，不设置DOCKER_HOST)
unset DOCKER_HOST 

3. 修改 ~/.bashrc文件

vi ~/.bashrc
# 在最下面添加一行：
export DOCKER_HOST='unix:///var/run/docker.sock'
source .bashrc
```

```bash
sudo gitlab-runner register -n \
  --url https://gitlab.intsig.net/ \
  --registration-token DD9f3deZ_Ezm9CSDBWqr \
  --executor docker \
  --description "TransferRunner" \
  --docker-image "docker:19.03.12" \
  --docker-volumes /var/run/docker.sock:/var/run/docker.sock
```

### 安装busted

Centos安装（没用）

```bash
yum install luarocks
luarocks install busted
```

使用docker启动

```bash
docker run -it harbor.intsig.net/dongfei_gao/http_test bash
```

起一个容器

```bash
docker run -it --name busted \
-v /opt/docker/openresty/nginx/conf:/home \
da8e1ba2b0f1 bash
```



### Ubuntu安装Vscode

```bash
wget https://vscode.cdn.azure.cn/stable/f80445acd5a3dadef24aa209168452a3d97cc326/code_1.64.2-1644445741_amd64.deb

sudo apt-get  install dpkg
sudo dpkg -i code_1.64.2-1644445741_amd64.deb 
code
```

### OpenResty安装lua-resty-http

```bash
cd /usr/example/lualib/resty/  
wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http_headers.lua  
wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http.lua
```

如果安装了luarocks，执行命令``luarocks install lua-resty-http``

直接编辑文件

``http.lua``

```lua
local http_headers = require "resty.http_headers"

local ngx = ngx
local ngx_socket_tcp = ngx.socket.tcp
local ngx_req = ngx.req
local ngx_req_socket = ngx_req.socket
local ngx_req_get_headers = ngx_req.get_headers
local ngx_req_get_method = ngx_req.get_method
local str_lower = string.lower
local str_upper = string.upper
local str_find = string.find
local str_sub = string.sub
local tbl_concat = table.concat
local tbl_insert = table.insert
local ngx_encode_args = ngx.encode_args
local ngx_re_match = ngx.re.match
local ngx_re_gmatch = ngx.re.gmatch
local ngx_re_sub = ngx.re.sub
local ngx_re_gsub = ngx.re.gsub
local ngx_re_find = ngx.re.find
local ngx_log = ngx.log
local ngx_DEBUG = ngx.DEBUG
local ngx_ERR = ngx.ERR
local ngx_var = ngx.var
local ngx_print = ngx.print
local ngx_header = ngx.header
local co_yield = coroutine.yield
local co_create = coroutine.create
local co_status = coroutine.status
local co_resume = coroutine.resume
local setmetatable = setmetatable
local tonumber = tonumber
local tostring = tostring
local unpack = unpack
local rawget = rawget
local select = select
local ipairs = ipairs
local pairs = pairs
local pcall = pcall
local type = type


-- http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.5.1
local HOP_BY_HOP_HEADERS = {
    ["connection"]          = true,
    ["keep-alive"]          = true,
    ["proxy-authenticate"]  = true,
    ["proxy-authorization"] = true,
    ["te"]                  = true,
    ["trailers"]            = true,
    ["transfer-encoding"]   = true,
    ["upgrade"]             = true,
    ["content-length"]      = true, -- Not strictly hop-by-hop, but Nginx will deal
                                    -- with this (may send chunked for example).
}


local EXPECTING_BODY = {
    POST  = true,
    PUT   = true,
    PATCH = true,
}


-- Reimplemented coroutine.wrap, returning "nil, err" if the coroutine cannot
-- be resumed. This protects user code from infinite loops when doing things like
-- repeat
--   local chunk, err = res.body_reader()
--   if chunk then -- <-- This could be a string msg in the core wrap function.
--     ...
--   end
-- until not chunk
local co_wrap = function(func)
    local co = co_create(func)
    if not co then
        return nil, "could not create coroutine"
    else
        return function(...)
            if co_status(co) == "suspended" then
                return select(2, co_resume(co, ...))
            else
                return nil, "can't resume a " .. co_status(co) .. " coroutine"
            end
        end
    end
end


-- Returns a new table, recursively copied from the one given.
--
-- @param   table   table to be copied
-- @return  table
local function tbl_copy(orig)
    local orig_type = type(orig)
    local copy
    if orig_type == "table" then
        copy = {}
        for orig_key, orig_value in next, orig, nil do
            copy[tbl_copy(orig_key)] = tbl_copy(orig_value)
        end
    else -- number, string, boolean, etc
        copy = orig
    end
    return copy
end


local _M = {
    _VERSION = '0.17.0-beta.1',
}
_M._USER_AGENT = "lua-resty-http/" .. _M._VERSION .. " (Lua) ngx_lua/" .. ngx.config.ngx_lua_version

local mt = { __index = _M }


local HTTP = {
    [1.0] = " HTTP/1.0\r\n",
    [1.1] = " HTTP/1.1\r\n",
}


local DEFAULT_PARAMS = {
    method = "GET",
    path = "/",
    version = 1.1,
}


local DEBUG = false


function _M.new(_)
    local sock, err = ngx_socket_tcp()
    if not sock then
        return nil, err
    end
    return setmetatable({ sock = sock, keepalive = true }, mt)
end


function _M.debug(d)
    DEBUG = (d == true)
end


function _M.set_timeout(self, timeout)
    local sock = self.sock
    if not sock then
        return nil, "not initialized"
    end

    return sock:settimeout(timeout)
end


function _M.set_timeouts(self, connect_timeout, send_timeout, read_timeout)
    local sock = self.sock
    if not sock then
        return nil, "not initialized"
    end

    return sock:settimeouts(connect_timeout, send_timeout, read_timeout)
end

do
    local aio_connect = require "resty.http_connect"
    -- Function signatures to support:
    -- ok, err, ssl_session = httpc:connect(options_table)
    -- ok, err = httpc:connect(host, port, options_table?)
    -- ok, err = httpc:connect("unix:/path/to/unix.sock", options_table?)
    function _M.connect(self, options, ...)
        if type(options) == "table" then
            -- all-in-one interface
            return aio_connect(self, options)
        else
            -- backward compatible
            return self:tcp_only_connect(options, ...)
        end
    end
end

function _M.tcp_only_connect(self, ...)
    ngx_log(ngx_DEBUG, "Use of deprecated `connect` method signature")

    local sock = self.sock
    if not sock then
        return nil, "not initialized"
    end

    self.host = select(1, ...)
    self.port = select(2, ...)

    -- If port is not a number, this is likely a unix domain socket connection.
    if type(self.port) ~= "number" then
        self.port = nil
    end

    self.keepalive = true
    self.ssl = false

    return sock:connect(...)
end


function _M.set_keepalive(self, ...)
    local sock = self.sock
    if not sock then
        return nil, "not initialized"
    end

    if self.keepalive == true then
        return sock:setkeepalive(...)
    else
        -- The server said we must close the connection, so we cannot setkeepalive.
        -- If close() succeeds we return 2 instead of 1, to differentiate between
        -- a normal setkeepalive() failure and an intentional close().
        local res, err = sock:close()
        if res then
            return 2, "connection must be closed"
        else
            return res, err
        end
    end
end


function _M.get_reused_times(self)
    local sock = self.sock
    if not sock then
        return nil, "not initialized"
    end

    return sock:getreusedtimes()
end


function _M.close(self)
    local sock = self.sock
    if not sock then
        return nil, "not initialized"
    end

    return sock:close()
end


local function _should_receive_body(method, code)
    if method == "HEAD" then return nil end
    if code == 204 or code == 304 then return nil end
    if code >= 100 and code < 200 then return nil end
    return true
end


function _M.parse_uri(_, uri, query_in_path)
    if query_in_path == nil then query_in_path = true end

    local m, err = ngx_re_match(
        uri,
        [[^(?:(http[s]?):)?//((?:[^\[\]:/\?]+)|(?:\[.+\]))(?::(\d+))?([^\?]*)\??(.*)]],
        "jo"
    )

    if not m then
        if err then
            return nil, "failed to match the uri: " .. uri .. ", " .. err
        end

        return nil, "bad uri: " .. uri
    else
        -- If the URI is schemaless (i.e. //example.com) try to use our current
        -- request scheme.
        if not m[1] then
            -- Schema-less URIs can occur in client side code, implying "inherit
            -- the schema from the current request". We support it for a fairly
            -- specific case; if for example you are using the ESI parser in
            -- ledge (https://github.com/ledgetech/ledge) to perform in-flight
            -- sub requests on the edge based on instructions found in markup,
            -- those URIs may also be schemaless with the intention that the
            -- subrequest would inherit the schema just like JavaScript would.
            local scheme = ngx_var.scheme
            if scheme == "http" or scheme == "https" then
                m[1] = scheme
            else
                return nil, "schemaless URIs require a request context: " .. uri
            end
        end

        if m[3] then
            m[3] = tonumber(m[3])
        else
            if m[1] == "https" then
                m[3] = 443
            else
                m[3] = 80
            end
        end
        if not m[4] or "" == m[4] then m[4] = "/" end

        if query_in_path and m[5] and m[5] ~= "" then
            m[4] = m[4] .. "?" .. m[5]
            m[5] = nil
        end

        return m, nil
    end
end


local function _format_request(self, params)
    local version = params.version
    local headers = params.headers or {}

    local query = params.query or ""
    if type(query) == "table" then
        query = "?" .. ngx_encode_args(query)
    elseif query ~= "" and str_sub(query, 1, 1) ~= "?" then
        query = "?" .. query
    end

    -- Initialize request
    local req = {
        str_upper(params.method),
        " ",
        self.path_prefix or "",
        params.path,
        query,
        HTTP[version],
        -- Pre-allocate slots for minimum headers and carriage return.
        true,
        true,
        true,
    }
    local c = 7 -- req table index it's faster to do this inline vs table.insert

    -- Append headers
    for key, values in pairs(headers) do
        key = tostring(key)

        if type(values) == "table" then
            for _, value in pairs(values) do
                req[c] = key .. ": " .. tostring(value) .. "\r\n"
                c = c + 1
            end

        else
            req[c] = key .. ": " .. tostring(values) .. "\r\n"
            c = c + 1
        end
    end

    -- Close headers
    req[c] = "\r\n"

    return tbl_concat(req)
end


local function _receive_status(sock)
    local line, err = sock:receive("*l")
    if not line then
        return nil, nil, nil, err
    end

    local version = tonumber(str_sub(line, 6, 8))
    if not version then
        return nil, nil, nil,
               "couldn't parse HTTP version from response status line: " .. line
    end

    local status = tonumber(str_sub(line, 10, 12))
    if not status then
        return nil, nil, nil,
               "couldn't parse status code from response status line: " .. line
    end

    local reason = str_sub(line, 14)

    return status, version, reason
end


local function _receive_headers(sock)
    local headers = http_headers.new()

    repeat
        local line, err = sock:receive("*l")
        if not line then
            return nil, err
        end

        local m, err = ngx_re_match(line, "([^:\\s]+):\\s*(.*)", "jo")
        if err then ngx_log(ngx_ERR, err) end

        if not m then
            break
        end

        local key = m[1]
        local val = m[2]
        if headers[key] then
            if type(headers[key]) ~= "table" then
                headers[key] = { headers[key] }
            end
            tbl_insert(headers[key], tostring(val))
        else
            headers[key] = tostring(val)
        end
    until ngx_re_find(line, "^\\s*$", "jo")

    return headers, nil
end


local function transfer_encoding_is_chunked(headers)
    local te = headers["Transfer-Encoding"]
    if not te then
        return false
    end

    -- Handle duplicate headers
    -- This shouldn't happen but can in the real world
    if type(te) ~= "string" then
        te = tbl_concat(te, ",")
    end

    return str_find(str_lower(te), "chunked", 1, true) ~= nil
end
_M.transfer_encoding_is_chunked = transfer_encoding_is_chunked


local function _chunked_body_reader(sock, default_chunk_size)
    return co_wrap(function(max_chunk_size)
        local remaining = 0
        local length
        max_chunk_size = max_chunk_size or default_chunk_size

        repeat
            -- If we still have data on this chunk
            if max_chunk_size and remaining > 0 then

                if remaining > max_chunk_size then
                    -- Consume up to max_chunk_size
                    length = max_chunk_size
                    remaining = remaining - max_chunk_size
                else
                    -- Consume all remaining
                    length = remaining
                    remaining = 0
                end
            else -- This is a fresh chunk

                -- Receive the chunk size
                local str, err = sock:receive("*l")
                if not str then
                    co_yield(nil, err)
                end

                length = tonumber(str, 16)

                if not length then
                    co_yield(nil, "unable to read chunksize")
                end

                if max_chunk_size and length > max_chunk_size then
                    -- Consume up to max_chunk_size
                    remaining = length - max_chunk_size
                    length = max_chunk_size
                end
            end

            if length > 0 then
                local str, err = sock:receive(length)
                if not str then
                    co_yield(nil, err)
                end

                max_chunk_size = co_yield(str) or default_chunk_size

                -- If we're finished with this chunk, read the carriage return.
                if remaining == 0 then
                    sock:receive(2) -- read \r\n
                end
            else
                -- Read the last (zero length) chunk's carriage return
                sock:receive(2) -- read \r\n
            end

        until length == 0
    end)
end


local function _body_reader(sock, content_length, default_chunk_size)
    return co_wrap(function(max_chunk_size)
        max_chunk_size = max_chunk_size or default_chunk_size

        if not content_length and max_chunk_size then
            -- We have no length, but wish to stream.
            -- HTTP 1.0 with no length will close connection, so read chunks to the end.
            repeat
                local str, err, partial = sock:receive(max_chunk_size)
                if not str and err == "closed" then
                    co_yield(partial, err)
                end

                max_chunk_size = tonumber(co_yield(str) or default_chunk_size)
                if max_chunk_size and max_chunk_size < 0 then max_chunk_size = nil end

                if not max_chunk_size then
                    ngx_log(ngx_ERR, "Buffer size not specified, bailing")
                    break
                end
            until not str

        elseif not content_length then
            -- We have no length but don't wish to stream.
            -- HTTP 1.0 with no length will close connection, so read to the end.
            co_yield(sock:receive("*a"))

        elseif not max_chunk_size then
            -- We have a length and potentially keep-alive, but want everything.
            co_yield(sock:receive(content_length))

        else
            -- We have a length and potentially a keep-alive, and wish to stream
            -- the response.
            local received = 0
            repeat
                local length = max_chunk_size
                if received + length > content_length then
                    length = content_length - received
                end

                if length > 0 then
                    local str, err = sock:receive(length)
                    if not str then
                        co_yield(nil, err)
                    end
                    received = received + length

                    max_chunk_size = tonumber(co_yield(str) or default_chunk_size)
                    if max_chunk_size and max_chunk_size < 0 then max_chunk_size = nil end

                    if not max_chunk_size then
                        ngx_log(ngx_ERR, "Buffer size not specified, bailing")
                        break
                    end
                end

            until length == 0
        end
    end)
end


local function _no_body_reader()
    return nil
end


local function _read_body(res)
    local reader = res.body_reader

    if not reader then
        -- Most likely HEAD or 304 etc.
        return nil, "no body to be read"
    end

    local chunks = {}
    local c = 1

    local chunk, err
    repeat
        chunk, err = reader()

        if err then
            return nil, err, tbl_concat(chunks) -- Return any data so far.
        end
        if chunk then
            chunks[c] = chunk
            c = c + 1
        end
    until not chunk

    return tbl_concat(chunks)
end


local function _trailer_reader(sock)
    return co_wrap(function()
        co_yield(_receive_headers(sock))
    end)
end


local function _read_trailers(res)
    local reader = res.trailer_reader
    if not reader then
        return nil, "no trailers"
    end

    local trailers = reader()
    setmetatable(res.headers, { __index = trailers })
end


local function _send_body(sock, body)
    if type(body) == "function" then
        repeat
            local chunk, err, partial = body()

            if chunk then
                local ok, err = sock:send(chunk)

                if not ok then
                    return nil, err
                end
            elseif err ~= nil then
                return nil, err, partial
            end

        until chunk == nil
    elseif body ~= nil then
        local bytes, err = sock:send(body)

        if not bytes then
            return nil, err
        end
    end
    return true, nil
end


local function _handle_continue(sock, body)
    local status, version, reason, err = _receive_status(sock) --luacheck: no unused
    if not status then
        return nil, nil, err
    end

    -- Only send body if we receive a 100 Continue
    if status == 100 then
        local ok, err = sock:receive("*l") -- Read carriage return
        if not ok then
            return nil, nil, err
        end
        _send_body(sock, body)
    end
    return status, version, err
end


function _M.send_request(self, params)
    -- Apply defaults
    setmetatable(params, { __index = DEFAULT_PARAMS })

    local sock = self.sock
    local body = params.body
    local headers = http_headers.new()

    -- We assign one-by-one so that the metatable can handle case insensitivity
    -- for us. You can blame the spec for this inefficiency.
    local params_headers = params.headers or {}
    for k, v in pairs(params_headers) do
        headers[k] = v
    end

    if not headers["Proxy-Authorization"] then
        -- TODO: next major, change this to always override the provided
        -- header. Can't do that yet because it would be breaking.
        -- The connect method uses self.http_proxy_auth in the poolname so
        -- that should be leading.
        headers["Proxy-Authorization"] = self.http_proxy_auth
    end

    -- Ensure we have appropriate message length or encoding.
    do
        local is_chunked = transfer_encoding_is_chunked(headers)

        if is_chunked then
            -- If we have both Transfer-Encoding and Content-Length we MUST
            -- drop the Content-Length, to help prevent request smuggling.
            -- https://tools.ietf.org/html/rfc7230#section-3.3.3
            headers["Content-Length"] = nil

        elseif not headers["Content-Length"] then
            -- A length was not given, try to calculate one.

            local body_type = type(body)

            if body_type == "function" then
                return nil, "Request body is a function but a length or chunked encoding is not specified"

            elseif body_type == "table" then
                local length = 0
                for _, v in ipairs(body) do
                    length = length + #tostring(v)
                end
                headers["Content-Length"] = length

            elseif body == nil and EXPECTING_BODY[str_upper(params.method)] then
                headers["Content-Length"] = 0

            elseif body ~= nil then
                headers["Content-Length"] = #tostring(body)
            end
        end
    end

    if not headers["Host"] then
        if (str_sub(self.host, 1, 5) == "unix:") then
            return nil, "Unable to generate a useful Host header for a unix domain socket. Please provide one."
        end
        -- If we have a port (i.e. not connected to a unix domain socket), and this
        -- port is non-standard, append it to the Host header.
        if self.port then
            if self.ssl and self.port ~= 443 then
                headers["Host"] = self.host .. ":" .. self.port
            elseif not self.ssl and self.port ~= 80 then
                headers["Host"] = self.host .. ":" .. self.port
            else
                headers["Host"] = self.host
            end
        else
            headers["Host"] = self.host
        end
    end
    if not headers["User-Agent"] then
        headers["User-Agent"] = _M._USER_AGENT
    end
    if params.version == 1.0 and not headers["Connection"] then
        headers["Connection"] = "Keep-Alive"
    end

    params.headers = headers

    -- Format and send request
    local req = _format_request(self, params)
    if DEBUG then ngx_log(ngx_DEBUG, "\n", req) end
    local bytes, err = sock:send(req)

    if not bytes then
        return nil, err
    end

    -- Send the request body, unless we expect: continue, in which case
    -- we handle this as part of reading the response.
    if headers["Expect"] ~= "100-continue" then
        local ok, err, partial = _send_body(sock, body)
        if not ok then
            return nil, err, partial
        end
    end

    return true
end


function _M.read_response(self, params)
    local sock = self.sock

    local status, version, reason, err

    -- If we expect: continue, we need to handle this, sending the body if allowed.
    -- If we don't get 100 back, then status is the actual status.
    if params.headers["Expect"] == "100-continue" then
        local _status, _version, _err = _handle_continue(sock, params.body)
        if not _status then
            return nil, _err
        elseif _status ~= 100 then
            status, version, err = _status, _version, _err -- luacheck: no unused
        end
    end

    -- Just read the status as normal.
    if not status then
        status, version, reason, err = _receive_status(sock)
        if not status then
            return nil, err
        end
    end


    local res_headers, err = _receive_headers(sock)
    if not res_headers then
        return nil, err
    end

    -- keepalive is true by default. Determine if this is correct or not.
    local ok, connection = pcall(str_lower, res_headers["Connection"])
    if ok then
        if (version == 1.1 and str_find(connection, "close", 1, true)) or
           (version == 1.0 and not str_find(connection, "keep-alive", 1, true)) then
            self.keepalive = false
        end
    else
        -- no connection header
        if version == 1.0 then
            self.keepalive = false
        end
    end

    local body_reader = _no_body_reader
    local trailer_reader, err
    local has_body = false

    -- Receive the body_reader
    if _should_receive_body(params.method, status) then
        has_body = true

        if version == 1.1 and transfer_encoding_is_chunked(res_headers) then
            body_reader, err = _chunked_body_reader(sock)
        else
            local ok, length = pcall(tonumber, res_headers["Content-Length"])
            if not ok then
                -- No content-length header, read until connection is closed by server
                length = nil
            end

            body_reader, err = _body_reader(sock, length)
        end
    end

    if res_headers["Trailer"] then
        trailer_reader, err = _trailer_reader(sock)
    end

    if err then
        return nil, err
    else
        return {
            status = status,
            reason = reason,
            headers = res_headers,
            has_body = has_body,
            body_reader = body_reader,
            read_body = _read_body,
            trailer_reader = trailer_reader,
            read_trailers = _read_trailers,
        }
    end
end


function _M.request(self, params)
    params = tbl_copy(params) -- Take by value
    local res, err = self:send_request(params)
    if not res then
        return res, err
    else
        return self:read_response(params)
    end
end


function _M.request_pipeline(self, requests)
    requests = tbl_copy(requests) -- Take by value

    for _, params in ipairs(requests) do
        if params.headers and params.headers["Expect"] == "100-continue" then
            return nil, "Cannot pipeline request specifying Expect: 100-continue"
        end

        local res, err = self:send_request(params)
        if not res then
            return res, err
        end
    end

    local responses = {}
    for i, params in ipairs(requests) do
        responses[i] = setmetatable({
            params = params,
            response_read = false,
        }, {
            -- Read each actual response lazily, at the point the user tries
            -- to access any of the fields.
            __index = function(t, k)
                local res, err
                if t.response_read == false then
                    res, err = _M.read_response(self, t.params)
                    t.response_read = true

                    if not res then
                        ngx_log(ngx_ERR, err)
                    else
                        for rk, rv in pairs(res) do
                            t[rk] = rv
                        end
                    end
                end
                return rawget(t, k)
            end,
        })
    end
    return responses
end


function _M.request_uri(self, uri, params)
    params = tbl_copy(params or {}) -- Take by value
    if self.proxy_opts then
        params.proxy_opts = tbl_copy(self.proxy_opts or {})
    end

    do
        local parsed_uri, err = self:parse_uri(uri, false)
        if not parsed_uri then
            return nil, err
        end

        local path, query
        params.scheme, params.host, params.port, path, query = unpack(parsed_uri)
        params.path = params.path or path
        params.query = params.query or query
        params.ssl_server_name = params.ssl_server_name or params.host
    end

    do
        local proxy_auth = (params.headers or {})["Proxy-Authorization"]
        if proxy_auth and params.proxy_opts then
            params.proxy_opts.https_proxy_authorization = proxy_auth
            params.proxy_opts.http_proxy_authorization = proxy_auth
        end
    end

    local ok, err = self:connect(params)
    if not ok then
        return nil, err
    end

    local res, err = self:request(params)
    if not res then
        self:close()
        return nil, err
    end

    local body, err = res:read_body()
    if not body then
        self:close()
        return nil, err
    end

    res.body = body

    if params.keepalive == false then
        local ok, err = self:close()
        if not ok then
            ngx_log(ngx_ERR, err)
        end

    else
        local ok, err = self:set_keepalive(params.keepalive_timeout, params.keepalive_pool)
        if not ok then
            ngx_log(ngx_ERR, err)
        end

    end

    return res, nil
end


function _M.get_client_body_reader(_, chunksize, sock)
    chunksize = chunksize or 65536

    if not sock then
        local ok, err
        ok, sock, err = pcall(ngx_req_socket)

        if not ok then
            return nil, sock -- pcall err
        end

        if not sock then
            if err == "no body" then
                return nil
            else
                return nil, err
            end
        end
    end

    local headers = ngx_req_get_headers()
    local length = headers.content_length
    if length then
        return _body_reader(sock, tonumber(length), chunksize)
    elseif transfer_encoding_is_chunked(headers) then
        -- Not yet supported by ngx_lua but should just work...
        return _chunked_body_reader(sock, chunksize)
    else
        return nil
    end
end


function _M.set_proxy_options(self, opts)
    -- TODO: parse and cache these options, instead of parsing them
    -- on each request over and over again (lru-cache on module level)
    self.proxy_opts = tbl_copy(opts) -- Take by value
end


function _M.get_proxy_uri(self, scheme, host)
    if not self.proxy_opts then
        return nil
    end

    -- Check if the no_proxy option matches this host. Implementation adapted
    -- from lua-http library (https://github.com/daurnimator/lua-http)
    if self.proxy_opts.no_proxy then
        if self.proxy_opts.no_proxy == "*" then
            -- all hosts are excluded
            return nil
        end

        local no_proxy_set = {}
        -- wget allows domains in no_proxy list to be prefixed by "."
        -- e.g. no_proxy=.mit.edu
        for host_suffix in ngx_re_gmatch(self.proxy_opts.no_proxy, "\\.?([^,]+)", "jo") do
            no_proxy_set[host_suffix[1]] = true
        end

        -- From curl docs:
        -- matched as either a domain which contains the hostname, or the
        -- hostname itself. For example local.com would match local.com,
        -- local.com:80, and www.local.com, but not www.notlocal.com.
        --
        -- Therefore, we keep stripping subdomains from the host, compare
        -- them to the ones in the no_proxy list and continue until we find
        -- a match or until there's only the TLD left
        repeat
            if no_proxy_set[host] then
                return nil
            end

            -- Strip the next level from the domain and check if that one
            -- is on the list
            host = ngx_re_sub(host, "^[^.]+\\.", "", "jo")
        until not ngx_re_find(host, "\\.", "jo")
    end

    if scheme == "http" and self.proxy_opts.http_proxy then
        return self.proxy_opts.http_proxy
    end

    if scheme == "https" and self.proxy_opts.https_proxy then
        return self.proxy_opts.https_proxy
    end

    return nil
end


-- ----------------------------------------------------------------------------
-- The following functions are considered DEPRECATED and may be REMOVED in
-- future releases. Please see the notes in `README.md`.
-- ----------------------------------------------------------------------------

function _M.ssl_handshake(self, ...)
    ngx_log(ngx_DEBUG, "Use of deprecated function `ssl_handshake`")

    local sock = self.sock
    if not sock then
        return nil, "not initialized"
    end

    self.ssl = true

    return sock:sslhandshake(...)
end


function _M.connect_proxy(self, proxy_uri, scheme, host, port, proxy_authorization)
    ngx_log(ngx_DEBUG, "Use of deprecated function `connect_proxy`")

    -- Parse the provided proxy URI
    local parsed_proxy_uri, err = self:parse_uri(proxy_uri, false)
    if not parsed_proxy_uri then
        return nil, err
    end

    -- Check that the scheme is http (https is not supported for
    -- connections between the client and the proxy)
    local proxy_scheme = parsed_proxy_uri[1]
    if proxy_scheme ~= "http" then
        return nil, "protocol " .. proxy_scheme .. " not supported for proxy connections"
    end

    -- Make the connection to the given proxy
    local proxy_host, proxy_port = parsed_proxy_uri[2], parsed_proxy_uri[3]
    local c, err = self:tcp_only_connect(proxy_host, proxy_port)
    if not c then
        return nil, err
    end

    if scheme == "https" then
        -- Make a CONNECT request to create a tunnel to the destination through
        -- the proxy. The request-target and the Host header must be in the
        -- authority-form of RFC 7230 Section 5.3.3. See also RFC 7231 Section
        -- 4.3.6 for more details about the CONNECT request
        local destination = host .. ":" .. port
        local res, err = self:request({
            method = "CONNECT",
            path = destination,
            headers = {
                ["Host"] = destination,
                ["Proxy-Authorization"] = proxy_authorization,
            }
        })

        if not res then
            return nil, err
        end

        if res.status < 200 or res.status > 299 then
            return nil, "failed to establish a tunnel through a proxy: " .. res.status
        end
    end

    return c, nil
end


function _M.proxy_request(self, chunksize)
    ngx_log(ngx_DEBUG, "Use of deprecated function `proxy_request`")

    return self:request({
        method = ngx_req_get_method(),
        path = ngx_re_gsub(ngx_var.uri, "\\s", "%20", "jo") .. ngx_var.is_args .. (ngx_var.query_string or ""),
        body = self:get_client_body_reader(chunksize),
        headers = ngx_req_get_headers(),
    })
end


function _M.proxy_response(_, response, chunksize)
    ngx_log(ngx_DEBUG, "Use of deprecated function `proxy_response`")

    if not response then
        ngx_log(ngx_ERR, "no response provided")
        return
    end

    ngx.status = response.status

    -- Filter out hop-by-hop headeres
    for k, v in pairs(response.headers) do
        if not HOP_BY_HOP_HEADERS[str_lower(k)] then
            ngx_header[k] = v
        end
    end

    local reader = response.body_reader

    repeat
        local chunk, ok, read_err, print_err

        chunk, read_err = reader(chunksize)
        if read_err then
            ngx_log(ngx_ERR, read_err)
        end

        if chunk then
            ok, print_err = ngx_print(chunk)
            if not ok then
                ngx_log(ngx_ERR, print_err)
            end
        end

        if read_err or print_err then
            break
        end
    until not chunk
end


return _M
```

``http_headers.lua``

```lua
local rawget, rawset, setmetatable =
    rawget, rawset, setmetatable

local str_lower = string.lower

local _M = {
    _VERSION = '0.17.0-beta.1',
}


-- Returns an empty headers table with internalised case normalisation.
function _M.new()
    local mt = {
        normalised = {},
    }

    mt.__index = function(t, k)
        return rawget(t, mt.normalised[str_lower(k)])
    end

    mt.__newindex = function(t, k, v)
        local k_normalised = str_lower(k)

        -- First time seeing this header field?
        if not mt.normalised[k_normalised] then
            -- Create a lowercased entry in the metatable proxy, with the value
            -- of the given field case
            mt.normalised[k_normalised] = k

            -- Set the header using the given field case
            rawset(t, k, v)
        else
            -- We're being updated just with a different field case. Use the
            -- normalised metatable proxy to give us the original key case, and
            -- perorm a rawset() to update the value.
            rawset(t, mt.normalised[k_normalised], v)
        end
    end

    return setmetatable({}, mt)
end


return _M
```

### test::nginx

#### 安装

```bash
apt-get update && apt-get install wget cpanminus git -y && cpanm --notest Test::Nginx IPC::Run > build.log 2>&1 || (cat build.log && exit 1)
&& git clone https://github.com/openresty/test-nginx.git && cd test-nginx && perl Makefile.PL && make && make install
```

测试命令

```bash
cd /usr/local/openresty/nginx/conf
prove -I ~/test-nginx/lib/ -r t
```

ci

```
image: docker:latest

services:
  - docker:dind

stages:
  - test

test:
  # image: harbor.intsig.net/base/openresty:1.17.8.2-ubuntu20.04
  image: openresty/openresty
  tags:
    - openresty
  before_script:
    - cp -r $CI_PROJECT_DIR/conf /usr/local/openresty/nginx/conf
    - echo 'deb http://mirrors.aliyun.com/debian/ stretch main non-free contrib
      deb-src http://mirrors.aliyun.com/debian/ stretch main non-free contrib
      deb http://mirrors.aliyun.com/debian-security stretch/updates main
      deb-src http://mirrors.aliyun.com/debian-security stretch/updates main
      deb http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib
      deb-src http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib
      deb http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib
      deb-src http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib' > /etc/apt/sources.list
    - apt-get update 
    # - apt-get install perl-base=5.24.1-3+deb9u7 libgdbm3 netbase rename perl-modules-5.24 libperl5.24 perl -y 
    - apt-get install perl-base=5.24.1-3+deb9u7 -y --allow-downgrades
    - apt-get install perl-modules-5.24=5.24.1-3+deb9u7 -y
    - apt-get install libperl5.24=5.24.1-3+deb9u7 -y
    - apt-get install perl -y
    - apt-get install cpanminus -y
    - apt-get install git -y 
    # - apt-get install libc6=2.24-11+deb9u4 --allow-downgrades
    # - apt-get install libc-dev-bin=2.24-11+deb9u4
    # - apt-get install libc6-dev 
    # - apt-get install gcc automake autoconf libtool make -y
    - apt-get install make -y
    - cpanm --notest Test::Nginx IPC::Run > build.log 2>&1 || (cat build.log && exit 1) 
    - cd ~ && git clone https://github.com/openresty/test-nginx.git && cd test-nginx && perl Makefile.PL && make && make install
  script:
    - cd /usr/local/openresty/nginx/conf
    - prove -I ~/test-nginx/lib/ -r t
```



### busted

#### 安装

```
apt-get update && apt-get install vim luarocks -y && luarocks --server=http://rocks.moonscript.org install lyaml && luarocks install luasocket && luarocks install lpeg && luarocks install ansicolors
```

### gitlab runner mac docker安装

Option 1: Use local system volume mounts to start the Runner container

```bash
docker run -d --name gitlab-runner --restart always \
     -v /Users/Shared/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest
```

Option 2: Use Docker volumes to start the Runner container

1. Create the Docker volume:

```bash
docker volume create gitlab-runner-config
```

2. Start the GitLab Runner container using the volume we just created:

```bash
docker run -d --name gitlab-runner --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v gitlab-runner-config:/etc/gitlab-runner \
    gitlab/gitlab-runner:latest
```

Register

```bash
docker run --rm -v /Users/Shared/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register \
  --non-interactive \
  --executor "ubuntu" \
  --docker-image ubuntu:latest \
  --url "https://gitlab.intsig.net/" \
  --registration-token "DD9f3deZ_Ezm9CSDBWqr" \
  --description "ubuntu-runner" \
  --tag-list "ubuntu" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

## OpenResty代码

### cjson.safe和resty.http，环境变量设置

示例

```nginx
events {
    worker_connections  204800;
}

env SRC;
env DST;
```

```lua
local cjson = require("cjson.safe")
local shell = require("resty.shell")
local httpc = require("resty.http").new()

-- get env
-- accept src
-- 10.0.2.15:5000
local src = os.getenv("SRC")
local dst = os.getenv("DST")
local registry_src = "docker://" .. src .. "/"
local registry_dst = "docker://" .. dst .. "/"

-- get registry images
local baseURI = "http://" .. src .. "/v2/"
local res, err = httpc:request_uri( baseURI .. "_catalog", {
    method = "GET",
})

if not res then
    ngx.log(ngx.ERR, "request failed: ", err)
    return
end

local status = res.status
local length = res.headers["Content-Length"]
local body   = res.body

ngx.say("status: ", status .. " length: ", length .. " body: ", body)

local str = cjson.decode(body)

-- get  images of the registry
local images = str.repositories

-- display every image
for _, image in pairs(images) do
	ngx.say(image)
end
```

resty.http

```lua
-- get this image tags
local res, err = httpc:request_uri(baseURI .. image .. "/tags/list" , {
        method = "GET",
    })
if not res then
    ngx.log(ngx.ERR, "request failed: ", err)
    return
end
local status = res.status
local length = res.headers["Content-Length"]
local body   = res.body

-- print return info of tags
ngx.say("status: ", status .. " length: ", length .. " body: ", body)
local str = cjson.decode(res.body)
local tags = str.tags
```



### resty.shell

示例

```lua
for _, tag in pairs(tags) do
            local str = "skopeo --insecure-policy copy --dest-tls-verify=false --src-tls-verify=false " .. registry_src .. image .. ":" .. tag .. " " .. registry_dst .. image .. ":" .. tag
            ngx.say(str)
            local ok, stdout, stderr, reason, status =
                shell.run(str)
            if not ok then
                ngx.say("transfer failed")
                return -1
            end
            ngx.say("ok:", ok)
            ngx.say("stdout:" , stdout)
            ngx.say("stderr:", stderr)
            ngx.say("reason:", reason)
            ngx.say("status:", status)
            ngx.say(tag)
        end
```

### git rebase处理

```
git rebase aws
git rebase font_color
git rebase --continue
git rebase --continue

git remote -v    查看分支信息，没什么用
```

开始reset

```bash
louis@Louis-2 tee % git reset --hard FETCH_HEAD

HEAD is now at f92f35c Merge branch 'font_color' into 'aws'
```

再重新去已经被修改的分支拉最新代码

```bash
git pull origin aws
```

查看现在的状态

```bash
louis@Louis-2 tee % git status
On branch font_color
Last command done (1 command done):
   pick d085ff6 feat: filter font color
Next commands to do (10 remaining commands):
   pick 9d00ef7 feat: fliter font color
   pick 8f4125b feat: filter font color
  (use "git rebase --edit-todo" to view and edit)
You are currently editing a commit while rebasing branch 'font_color' on 'f92f35c'.
  (use "git commit --amend" to amend the current commit)
  (use "git rebase --continue" once you are satisfied with your changes)
```

想切换到自己的分支，提示不行

```bash
louis@Louis-2 tee % git checkout font_color
nginx/lib/filter.lua: needs merge
nginx/t/filter.t: needs merge
error: you need to resolve your current index first
```

那只能merge了

```bash
louis@Louis-2 tee % git reset --merge
louis@Louis-2 tee % git checkout font_color
Already on 'font_color'
```

执行一下rebase continue，因为上面git status提示，如果想改变，就continue，跟着提示来continue，可能要两次

```bash
louis@Louis-2 tee % git rebase --continue
git rebase --continue
```

如果开着vscode的话，会提示要不要合并，点both accept，然后再查看git status

接着就能add，commit，push了

## OpenResty搭建文件服务器

### 启动

文件位置:`/usr/local/openresty/nginx/conf/lua/update.lua`

```lua
-- upload.lua
--==========================================
-- 文件上传
--==========================================
-- 导入模块
local upload = require "resty.upload"
local cjson = require "cjson"
-- 定义回复的结构
-- 基本结构
local function response(status,msg,data)
    local res = {}
    res["status"] = status
    res["msg"] = msg
    res["data"] = data
    local jsonData = cjson.encode(res)
    return jsonData
end
-- 默认成功的response
local function success()
    return response(0,"success",nil)
end
-- 默认带数据的response
local function successWithData(data)
    return response(0,"success",data)
end
-- 失败的response
local function failed( msg )
    return response(-1,msg,nil)
end
-- end
local chunk_size = 4096
-- 获取请求的form
local form, err = upload:new(chunk_size)
if not form then
    ngx.log(ngx.ERR, "failed to new upload: ", err)
    ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR) 
    ngx.say(failed("没有获取到文件"))
end
form:set_timeout(1000)
-- 定义 字符串 split 分割 属性
string.split = function(s, p)
    local rt= {}
    string.gsub(s, '[^'..p..']+', function(w) table.insert(rt, w) end )
    return rt
end
-- 定义 支持字符串前后 trim 属性
string.trim = function(s)
    return (s:gsub("^%s*(.-)%s*$", "%1"))
end
-- 文件保存的根路径
local saveRootPath = ngx.var.store_dir
-- 保存的文件对象
local fileToSave
-- 标识文件是否成功保存
local ret_save = false
-- 实际收到的文件名
local rawFileName
-- 实际保存的文件名
local filename
-- 开始处理数据
while true do
    -- 读取数据
    local typ, res, err = form:read()
    if not typ then
        ngx.say(failed(err))
        return
    end
    -- 开始读取 http header
    if typ == "header" then
        -- 解析出本次上传的文件名
        local key = res[1]
        local value = res[2]
        if key == "Content-Disposition" then
            -- 解析出本次上传的文件名
            -- form-data; name="keyName"; filename="xxx.xx"
            local kvlist = string.split(value, ';')
            for _, kv in ipairs(kvlist) do
                local seg = string.trim(kv)
                if seg:find("filename") then
                    local kvfile = string.split(seg, "=")
                    -- 实际的文件名
                    rawFileName = string.sub(kvfile[2], 2, -2)
                    -- 重命文件名
                    filename = os.time().."-"..rawFileName
                    if filename then
                        -- 打开(创建)文件
                        fileToSave = io.open(saveRootPath .."/" .. filename, "w+")
                        if not fileToSave then
                            -- ngx.say("failed to open file ", filename)
                            ngx.say(failed("打开文件失败"))
                            return
                        end
                        break
                    end
                end
            end
        end
    elseif typ == "body" then
        -- 开始读取 http body
        if fileToSave then
            -- 写入文件内容
            fileToSave:write(res)
        end
    elseif typ == "part_end" then
        -- 文件写结束，关闭文件
        if fileToSave then
            fileToSave:close()
            fileToSave = nil
        end
         
        ret_save = true
        -- 文件读取结束
    elseif typ == "eof" then 
        break
    else
        ngx.log(ngx.INFO, "do other things")
    end
end
if ret_save then
    local uploadData = {}
    uploadData["file"] = rawFileName
    uploadData["url"] = "http://xxx.com/download/"..filename
    ngx.say(successWithData(uploadData))
else
    ngx.say(failed("系统异常"))
end
```

文件位置:`/usr/local/openresty/nginx/conf/nginx.conf`

```lua
user root;
worker_processes  20;
error_log  logs/error.log notice;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    server {
        listen       8080;
        server_name  localhost;
        server_tokens off;
        # 最大允许上传的文件大小
        client_max_body_size 300m; 
        set $store_dir "/var/www/download"; # 文件存储路径
        # 文件上传接口：http://xxx.com/uploadfile
        add_header Strict-Transport-Security 'max-age=15552000';
        add_header X-Content-Type-Options 'nosniff';
        add_header X-Robots-Tag 'none';
        add_header X-Download-Options 'noopen';
        add_header X-Permitted-Cross-Domain-Policies 'none';
        add_header X-XSS-Protection '1;mode=block';
        add_header X-Frame-Options SAMEORIGIN;
        location /uploadfile {
            # 实现文件上传的逻辑
          content_by_lua_file conf/lua/update.lua; 
        }
        # 文件下载入口: http://xxx.com/download
        location /download {
            alias /var/www/download;
            autoindex on;
            autoindex_localtime on; 
        }
        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

