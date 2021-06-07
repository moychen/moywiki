<img src="images/runtime/image-20210521164237232.png" alt="image-20210521164237232" style="zoom:67%;" />

![image-20210521164456481](images/runtime/image-20210521164456481.png)

```
//go:nosplit
//go:noscape
//go:linkname timeSleep time.Sleep
//go:nowritebarrierrec
```

