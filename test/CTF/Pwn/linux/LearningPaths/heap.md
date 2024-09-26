### 内存布局

进程初始部分的低地址是一部分保护空间，这段空间的作用有

1. 空闲内存：在程序加载和运行之前，这段内存空间通常是未被使用的空闲内存。它可以用于后续的内存分配和动态内存管理
2. 内存映射：在一些特殊情况下，这段内存空间可能会被用于内存映射，将磁盘上的文件映射到内存中，以便程序可以直接访问文件内容
3. 内核空间扩展：在某些情况下，这段内存空间可能会被用于扩展内核空间的一部分，以提供更多的内核功能和数据结构

接下来的内容是Text Segment与Data Segment，在Data Segment的结尾是bss段（未初始化数据段），bss段之后是堆区，堆区之后是映射区（内存映射空间的主要作用是用于映射文件到内存或者用于实现其他特殊的内存映射需求），接着是栈区，栈区后的高地址空间是内核空间，每个进程通过系统调用接口与内核进行交互，多个进程共享内核空间

### 堆区的扩展

堆段处于bss段的结尾，函数brk，sbrk修改数据段结束的位置，用来扩展堆，函数原型

```
#include<unistd.h>
int brk(void * addr); 
void * sbrk(intptr_t c);
```

brk将数据段结尾改变为addr，sbrk将数据段扩展increment个字节，是libc中对brk的封装

当brk无法扩展堆时，堆管理器会使用 mmap() 这个系统调用从映射区寻找可用的内存空间

这种方法是进程级别的内存管理机制，不依赖于线程的身份，所以无论线程是否是主线程，堆空间的初始化都使用brk

### 堆管理器arena & heap

一个线程对应一个arena（堆管理器），一个arena对应多个heap，即一个arena可以用来管理多个线程的堆分配，arena的数量由cpu内核数量决定

通常情况下一个线程只有一个堆

| system | arena number            |
| ------ | ----------------------- |
| 32bit  | 2 x cpu_core_number + 1 |
| 64bit  | 8 x cpu_core_number + 1 |

**arena数据结构malloc_state：**

```c
struct malloc_state {
  // Flags (formerly in max_fast)
  int flags;

  // Set if the fastbin chunks contain recently 
  // inserted free blocks, Note this is a bool 
  // but not all targets support atomics on booleans
  int have_fastchunks;

  // Fastbins
  mfastbinptr fastbinsY[NFASTBINS];

  // Base of the topmost chunk -- not otherwise kept in a bin
  mchunkptr top;

  // The remainder from the most recent split of a small request
  mchunkptr last_remainder;

  // Normal bins packed as described above
  // NBINS = 127
  mchunkptr bins[NBINS * 2 - 2];

  // Bitmap of bins
  unsigned int binmap[BINMAPSIZE];

  // Linked list
  struct malloc_state *next;

  // Linked list for free arenas.  
  struct malloc_state *next_free;

  // Number of threads attached to this arena
  INTERNAL_SIZE_T attached_threads;

  // Memory allocated from the system in this arena
  INTERNAL_SIZE_T system_mem;
  INTERNAL_SIZE_T max_system_mem;
};
```

**heap数据结构heap_info：**

```c
typedef struct _heap_info {
  // Arena for this heap
  struct malloc_state *ar_ptr;

  // Previous heap
  struct _heap_info *prev;

  // Current size in bytes
  size_t size;

  // Size in bytes that has been mprotected PROT_READ|PROT_WRITE
  size_t mprotect_size;

  // padding
  char pad[];
} heap_info;
```

如果一个arena用到多个heap，那么这些heap通过prev这个指针连接起来，并且通过ar_ptr这个指针指向所属的arena

### chunk & bin

#### malloc_chunk

INTERNAL_SIZE_T，在64位操作系统中是64位无符号整数，32位操作系统中是32位无符号整数

```c
struct malloc_chunk {

  INTERNAL_SIZE_T      mchunk_prev_size;  /* Size of previous chunk (if free).  */
  INTERNAL_SIZE_T      mchunk_size;       /* Size in bytes, including overhead. */

  struct malloc_chunk* fd;         /* double links -- used only if free. */
  struct malloc_chunk* bk;

  /* Only used for large blocks: pointer to next larger size.  */
  struct malloc_chunk* fd_nextsize; /* double links -- used only if free. */
  struct malloc_chunk* bk_nextsize;
};
```

mchunk_prev_size（物理相邻的前一地址chunk）

* 在previous chunk是free chunk的时候：这个字段的值是前一个chunk的size
* 在previous chunk是allocated chunk的时候：这个字段里的值可能是前一个chunk的payload（有效数据）

mchunk_size（32 位系统中，SIZE_SZ 是 4；64 位系统中，SIZE_SZ 是 8）

本chunk的大小，大小必须是 2 * SIZE_SZ 的整数倍，如果申请的内存大小不是 2 * SIZE_SZ 的整数倍，会被转换满足大小的最小的 2 * SIZE_SZ 的倍数

如果该chunk大小最小即8字节，该字段最后一个字节为00001000，该字段的低三个比特位对 chunk 的大小没有影响，从高到低分别表示为：

* `NON_MAIN_ARENA`，记录当前 chunk 是否不属于主线程，1 表示不属于，0 表示属于
* `IS_MAPPED`，记录当前 chunk 是否是由 mmap 分配的
* `PREV_INUSE`，记录前一个 chunk 块是否被分配。一般来说，堆中第一个被分配的内存块的 size 字段的 P 位都会被设置为 1，以便于防止访问前面的非法内存。当一个 chunk 的 size 的 P 位为 0 时，我们能通过 prev_size 字段来获取上一个 chunk 的大小以及地址。这也方便进行空闲 chunk 之间的合并

fd，bk：chunk 处于分配状态时，从 fd 字段开始是用户的数据。chunk 空闲时，通过 fd 和 bk 可以将空闲的 chunk 块加入到空闲的 chunk 块链表进行统一管理

* `fd` 指向下一个（非物理相邻）空闲的 chunk
* `bk` 指向上一个（非物理相邻）空闲的 chunk

fd_nextsize，bk_nextsize ：chunk空闲时使用，且只用于large chunk。一般空闲的 large chunk 在 fd 的遍历顺序中，按照由大到小的顺序排列。这样做可以避免在寻找合适 chunk 时挨个遍历

* fd_nextsize 指向前一个与当前 chunk 大小不同的第一个空闲块，不包含 bin 的头指针
* fd_nextsize 指向后一个与当前 chunk 大小不同的第一个空闲块，不包含 bin 的头指针

```
bin是一个空闲链表，用于将malloc_chunk连接
```

