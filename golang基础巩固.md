# Map



# Slice



# Channal

## 常见问题

### fatal error: all goroutines are asleep - deadlock!

```
fatal error: all goroutines are asleep - deadlock!
```

- 情况1

  channal**未初始化分配空间**

  ```shell
  goroutine 1 [chan receive (nil chan)]:
  main.main()
  	C:/代码/Lesson/goSourceLearning/channelLearning/循环打印/循环打印123/channal/main.go:15 +0xb7
  ```

  分配空间即可

  ```go
  chan1 := make(chan int, 1)
  ```

- 情况2

  channal**消息满，却收新消息**

  ```
  goroutine 1 [chan send]:
  main.main()
  	C:/代码/Lesson/goSourceLearning/channelLearning/循环打印/循环打印123/channal/main.go:19 +0x7b
  
  ```

  分配足够的缓冲空间

  ```go
  chan1 := make(chan int, 3)
  ```

- 情况3

  channal**消息不足，却发送消息**

  ```
  goroutine 1 [chan receive]:
  main.main()
  	C:/代码/Lesson/goSourceLearning/channelLearning/循环打印/循环打印ABC/channal/main.go:19 +0x17f
  ```

  发送消息到channal

  ```go
  chanA <- struct{}{}
  ```

  