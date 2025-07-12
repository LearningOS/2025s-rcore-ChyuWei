# 实验报告

## 1. 功能介绍

- 完成syscall_trace的功能,这个系统调用有三种功能，根据 trace_request 的值不同，执行不同的操作：
    1. 如果 trace_request 为 0，则 id 应被视作 *const u8 ，表示读取当前任务 id 地址处一个字节的无符号整数值。此时应忽略 data 参数。返回值为 id 地址处的值。
    2. 如果 trace_request 为 1，则 id 应被视作 *mut u8 ，表示写入 data （作为 u8，即只考虑最低位的一个字节）到该用户程序 id 地址处。返回值应为0。
    3. 如果 trace_request 为 2，表示查询当前任务调用编号为 id 的系统调用的次数，返回值为这个调用次数。本次调用也计入统计 。
    4. 否则，忽略其他参数，返回值为 -1

1. 直接将id转为指针， unsafe读取
2. 直接将id转为指针， unsafe写入
3. 在TaskControlBlock增加syscall_count, 类型是[u16; 512], 用于记录当前任务不同系统调用的次数
    - 修改TaskManager初始化代码, 并增加获取/修改当前任务syscall_count的函数
    - 在syscall函数增加当前任务对应系统调用的次数

## 2. 问答题

```
    1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容（运行 三个 bad 测例 (ch2b_bad_*.rs) ）， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。
```
- bad_address 不太理解有啥问题，目前没有内存保护
- bad_instructions 执行了sret，是S特权指令
- bad_register 读了S特权才能读取的寄存器


```
    深入理解 trap.S 中两个函数 __alltraps 和 __restore 的作用，并回答如下问题:

    L40：刚进入 __restore 时，sp 代表了什么值。请指出 __restore 的两种使用情景。  
```
- sp指向内核栈，放着TrapContext  
- __restore 使用场景
    - 用户系统调用后返回用户态程序
    - 程序第一次运行

```
    L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。
        ld t0, 32*8(sp)
        ld t1, 33*8(sp)
        ld t2, 2*8(sp)
        csrw sstatus, t0
        csrw sepc, t1
        csrw sscratch, t2
```
- sstatus 设置为 用户态特权级
- sepc 用户态syscall后的指令地址
- sscratch 提前读用户态sp到sscratch中，后续设置给sp
    
```
    L50-L56：为何跳过了 x2 和 x4？

        ld x1, 1*8(sp)
        ld x3, 3*8(sp)
        .set n, 5
        .rept 27
        LOAD_GP %n
        .set n, n+1
        .endr
```
- x2是sp， 已经读到sscratch，现在不能直接修改，需要到内核栈弹出TrapContext后,  才能修改sp
- x4用不上

```
    L60：该指令之后，sp 和 sscratch 中的值分别有什么意义？
        csrrw sp, sscratch, sp
```
- 交换sp和sscratch
- sp指向用户栈, sscratch指向内核栈

```
    __restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？
```
- sret

```
    L13：该指令之后，sp 和 sscratch 中的值分别有什么意义？
        csrrw sp, sscratch, sp
```
- sp指向内核栈, sscratch指向用户栈

```
    从 U 态进入 S 态是哪一条指令发生的？
```
- ecall




## 3. 荣誉准则