chunk被分配时：mchunk_prev_size（具体内容取决于前一个chunk是否被占用），mchunk_size，有效数据

chunk空闲时：mchunk_prev_size（具体内容取决于前一个chunk是否被占用），mchunk_size，fd，bk，fd_nextsize,fd_nextsize(这两个字段是否被使用取决于该空闲chunk是否被归到large bin中)

#### bin

一个链表通过将free chunk连接用来管理free chunk，该链表被称为bin

在arena数据结构malloc_state中，有两个bins数组，他们由多个bin组成，bin为单链表的fastbinsY和bin为双链表的bins

##### fastbinsY

分配速度最快的结构，各个bin中chunk大小如下：

| 16 | 24 | 32 | 40 | 48 | 56  | 64  | 72  | 80  | 88  | 32位 |
| -- | -- | -- | -- | -- | --- | --- | --- | --- | --- | ---- |
| 32 | 48 | 64 | 80 | 96 | 112 | 128 | 144 | 160 | 176 |      |

* 由10个malloc_chunk指针构成的数组
* 每个fastbin都是单链表
* 同一个fastbin中的free chunk大小相同，各个fast bin中的free chunk大小按size_sz字节递增
* fastbin的P标志位都是1，通常情况下不对free chunk进行merge（在malloc_cosnolidate中会对fastbin中的chunk合并）
* 每个 bin 采取 LIFO 策略 ，最近释放的 chunk 会更早地被分配，添加空闲chunk和分配chunk都在链表头进行，最后一个chunk的fd指针会指向下一个fastbin

根据chunk大小来找到对应的fastbinsY数组下标是由宏 `fastbin_index`实现的

`#define fastbin_index(sz)    ((((unsigned int) (sz)) >> (SIZE_SZ == 8 ? 4 : 3)) - 2)`

在64位下：

```
size:36      fastbin_index:0
...
size:47      fastbin_index:0
size:48      fastbin_index:1
...
size:63      fastbin_index:1
size:64      fastbin_index:2
...
size:79      fastbin_index:2
size:80      fastbin_index:3
...
size:95      fastbin_index:3
size:96      fastbin_index:4
...
size:111      fastbin_index:4
size:112      fastbin_index:5
...
size:127      fastbin_index:5
size:128      fastbin_index:6
...
size:143      fastbin_index:6
size:144      fastbin_index:7
...
size:159      fastbin_index:7
size:160      fastbin_index:8
...
size:175      fastbin_index:8
size:176      fastbin_index:9
```

由于内存对齐，各个fast bin中的free chunk大小按size_sz字节递增

##### bins

`mchunkptr bins[NBINS * 2 - 2];`

NBINS为127，共有1个unsorted bin，62个small bin，62个large bin

使用双向链表来管理，即第一个元素存放fd指针，第二个元素存放bk指针

各个bin的种类和存放位置如下：

| unsorted bin | small bin | ...        | small bin | large bin | ...          | large bin |
| :----------: | --------- | ---------- | --------- | --------- | ------------ | --------- |
|     0,1     | 2,3       | 4-123      | 124,125   | 126,127   | 128-249      | 250,251   |
|     bin1     | bin2      | bin3-bin62 | bin63     | bin64     | bin65-bin125 | bin126    |

step为1的位运算

###### unsorted bin

相当于bin的缓存，让malloc重新利用最近free的chunk。仅有一个bin，bin中的chunk size可以不同。

FIFO的分配策略，越早被释放越早被重新使用。

unsorted bin中第一个chunk的bk和最后一个chunk的fd都指向main_arena+48（32位）或main_arena+88（64位）的位置

###### small bin

同一个small bin的chunk size相同。一共62个small bin，chunk size大小从16字节开始，8字节递增，最大504字节

FIFO的分配策略，越早被释放越早被重新使用。将新释放的chunk添加到链表的前端，分配操作就从链表的尾端中获取chunk。

###### large bin

同small bin，chunk大小见下

**各个bin中chunk大小如下：**

| idx       | category | step  | bin size range         |
| --------- | -------- | ----- | ---------------------- |
| 1         | unsorted | N/A   | N/A                    |
| [2,63]    | small    | 1<<3  | [16, 504] [0x10,0x1f8] |
| [64,94]   | large    | 1<<6  | [512, 512+30*step]     |
| [95,111]  | large    | 1<<9  | [2496, 2496+16*step]   |
| [112,120] | large    | 1<<12 | [11200, 11200+8*step]  |
| [121,123] | large    | 1<<15 | [48064, 48064+2*step]  |
| [124,125] | large    | 1<<18 | [146368, 146368+step]  |
| 126       | large    | N/A   | [670656, -]            |

##### tcachebins

在libc2.26后引入了tcachebins的一部分，在libc-2.27中正式引入

第一次使用tcachebin时，会在堆上分配0x290大小的tcache_perthread_struct结构体，其中：

0x10的chunk头；0x2(uint16_t) * 64(TCACHE_MAX_BINS)；0x8(tcache_entry *) \* 64(TCACHE_MAX_BINS);

前0x2*64字节储存每个bin中的chunk数，后0x8\*64个字节储存tcache_entry指针，这个指针指向的是这个chunk偏移0x10处，即规定将用户内存起始位置作为tcache_entry，由于防止double free保护机制，每个tcache_entry的第二个成员key指向tcache_perthread_struct

每次释放tcache chunk纳入bin时，都从头插入




#### 特殊chunk

##### top chunk

位于堆的末尾，当所有的bin都无法满足分配要求时，就要从这块区域里来分配，分配的空间返给用户，剩余部分形成新的top chunk，如果top chunk的空间也不满足用户的请求，就要使用brk或者mmap来向系统申请更多的堆空间。在free chunk的时候，如果chunk size不属于fastbin的范围，就要考虑是不是和top chunk挨着，如果挨着，就要merge到top chunk中。

##### last remainder chunk

这个特殊chunk是被维护在unsorted bin中的，形成方式有两种：

1. 当申请的size属于small bin，但是对应的small bin为空，有大于申请的size的small bin为非空，就从此small bin上分配，将该samll bin的空闲chunk分割，一部分返给用户，一部分形成last remainder chunk，插入到unsorted bin中
2. 在使用fast bin和small bin分配失败后，会尝试从unsorted bin中分配，当满足以下条件时：

   * 申请的size在small bin的范围内
   * unsorted bin仅有一个free chunk
   * 此free chunk是last reminder chunk
   * 此free chunk大小满足分配需求

   将该last remainder chunk分割，一部分返给用户，剩余的形成新的last remainder chunk，插入到unsorted bin中

### malloc&free

