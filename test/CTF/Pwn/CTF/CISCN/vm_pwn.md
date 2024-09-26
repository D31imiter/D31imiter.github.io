```
__int64 __fastcall vm_id::run(vm_id *this, vm *a2)
{
  char *v2; // rax
  char *v3; // rax
  int v4; // eax
  char *v5; // rax
  char *v6; // rax
  int v7; // eax
  char *v8; // rax
  char v10; // [rsp+18h] [rbp-18h]
  char v11; // [rsp+19h] [rbp-17h]
  char v12; // [rsp+1Ah] [rbp-16h]
  char v13; // [rsp+1Ah] [rbp-16h]
  char v14; // [rsp+1Ah] [rbp-16h]
  char v15; // [rsp+1Ah] [rbp-16h]
  char v16; // [rsp+1Ah] [rbp-16h]
  char v17; // [rsp+1Bh] [rbp-15h]
  unsigned int v18; // [rsp+1Ch] [rbp-14h]
  _BYTE *v19; // [rsp+20h] [rbp-10h]
  char *v20; // [rsp+20h] [rbp-10h]
  char *v21; // [rsp+20h] [rbp-10h]

  v2 = (char *)*(vm[6]+vm[5]);	// v2 = *(vm[6]+vm[5])，read输入的地方的+vm[5]偏移
  v19 = (char *)*(vm[6]+vm[5]) + 1;
  v10 = *((char *)*(vm[6]+vm[5]));
  v18 = 1;
  if ( *v2 <= 0 || v10 > 8 )
  {
    if ( v10 <= 8 || v10 > 10 )
    {
      if ( v10 && v10 != 11 )
      {
        *((_QWORD *)this + 1) = -1LL;
      }
      else
      {
        *((_QWORD *)this + 1) = v10;
        *((_QWORD *)this + 2) = 0LL;
        *((_QWORD *)this + 3) = 0LL;
        *((_QWORD *)this + 4) = 0LL;
      }
    }
    else
    {
      v8 = v2 + 1;
      v21 = v19 + 1;
      v17 = *v8;
      v18 = 2;
      *((_QWORD *)this + 2) = *v8;
      if ( (v17 & 3) == 2 )
      {
        v18 = 3;
        v16 = *v21;
        if ( (unsigned int)vm_id::check_regs(this, *v21, a2) )
        {
          *((_QWORD *)this + 1) = v10;
          *((_QWORD *)this + 3) = v16;
          *((_QWORD *)this + 4) = 0LL;
        }
        else
        {
          *((_QWORD *)this + 1) = -1LL;
        }
      }
      else
      {
        *((_QWORD *)this + 1) = -1LL;
      }
      if ( (*((_QWORD *)a2 + 4) & 7LL) != 0 )
        *((_QWORD *)this + 1) = -1LL;
      if ( v10 == 9 )
      {
        if ( *((_QWORD *)a2 + 4) >= *((_QWORD *)a2 + 11) || *((_QWORD *)a2 + 4) <= 7uLL )
          *((_QWORD *)this + 1) = -1LL;
      }
      else if ( (unsigned __int64)(*((_QWORD *)a2 + 11) - 8LL) < *((_QWORD *)a2 + 4) )
      {
        *((_QWORD *)this + 1) = -1LL;
      }
    }
  }
  else
  {
    v3 = v2 + 1;
    v20 = v19 + 1;
    v11 = *v3;
    v18 = 2;
    *((_QWORD *)this + 2) = *v3;
    v4 = v11 & 3;
    if ( v4 == 2 )
    {
      v18 = 3;
      v5 = v20++;
      v12 = *v5;
      if ( (unsigned int)vm_id::check_regs(this, *v5, a2) )
      {
        *((_QWORD *)this + 1) = v10;
        *((_QWORD *)this + 3) = v12;
      }
      else
      {
        *((_QWORD *)this + 1) = -1LL;
      }
    }
    else if ( v4 == 3 )
    {
      v18 = 3;
      v6 = v20++;
      v13 = *v6;
      if ( (unsigned int)vm_id::check_addr(this, *((_QWORD *)a2 + *v6), a2) )
      {
        *((_QWORD *)this + 1) = v10;
        *((_QWORD *)this + 3) = v13;
      }
      else
      {
        *((_QWORD *)this + 1) = -1LL;
      }
    }
    else
    {
      *((_QWORD *)this + 1) = -1LL;
    }
    if ( *((_QWORD *)this + 1) != -1LL )
    {
      v7 = (v11 >> 2) & 3;
      if ( v7 == 3 )
      {
        ++v18;
        v15 = *v20;
        if ( (unsigned int)vm_id::check_addr(this, *((_QWORD *)a2 + *v20), a2) )
          *((_QWORD *)this + 4) = v15;
        else
          *((_QWORD *)this + 1) = -1LL;
      }
      else
      {
        if ( ((v11 >> 2) & 3u) > 3 )
        {
LABEL_25:
          *((_QWORD *)this + 1) = -1LL;
          goto LABEL_45;
        }
        if ( v7 == 1 )
        {
          v18 += 8;
          *((_QWORD *)this + 4) = *(_QWORD *)v20;
        }
        else
        {
          if ( v7 != 2 )
            goto LABEL_25;
          ++v18;
          v14 = *v20;
          if ( (unsigned int)vm_id::check_regs(this, *v20, a2) )
            *((_QWORD *)this + 4) = v14;
          else
            *((_QWORD *)this + 1) = -1LL;
        }
      }
    }
  }
LABEL_45:
  *(_DWORD *)this = 1;
  return v18;
}
```
