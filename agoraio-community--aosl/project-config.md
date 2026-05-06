---
trigger: always_on
description: **Description:** AOSL is a cross-platform low-level general-purpose component library providing abstract implementations for threads, IO, logging, memory, networking, message queues, and more.
---

# AOSL (Advanced Operating System Layer) SDK

**Version:** 1.0.0  
**Description:** AOSL is a cross-platform low-level general-purpose component library providing abstract implementations for threads, IO, logging, memory, networking, message queues, and more.

---

## Module Index

| Module | Header File | Description |
|--------|-------------|-------------|
| Initialization | aosl.h | Library initialization and cleanup |
| Atomic Operations | aosl_atomic.h | Atomic variable operations and memory barriers |
| Memory Management | aosl_mm.h | Memory allocation and statistics |
| Logging System | aosl_log.h | Leveled log output |
| Time Management | aosl_time.h | Timestamp retrieval and sleep |
| Thread Synchronization | aosl_thread.h | Locks, condition variables, events |
| Multiplex Queue | aosl_mpq.h | Message queue core |
| MPQ Pool | aosl_mpqp.h | Thread pool |
| MPQ Timer | aosl_mpq_timer.h | Timers |
| MPQ File Descriptor | aosl_mpq_fd.h | IO event monitoring |
| MPQ Network | aosl_mpq_net.h | Socket networking |
| Reference Object | aosl_ref.h | Reference counting and read-write locks |
| Linked List | aosl_list.h | Doubly linked list |
| Red-Black Tree | aosl_rbtree.h | Red-black tree |
| Packet Buffer | aosl_psb.h | Network packet buffering |
| Socket | aosl_socket.h | Socket address and byte order |

---

## 1. Initialization and Cleanup

```c
void aosl_ctor(void);           // Initialize AOSL library (must be called first)
void aosl_dtor(void);           // Cleanup AOSL library (call before program exit)
```

---

## 2. Atomic Operations (aosl_atomic.h)

### Basic Operations
```c
int  aosl_atomic_read(const aosl_atomic_t *v);           // Atomic read
void aosl_atomic_set(aosl_atomic_t *v, int i);            // Atomic set
void aosl_atomic_inc(aosl_atomic_t *v);                    // Atomic increment
void aosl_atomic_dec(aosl_atomic_t *v);                    // Atomic decrement
```

### Arithmetic Operations
```c
int  aosl_atomic_add_return(int i, aosl_atomic_t *v);      // Add and return result
int  aosl_atomic_sub_return(int i, aosl_atomic_t *v);      // Subtract and return result
int  aosl_atomic_inc_and_test(aosl_atomic_t *v);           // Increment and test if zero
int  aosl_atomic_dec_and_test(aosl_atomic_t *v);           // Decrement and test if zero
```

### Exchange Operations
```c
int  aosl_atomic_cmpxchg(aosl_atomic_t *v, int oldval, int newval);  // Compare and swap (CAS)
int  aosl_atomic_xchg(aosl_atomic_t *v, int newval);                   // Exchange
```

### Memory Barriers
```c
void aosl_mb(void);    // Full memory barrier
void aosl_rmb(void);   // Read memory barrier
void aosl_wmb(void);   // Write memory barrier
```

---

## 3. Memory Management (aosl_mm.h)

### Memory Allocation
```c
void *aosl_malloc(size_t size);              // Allocate memory
void *aosl_calloc(size_t nmemb, size_t size); // Allocate and zero-initialize
void *aosl_realloc(void *ptr, size_t size);   // Reallocate
char *aosl_strdup(const char *s);             // Duplicate string
void  aosl_free(void *ptr);                   // Free memory
```

### Memory Statistics
```c
size_t aosl_memused(void);       // Get used memory size
void   aosl_memdump(void);        // Print memory allocation info (debug)
```

---

## 4. Logging System (aosl_log.h)

### Log Levels
```c
AOSL_LOG_EMERG    (0)  // System is unusable
AOSL_LOG_ALERT    (1)  // Action must be taken immediately
AOSL_LOG_CRIT     (2)  // Critical conditions
AOSL_LOG_ERROR    (3)  // Error
AOSL_LOG_WARNING  (4)  // Warning
AOSL_LOG_NOTICE   (5)  // Notice
AOSL_LOG_INFO     (6)  // Informational
AOSL_LOG_DEBUG    (7)  // Debug
```

### Log Functions
```c
void aosl_set_vlog_func(aosl_vlog_t vlog);  // Set custom log function
int  aosl_get_log_level(void);               // Get current log level
void aosl_set_log_level(int level);          // Set log level
void aosl_log(int level, const char *fmt, ...);  // Output log message
```

### Convenience Macros
```c
AOSL_LOG_DBG(fmt, ...)  // Debug log
AOSL_LOG_INF(fmt, ...)  // Info log
AOSL_LOG_WRN(fmt, ...)  // Warning log
AOSL_LOG_ERR(fmt, ...)  // Error log
```

---

## 5. Time Management (aosl_time.h)

### Get Time
```c
aosl_ts_t aosl_tick_now(void);    // Get tick count
aosl_ts_t aosl_tick_ms(void);     // Get millisecond timestamp
aosl_ts_t aosl_tick_us(void);     // Get microsecond timestamp
aosl_ts_t aosl_time_sec(void);    // Get Unix timestamp in seconds
aosl_ts_t aosl_time_ms(void);     // Get Unix timestamp in milliseconds
```

### Sleep
```c
void aosl_msleep(uint64_t ms);    // Sleep for specified milliseconds
```

---

## 6. Thread Synchronization (aosl_thread.h)

### Thread Local Storage (TLS)
```c
int  aosl_tls_key_create(aosl_tls_key_t *key);   // Create TLS key
void *aosl_tls_key_get(aosl_tls_key_t key);       // Get TLS value
int  aosl_tls_key_set(aosl_tls_key_t key, void *value);  // Set TLS value
int  aosl_tls_key_delete(aosl_tls_key_t key);     // Delete TLS key
```

### Mutex Lock
```c
aosl_lock_t aosl_lock_create(void);        // Create lock
void        aosl_lock_lock(aosl_lock_t lock);      // Lock (blocking)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgoraIO-Community/aosl](https://github.com/AgoraIO-Community/aosl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
