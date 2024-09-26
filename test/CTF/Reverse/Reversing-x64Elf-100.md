### Reversing-x64Elf-100

1. ida分析

   ~~~ c
   __int64 __fastcall main(int a1, char **a2, char **a3)
   {
     char s[264]; // [rsp+0h] [rbp-110h] BYREF
     unsigned __int64 v5; // [rsp+108h] [rbp-8h]
   
     v5 = __readfsqword(0x28u);
     printf("Enter the password: ");
     if ( !fgets(s, 255, stdin) )
       return 0LL;
     if ( (unsigned int)sub_4006FD((__int64)s) )
     {
       puts("Incorrect password!");
       return 1LL;
     }
     else
     {
       puts("Nice!");
       return 0LL;
     }
   }
   ~~~

   sub_4006FD函数

   ~~~ c
   __int64 __fastcall sub_4006FD(__int64 a1)
   {
     int i; // [rsp+14h] [rbp-24h]
     __int64 v3[4]; // [rsp+18h] [rbp-20h]
   
     v3[0] = (__int64)"Dufhbmf";
     v3[1] = (__int64)"pG`imos";
     v3[2] = (__int64)"ewUglpt";
     for ( i = 0; i <= 11; ++i )
     {
       if ( *(char *)(v3[i % 3] + 2 * (i / 3)) - *(char *)(i + a1) != 1 )
         return 1LL;
     }
     return 0LL;
   }
   ~~~

   让`*(char *)(i + a1)+1==*(char *)(v3[i % 3] + 2 * (i / 3))`即可，也就是说

   `*(char *)(v3[i % 3] + 2 * (i / 3))-1==[a1+i]`

2. 把`*(char *)(v3[i % 3] + 2 * (i / 3))-1`打印出来即可

3. ~~~ c
   int main()
   {
   	int i = 0;
   	__int64 v3[4]; // [rsp+18h] [rbp-20h]
   
   	v3[0] = (__int64)"Dufhbmf";
   	v3[1] = (__int64)"pG`imos";
   	v3[2] = (__int64)"ewUglpt";
   	for (i = 0; i <= 11; ++i)
   	{
   		printf("%c", *(char*)(v3[i % 3] + 2 * (i / 3)) - 1);
   	}
   	return 0;
   }
   ~~~

   Code_Talkers