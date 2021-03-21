## LockSupport

```java
public static void unpark(Thread thread) {
    if (thread != null)
        UNSAFE.unpark(thread);
}
public static void park(Object blocker) {
	Thread t = Thread.currentThread();
    setBlocker(t, blocker);
    UNSAFE.park(false, 0L);
    setBlocker(t, null);
}
private static void setBlocker(Thread t, Object arg) {
    // Even though volatile, hotspot doesn't need a write barrier here.
    UNSAFE.putObject(t, parkBlockerOffset, arg);
}
/**
 * The argument supplied to the current call to
 * java.util.concurrent.locks.LockSupport.park.
 * Set by (private) java.util.concurrent.locks.LockSupport.setBlocker
 * Accessed using java.util.concurrent.locks.LockSupport.getBlocker
 */
 volatile Object parkBlocker;
```