在glibc中malloc和free只是别名，真正的现实是__libc_malloc，__libc_free

```
// malloc/malloc.c
strong_alias (__libc_malloc, malloc)
strong_alias(__libc_free, free)
```

strong_alias宏：

```
// include/libc-symbols.h
#define strong_alias(a, b) _strong_alias(a, b)
#define _strong_alias(a, b) \
  extern __typeof(a) b __attribute__ ((alias (#a)));
```

`__typeof()`是标准c的扩展，用于获取变量的数据类型，获取 `a` 的数据类型来创建变量 `b `。`__attribute__ ((alias (#a)))`: 这是一个 GNU C 扩展，用于将变量 `b `的定义链接到变量 `a `。具体来说，`__attribute__((alias))`允许你将 `b `视为 `a `的别名，使得访问 `b `实际上就是访问 `a `。`#a` 是一个预处理器的操作，将变量名 `a `转换为字符串常量。所以 `#a `将 `a ` 的名称作为字符串。`__attribute__((alias))`的参数是一个字符串常量，该字符串表示 `b `应该链接到的目标。因此，这里的 `(alias (#a))`意味着将 `b `链接到具有与变量 `a `相同名称的目标，即将 `b `视为 `a` 的别名。

这行代码的作用是创建一个新的变量 `b`，并将其链接到变量 `a`，使得访问 `b` 等同于访问 `a`，并且它们具有相同的数据类型。

**涉及到的宏**

```c
#define chunk2mem(p)   ((Void_t*)((char*)(p) + 2*SIZE_SZ))
#define mem2chunk(mem) ((mchunkptr)((char*)(mem) - 2*SIZE_SZ))
```

chunk2mem，chunk to mem，chunk指针转化为用户内存指针

mem2chunk，mem to chunk，用户内存指针转化为chunk指针

#### __libc_malloc

```
void* __libc_malloc (size_t bytes)
{
  mstate ar_ptr;
  void *victim;

  void *(*hook) (size_t, const void *)
    = atomic_forced_read (__malloc_hook);
  if (__builtin_expect (hook != NULL, 0))
    return (*hook)(bytes, RETURN_ADDRESS (0));

  arena_get (ar_ptr, bytes);

  victim = _int_malloc (ar_ptr, bytes);
  /* Retry with another arena only if we were able to find a usable arena
     before.  */
  if (!victim && ar_ptr != NULL)
    {
      LIBC_PROBE (memory_malloc_retry, 1, bytes);
      ar_ptr = arena_get_retry (ar_ptr, bytes);
      victim = _int_malloc (ar_ptr, bytes);
    }

  if (ar_ptr != NULL)
    __libc_lock_unlock (ar_ptr->mutex);

  assert (!victim || chunk_is_mmapped (mem2chunk (victim)) ||
          ar_ptr == arena_for_chunk (mem2chunk (victim)));
  return victim;
}
```

`mstate ar_ptr;`声明了一个名为 `ar_ptr` 的变量，它是一个指向arena的指针

`void* victim;`指向分配的内存块的指针

`void *(*hook) (size_t, const void *) = atomic_forced_read (__malloc_hook);`定义了一个 `hook` 函数指针，`atomic_forced_read` 函数来读取 `__malloc_hook` 变量的值，该变量是用户设置的内存分配钩子函数。

`if (__builtin_expect (hook != NULL, 0))     return (*hook)(bytes, RETURN_ADDRESS (0));`如果用户设置了钩子函数，且钩子函数不为  `NULL`，则调用钩子函数并返回结果，否则继续执行下面的代码

`arena_get (ar_ptr, bytes);`  `arena_get`函数用于获取一个arena，并将其地址存储在 `ar_ptr` 中

`victim = _int_malloc (ar_ptr, bytes);` `_int_malloc` 是 glibc 内部用于执行实际的内存分配操作的函数。调用 `_int_malloc` 函数来尝试分配 `bytes` 字节大小的内存块，并将分配的内存块的地址存储在 `victim` 中。

```
if (!victim && ar_ptr != NULL)
{
    LIBC_PROBE (memory_malloc_retry, 1, bytes);
    ar_ptr = arena_get_retry (ar_ptr, bytes);
    victim = _int_malloc (ar_ptr, bytes);
}
```

如果 `_int_malloc`分配失败且 `ar_ptr`不为空，执行以下代码

`LIBC_PROBE (memory_malloc_retry, 1, bytes);`宏，用于执行性能分析和调试操作。它会记录内存分配的重试操作，以帮助诊断内存分配问题

`ar_ptr = arena_get_retry (ar_ptr, bytes);` `arena_get_retry` 函数用于尝试从其他 arena 中获取内存，并将新的 arena 地址存储在 `ar_ptr` 中

`victim = _int_malloc (ar_ptr, bytes);`再次调用 `_int_malloc`函数分配

`if (ar_ptr != NULL)     __libc_lock_unlock (ar_ptr->mutex);`如果arena不为空则释放之前获取的 arena 的互斥锁

`assert (!victim || chunk_is_mmapped (mem2chunk (victim)) || ar_ptr == arena_for_chunk (mem2chunk (victim)));`这是一个断言语句，用于调试时检查，它检查分配的内存块是否符合预期的条件，包括是否是通过内存映射（mmap）分配的，以及是否属于正确的 arena。

最后返回 `victim` 指针

##### _int_malloc

