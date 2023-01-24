---
title:  "스프링 배치 Job"
categories: spring-batch
toc: true
toc_sticky: true
toc_label: "Contents"
---

## Job
- Job은 배치 계층 구조에서 가장 상위에 있는 개념으로 하나의 Job은 하나의 배치작업을 의미한다. (ex: 일일 정산로그수집)
- 배치 Job은 최상위 인터페이스로 스프링 배치는 다양한 기본 구현체를 제공한다.
- Job은 하나 이상의 Step으로 구성해야 한다.

### Job의 기본 구현체

<img src="/public/img/1205_springbatch_job_instance.png" width="900" object-fit="cover" alt="" />

- <span style='background-color:#fff5b1'>SimpleJob</span>
  - 순차적으로 Step을 실행시키는 Job

- <span style='background-color:#fff5b1'>FlowJob</span>
  - 특정한 조건과 흐름에 따라 Step을 실행시키는 Job

---

<img src="/public/img/1205_springbatch_simplejob.png" width="900" object-fit="cover" alt="" />

> 위의 SimpleJob의 계층구조를 보면 AbstratJob을 상속받고 있다.
> 또, AbstractJob은 최상위 인터페이스인 Job을 구현하고 있다.   

### AbstractJob

<img src="/public/img/1205_abstractjob.png" width="900" object-fit="cover" alt="" />

- name : Job 이름
- restartable : 재시작 여부(default=true)
- JobRepository : 메타데이터 저장소
- JobExecutionListener : Job 이벤트 리스너
- JobParametersIncrement : JobParameter 증가
- JobParametersValidator : JobParameter 검증기
- SimpleStepHandler : Step 실행 핸들러

### Job 구성하기

```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JobConfiguration {

    /**
     * 스프링 배치는 Job과 Step을 생성할 수 있는 빌더 팩터리를 제공하는데,
     * 런타임 시점에 JobBuilderfactory, StepBuilderFactory를 빈으로 등록해준다.
     */
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;


    /**
     * "job"이라는 이름을 갖는 Job을 생성하며
     * start()와 next()를 호출해서 두 개의 Step을 등록한다.
     *
     * jobBuilderFactory의 build()를 호출하면 Job 인스턴스를 리턴한다.
     */
    @Bean
    public Job job() {
        return jobBuilderFactory.get("job")
                .start(step1())
                .next(step2())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet((stepContribution, chunkContext) -> {
                    System.out.println("step1 was executed");
                    return RepeatStatus.FINISHED;
                })
                .build();

    }

    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet((stepContribution, chunkContext) -> {
                    System.out.println("step2 was executed");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

---

## JobInstance

- Job이 실행될 때 생성되는 Job의 논리적 실행 단위 객체로서 고유하게 식별 가능한 작업을 나타낸다.
- Job은 실행되는 시점에 따라 처리하는 내용이 따르기 떄문에 Job의 실행을 구분해야 하기 위해서 JobInstance를 사용한다.

### JobInstance의 생성과 실행
> JobInstance의 생성과 실행에 대해 알아보기 전에 JobLauncher를 알아야 한다. 
> **<span style="color:red;">JobLauncher</span>**는 Job을 실행하는 역할을 하는데 이때 Job과 JobParameters를 인자로 넘겨야 한다.

```java
@Component
public class JobParameterTest implements ApplicationRunner {

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private Job job;

