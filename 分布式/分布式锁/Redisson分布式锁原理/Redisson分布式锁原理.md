Redisson分布式锁的使用非常简单，需要要很简单的几行代码即可搞定。

```java
RLock lock = redisson.getLock("lock");
//加锁
lock.lock();
//释放锁
lock.unlock();
```



下面从lock()和unlock()方法来看一看Redisson分布式锁的实现。



# 加锁

```java
    public RLock getLock(String name) {
        return new RedissonLock(connectionManager.getCommandExecutor(), name, id);
    }
```



通过lock方法一步步的往下追，可以发现最终Redisson是通过tryAcquireAsync方法来实现加锁的逻辑，它通过执行lua脚本来获取锁，其代码如下

```java
    <T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
        internalLockLeaseTime = unit.toMillis(leaseTime);

        return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
                  "if (redis.call('exists', KEYS[1]) == 0) then " +
                      "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                      "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                      "return nil; " +
                  "end; " +
                  "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                      "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                      "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                      "return nil; " +
                  "end; " +
                  "return redis.call('pttl', KEYS[1]);",
                    Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
    }
```



KEYS1[1] :  加锁key，对应Collections.singletonList(getName),也就是redisson.getLock("lock");中的参数“lock”。

ARGV[1] : key的过期时间，默认是30s.

ARGV[2] : 客户端的ID



下面对这段lua脚本做一些简单的介绍：

redis.call（）用来执行redis的命令，比如第一行的redis.call('exists', KEYS[1])，意思是对KEYS[1]执行exists命令。

1. 如果KEYS[1]不存在，执行hset命令，将KEYS[1]下的ARGV[2]的值设置为1
2. 设置KEYS[1]下的ARGV[2]的过期时间为30s
3. 如果KEYS[1]已存在，并且KEYS[1]下存在ARGV[2]，那么本次加锁操作就相当于一个锁重入的操作，于是对KEYS[1]下的ARGV[2]的值执行加1操作。
4. 设置KEYS[1]下的ARGV[2]的过期时间为30s
5. 返回KEYS[1]的剩余过期时间。
   

用来加锁的这段脚本有两种返回值类型：nil（null）和pttl（剩余过期时间），返回null代表着本次加锁成功，返回pttl表示本次加锁失败。



这里可能会比较疑惑为什么要用hash类型而不是string类型，原因很简单，string类型KV只能保存两个有效信息，无法同时保存锁key、客户端ID和锁重入次数，因此无法进行锁重入的判断。



加锁以后在Redis的存在形式如下：

```java
lock:{
    "6794c9d0-0805-4387-87cd-6a719d6b6278:1" : 1
}
```



# 锁互斥

上面对加锁操作进行了简单的解释，当客户端A执行了上述加锁操作以后，客户端A就持有了锁，那么当客户端B再来执行上述加锁操作时，首先在第一个if判断那里会发现要获取的锁已经被持有了，然后就进行到了第二个if判断，判断持有的该锁的客户端是不是客户端B，结果很显然，lock锁key的hash结构中并不包含客户端B的ID，因此客户端B加锁失败。

但是还没完，尽管加锁失败，但还是需要获取到lock锁的剩余过期时间，然后客户端B会根据这个过期时间进行循环判断，不停的尝试加锁。这里可能会有一个问题，当大量线程并发的获取锁时，可能会有大量的线程在这里进行自旋操作。



# 超时处理

客户端获取到锁以后，就可以执行它的业务逻辑了，可是如果业务执行时间比较长，锁的默认到期时间到了还是没有执行完该怎么办呢？

调大默认过期时间是一种解决办法，可是如果大部分的执行时间都很短，为了应对偶发性的执行超时，去调大过期时间，无疑是得不偿失的。

Redisson通过watch dog机制来自动延长过期时间:

```java
    private RFuture<Boolean> tryAcquireOnceAsync(long leaseTime, TimeUnit unit, final long threadId) {
        if (leaseTime != -1) {
            return tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_NULL_BOOLEAN);
        }
        RFuture<Boolean> ttlRemainingFuture = tryLockInnerAsync(commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_NULL_BOOLEAN);
        ttlRemainingFuture.addListener(new FutureListener<Boolean>() {
            @Override
            public void operationComplete(Future<Boolean> future) throws Exception {
                if (!future.isSuccess()) {
                    return;
                }

                Boolean ttlRemaining = future.getNow();
                // lock acquired
                if (ttlRemaining) {
                    scheduleExpirationRenewal(threadId);
                }
            }
        });
        return ttlRemainingFuture;
    }
```