```c
static void*
_int_malloc (mstate av, size_t bytes)
{
  INTERNAL_SIZE_T nb;               /* normalized request size */
  unsigned int idx;                 /* associated bin index */
  mbinptr bin;                      /* associated bin */

  mchunkptr victim;                 /* inspected/selected chunk */
  INTERNAL_SIZE_T size;             /* its size */
  int victim_index;                 /* its bin index */

  mchunkptr remainder;              /* remainder from a split */
  unsigned long remainder_size;     /* its size */

  unsigned int block;               /* bit map traverser */
  unsigned int bit;                 /* bit map traverser */
  unsigned int map;                 /* current word of binmap */

  mchunkptr fwd;                    /* misc temp for linking */
  mchunkptr bck;                    /* misc temp for linking */

  const char *errstr = NULL;

  /*
     Convert request size to internal form by adding SIZE_SZ bytes
     overhead plus possibly more to obtain necessary alignment and/or
     to obtain a size of at least MINSIZE, the smallest allocatable
     size. Also, checked_request2size traps (returning 0) request sizes
     that are so large that they wrap around zero when padded and
     aligned.
   */

  checked_request2size (bytes, nb);

  /* There are no usable arenas.  Fall back to sysmalloc to get a chunk from
     mmap.  */
  if (__glibc_unlikely (av == NULL))
    {
      void *p = sysmalloc (nb, av);
      if (p != NULL)
	alloc_perturb (p, bytes);
      return p;
    }

  /*
     If the size qualifies as a fastbin, first check corresponding bin.
     This code is safe to execute even if av is not yet initialized, so we
     can try it without checking, which saves some time on this fast path.
   */

  if ((unsigned long) (nb) <= (unsigned long) (get_max_fast ()))
    {
      idx = fastbin_index (nb);
      mfastbinptr *fb = &fastbin (av, idx);
      mchunkptr pp = *fb;
      do
        {
          victim = pp;
          if (victim == NULL)
            break;
        }
      while ((pp = catomic_compare_and_exchange_val_acq (fb, victim->fd, victim))
             != victim);
      if (victim != 0)
        {
          if (__builtin_expect (fastbin_index (chunksize (victim)) != idx, 0))
            {
              errstr = "malloc(): memory corruption (fast)";
            errout:
              malloc_printerr (check_action, errstr, chunk2mem (victim), av);
              return NULL;
            }
          check_remalloced_chunk (av, victim, nb);
          void *p = chunk2mem (victim);
          alloc_perturb (p, bytes);
          return p;
        }
    }

  /*
     If a small request, check regular bin.  Since these "smallbins"
     hold one size each, no searching within bins is necessary.
     (For a large request, we need to wait until unsorted chunks are
     processed to find best fit. But for small ones, fits are exact
     anyway, so we can check now, which is faster.)
   */

  if (in_smallbin_range (nb))
    {
      idx = smallbin_index (nb);
      bin = bin_at (av, idx);

      if ((victim = last (bin)) != bin)
        {
          if (victim == 0) /* initialization check */
            malloc_consolidate (av);
          else
            {
              bck = victim->bk;
	if (__glibc_unlikely (bck->fd != victim))
                {
                  errstr = "malloc(): smallbin double linked list corrupted";
                  goto errout;
                }
              set_inuse_bit_at_offset (victim, nb);
              bin->bk = bck;
              bck->fd = bin;

              if (av != &main_arena)
		set_non_main_arena (victim);
              check_malloced_chunk (av, victim, nb);
              void *p = chunk2mem (victim);
              alloc_perturb (p, bytes);
              return p;
            }
        }
    }

  /*
     If this is a large request, consolidate fastbins before continuing.
     While it might look excessive to kill all fastbins before
     even seeing if there is space available, this avoids
     fragmentation problems normally associated with fastbins.
     Also, in practice, programs tend to have runs of either small or
     large requests, but less often mixtures, so consolidation is not
     invoked all that often in most programs. And the programs that
     it is called frequently in otherwise tend to fragment.
   */

  else
    {
      idx = largebin_index (nb);
      if (have_fastchunks (av))
        malloc_consolidate (av);
    }

  /*
     Process recently freed or remaindered chunks, taking one only if
     it is exact fit, or, if this a small request, the chunk is remainder from
     the most recent non-exact fit.  Place other traversed chunks in
     bins.  Note that this step is the only place in any routine where
     chunks are placed in bins.

     The outer loop here is needed because we might not realize until
     near the end of malloc that we should have consolidated, so must
     do so and retry. This happens at most once, and only when we would
     otherwise need to expand memory to service a "small" request.
   */

  for (;; )
    {
      int iters = 0;
      while ((victim = unsorted_chunks (av)->bk) != unsorted_chunks (av))
        {
          bck = victim->bk;
          if (__builtin_expect (chunksize_nomask (victim) <= 2 * SIZE_SZ, 0)
              || __builtin_expect (chunksize_nomask (victim)
				   > av->system_mem, 0))
            malloc_printerr (check_action, "malloc(): memory corruption",
                             chunk2mem (victim), av);
          size = chunksize (victim);

          /*
             If a small request, try to use last remainder if it is the
             only chunk in unsorted bin.  This helps promote locality for
             runs of consecutive small requests. This is the only
             exception to best-fit, and applies only when there is
             no exact fit for a small chunk.
           */

          if (in_smallbin_range (nb) &&
              bck == unsorted_chunks (av) &&
              victim == av->last_remainder &&
              (unsigned long) (size) > (unsigned long) (nb + MINSIZE))
            {
              /* split and reattach remainder */
              remainder_size = size - nb;
              remainder = chunk_at_offset (victim, nb);
              unsorted_chunks (av)->bk = unsorted_chunks (av)->fd = remainder;
              av->last_remainder = remainder;
              remainder->bk = remainder->fd = unsorted_chunks (av);
              if (!in_smallbin_range (remainder_size))
                {
                  remainder->fd_nextsize = NULL;
                  remainder->bk_nextsize = NULL;
                }

              set_head (victim, nb | PREV_INUSE |
                        (av != &main_arena ? NON_MAIN_ARENA : 0));
              set_head (remainder, remainder_size | PREV_INUSE);
              set_foot (remainder, remainder_size);

              check_malloced_chunk (av, victim, nb);
              void *p = chunk2mem (victim);
              alloc_perturb (p, bytes);
              return p;
            }

          /* remove from unsorted list */
          unsorted_chunks (av)->bk = bck;
          bck->fd = unsorted_chunks (av);

          /* Take now instead of binning if exact fit */

          if (size == nb)
            {
              set_inuse_bit_at_offset (victim, size);
              if (av != &main_arena)
		set_non_main_arena (victim);
              check_malloced_chunk (av, victim, nb);
              void *p = chunk2mem (victim);
              alloc_perturb (p, bytes);
              return p;
            }

          /* place chunk in bin */

          if (in_smallbin_range (size))
            {
              victim_index = smallbin_index (size);
              bck = bin_at (av, victim_index);
              fwd = bck->fd;
            }
          else
            {
              victim_index = largebin_index (size);
              bck = bin_at (av, victim_index);
              fwd = bck->fd;

              /* maintain large bins in sorted order */
              if (fwd != bck)
                {
                  /* Or with inuse bit to speed comparisons */
                  size |= PREV_INUSE;
                  /* if smaller than smallest, bypass loop below */
                  assert (chunk_main_arena (bck->bk));
                  if ((unsigned long) (size)
		      < (unsigned long) chunksize_nomask (bck->bk))
                    {
                      fwd = bck;
                      bck = bck->bk;

                      victim->fd_nextsize = fwd->fd;
                      victim->bk_nextsize = fwd->fd->bk_nextsize;
                      fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
                    }
                  else
                    {
                      assert (chunk_main_arena (fwd));
                      while ((unsigned long) size < chunksize_nomask (fwd))
                        {
                          fwd = fwd->fd_nextsize;
			  assert (chunk_main_arena (fwd));
                        }

                      if ((unsigned long) size
			  == (unsigned long) chunksize_nomask (fwd))
                        /* Always insert in the second position.  */
                        fwd = fwd->fd;
                      else
                        {
                          victim->fd_nextsize = fwd;
                          victim->bk_nextsize = fwd->bk_nextsize;
                          fwd->bk_nextsize = victim;
                          victim->bk_nextsize->fd_nextsize = victim;
                        }
                      bck = fwd->bk;
                    }
                }
              else
                victim->fd_nextsize = victim->bk_nextsize = victim;
            }

          mark_bin (av, victim_index);
          victim->bk = bck;
          victim->fd = fwd;
          fwd->bk = victim;
          bck->fd = victim;

#define MAX_ITERS       10000
          if (++iters >= MAX_ITERS)
            break;
        }

      /*
         If a large request, scan through the chunks of current bin in
         sorted order to find smallest that fits.  Use the skip list for this.
       */

      if (!in_smallbin_range (nb))
        {
          bin = bin_at (av, idx);

          /* skip scan if empty or largest chunk is too small */
          if ((victim = first (bin)) != bin
	      && (unsigned long) chunksize_nomask (victim)
	        >= (unsigned long) (nb))
            {
              victim = victim->bk_nextsize;
              while (((unsigned long) (size = chunksize (victim)) <
                      (unsigned long) (nb)))
                victim = victim->bk_nextsize;

              /* Avoid removing the first entry for a size so that the skip
                 list does not have to be rerouted.  */
              if (victim != last (bin)
		  && chunksize_nomask (victim)
		    == chunksize_nomask (victim->fd))
                victim = victim->fd;

              remainder_size = size - nb;
              unlink (av, victim, bck, fwd);

              /* Exhaust */
              if (remainder_size < MINSIZE)
                {
                  set_inuse_bit_at_offset (victim, size);
                  if (av != &main_arena)
		    set_non_main_arena (victim);
                }
              /* Split */
              else
                {
                  remainder = chunk_at_offset (victim, nb);
                  /* We cannot assume the unsorted list is empty and therefore
                     have to perform a complete insert here.  */
                  bck = unsorted_chunks (av);
                  fwd = bck->fd;
	  if (__glibc_unlikely (fwd->bk != bck))
                    {
                      errstr = "malloc(): corrupted unsorted chunks";
                      goto errout;
                    }
                  remainder->bk = bck;
                  remainder->fd = fwd;
                  bck->fd = remainder;
                  fwd->bk = remainder;
                  if (!in_smallbin_range (remainder_size))
                    {
                      remainder->fd_nextsize = NULL;
                      remainder->bk_nextsize = NULL;
                    }
                  set_head (victim, nb | PREV_INUSE |
                            (av != &main_arena ? NON_MAIN_ARENA : 0));
                  set_head (remainder, remainder_size | PREV_INUSE);
                  set_foot (remainder, remainder_size);
                }
              check_malloced_chunk (av, victim, nb);
              void *p = chunk2mem (victim);
              alloc_perturb (p, bytes);
              return p;
            }
        }

      /*
         Search for a chunk by scanning bins, starting with next largest
         bin. This search is strictly by best-fit; i.e., the smallest
         (with ties going to approximately the least recently used) chunk
         that fits is selected.

         The bitmap avoids needing to check that most blocks are nonempty.
         The particular case of skipping all bins during warm-up phases
         when no chunks have been returned yet is faster than it might look.
       */

      ++idx;
      bin = bin_at (av, idx);
      block = idx2block (idx);
      map = av->binmap[block];
      bit = idx2bit (idx);

      for (;; )
        {
          /* Skip rest of block if there are no more set bits in this block.  */
          if (bit > map || bit == 0)
            {
              do
                {
                  if (++block >= BINMAPSIZE) /* out of bins */
                    goto use_top;
                }
              while ((map = av->binmap[block]) == 0);

              bin = bin_at (av, (block << BINMAPSHIFT));
              bit = 1;
            }

          /* Advance to bin with set bit. There must be one. */
          while ((bit & map) == 0)
            {
              bin = next_bin (bin);
              bit <<= 1;
              assert (bit != 0);
            }

          /* Inspect the bin. It is likely to be non-empty */
          victim = last (bin);

          /*  If a false alarm (empty bin), clear the bit. */
          if (victim == bin)
            {
              av->binmap[block] = map &= ~bit; /* Write through */
              bin = next_bin (bin);
              bit <<= 1;
            }

          else
            {
              size = chunksize (victim);

              /*  We know the first chunk in this bin is big enough to use. */
              assert ((unsigned long) (size) >= (unsigned long) (nb));

              remainder_size = size - nb;

              /* unlink */
              unlink (av, victim, bck, fwd);

              /* Exhaust */
              if (remainder_size < MINSIZE)
                {
                  set_inuse_bit_at_offset (victim, size);
                  if (av != &main_arena)
		    set_non_main_arena (victim);
                }

              /* Split */
              else
                {
                  remainder = chunk_at_offset (victim, nb);

                  /* We cannot assume the unsorted list is empty and therefore
                     have to perform a complete insert here.  */
                  bck = unsorted_chunks (av);
                  fwd = bck->fd;
	  if (__glibc_unlikely (fwd->bk != bck))
                    {
                      errstr = "malloc(): corrupted unsorted chunks 2";
                      goto errout;
                    }
                  remainder->bk = bck;
                  remainder->fd = fwd;
                  bck->fd = remainder;
                  fwd->bk = remainder;

                  /* advertise as last remainder */
                  if (in_smallbin_range (nb))
                    av->last_remainder = remainder;
                  if (!in_smallbin_range (remainder_size))
                    {
                      remainder->fd_nextsize = NULL;
                      remainder->bk_nextsize = NULL;
                    }
                  set_head (victim, nb | PREV_INUSE |
                            (av != &main_arena ? NON_MAIN_ARENA : 0));
                  set_head (remainder, remainder_size | PREV_INUSE);
                  set_foot (remainder, remainder_size);
                }
              check_malloced_chunk (av, victim, nb);
              void *p = chunk2mem (victim);
              alloc_perturb (p, bytes);
              return p;
            }
        }

    use_top:
      /*
         If large enough, split off the chunk bordering the end of memory
         (held in av->top). Note that this is in accord with the best-fit
         search rule.  In effect, av->top is treated as larger (and thus
         less well fitting) than any other available chunk since it can
         be extended to be as large as necessary (up to system
         limitations).

         We require that av->top always exists (i.e., has size >=
         MINSIZE) after initialization, so if it would otherwise be
         exhausted by current request, it is replenished. (The main
         reason for ensuring it exists is that we may need MINSIZE space
         to put in fenceposts in sysmalloc.)
       */

      victim = av->top;
      size = chunksize (victim);

      if ((unsigned long) (size) >= (unsigned long) (nb + MINSIZE))
        {
          remainder_size = size - nb;
          remainder = chunk_at_offset (victim, nb);
          av->top = remainder;
          set_head (victim, nb | PREV_INUSE |
                    (av != &main_arena ? NON_MAIN_ARENA : 0));
          set_head (remainder, remainder_size | PREV_INUSE);

          check_malloced_chunk (av, victim, nb);
          void *p = chunk2mem (victim);
          alloc_perturb (p, bytes);
          return p;
        }

      /* When we are using atomic ops to free fast chunks we can get
         here for all block sizes.  */
      else if (have_fastchunks (av))
        {
          malloc_consolidate (av);
          /* restore original bin index */
          if (in_smallbin_range (nb))
            idx = smallbin_index (nb);
          else
            idx = largebin_index (nb);
        }

      /*
         Otherwise, relay to handle system-dependent cases
       */
      else
        {
          void *p = sysmalloc (nb, av);
          if (p != NULL)
            alloc_perturb (p, bytes);
          return p;
        }
    }
}
```

