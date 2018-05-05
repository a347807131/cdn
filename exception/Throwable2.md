# Throwable类解读

## 相关方法解读

### initCause(构造异常链)

```java
public synchronized Throwable initCause(Throwable cause) {

    //一旦通过初始化或者initCause设置了cause，就不能再次设置了  
    if (this.cause != this)
        throw new IllegalStateException("Can't overwrite cause with " +Objects.toString(cause, "a null"), this);
        
    //一旦通过初始化或者initCause设置了cause，就不能再次设置了  
    if (cause == this)
        throw new IllegalArgumentException("Self-causation not permitted", this);
    this.cause = cause;
    return this;
}
```
疑惑1：条件this.cause==this && cause!=this的意义？

疑惑2: 为什么会用sysnchronized关键字？

### printStackTrace(打印异常栈信息)
```java
//先打印thia，再打印方法调用的栈轨迹，最后打印cause 
private void printStackTrace(PrintStreamOrWriter s) {
    // Guard against malicious overrides of Throwable.equals by
    // using a Set with identity equality semantics.
    Set<Throwable> dejaVu = Collections.newSetFromMap(new IdentityHashMap<>());
    dejaVu.add(this);
    
    //对输出进行加锁
    synchronized (s.lock()) {
        // Print our stack trace
        s.println(this);
        //获取在初始化异常栈时生成的异常栈信息
        StackTraceElement[] trace = getOurStackTrace();
        迭代打印出异常栈信息
        for (StackTraceElement traceElement : trace)
            s.println("\tat " + traceElement);
    
        //迭代打印出被抑制的异常
        for (Throwable se : getSuppressed())
            se.printEnclosedStackTrace(s, trace, SUPPRESSED_CAPTION, "\t", dejaVu);
    
        // Print cause, if any
        Throwable ourCause = getCause();
        if (ourCause != null)
            //将栈跟踪信息作为一个指定栈跟踪信息的封闭异常打印出来
            ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, "", dejaVu);
    }
} 
```
