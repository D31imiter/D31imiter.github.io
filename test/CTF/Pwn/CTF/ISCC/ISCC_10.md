```
size_t __fastcall sub_400BCA(__int64 a1, size_t a2)
{
  size_t result; // rax
  int v3; // [rsp+1Ch] [rbp-14h]
  size_t i; // [rsp+20h] [rbp-10h]

  for ( i = 0LL; ; i += v3 )
  {
    result = i;
    if ( i >= a2 )
      break;
    v3 = read(0, (void *)(a1 + i), a2);
    if ( v3 <= 0 )
    {
      write(1, "error\n", 6uLL);
      sub_400AD6(1u);
    }
  }
  return result;
}
```

a1为chunk地址，a2为chunk大小，如果说输入的字符数正好为a2，则v3=v2，i=v2，break

如果说输入的字符数正好为a2-1，i=v2-1，又会执行v3 = read(0, (void *)(a1 + i), a2);

导致堆溢出
