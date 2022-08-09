pthread_mutex_timedlock()

当线程试图获取一个已加锁的互斥量时，pthread_mutex_timedlock互斥量原语允许绑定线程阻塞时间。pthread_mutex_timedlock()和pthread_mutex_lock()是等价的，但在到达超时时间时，pthread_mutex_timedlock不会对互斥量进行加锁，而是返回错误码ETIMEDOUT。

```c
#include <pthread.h>
#include <time.h>

int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex, const struct timespec *restrict tsptr);
//成功返回0，失败错误码
```

超过指定愿意等待的绝对时间（与相对时间对比而言，指定在时间X之前可以阻塞等待，而不是说愿意阻塞Y秒）。这个超时时间使用timespec结构体表示的，用秒和纳秒来描述时间

示例：

```c
struct timeval now;
struct timespec timeout;

pthread_mutex_lock(&mutex);
timeout.tv_sec = now.tv_sec + 5;
timeout.tv_nsec = (now.tv_usec + 3 * 1000) * 1000;
if (ETIMEOUT == pthread_mutex_timedlock(&cond, &mutex, &timeout)
{
	printf("timeout!\n");
}
pthread_mutex_unlock(&mutex);
```



书上示例

```
#include "apue.h"
#include <pthread.h>

int main()
{
	int err;
	struct timespec tout;
	struct tm *tmp;
	char buf[64];
	pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
	
	pthread_mutex_lock(&lock);
	printf("mutex is locked\n");
	clock_gettime(CLOCK_REALTIME, &tout);
	tmp = localtime(&tout.tv_sec);
	strftime(buf, sizeof(buf), "%r", tmp);
	printf("current time");
}
```