* 定义用到的变量:

```c
	INTERNAL_SIZE_T nb;               /* normalized request size */
	unsigned int idx;                 /* associated bin index */
	mbinptr bin;                      /* associated bin */

	mchunkptr victim;                 /* inspected/selected chunk */
	INTERNAL_SIZE_T size;             /* its size */
	int victim_index;                 /* its bin index */

	mchunkptr remainder;              /* remainder from a split */
	unsigned long remainder_size;     /* its size */

	unsigned int block;               /* bit map traverser */
	unsigned int bit;                 /* bit map traverser */
	unsigned int map;                 /* current word of binmap */

	mchunkptr fwd;                    /* misc temp for linking */
	mchunkptr bck;                    /* misc temp for linking */

	const char* errstr = NULL;
```

* `checked_request2size(bytes, nb);`内存对齐
* 如果没有对应的arena就使用mmap分配

  ```c
  if (__glibc_unlikely(av == NULL))
  {
  	void* p = sysmalloc(nb, av);
  	if (p != NULL)
  		alloc_perturb(p, bytes);
  	return p;
  }
  ```
* 分配的字节小于等于fastbin中chunk的最大值，先用fastbinsY中的chunk分配

  ```
  if ((unsigned long)(nb) <= (unsigned long)(get_max_fast()))
  {
  	idx = fastbin_index(nb);
  	mfastbinptr* fb = &fastbin(av, idx);
  	mchunkptr pp = *fb;
  	do
  	{
  		victim = pp;
  		if (victim == NULL)
  			break;
  	} while ((pp = catomic_compare_and_exchange_val_acq(fb, victim->fd, victim)) != victim);
  	if (victim != 0)
  	{
  		if (__builtin_expect(fastbin_index(chunksize(victim)) != idx, 0))
  		{
  			errstr = "malloc(): memory corruption (fast)";
  		errout:
  			malloc_printerr(check_action, errstr, chunk2mem(victim), av);
  			return NULL;
  		}
  		check_remalloced_chunk(av, victim, nb);
  		void* p = chunk2mem(victim);
  		alloc_perturb(p, bytes);
  		return p;
  	}
  }
  ```
