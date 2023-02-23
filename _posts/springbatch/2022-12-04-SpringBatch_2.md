---
title:  "스프링 배치 초기화 과정"
categories:
- spring batch

toc: true
toc_sticky: true
toc_label: "Contents"
---
## @EnableBatchProcessing
- @EnableBatchProcessing를 추가하면 스프링 배치가 작동하기 위한 4개의 설정 클래스를
실행시키며 스프링 배치의 초기화 과정 및 구성이 이루어진다. 

## 스프링 배치 초기화 설정 클래스 

1. BatchAutoConfiguration
   - 스프링 배치가 초기화 될 때 자동으로 실행되는 설정 클래스
   - Job을 수행하는 JobLauncherApplicationRunner 빈을 생성한다.
   
```java
//  BatchAutoConfiguration에서 JobLauncherApplicationRunner만 생성하는 부분을 발췌
public class BatchAutoConfiguration {
    public JobLauncherApplicationRunner jobLauncherApplicationRunner(JobLauncher jobLauncher, JobExplorer jobExplorer, JobRepository jobRepository, BatchProperties properties) {
        JobLauncherApplicationRunner runner = new JobLauncherApplicationRunner(jobLauncher, jobExplorer, jobRepository);
        String jobNames = properties.getJob().getNames();
        if (StringUtils.hasText(jobNames)) {
            runner.setJobNames(jobNames);
        }

        return runner;
    }
}
```
---

2. SimpleBatchConfiguration
   - JobBuilderFactory와 StepBuilderFactory 생성 
   - 스프링 배치의 주요 구성 요소를 생성 - 프록시 객체로 생성된다. 

3. BatchConfigurerConfiguration
   1. BasicBatchConfigurer
      - SimpleBatchConfiguration에서 생성한 프록시 객체의 실대 대상 객체를 생성하는 클래스
   2. JpaBatchConfigurer
      - JPA와 관련된 객체를 생성하는 설정 클래스다.

---

## 스프링 배치 시작하기

```java
@Configuration
@RequiredArgsConstructor
public class HelloJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job helloJob() {
        return jobBuilderFactory.get("helloJob")
                .start(helloStep())
                .next(helloStep2())
                .build();
    }

    @Bean
    public Step helloStep() {
        return stepBuilderFactory.get("hellpStep1")
                .tasklet(((stepContribution, chunkContext) -> {
                    System.out.println("===========================");
                    System.out.println(" >> Hello Spring Batch step 1");
                    System.out.println("===========================");
                    return RepeatStatus.FINISHED;
                }))
                .build();
    }


    @Bean
    public Step helloStep2() {
        return stepBuilderFactory.get("hellpStep2")
                .tasklet(((stepContribution, chunkContext) -> {
                    System.out.println("===========================");
                    System.out.println(" >> Hello Spring Batch step 2");
                    System.out.println("===========================");
                    return RepeatStatus.FINISHED;
                }))
                .build();
    }
}

```

- 스프링 배치에 필요한 Job, Step 등을 빈으로 등록한다ㅏ.
- JobBuilderFactory
  - Job을 생성하는 빌더 팩토리
- StepBuilderFactory
  - Step을 생성하는 빌더 팩토리

> 위의 코드는 JobBuilderFactory와 StepBuilderFactory를 활용해 
> 배치 작업에 필요한 Job, Step 등을 생성해 빈으로 등록한다.

<img src="/public/img/1204_batch_stream.png" width="500" object-fit="cover" alt="" />