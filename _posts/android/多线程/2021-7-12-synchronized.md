---
published: false
---

# synchronized
[synchronized](https://www.cnblogs.com/aspirant/p/11470858.html)

### 对象 MarkWork 部分
![对象 MarkWork](https://raw.githubusercontent.com/ooftf/Material/master/img/blog/1627210940(1).png)

### cas 底层实现
![cas 底层实现](https://raw.githubusercontent.com/ooftf/Material/master/img/blog/aea66353f5ec1084df3a2ad81a63a2c.png)

### _asm_
表示接下来要执行汇编指令

#### "cmpxchgl %1.(3%)" 
cmpxchg 这是一个 cas 汇编指令
#### LOCK_IF_MP

MP: Multi Processor 多处理器

如果是多处理器，锁定总线，只允许我通过总线传输数据
 