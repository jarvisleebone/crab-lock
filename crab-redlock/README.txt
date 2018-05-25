获取锁：使用redis，Redlock算法
    在每个redis master上轮流获取锁
    SET lock_key request_sign NX PX 10000
        lock_key： 锁key
        request_sign：请求签名，保证所有请求唯一
        NX：当lock_key不存在时才写入
        PX：超时时间（毫秒） # 该时间为预留给业务处理的时间

    在单个master获取锁的超时时间需要远小于锁过期时间（为了给客户端的业务处理留出足够时间）
    例：
    锁过期时间：10秒
    单台master获取锁超时时间：5-50毫秒

    计算总的锁获取时间（在所有master获取锁花费的时间总和）
    锁超时时间 - 锁获取时间 = 留给业务处理的时间

    if 锁获取时间 < 锁超时时间 and 成功获取锁的master数量 >= N/2+1: 锁获取成功
    else: 锁获取失败

释放锁：
    在每个redis master上轮流释放锁
    request_sign = GET lock_key
    if request_id == request_sign: DEL lock_key