上面已经说过了tryLockInnerAsync方法用来进行加锁操作，那么当加锁成功以后，每隔10s，就会检查一下当前客户端线程是否还持有这个锁，如果依然还持有这个锁，就会不断的延长key的过期时间。实现如下：

```java
    private void scheduleExpirationRenewal(final long threadId) {
        if (expirationRenewalMap.containsKey(getEntryName())) {
            return;
        }

        Timeout task = commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
            @Override
            public void run(Timeout timeout) throws Exception {
                
                RFuture<Boolean> future = commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
                        "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                            "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                            "return 1; " +
                        "end; " +
                        "return 0;",
                          Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
                
                future.addListener(new FutureListener<Boolean>() {
                    @Override
                    public void operationComplete(Future<Boolean> future) throws Exception {
                        expirationRenewalMap.remove(getEntryName());
                        if (!future.isSuccess()) {
                            log.error("Can't update lock " + getName() + " expiration", future.cause());
                            return;
                        }
                        
                        if (future.getNow()) {
                            // reschedule itself
                            scheduleExpirationRenewal(threadId);
                        }
                    }
                });
            }
        }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);

        if (expirationRenewalMap.putIfAbsent(getEntryName(), task) != null) {
            task.cancel();
        }
    }
```







# 释放锁

释放锁操作也是通过lua脚本来实现的，如下:

```java
    protected RFuture<Boolean> unlockInnerAsync(long threadId) {
        return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
                "if (redis.call('exists', KEYS[1]) == 0) then " +
                    "redis.call('publish', KEYS[2], ARGV[1]); " +
                    "return 1; " +
                "end;" +
                "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
                    "return nil;" +
                "end; " +
                "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
                "if (counter > 0) then " +
                    "redis.call('pexpire', KEYS[1], ARGV[2]); " +
                    "return 0; " +
                "else " +
                    "redis.call('del', KEYS[1]); " +
                    "redis.call('publish', KEYS[2], ARGV[1]); " +
                    "return 1; "+
                "end; " +
                "return nil;",
                Arrays.<Object>asList(getName(), getChannelName()), LockPubSub.unlockMessage, internalLockLeaseTime, getLockName(threadId));

    }
```

这段脚本的过程如下：

1. 如果锁KEYS[1]不存在，发送一条消息到频道KEYS[2]，内容是ARGV[1],然后返回1.
2. 如果锁KEYS[1]存在，但是KEYS[1]下不存在ARGV[3],说明现在有客户端线程持有锁，但是持有锁的客户端不是当前客户端，返回nil，也就是null.
3. 如果锁KEYS[1]存在并且KEYS[1]下存在ARGV[3]，说明有客户端线程持有锁，并且这个客户端就是当前客户端，这个时候要将ARGV[3]的值进行减1操作，如果减1后的值大于0，延长锁的到期时间，然后返回0，如果等于0，就把代表锁KEYS[1]的键删除，然后发送一条消息到频道KEYS[2]，内容是ARGV[1]，然后返回1.



再来看看释放锁操作的返回值有什么用，该返回值一共有三种情况：null、0、1

```java
    @Override
    public void unlock() {
        Boolean opStatus = get(unlockInnerAsync(Thread.currentThread().getId()));
        if (opStatus == null) {
            //返回值为null时，说明该客户端不是持有锁的客户端，因此不能释放锁
            throw new IllegalMonitorStateException("attempt to unlock lock, not locked by current thread by node id: "
                    + id + " thread-id: " + Thread.currentThread().getId());
        }
       
        if (opStatus) {
            cancelExpirationRenewal();
        }

//        Future<Void> future = unlockAsync();
//        future.awaitUninterruptibly();
//        if (future.isSuccess()) {
//            return;
//        }
//        if (future.cause() instanceof IllegalMonitorStateException) {
//            throw (IllegalMonitorStateException)future.cause();
//        }
//        throw commandExecutor.convertException(future);
    }

```



1. 当返回值为null时，说明该客户端不是持有锁的客户端，因此不能释放锁，这里抛出了异常。
2. 当返回值为0时，说明存在客户端锁重入的情况，本次释放锁的操作并没有将锁完全释放。
3. 当返回值为1时，说明已经将锁完全释放掉，这里执行了 cancelExpirationRenewal()方法。



```java
    void cancelExpirationRenewal() {
        Timeout task = expirationRenewalMap.remove(getEntryName());
        if (task != null) {
            task.cancel();
        }
    }
```

cancelExpirationRenewal方法的作用是将加锁时增加的watchdog监视器给取消掉。