* 分配的字节在smallbin的范围内，用smallbin中的chunk分配

  ```
  if (in_smallbin_range(nb))
  {
  	idx = smallbin_index(nb);
  	bin = bin_at(av, idx);

  	if ((victim = last(bin)) != bin)	//分配的字节所属的smallbin不为最后一个smallbin（最大的一个smallbin），
  	{
  		if (victim == 0) /* initialization check */	//smallbin没有初始化即进行一次malloc_consolidate
  			malloc_consolidate(av);
  		else
  		{
  			bck = victim->bk;
  			if (__glibc_unlikely(bck->fd != victim))	// bk指针指向chunk的fd是否为本chunk，前后chunk的检查
  			{
  				errstr = "malloc(): smallbin double linked list corrupted";
  				goto errout;
  			}
  			set_inuse_bit_at_offset(victim, nb);	//设置该chunk为已分配状态
  			bin->bk = bck;				//从bin上脱去该chunk
  			bck->fd = bin;

  			if (av != &main_arena)		//判断并更改NON_MAIN_ARENA位
  				set_non_main_arena(victim);
  			check_malloced_chunk(av, victim, nb);	//对内存的检查
  			void* p = chunk2mem(victim);	//将chunk指针转化为用户指针返回
  			alloc_perturb(p, bytes);	//在分配的内存块中填充随机数据，通常用于调试和检测内存访问错误。在生产版本中被关闭
  			return p;
  		}
  	}
  }
  ```
* 

#### __libc_free

```c
void  __libc_free (void *mem)
{
  mstate ar_ptr;
  mchunkptr p;                          /* chunk corresponding to mem */

  void (*hook) (void *, const void *)
    = atomic_forced_read (__free_hook);
  if (__builtin_expect (hook != NULL, 0))
    {
      (*hook)(mem, RETURN_ADDRESS (0));
      return;
    }

  if (mem == 0)                              /* free(0) has no effect */
    return;

  p = mem2chunk (mem);

  if (chunk_is_mmapped (p))                       /* release mmapped memory. */
    {
      /* See if the dynamic brk/mmap threshold needs adjusting.
	 Dumped fake mmapped chunks do not affect the threshold.  */
      if (!mp_.no_dyn_threshold
          && chunksize_nomask (p) > mp_.mmap_threshold
          && chunksize_nomask (p) <= DEFAULT_MMAP_THRESHOLD_MAX
	  && !DUMPED_MAIN_ARENA_CHUNK (p))
        {
          mp_.mmap_threshold = chunksize (p);
          mp_.trim_threshold = 2 * mp_.mmap_threshold;
          LIBC_PROBE (memory_mallopt_free_dyn_thresholds, 2,
                      mp_.mmap_threshold, mp_.trim_threshold);
        }
      munmap_chunk (p);
      return;
    }

  ar_ptr = arena_for_chunk (p);
  _int_free (ar_ptr, p, 0);
}
```

`mstate ar_ptr;` 声明了一个名为 `ar_ptr` 的变量，它是一个指向arena的指针

`mchunkptr p;` 声明了一个类型为 `mchunkptr` 的变量 `p`，它是一个指向内存块（chunk）的指针，用于表示要释放的内存

