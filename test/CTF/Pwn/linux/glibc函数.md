#### write

`ssize_t write (int fd, const void * buf, size_t count);`

```
write()会把参数buf 所指的内存写入count 个字节到参数fd 所指的文件内. 当然, 文件读写位置也会随之移动。
write函数的特点在于其输出完全由其参数size决定，只要目标地址可读，size填多少就输出多少，不会受到诸如‘\0’, ‘\n’之类的字符影响。
fd = 1为标准输出，如果顺利write()会返回实际写入的字节数. 当有错误发生时则返回-1, 错误代码存入errno 中.
```

#### read

`ssize_t read (int fd, const void * buf, size_t count);`

```
read()会把fd所指的文件内的内容写入count 个字节到参数buf.
fd=0时，从标准输入读取
```
