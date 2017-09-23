http包使用总结

## type Request

### 

## type Response 

### 注意点1: Body 字段
Body 字段类型为 io.ReadCloser，Body字段无论任何情况，都不会为空，哪怕相应报文没有Body，或者相应报文有长度为0的body。

注：调用者要负责在读取body体之后，对body体进行关闭。