```c
void (*hook) (void *, const void *)
    = atomic_forced_read (__free_hook);
if (__builtin_expect (hook != NULL, 0))
{
    (*hook)(mem, RETURN_ADDRESS (0));
    return;
}
```

定义了一个 `hook` 函数指针，`atomic_forced_read` 函数来读取 `__free_hook` 变量的值，该变量是用户设置的内存分配钩子函数。如果用户设置了钩子函数，且钩子函数不为  `NULL`，则调用钩子函数并返回结果，否则继续执行下面的代码

`if (mem == 0)   return;` 如果为空指针，什么也不做

`p = mem2chunk (mem);`获取chunk指针p

```c
  if (chunk_is_mmapped (p))                       /* release mmapped memory. */
    {
      /* See if the dynamic brk/mmap threshold needs adjusting.
	 Dumped fake mmapped chunks do not affect the threshold.  */
      if (!mp_.no_dyn_threshold
          && chunksize_nomask (p) > mp_.mmap_threshold
          && chunksize_nomask (p) <= DEFAULT_MMAP_THRESHOLD_MAX
	  && !DUMPED_MAIN_ARENA_CHUNK (p))
        {
          mp_.mmap_threshold = chunksize (p);
          mp_.trim_threshold = 2 * mp_.mmap_threshold;
          LIBC_PROBE (memory_mallopt_free_dyn_thresholds, 2,
                      mp_.mmap_threshold, mp_.trim_threshold);
        }
      munmap_chunk (p);
      return;
    }
```

判断chunk是否由mmap分配，如果由mmap分配则由mummap

`ar_ptr = arena_for_chunk (p);`获取chunk对应的arena
`_int_free (ar_ptr, p, 0);`调用_int_free释放内存块

##### _int_free

```c
static void
_int_free (mstate av, mchunkptr p, int have_lock)
{
  INTERNAL_SIZE_T size;        /* its size */
  mfastbinptr *fb;             /* associated fastbin */
  mchunkptr nextchunk;         /* next contiguous chunk */
  INTERNAL_SIZE_T nextsize;    /* its size */
  int nextinuse;               /* true if nextchunk is used */
  INTERNAL_SIZE_T prevsize;    /* size of previous contiguous chunk */
  mchunkptr bck;               /* misc temp for linking */
  mchunkptr fwd;               /* misc temp for linking */

  const char *errstr = NULL;
  int locked = 0;

  size = chunksize (p);

  /* Little security check which won't hurt performance: the
     allocator never wrapps around at the end of the address space.
     Therefore we can exclude some size values which might appear
     here by accident or by "design" from some intruder.  */
  if (__builtin_expect ((uintptr_t) p > (uintptr_t) -size, 0)
      || __builtin_expect (misaligned_chunk (p), 0))
    {
      errstr = "free(): invalid pointer";
    errout:
      if (!have_lock && locked)
        __libc_lock_unlock (av->mutex);
      malloc_printerr (check_action, errstr, chunk2mem (p), av);
      return;
    }
  /* We know that each chunk is at least MINSIZE bytes in size or a
     multiple of MALLOC_ALIGNMENT.  */
  if (__glibc_unlikely (size < MINSIZE || !aligned_OK (size)))
    {
      errstr = "free(): invalid size";
      goto errout;
    }

  check_inuse_chunk(av, p);

  /*
    If eligible, place chunk on a fastbin so it can be found
    and used quickly in malloc.
  */

  if ((unsigned long)(size) <= (unsigned long)(get_max_fast ())

#if TRIM_FASTBINS
      /*
	If TRIM_FASTBINS set, don't place chunks
	bordering top into fastbins
      */
      && (chunk_at_offset(p, size) != av->top)
#endif
      ) {

    if (__builtin_expect (chunksize_nomask (chunk_at_offset (p, size))
			  <= 2 * SIZE_SZ, 0)
	|| __builtin_expect (chunksize (chunk_at_offset (p, size))
			     >= av->system_mem, 0))
      {
	/* We might not have a lock at this point and concurrent modifications
	   of system_mem might have let to a false positive.  Redo the test
	   after getting the lock.  */
	if (have_lock
	    || ({ assert (locked == 0);
		  __libc_lock_lock (av->mutex);
		  locked = 1;
		  chunksize_nomask (chunk_at_offset (p, size)) <= 2 * SIZE_SZ
		    || chunksize (chunk_at_offset (p, size)) >= av->system_mem;
	      }))
	  {
	    errstr = "free(): invalid next size (fast)";
	    goto errout;
	  }
	if (! have_lock)
	  {
	    __libc_lock_unlock (av->mutex);
	    locked = 0;
	  }
      }

    free_perturb (chunk2mem(p), size - 2 * SIZE_SZ);

    set_fastchunks(av);
    unsigned int idx = fastbin_index(size);
    fb = &fastbin (av, idx);

    /* Atomically link P to its fastbin: P->FD = *FB; *FB = P;  */
    mchunkptr old = *fb, old2;
    unsigned int old_idx = ~0u;
    do
      {
	/* Check that the top of the bin is not the record we are going to add
	   (i.e., double free).  */
	if (__builtin_expect (old == p, 0))
	  {
	    errstr = "double free or corruption (fasttop)";
	    goto errout;
	  }
	/* Check that size of fastbin chunk at the top is the same as
	   size of the chunk that we are adding.  We can dereference OLD
	   only if we have the lock, otherwise it might have already been
	   deallocated.  See use of OLD_IDX below for the actual check.  */
	if (have_lock && old != NULL)
	  old_idx = fastbin_index(chunksize(old));
	p->fd = old2 = old;
      }
    while ((old = catomic_compare_and_exchange_val_rel (fb, p, old2)) != old2);

    if (have_lock && old != NULL && __builtin_expect (old_idx != idx, 0))
      {
	errstr = "invalid fastbin entry (free)";
	goto errout;
      }
  }

  /*
    Consolidate other non-mmapped chunks as they arrive.
  */

  else if (!chunk_is_mmapped(p)) {
    if (! have_lock) {
      __libc_lock_lock (av->mutex);
      locked = 1;
    }

    nextchunk = chunk_at_offset(p, size);

    /* Lightweight tests: check whether the block is already the
       top block.  */
    if (__glibc_unlikely (p == av->top))
      {
	errstr = "double free or corruption (top)";
	goto errout;
      }
    /* Or whether the next chunk is beyond the boundaries of the arena.  */
    if (__builtin_expect (contiguous (av)
			  && (char *) nextchunk
			  >= ((char *) av->top + chunksize(av->top)), 0))
      {
	errstr = "double free or corruption (out)";
	goto errout;
      }
    /* Or whether the block is actually not marked used.  */
    if (__glibc_unlikely (!prev_inuse(nextchunk)))
      {
	errstr = "double free or corruption (!prev)";
	goto errout;
      }

    nextsize = chunksize(nextchunk);
    if (__builtin_expect (chunksize_nomask (nextchunk) <= 2 * SIZE_SZ, 0)
	|| __builtin_expect (nextsize >= av->system_mem, 0))
      {
	errstr = "free(): invalid next size (normal)";
	goto errout;
      }

    free_perturb (chunk2mem(p), size - 2 * SIZE_SZ);

    /* consolidate backward */
    if (!prev_inuse(p)) {
      prevsize = prev_size (p);
      size += prevsize;
      p = chunk_at_offset(p, -((long) prevsize));
      unlink(av, p, bck, fwd);
    }

    if (nextchunk != av->top) {
      /* get and clear inuse bit */
      nextinuse = inuse_bit_at_offset(nextchunk, nextsize);

      /* consolidate forward */
      if (!nextinuse) {
	unlink(av, nextchunk, bck, fwd);
	size += nextsize;
      } else
	clear_inuse_bit_at_offset(nextchunk, 0);

      /*
	Place the chunk in unsorted chunk list. Chunks are
	not placed into regular bins until after they have
	been given one chance to be used in malloc.
      */

      bck = unsorted_chunks(av);
      fwd = bck->fd;
      if (__glibc_unlikely (fwd->bk != bck))
	{
	  errstr = "free(): corrupted unsorted chunks";
	  goto errout;
	}
      p->fd = fwd;
      p->bk = bck;
      if (!in_smallbin_range(size))
	{
	  p->fd_nextsize = NULL;
	  p->bk_nextsize = NULL;
	}
      bck->fd = p;
      fwd->bk = p;

      set_head(p, size | PREV_INUSE);
      set_foot(p, size);

      check_free_chunk(av, p);
    }

    /*
      If the chunk borders the current high end of memory,
      consolidate into top
    */

    else {
      size += nextsize;
      set_head(p, size | PREV_INUSE);
      av->top = p;
      check_chunk(av, p);
    }

    /*
      If freeing a large space, consolidate possibly-surrounding
      chunks. Then, if the total unused topmost memory exceeds trim
      threshold, ask malloc_trim to reduce top.

      Unless max_fast is 0, we don't know if there are fastbins
      bordering top, so we cannot tell for sure whether threshold
      has been reached unless fastbins are consolidated.  But we
      don't want to consolidate on each free.  As a compromise,
      consolidation is performed if FASTBIN_CONSOLIDATION_THRESHOLD
      is reached.
    */

    if ((unsigned long)(size) >= FASTBIN_CONSOLIDATION_THRESHOLD) {
      if (have_fastchunks(av))
	malloc_consolidate(av);

      if (av == &main_arena) {
#ifndef MORECORE_CANNOT_TRIM
	if ((unsigned long)(chunksize(av->top)) >=
	    (unsigned long)(mp_.trim_threshold))
	  systrim(mp_.top_pad, av);
#endif
      } else {
	/* Always try heap_trim(), even if the top chunk is not
	   large, because the corresponding heap might go away.  */
	heap_info *heap = heap_for_ptr(top(av));

	assert(heap->ar_ptr == av);
	heap_trim(heap, mp_.top_pad);
      }
    }

    if (! have_lock) {
      assert (locked);
      __libc_lock_unlock (av->mutex);
    }
  }
  /*
    If the chunk was allocated via mmap, release via munmap().
  */

  else {
    munmap_chunk (p);
  }
}
```

