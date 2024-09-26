```
struct cred {
    atomic_t    usage;
#ifdef CONFIG_DEBUG_CREDENTIALS
    atomic_t    subscribers;    /* number of processes subscribed */
    void        *put_addr;
    unsigned    magic;
#define CRED_MAGIC  0x43736564
#define CRED_MAGIC_DEAD 0x44656144
#endif
    kuid_t      uid;        /* real UID of the task */
    kgid_t      gid;        /* real GID of the task */
    kuid_t      suid;       /* saved UID of the task */
    kgid_t      sgid;       /* saved GID of the task */
    kuid_t      euid;       /* effective UID of the task */
    kgid_t      egid;       /* effective GID of the task */
    kuid_t      fsuid;      /* UID for VFS ops */
    kgid_t      fsgid;      /* GID for VFS ops */
    unsigned    securebits; /* SUID-less security management */
    kernel_cap_t    cap_inheritable; /* caps our children can inherit */
    kernel_cap_t    cap_permitted;  /* caps we're permitted */
    kernel_cap_t    cap_effective;  /* caps we can actually use */
    kernel_cap_t    cap_bset;   /* capability bounding set */
    kernel_cap_t    cap_ambient;    /* Ambient capability set */
#ifdef CONFIG_KEYS
    unsigned char   jit_keyring;    /* default keyring to attach requested
                     * keys to */
    struct key __rcu *session_keyring; /* keyring inherited over fork */
    struct key  *process_keyring; /* keyring private to this process */
    struct key  *thread_keyring; /* keyring private to this thread */
    struct key  *request_key_auth; /* assumed request_key authority */
#endif
#ifdef CONFIG_SECURITY
    void        *security;  /* subjective LSM security */
#endif
    struct user_struct *user;   /* real user ID subscription */
    struct user_namespace *user_ns; /* user_ns the caps and keyrings are relative to. */
    struct group_info *group_info;  /* supplementary groups for euid/fsgid */
    struct rcu_head rcu;        /* RCU deletion hook */
} __randomize_layout;
```


* `usage`：原子计数器，用于跟踪对 `struct cred` 的引用计数。
* `subscribers`（仅在启用了 `CONFIG_DEBUG_CREDENTIALS` 时可用）：订阅此凭据的进程数量。
* `put_addr`（仅在启用了 `CONFIG_DEBUG_CREDENTIALS` 时可用）：最后一次释放此凭据的进程的地址。
* `magic`（仅在启用了 `CONFIG_DEBUG_CREDENTIALS` 时可用）：标识此凭据的魔术数。
* `uid`：进程的实际用户标识（real UID）。
* `gid`：进程的实际组标识（real GID）。
* `suid`：进程的保存的用户标识（saved UID）。
* `sgid`：进程的保存的组标识（saved GID）。
* `euid`：进程的有效用户标识（effective UID）。
* `egid`：进程的有效组标识（effective GID）。
* `fsuid`：用于 VFS（Virtual Filesystem）操作的用户标识。
* `fsgid`：用于 VFS 操作的组标识。
* `securebits`：用于管理无 SUID 权限的安全位（security bits）。
* `cap_inheritable`：子进程可以继承的能力集。
* `cap_permitted`：进程允许拥有的能力集。
* `cap_effective`：进程实际可以使用的能力集。
* `cap_bset`：能力的边界集（capability bounding set）。
* `cap_ambient`：环境能力集（ambient capability set）。
* `jit_keyring`（仅在启用了 `CONFIG_KEYS` 时可用）：默认的密钥环，用于附加请求的密钥。
* `session_keyring`（仅在启用了 `CONFIG_KEYS` 时可用）：继承而来的密钥环。
* `process_keyring`（仅在启用了 `CONFIG_KEYS` 时可用）：进程私有的密钥环。
* `thread_keyring`（仅在启用了 `CONFIG_KEYS` 时可用）：线程私有的密钥环。
* `request_key_auth`（仅在启用了 `CONFIG_KEYS` 时可用）：用于请求密钥的授权。
* `security`（仅在启用了 `CONFIG_SECURITY` 时可用）：主观 LSM（Linux Security Module）安全数据。
* `user`：真实用户 ID 订阅。
* `user_ns`：与能力和密钥环相关的用户命名空间。
* `group_info`：进程的附加组信息。
* `rcu`：RCU（Read-Copy Update）删除钩子。
