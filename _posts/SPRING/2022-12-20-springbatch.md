---
layout: post
title: '[JAVA, SPRING BATCH] SPRING BATCH ? '
subtitle: '스프링 배치, Job, Step, Chunk, Tasklet, JobRepository'
date: 2022-07-27 12:00:00 +0900
categories: 'SPRING'
background: '/img/posts/etc/spring.jpg'
---

## Spring batch

- 배치 작업(일괄 처리 작업)을 위해 만들어진 스프링 프레임워크입니다.
> - 정해진 시간에 자동으로 진행되는 것들을 배치 작업이라 할 수 있습니다.
- 가볍고, 빠르고, 실행 중 오류가 생기면 그 시점부터 다시 시작할 수 있다는(Restartability) 장점이 있습니다. 
￼
<br>

### 구성 요소

![spring batch 구성](https://user-images.githubusercontent.com/88040158/208605222-2460da08-61e3-4a27-acda-6c516f732971.png)


#### Job(Flow) 
- 배치 작업 단위, 최소 하나의 step을 가져야 합니다. 대략 2~10개 정도의 step 을 가집니다.
> - 만약 step이 10개 이상 이상이면 리팩토링 과정으로 job을 쪼개 책임을 나누는 것이 좋습니다. 

<br>

#### Step 
- 실질적인 배치 로직, 하나의 트랜잭션으로 이해하면 됩니다.
> - Chunk Processing, Tasklet 중 하나로 구성됩니다. 

<br>

#### Chunk Processing 

- `읽고, 가공하고, 쓰는` 덩어리 로직입니다.  
> - ItemReader(인터페이스) : 데이터 읽고 리턴합니다(DB는 한 row, 파일은 한 라인, JSON Array는 하나의 element), `read()` 메소드를 가지고 있습니다.
> > - 다양한 인터페이스 구현체들을 가지고 있으니 적절하게 사용하면 좋을 것 같습니다, FlatFileItemReader, HibernateCursorItemReader,  JdbcCursorItemReader, JsonItemReader,  MongoItemReader
> > -  [ItemReader 공식문서](https://docs.spring.io/spring-batch/docs/current/api/org/springframework/batch/item/ItemReader.html)
> - ItemProcessor(인터페이스)  :  ItemReader에게서 객체를 받아 원하는 방식으로 가공 후 ItemWriter에게 넘겨주는 역할을 하고 한 번에 하나의 아이템을 처리합니다. `process(Input 객체)` 메소드를 기지고 있습니다.
> >  - Input, Output을 정해주어야 합니다. Input은 ItemReader의 매개변수 타입, Output은 ItemWriter의 매개변수 타입과 같아야 합니다.
> > - 가공 단계가 필요하지 않을 때는 ItemReader에서 ItemWriter로 바로 넘겨도 상관없습니다. 
> - ItemWriter(인터페이스) :  넘어온 데이터를 리스트에 차곡차곡 쌓아놓습니다. commit-interval 프로퍼티(step에서 설정 가능)의 정의된 숫자에 의해 `write(리스트 객체)` 메소드가 실행됩니다.
> > - 다양한 인터페이스 구현체들을 가지고 있으니 적절하게 사용하면 좋을 것 같습니다. CompositeItemWriter, FlatFileItemWriter, HibernateItemWriter,  JdbcBatchItemWriter, JsonFileItemWriter, MongoItemWriter

<br>

#### Tasklet 
- chunk (reader + processor + writer) 가 아닌 다른 로직을 처리할 때 사용되고 딱 한 번만 실행됩니다. 
> - Tasklet 인터에이스를 구현하여 사용하면 되고 해당 step이 실행되면 execute 메소드가 한 번 실행되고 끝납니다.


![jobrepository 메타데이터](https://user-images.githubusercontent.com/88040158/208605386-fa9a7dac-2358-4dd6-b1ce-330505b29d61.png)

<br>

- JobRepository 

- 6개의 메타데이터 테이블을 통해 배치에 관한 것들을 RDB에 모두 기록한다.

> - BATCH_JOB_INSTANCE : job이 실행되면 job 이름, 파라미터를 직렬화해서 저장합니다.
> - BATCH_JOB_EXECUTION : job 실행 내용(성공, 실패 모두)을 담고 있습니다.
> - BATCH_JOB_EXECUTION_PARAMS : job 파라미터가 저장됩니다.
> - BATCH_JOB_EXECUTION_CONTEXT : job 안의 객체들(tasklet, step)이 정보를 교환할 때 사용합니다. 
> - BATCH_STEP_EXECUTION : step 실행 내용(성공, 실패 모두)을 담고 있습니다.
> - BATCH_STEP_EXECUTION_CONTEXT : step 안의 객체들 (reader, processor, writer)이 정보를 교환할 때 사용합니다. 
> > - [스프링배치 메타데이터 공식문서](https://docs.spring.io/spring-batch/docs/current/reference/html/schema-appendix.html)

<br>

Reference:

- [Spring Batch] Intro - 스프링 배치 기본 개념 익히기 _ Fwantastic](https://www.fwantastic.com/2019/12/spring-batch-intro.html)
- [spring batch 공식문서](https://docs.spring.io/spring-batch/docs/current/reference/html/domain.html#domainLanguageOfBatch)