#### malloc_cosnolidate

```c
static void malloc_consolidate(mstate av)
{
  mfastbinptr*    fb;                 /* current fastbin being consolidated */
  mfastbinptr*    maxfb;              /* last fastbin (for loop control) */
  mchunkptr       p;                  /* current chunk being consolidated */
  mchunkptr       nextp;              /* next chunk to consolidate */
  mchunkptr       unsorted_bin;       /* bin header */
  mchunkptr       first_unsorted;     /* chunk to link to */

  /* These have same use as in free() */
  mchunkptr       nextchunk;
  INTERNAL_SIZE_T size;
  INTERNAL_SIZE_T nextsize;
  INTERNAL_SIZE_T prevsize;
  int             nextinuse;
  mchunkptr       bck;
  mchunkptr       fwd;

  /*
    If max_fast is 0, we know that av hasn't
    yet been initialized, in which case do so below
  */

  if (get_max_fast () != 0) {
    clear_fastchunks(av);

    unsorted_bin = unsorted_chunks(av);

    /*
      Remove each chunk from fast bin and consolidate it, placing it
      then in unsorted bin. Among other reasons for doing this,
      placing in unsorted bin avoids needing to calculate actual bins
      until malloc is sure that chunks aren't immediately going to be
      reused anyway.
    */

    maxfb = &fastbin (av, NFASTBINS - 1);
    fb = &fastbin (av, 0);
    do {
      p = atomic_exchange_acq (fb, NULL);
      if (p != 0) {
	do {
	  check_inuse_chunk(av, p);
	  nextp = p->fd;

	  /* Slightly streamlined version of consolidation code in free() */
	  size = chunksize (p);
	  nextchunk = chunk_at_offset(p, size);
	  nextsize = chunksize(nextchunk);

	  if (!prev_inuse(p)) {
	    prevsize = prev_size (p);
	    size += prevsize;
	    p = chunk_at_offset(p, -((long) prevsize));
	    unlink(av, p, bck, fwd);
	  }

	  if (nextchunk != av->top) {
	    nextinuse = inuse_bit_at_offset(nextchunk, nextsize);

	    if (!nextinuse) {
	      size += nextsize;
	      unlink(av, nextchunk, bck, fwd);
	    } else
	      clear_inuse_bit_at_offset(nextchunk, 0);

	    first_unsorted = unsorted_bin->fd;
	    unsorted_bin->fd = p;
	    first_unsorted->bk = p;

	    if (!in_smallbin_range (size)) {
	      p->fd_nextsize = NULL;
	      p->bk_nextsize = NULL;
	    }

	    set_head(p, size | PREV_INUSE);
	    p->bk = unsorted_bin;
	    p->fd = first_unsorted;
	    set_foot(p, size);
	  }

	  else {
	    size += nextsize;
	    set_head(p, size | PREV_INUSE);
	    av->top = p;
	  }

	} while ( (p = nextp) != 0);

      }
    } while (fb++ != maxfb);
  }
  else {
    malloc_init_state(av);
    check_malloc_state(av);
  }
}
```
