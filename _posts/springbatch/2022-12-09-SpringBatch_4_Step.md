---
title: "스프링 배치 Step"
categories: spring-batch
toc: true
toc_sticky: true
toc_label: "Contents"
---

## Step
- Job을 구성하는 하나의 단계로서 실제 배치 처리를 정의하고 관련된 모든 정보를 가지고 있는 객체
- 입력과 처리 그리고 출력과 관련된 비즈니스 로직을 포함하는 모든 설정들을 담고 있다.
- 모든 Job은 하나 이상의 Step으로 구성된다.


### Step 구현체

#### TaskletStep
- 가장 기본이 되는 클래스로서 Tasklet 타입의 구현체들을 제어.
#### PartitionStep
- Multi Thread 방식으로 Step을 여러 개의 스레드로 분리해서 실행한다.
#### JobStep
- Step 내에서 Job을 실행할 수 있는 Step.
#### FlowStep
- Step 내에서 Flow를 갖고 실행할 수 있다.

---

### Step의 계층 구조
- 위의 4개의 구현체 중 TaskletStep의 계층 구조를 살펴보자.

<img src="/public/img/1209_tasklet.png" width="400" object-fit="cover" alt="" />

> Step의 각 구현체들은 모두 AbstractStep을 상속받고 있으면 각 Step에 맞게 구현하고 있다.

---

## StepExecution
- Step에 대한 한번의 시도를 의미하며 Step 실행 중에 발생한 정보들을 저장한다.
- 매번의 시도마다 생성되며 각 Step 별로 생성된다.
- Job이 재시작 하더라도 이미 성공적으로 완료된 Step은 재 실행되지 않고 실패한 Step만 실행된다.
- 만약 이전 단계의 Step이 실패하면 현재 Step은 애초에 실행되지 않기 떄문에 StepExecution을 생성하지 않는다.

```java
public class StepExecution extends Entity {
    private final JobExecution jobExecution; // JobExecution 객체 
    private final String stepName; // Step 이름 
    private volatile BatchStatus status; // 실행 상태 Enum 클래스
    private volatile int readCount; // 성공적으로 read한 아이템 수
    private volatile int writeCount; // 성공적으로 write한 아이템 수 
    private volatile int commitCount; // 실행 중에 커밋된 트랜잭션 수 
    private volatile int rollbackCount; // 트랜잭셩 중 롤백된 횟수
    private volatile int readSkipCount; // read에 실패해서 스킵된 횟수
    private volatile int processSkipCount; // process에 실패해서 스킵된 횟수
    private volatile int writeSkipCount; // write에 실패해서 스킵된 횟수
    private volatile Date startTime; // Job을 실행할 때의 시스템 시간
    private volatile Date endTime; // 성공 여부와 관계없이 종료된 시간
    private volatile Date lastUpdated; // jobExecution이 마지막 저장될 때의 시스템 시간
    private volatile ExecutionContext executionContext; // 실행되는 동안 유지해야 하는 데이터를 저장
    private volatile ExitStatus exitStatus; // 실행결과를 나타내는 클래스로 종료코드를 포함한다.
    private volatile boolean terminateOnly; //
    private volatile int filterCount; // ItemProcessor에 의해 필터딩뢴 아이템 수
    private transient volatile List<Throwable> failureExceptions; // Job 실행 중에 발생한 예외 리스트를 저장
}
```


### StepExecution 예시
> 간단한 예제를 통해 StepExecution를 알아보자.
> 예시에서는 하나의 Job이 세 개의 Step을 구성되는 첫 번째 실행에서는 일부러 두 번째 Step을 실패시켜서 세 번째 Step의 StepExecution이 생성되지 않는 것을 확인하고
> 재실행을 통해서 세 번째 Step의 실행을 확인하려 한다.

```java
@Configuration
@RequiredArgsConstructor
public class StepExecutionTest {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;


    @Bean
    public Job job() {
        return jobBuilderFactory.get("stepExecutionTestJob")
                .start(firstStep())
                .next(secondStep())
                .next(thirdStep())
                .build();
    }

    @Bean
    public Step firstStep() {
        return stepBuilderFactory.get("firstStep")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("first Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step secondStep() {
        return stepBuilderFactory.get("firstStep")
                .tasklet((contribution, chunkContext) -> {
                    String jobName = contribution.getStepExecution().getJobExecution().getJobInstance().getJobName();
                    ExecutionContext jobExecutionContext = chunkContext.getStepContext().getStepExecution().getJobExecution().getExecutionContext();
                    System.out.println("second Step!");
                    if (!jobExecutionContext.containsKey("data")) {
                        jobExecutionContext.put("data", "data");
                        throw new IllegalArgumentException("can't find data");
                    }

                    return RepeatStatus.FINISHED;
                })
                .build();
    }

    @Bean
    public Step thirdStep() {
        return stepBuilderFactory.get("thirdStep")
                .tasklet((contribution, chunkContext) -> {
                    System.out.println("third Step!");
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

#### 첫 번째 실행

<img src="/public/img/1209_taskletTest_1.png" width="900" object-fit="cover" alt="" />

> 실행 결과를 보면 1, 2번째 Step은 실행됐지만 2번째 Step에서 예외가 마지막 Step이 실행되지 못하고 종료됐다. 
> 그럼 BATCH_STEP_EXECUTION 테이블의 첫 번째와 두 번째 Step만 잘 insert 됐는지 확인하자.

<img src="/public/img/1209_taskletTest_2.png" width="900" object-fit="cover" alt="" />


#### 두 번째 실행

<img src="/public/img/1209_taskletTest_3.png" width="900" object-fit="cover" alt="" />

> 두 번째 실행 결과를 보면 2, 3번째 Step이 재실행된 걸 확인할 수 있다.

<img src="/public/img/1209_taskletTest_4.png" width="900" object-fit="cover" alt="" />

---


## StepContribution

- 청크 프로세스의 변경 사항을 버퍼링한 후 StepExecution 상태를 업데이트하는 도메인 객체다.
- 청크 커밋 직전에 StepExecution의 apply 메서드를 호출하여 상태를 업데이트한다.
- 사용자 정의 종료코드를 새엇ㅇ해서 적용 할 수 있다.

```java
public class StepExecution extends Entity {
    public synchronized void apply(StepContribution contribution) {
        this.readSkipCount += contribution.getReadSkipCount();
        this.writeSkipCount += contribution.getWriteSkipCount();
        this.processSkipCount += contribution.getProcessSkipCount();
        this.filterCount += contribution.getFilterCount();
        this.readCount += contribution.getReadCount();
        this.writeCount += contribution.getWriteCount();
        this.exitStatus = this.exitStatus.and(contribution.getExitStatus());
    }
}
```


### references
[참조 : 스프링 배치 - Spring Boot 기반으로 개발하는 Spring Batch](https://www.inflearn.com/course/스프링-배치/dashboard)
