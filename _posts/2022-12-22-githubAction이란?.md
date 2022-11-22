---
layout: post
title:  "Github Action이란?"
date:   2022-11-22 10:42:00 +0900
categories: DevOps
---

> 공식문서를 바탕으로 정리했습니다.

### Github Action이란?
- Github Action은 빌드, 테스트, 개발의 파이프라인을 자동화할 수 있는 CI/CD 플랫폼입니다.
- Github Action은 단순한 DevOps를 넘어서, 저장소에 특정 이벤트가 발생했을 때 workflows를 실행할 수도 있습니다.
  예를 들어 저장소에 특정 라벨을 추가한 이슈를 추가했을 때도 workflows가 실행되게 설정할 수 있습니다.

### 용어 정리
![](https://velog.velcdn.com/images/dlandif22/post/fc037816-04b8-49dd-8e69-5c342658f5d6/image.png)

#### workflow
> workflow는 자동화하고 싶은 일련의 작업(파이프라인)을 정의하는 파일입니다.
작성한 workflow(runner)는 가상머신 위에서 실행된다.

- workflow는 여러 개의 작업(Job)으로 구성된다.
- workflow가 실행되는 특정 시점(Trigger=pull request, merge)을 지정하거나, 수동으로 실행할 수 있다.
- workflow는 저장소의 '/github/workflows'안에 저장되어야 하며, 저장소는 여러 개의 workflow를 저장할 수 있다.

#### Job
- Job은 동일한 runner 위에서 실행되며, 여러 단계(step)로 이루어진다.
- 각 단계들은 순서대로 실행되며 각각 의존관계를 갖는다.
- Job들은 기본적으로 서로 독립적이며 병렬 실행된다.(메모리도 독립적이다.)
    - 설정을 통해 서로 의존적이게(순서대로) 설정할 수 있다.(파이프라인)

#### Step
- 하나의 Job안에 여러 개의 step을 정의할 수 있다.

#### Events
- workflow를 실행(Triggere)하는 특정 Activity
    - pull request
    - merge
    - create issue
    - etc...

#### Runner
- runner는 작성한 workflow를 실행해주는 서버다.

### Example
```yml
name: Node.js CI  

on: # event trigger
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: npm install and build webpack 
        run:  |
          npm install
          npm run build
      - uses: actions/upload-artifact@master
        with:
          name: webpack artifacts
          path: public/
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: actions/download-artifact@master
      with:
        name: webpack artifacts
        path: public/
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: 12.x
    - name: npm install and test
      run: |
        npm install
        npm test


````
- 위의 예시의 name은 해당 workflow의 이름을 지정한다.
- 'on'은 workflow가 실행되는 특정 trigger를 지정할 수 있는데, 위의 예시에서는 main branch가 push 또는 pull_request 시에 실행된다.
- 'jobs'는 하나 이상의 job을 작성할 수 있는데, 해당 예시에서 build와 test job을 작성했다.

