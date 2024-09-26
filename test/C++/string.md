# string

1. str.c_str,该成员函数返回类型为const char*，指向字符串的首字符，只读

2. assign，赋值函数

3. str.at(n),str[n]都是访问对应下标的字符

4. str.back(),返回最后一个字符

5. str1.capacity()，返回字符串的大小，有可能有被提前分配好的为被利用的空间

5. 与capcity不同的是，str.size()只返回字符串长度，不返回分配的内存空间

6. `begin();end()`分别返回指向容器头的迭代器、指向容器尾的迭代器

   `cbegin();cend()`分别返回`const`类型的指向容器头的迭代器、指向容器尾的迭代器（const_iterator)

   `rbegin(); rend()` 返回逆序迭代器。`crbegin(); crend()` 返回`const`修饰的逆序迭代器。

   对于反向迭代器，`++` 运算将访问前一个元素，`--` 运算则访问下一个元素

7. str.clear(),清空字符串容器中的字符

9. str1.compare(str2),按照字典序比较str1str2，对象大于参数则返回1，相同返回0，小于返回（-1）

10. str1._Equal("")，完全相同则返回true（1），否则返回false（0）

9. str1.copy(str,n)，将string类str1的前n个字符复制到str中，char为字符数组char[]

10. str.data(),返回指向string类的第一个字符的const char*，const修饰的字符指针

11. str.empty(),判断str是否为空，如果为空返回1（true），不为空返回0（false）

13. str.erase(iterator),删除迭代器所在位置的字符

14. str1.find(str),查找str1中是否有与str匹配的子串，如果匹配成功返回下标（int），找不到返回-1,因为返回值为size_t类型，所以一般返回size_t，其实就是string::npos，错误码但事实size_t型的

15. rfind，rfind是从末尾开始找

16. find_first_of()查找子串中的某个字符最先出现的位置。find_first_of()不是全匹配，意味这只要找到一个属于字串的字符就成功了

17. find_last_of()是从字符串的后面往前面搜索。

17. str.find_first_not_of(字串，n)，从下标为n初开始查找同find_first_of，如果子串中有源串中下标为n的字符则继续向后查找一旦发现不一样就反会源串的下标

17. find_last_not_of()与find_first_not_of()相似，只不过查找顺序是从指定位置向前。

18. str1._Swap_data(str2)，或者是swap，将str1和str2中的内容交换

19. str1.substr(m, n)，返回从str1中截获从m下标开始的n个字符

20. str.shrink_to_fit ()在STL中 vector和string 是比较特殊的，clear()之后是不会释放内存空间的，也就是size()会清零，但capacity()不会改变，需要手动去释放，说明 clear() 没有释放内存。想释放空间的话，除了swap一个空string外，c++11里新加入的的std::basic_string::shrink_to_fit 也可以。

21.  str1.max_size(),获取能创建的最大字符串长度

22. [(18条消息) 使用reserve来避免不必要的内存重新分配_小罗tongxue的博客-CSDN博客_reserve方法](https://blog.csdn.net/weixin_44843859/article/details/109403803?ops_request_misc=%7B%22request%5Fid%22%3A%22167093256216800186545580%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=167093256216800186545580&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-109403803-null-null.142^v68^js_top,201^v4^add_ask,213^v2^t3_esquery_v2&utm_term=reserve&spm=1018.2226.3001.4187)

23. resize()强迫容器把它的容量改变到包含n个元素的状态。在调用resize之后，size将返回n。如果n比当前的大小(size)要小，则容器尾部的元素将会被析构。如果n比当前的大小要大，则通过默认的构造函数创建的新元素将被添加到容器的尾部。如果n比当前的容量（capacity）要大，那么在添加元素之前，将先重新分配内存。reserve强迫容器把它的容量变为至少是n，前提是n不小于当前的大小。这通常会导致内存重新分配，因为容量需要增加。如果n比当前容量要小，则vector忽略该调用，什么也不做；而string则可能把自己的容量减为size和n中的最大值。

24. str1.replace(iterator_begin,iterator_end,"ppp"),把string容器中从iterator_begin到,iterator_end的字符替换为，常量字符串“ppp”

27. str1.front()，返回字符串第一个字符

28. getline,string提供的方法相当，读取一行内容，相当于gets

29. insert：插入操作，很多方法

    str1.insert(2, str2, 4, 6)从str1的下表为2的位置插入从str2下标为4开始截获长度为6的字串

30. get_allocator()，获取配置器