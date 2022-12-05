---
layout: post
title:  "스프링 배치 Job"
date:   2022-12-05 15:30:00 +0900
categories: SpringBatch
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


### references
