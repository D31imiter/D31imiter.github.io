#### 概览

第一次使用tcachebin时，会在堆上分配0x290大小的tcache_perthread_struct结构体，其中：

0x10的chunk头；0x2(uint16_t) * 64(TCACHE_MAX_BINS)；0x8(tcache_entry *) \* 64(TCACHE_MAX_BINS);

前0x2*64字节储存每个bin中的chunk数，后0x8\*64个字节储存tcache_entry指针，这个指针指向的是这个chunk偏移0x10处，即规定将用户内存起始位置作为tcache_entry，由于防止double free保护机制，每个tcache_entry的第二个成员key指向tcache_perthread_struct

每次释放tcache chunk纳入bin时，都从头插入

#### 数据结构

##### tcache_entry

```
typedef struct tcache_entry
{
  struct tcache_entry *next;
  /* This field exists to detect double frees.  */
  struct tcache_perthread_struct *key;
} tcache_entry;
```

key关键字是2.29中首次引入，指向tcache_perthread_struct结构体的指针

##### tcache_perthread_struct

```
typedef struct tcache_perthread_struct
{
  uint16_t counts[TCACHE_MAX_BINS];
  tcache_entry *entries[TCACHE_MAX_BINS];
} tcache_perthread_struct;
```

* counts：每个bin中chunk的个数
* entries：指针数组，每个成员为指向tcachebin的指针

##### tcache

`static __thread tcache_perthread_struct *tcache = NULL;`

tcache是指向tcache_perthread_struct结构体的指针，静态变量

##### e

```
static struct malloc_par mp_ =
{
  .top_pad = DEFAULT_TOP_PAD,
  .n_mmaps_max = DEFAULT_MMAP_MAX,
  .mmap_threshold = DEFAULT_MMAP_THRESHOLD,
  .trim_threshold = DEFAULT_TRIM_THRESHOLD,
#define NARENAS_FROM_NCORES(n) ((n) * (sizeof (long) == 4 ? 2 : 8))
  .arena_test = NARENAS_FROM_NCORES (1)
#if USE_TCACHE
  ,
  .tcache_count = TCACHE_FILL_COUNT,
  .tcache_bins = TCACHE_MAX_BINS,
  .tcache_max_bytes = tidx2usize (TCACHE_MAX_BINS-1),
  .tcache_unsorted_limit = 0 /* No limit.  */
#endif
};
```

#### 宏

```
# define csize2tidx(x) (((x) - MINSIZE + MALLOC_ALIGNMENT - 1) / MALLOC_ALIGNMENT)
# define TCACHE_FILL_COUNT 7
# define TCACHE_MAX_BINS		64
```

#### 函数

##### tcache_put

```c
static __always_inline void tcache_put (mchunkptr chunk, size_t tc_idx)
{
	tcache_entry *e = (tcache_entry *)chunk2mem (chunk);	//将chunk转化用户内存，再将此地址处转为为tcache_entry结构体

  	/* Mark this chunk as "in the tcache" so the test in _int_free will detect a double free.  */
  	e->key = tcache;

  	e->next = tcache->entries[tc_idx];	//e的next指针指向对应tcachebin的第一个
  	tcache->entries[tc_idx] = e;		//更新头指针
  	++(tcache->counts[tc_idx]);		//记录数++
}
```

##### tcache_get

```c
static __always_inline void *tcache_get (size_t tc_idx)
{
  	tcache_entry *e = tcache->entries[tc_idx];	//对应索引的tcachebin的头
  	tcache->entries[tc_idx] = e->next;		//更新tcachebin
	--(tcache->counts[tc_idx]);			//记录数--
  	e->key = NULL;					//清空key
  	return (void *) e;				//返回用户指针，不用转化直接返回
}
```

#### malloc

##### fastbin

```c
#if USE_TCACHE
	/* While we're here, if we see other chunks of the same size, stash them in the tcache.  */
	size_t tc_idx = csize2tidx (nb);	//获取在tcachebin中的索引
	if (tcache && tc_idx < mp_.tcache_bins)	//tcache初始化，且索引在tcache范围内
	{
		mchunkptr tc_victim;

		/* While bin not empty and tcache not full, copy chunks.  */
		while (tcache->counts[tc_idx] < mp_.tcache_count && (tc_victim = *fb) != NULL)
		// mp_.tcache_bins初始化为TCACHE_FILL_COUNT(7)，tc_victim为fastbin的第一个chunk
		{
			if (SINGLE_THREAD_P)	//单线程环境直接unlink
				*fb = tc_victim->fd;
			else			//多线程环境调用特定的宏unlink
			{
				REMOVE_FB (fb, pp, tc_victim);
				if (__glibc_unlikely (tc_victim == NULL))
					break;
			}
			tcache_put (tc_victim, tc_idx);	//放入tcachebin
		    }
		}
#endif
```

还是先从fastbin头拿chunk，但是拿完后对更新后的fasbin做检查，第一个chunk符合标准就纳入对应tcachebin

##### smallbin

```c
#if USE_TCACHE
	/* While we're here, if we see other chunks of the same size, stash them in the tcache.  */
	size_t tc_idx = csize2tidx (nb);		//获取在tcachebin中的索引
	if (tcache && tc_idx < mp_.tcache_bins){	//tcache初始化，且索引在tcache范围内
		mchunkptr tc_victim;

		/* While bin not empty and tcache not full, copy chunks over.  */
		while (tcache->counts[tc_idx] < mp_.tcache_count && (tc_victim = last (bin)) != bin)
		// mp_.tcache_bins初始化为TCACHE_FILL_COUNT(7);tcachebin不为空,tc_victim对应smallbin的尾节点，且对应bin不为空
		{
			if (tc_victim != 0)
			{
				// 从尾部unlink，放入tcachebin
				bck = tc_victim->bk;
				set_inuse_bit_at_offset (tc_victim, nb);
					if (av != &main_arena)
						set_non_main_arena (tc_victim);
				bin->bk = bck;
				bck->fd = bin;
				tcache_put (tc_victim, tc_idx);
	            	}
		}
	    }
#endif
```

还是先从smallbin尾部拿chunk，但是拿完后对更新后的fasbin做检查，第一个chunk符合标准就纳入对应tcachebin

#### free

仅在开始时判断是否需要放入tcachebin

```c
#if USE_TCACHE
  {
    size_t tc_idx = csize2tidx (size);
    if (tcache != NULL && tc_idx < mp_.tcache_bins)
      {
	/* Check to see if it's already in the tcache.  */
	tcache_entry *e = (tcache_entry *) chunk2mem (p);

	/* This test succeeds on double free.  However, we don't 100%
	   trust it (it also matches random payload data at a 1 in
	   2^<size_t> chance), so verify it's not an unlikely
	   coincidence before aborting.  */
	if (__glibc_unlikely (e->key == tcache))
	  {
	    tcache_entry *tmp;
	    LIBC_PROBE (memory_tcache_double_free, 2, e, tc_idx);

	    for (tmp = tcache->entries[tc_idx];tmp;tmp = tmp->next)
		if (tmp == e)
			malloc_printerr ("free(): double free detected in tcache 2");
	    /* If we get here, it was a coincidence.  We've wasted a
	       few cycles, but don't abort.  */
	  }

	if (tcache->counts[tc_idx] < mp_.tcache_count)
	  {
	    tcache_put (p, tc_idx);
	    return;
	  }
      }
  }
#endif
```
