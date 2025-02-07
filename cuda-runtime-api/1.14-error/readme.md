# 知识点
1. 若cuda核函数出错，由于他是异步的，立即执行cudaPeekAtLastError只会拿到对输入参数校验是否正确的状态，而不会拿到核函数是否执行正确的状态
2. 因此需要等待核函数执行完毕后，才真的知道当前核函数是否出错，一般通过设备同步或者流同步进行等待
3. 错误分为可恢复和不可恢复两种：
    - 可恢复：
        - 参数配置错误等，例如block越界（一般最大值是1024），shared memory大小超出范围（一般是48KB）
        - 通过cudaGetLastError可以获取错误代码，同时把当前状态恢复为success
        - 该错误在调用核函数后可以立即通过cudaGetLastError/cudaPeekAtLastError拿到
        - 该错误在下一个函数调用的时候会覆盖
    - 不可恢复：
        - 核函数执行错误，比如说会有访问越界等等异常
        - 该错误则会传递到之后的所有cuda操作上
        - 错误状态通常需要等到核函数执行完毕才能够拿到，也就是有可能在后续的任何流程中突然异常（因为是异步的）
        