    @Override
    public void run(ApplicationArguments args) throws Exception {

        JobParameters jobParameters = new JobParametersBuilder()
                .addString("name", "user1")
                .addLong("seq", 2L)
                .addDate("date", new Date())
                .addDouble("age", 27.0)
                .toJobParameters();

        jobLauncher.run(job, jobParameters);
    }
}
```

**JobInstance는 Job과 JobParamter의 조합으로 생성된다. 자세한 규칙은 다음과 같다. **
- 처음 시작하는 Job + JobParameter일 경우 새롱운 JobInstance를 생성한다.
- 이전과 동일한 Job + JobParameter로 실행할 경우 이미 존재하는 JobInstance를 리턴한다.

<span style="color:red;">**순서는 다음과 같다.**</span>
1. JobLauncher에 의해서 Job(with JobParameter)을 실행한다.
2. JobRepository를 통해 BATCH_JOB_INSTANCE를 조회해 해당 Job + JobParameter에 해당하는 instance를 조회한다.
   1. instance가 존재한다면 기존 JobInstance를 리턴한다.
   2. instance가 존재하지 않다면 새로운 JobInstance를 생성한다.
   
    
---

## JobParameter
> Job은 JobLauncher에 의해서 Job을 실행할 때 Job과 함께 JobParameter를 함께 넘긴다. 그 이유는 
> Job은 항상 동일한 구성을 갖지만, 실행 시점에 따라 처리 내용이 다르기 때문에 JobParameter를 통해서 이를 구분하기 위해서다.

### 개념
- Job을 실행할 때 함께 포함되어 사용되는 파라미터를 가진 도메인 객체다.
- 하나의 Job에 존재할 수 있는 여러 JobInstance를 구분하기 위한 용도
- 생성된 JobParameter는 BATCH_JOB_EXECUTION_PARAM 테이블에 매핑된다.

---

```java
@Component
public class JobParameterTest implements ApplicationRunner {

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private Job job;

    @Override
    public void run(ApplicationArguments args) throws Exception {

        JobParameters jobParameters = new JobParametersBuilder()
                .addString("name", "user1")
                .addLong("seq", 2L)
                .addDate("date", new Date())
                .addDouble("age", 27.0)
                .toJobParameters();

        jobLauncher.run(job, jobParameters);

    }
}
```
> JobParamter는 JobParameterBuilder를 통해 생성할 수 있고, 해당 JobParameters가 저장된 테이블은 다음과 같다.

<img src="/public/img/1205_batchparam.png" width="900" object-fit="cover" alt="" />


---

## JobExecution
- JobInstance는 Job의 실행 시점에 Job을 구분하기 위한 용도인데,   
JobInstance(Job)의 실행 중에 생성된 상태 값(정보)들을 저장하고 있는 객체다.

### JobExecution의 특징
- JobExecution은 'FAILED' 또는 'COMPLETED' 등의 Job의 실행 결과 상태를 갖는다.
- JobExecution의 실행 상태가 'COMPLETED'면 JobInstace의 실행이 완료된 것으로 간주해서 해당 JobInstace의 재실행이 불가능하다.
- JobExecution의 실행 상태가 'FAILED'면 JobInstance의 실행이 완료되지 않은 것으로 간주해서 재실행이 가능하다.

```java
public class JobExecution extends Entity {
    private final JobParameters jobParameters; // JobParameters의 객체 저장
    private JobInstance jobInstance; // JobInstance 객체 저장
    private volatile Collection<StepExecution> stepExecutions; 
    private volatile BatchStatus status; // 실행 상태를 저장하는 Enum 클래스
    private volatile Date startTime; // 실행 시간
    private volatile Date createTime; // JobExecution이 처음 저장될 때의 시간
    private volatile Date endTime; // 성공 여부와 상관없이 종료 시간 저장
    private volatile Date lastUpdated; // JobExecution의 최종 저장된 시간 저장
    private volatile ExitStatus exitStatus; // 실행 결과를 나타내는 클래스
    private volatile ExecutionContext executionContext; // 실행하는 동안 유지할 데이터를 저장.
    private transient volatile List<Throwable> failureExceptions; // 실행 중 발생한 예외 저장
    private final String jobConfigurationName;
}
```


### references
[참조 : 스프링 배치 - Spring Boot 기반으로 개발하는 Spring Batch](https://www.inflearn.com/course/스프링-배치/dashboard)