- redisServer
```s
struct redisServer {
    // ...
    sds aof_buf;      /* AOF buffer, written before entering the event loop */
    // ...
}    
```
