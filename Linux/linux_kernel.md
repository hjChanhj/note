# linux内核函数

## 内存管理

### kmalloc

#### 函数原型

```c
static __always_inline void * kmalloc (size_t size, gfp_t flags);
```

kmalloc() 申请的内存位于物理内存映射区域，而且在**物理上是连续的**，它们与真实的物理地址只有一个固定的偏移。

因为存在较简单的转换关系，所以**对申请的内存大小有限制**，不能超过**128KB**（32KB X PAGE_SIZE）。

##### flags

- **GFP_ATOMIC** —— 分配内存的过程是一个原子过程，分配内存的过程不会被（高优先级进程或中断）打断
- **GFP_KERNEL** —— 正常分配内存；
- **GFP_DMA** —— 给 DMA 控制器分配内存，需要使用该标志（DMA要求分配虚拟地址和**物理地址连续**）

###### flags 的参考用法：

　|– 进程上下文，可以睡眠　　　　　GFP_KERNEL
　|– 进程上下文，不可以睡眠　　　　GFP_ATOMIC
　|　　|– 中断处理程序　　　　　　 GFP_ATOMIC
　|　　|– 软中断　　　　　　　　　 GFP_ATOMIC
　|　　|– Tasklet　　　　　　　　 GFP_ATOMIC
　|– 用于DMA的内存，可以睡眠　　  GFP_DMA | GFP_KERNEL
　|– 用于DMA的内存，不可以睡眠　　GFP_DMA | GFP_ATOMIC

kmalloc对应的释放函数是kfree。

### kfree

#### 函数原型

```c
void kfree(const void *objp);
```

### kzalloc

#### 函数原型

```c
/** * kzalloc - allocate memory. The memory is set to zero. * @size: how many bytes of memory are required. * @flags: the type of memory to allocate (see kmalloc). */
static inline void *kzalloc(size_t size, gfp_t flags) {
    return kmalloc(size, flags | __GFP_ZERO);
}
```

kzalloc() 函数是kmalloc() 函数的一个变种，参数及返回值是一样的。kzalloc() 实际上只是额外附加了 **__GFP_ZERO** 标志。所以它除了申请内核内存外，还会对申请到的内存内容**清零**。

kzalloc() 函数对应的释放函数也是kfree。

### vmalloc

```c
void *vmalloc(unsigned long size);
```

vmalloc() 函数则会在虚拟内存空间给出一块连续的内存区，但这片连续的虚拟内存在**物理内存中并不一定连续**。

由于 vmalloc() 没有保证申请到的是连续的物理内存，因此**对申请的内存大小没有限制**，如果需要申请较大的内存空间就需要用此函数了。

vmalloc对应的释放函数是vfree。

vmalloc() 和 vfree() 可以睡眠，因此不能从中断上下文或其他不允许阻塞情况下调用。

### vfree

#### 函数原型

```c
void vfree(const void *addr);
```

### vzalloc

#### 函数原型

```c
void *vzalloc(unsigned long size)
```

vzalloc和vmalloc的区别，与kzalloc和kmalloc的区别一样，在函数实现时flags多一个**__GFP_ZERO** 标志，会对申请到的内存内容**清零**。

vzalloc对应的释放函数也是vfree。

### kmalloc与vmalloc总结

#### 共同点

1. 用于申请**内核空间**的内存；
2. 内存以**字节**为单位进行分配；
3. 所分配的内存虚拟地址上连续；

#### 区别

|         | 物理内存连续 | 大小限制 | 是否可能产生阻塞 | 性能 |
| :-----: | :----------: | :------: | :--------------: | :--: |
| kmalloc |      是      |  <=128K  | 加GFP_ATOMIC时否 |  快  |
| vmalloc |      否      |  无限制  |        是        |  慢  |

一般情况下，内存只有在要被 DMA 访问的时候才需要物理上连续，但为了性能上的考虑，内核中一般使用 kmalloc()，而只有在需要获得大块内存时才使用 vmalloc()。例如，当模块被动态加载到内核当中时，就把模块装载到由 vmalloc() 分配的内存上。



### 参考

kmalloc,kzalloc,vmalloc:https://www.cnblogs.com/sky-heaven/p/7390370.html