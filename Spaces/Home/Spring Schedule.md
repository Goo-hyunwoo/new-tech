# @EnableScheduling

Spring Boot는 @EnableScheduling이라는 Annotation을 사용해서 주기 작업을 제공한다.
Annotation은 Spring Boot Starter Class 또는 Config에 작성한다.

### @Scheduled(...)
사용하고자 하는 메서드의 위에 선언하는 Annotation
...에는 스케줄링에 필요한 표현식이나 각종 스케줄링 주기에 대한 내용을 작성할 수 있다.

### 확인사항
Spring Boot의 스케줄링은 기본적으로 Thread Pool로 구현된다.
> org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler

특별한 설정을 해주지 않는다면, 스프링이 기본적으로 제공하는 Pool은 1개이다.
> 따라서 2개의 메소드에  @Sheduled 어노테이션을 사용하면 다른 스케줄러에 의해 방해받는다.

```java
@EnableScheduling
@Configuration
public class TaskSchedulerConfiguration implements SchedulingConfigurer {

  @Override
  public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
    ThreadPoolTaskScheduler threadPoolTaskScheduler = 
	    new ThreadPoolTaskScheduler();
    threadPoolTaskScheduler.setPoolSize(2);
    threadPoolTaskScheduler.setThreadNamePrefix("Batch-Scheduler-");
    threadPoolTaskScheduler.initialize();
    taskRegistrar.setScheduler(threadPoolTaskScheduler);
  }
}
```

![[Pasted image 20240409161415.png]]



