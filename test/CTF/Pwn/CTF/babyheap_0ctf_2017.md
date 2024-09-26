1. checksec

   ```
   ┌──(root㉿kali)-[/home/kali/Desktop]
   └─# checksec babyheap_0ctf_2017 
   [*] '/home/kali/Desktop/babyheap_0ctf_2017'
       Arch:     amd64-64-little
       RELRO:    Full RELRO
       Stack:    Canary found
       NX:       NX enabled
       PIE:      PIE enabled
   ```

   保护全开
2. 尝试运行，一个菜单，拥有四个功能，Allocate，Fill，Free，Dump，IDA静态分析

   * 主函数

     ```
     __int64 __fastcall main(__int64 a1, char **a2, char **a3)
     {
       __int64 v4; // [rsp+8h] [rbp-8h]

       v4 = sub_B70(a1, a2, a3);
       while ( 1 )
       {
         sub_CF4();
         switch ( sub_138C() )
         {
           case 1LL:
             sub_D48(v4);
             break;
           case 2LL:
             sub_E7F(v4);
             break;
           case 3LL:
             sub_F50(v4);
             break;
           case 4LL:
             sub_1051(v4);
             break;
           case 5LL:
             return 0LL;
           default:
             continue;
         }
       }
     }
     ```

     v4即结构体存放位置处
   * Allocate

     ```
     void __fastcall sub_D48(__int64 a1)
     {
       int i; // [rsp+10h] [rbp-10h]
       int v2; // [rsp+14h] [rbp-Ch]
       void *v3; // [rsp+18h] [rbp-8h]

       for ( i = 0; i <= 15; ++i )
       {
         if ( !*(_DWORD *)(24LL * i + a1) )
         {
           printf("Size: ");
           v2 = sub_138C();
           if ( v2 > 0 )
           {
             if ( v2 > 4096 )
               v2 = 4096;
             v3 = calloc(v2, 1uLL);
             if ( !v3 )
               exit(-1);
             *(_DWORD *)(24LL * i + a1) = 1;
             *(_QWORD *)(a1 + 24LL * i + 8) = v2;
             *(_QWORD *)(a1 + 24LL * i + 16) = v3;
             printf("Allocate Index %d\n", (unsigned int)i);
           }
           return;
         }
       }
     }
     ```

     size最多分配4096字节，该函数作用为，将对应下标的结构体的第一个元素标记为1（使用），第二个元素为chunk_size,第三个元素为用calloc分配返回的chunk的地址，将这些空间初始化为0
   * Fill

     ```
     __int64 __fastcall sub_E7F(__int64 a1)
     {
       __int64 result; // rax
       int v2; // [rsp+18h] [rbp-8h]
       int v3; // [rsp+1Ch] [rbp-4h]

       printf("Index: ");
       result = sub_138C();
       v2 = result;
       if ( (unsigned int)result <= 0xF )
       {
         result = *(unsigned int *)(24LL * (int)result + a1);
         if ( (_DWORD)result == 1 )
         {
           printf("Size: ");
           result = sub_138C();
           v3 = result;
           if ( (int)result > 0 )
           {
             printf("Content: ");
             return sub_11B2(*(_QWORD *)(24LL * v2 + a1 + 16), v3);
           }
         }
       }
       return result;
     }
     ```

     最多分配15个chunk，如果index<15，则继续处理。先获取对应结构体的第一个元素，如果为1则已经分配，继续处理。继续输入size，调用sub_11B2(chunk_addr, size)，再继续跟进就是自己实现的一个read。但是此步没有将输入的size和结构体中的元素进行比较就赋值，导致了堆溢出
   * Free
     所做的事就是将第结构体各个元素赋值为0，在第三个元素=0前释放chunk
   * Dump输入对应堆块内容
3. 堆利用思路，由于PIE保护开启，所以需要泄露libc基址
