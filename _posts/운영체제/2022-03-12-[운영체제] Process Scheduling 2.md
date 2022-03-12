---
layout: post
title: "[운영체제] Process Scheduling 2 (Synchronize)"
categories: 운영체제
tags: [운영체제, OS, Process Scheduling, Synchronize]
comments: true
---


# Process Scheduling 2 (Synchronize)

> 이화여대 반효경 교수님의 kocw 운영체제 강의를 들으면서 개인적으로 작성한 내용입니다.

<br>

**스케쥴링**

- FCFS(First-come First-service)
- Round Robin(RR)
    - CPU를 기다리는 시간이 CPU를 사용하려는 시간에 어느정도 비례하는 측면에서 공정한 스케쥴링이라고 할 수 있음.
    - 각 프로세스는 동일한 크기의 할당 시간(time quantum)을 가짐.
    - Process Context를 save하고, CPU를 다시 얻었을 때는 그 지점부터 재개할 수 있는 메커니즘을 제공한다는 점이 장점이다.
    
<br>

**Multilevel Queue**

- 우선순위
    
    Highest↑ 
    
    - System Processes (시스템 관련)
    - Interactive (사용자와 인터렉션)
    - interactive editing
    - batch (cpu만 오랫동안 사용하는 프로세스)
    - student processes
    
    Lowest↓
    
- 의문점
    - 프로세스를 어떤 queue에 넣을 것인가
    - 우선순위 큐에 따라만 cpu를 줄 것인가
        - 우선순위가 낮은 큐는 Starvation 을 겪을 수 있다.
- Ready queue 여러 개 분할
    - foreground(interactive)
    - background(batch - no human interaction)
- 큐 마다 독립적인 스케줄링 알고리즘
    - foreground 는 RR
    - background 는 FCFS
- 큐에 대해서도 스케줄링이 필요함
    - Fixed priority scheduling
    - Time slice
        - foreground / background 적절한 비율을 할당

<br>


**Multilevel Feedback Queue**

- 신분(?)을 극복 못하는 문제를 해결하기 위해..
    - 프로세스가 다른 큐로 이동이 가능함
- Queue의 수
- 승격 기준
- 처음 프로세스를 할당하는 큐 기준
    - 처음 들어오는 프로세스는 가장 우선순위가 높은 큐에 넣음
        - Time quantum을 낮게 줌 (Q_0 = 8ms)
        - 할당 시간 끝나도 안끝나면 아래 큐로 강등시키고(quantum도 낮춤) (Q_1 = 16ms)
        - 더 낮아지면 FCFS로 강등
    
<br>


**Multiple-Processor Scheduling**

- Homogeneous processor 의 경우?
- Load sharing
    - 부하를 적절히 공유하는 메커니즘이 필요함
    - CPU 마다 한 줄 또는 여러 줄을 세울 수도 있다.
- Symmetric Multiprocessing (SMP)
    - 모든 CPU가 대등해서 각 CPU가 알아서 스케줄링을 결정
- Asymmetric multiprocessing
    - CPU가 여러개 있는데, 그 중 하나의 CPU가 전체적인 control 을 담당하는 방식

<br>

**Real-Time Scheduling**

read-time job: 정해진 시간 내에 완수가 되어야 함 (deadline 존재)

- Hard real-time systems
- Soft real-time computing
    - 일반 프로세스에 비해 높은 priority를 가지나, 무조건 deadline 을 지키지는 않음

<br>

**Thread Scheduling**

구현 방식

- Local Scheduling
    - User level thread, 사용자 프로세스가 직접 스레드를 관리하고, OS는 스레드의 존재를 모름
    - OS의 입장에서는 해당 프로세스에 CPU를 줄지 말지 만 결정
        - 프로세스 내부에서 어떤 스레드에 CPU를 줄 지 결정
- Global Scheduling
    - Kernel level thread, 프로세스의 경우와 똑같이 커널의 단기 스케줄러가 결정
    - OS가 특정 알고리즘에 근거해서 어떤 스레드에 CPU를 줄 지 결정

<br>

**Algorithm Evaluation**

- Queueing models (굉장히 이론적인 방법)
    - service rate(처리율)와 arrival rate(도착률) 등을 통해 계산해서 각종 performance index 값을 계산
- Implementation (구현) & Measurement (성능 측정)
    - 실제 시스템에 알고리즘을 구현해서 성능을 측정 비교
    - 시간이 너무 오래걸리고 작업이 크다.
- Simulation (모의 실험)
    - 알고리즘을 시뮬레이션으로 작성하고 `trace` 입력을 통해 결과 비교

<br>

### Process Synchronization

Execution - Box ↔ Storage - Box

- 데이터 읽고 수정한 후 다시 저장하는 경우, Synchronization 문제가 발생한다.
- 저장소는 하나고, 여러 주체가 이에 접근할 때 Race Condition의 가능성이 있다.
- 프로세스는 일반적인 경우, 자기 주소공간만 접근하기 때문에 이런 문제가 발생할 일이 없다.
- 프로세스가 직접 실행하지 못하는 작업의 경우, OS에게 작업을 요청하는 시스템 콜을 한다.
    - 커널의 코드를 실행할 때는 커널의 데이터(공유 데이터)에 접근할 수 있다.
- Race Condition 이 발생하는 경우
    - Kernel 수행 중 인터럽트 발생 시
        
        (고급 언어에서의 변수값 변경은 CPU 내부에서는 여러개의 instruction으로 실행된다. 메모리 내 변수 값을 CPU 내부의 레지스터에 저장시키고, 레지스터 값을 수정하고 이를 다시 메모리의 변수 위치에 쓴다.)
        
        ex. count++의 경우
        
        - 변수를 CPU로 읽어들이는(load) 중에 인터럽트가 들어오면, 인터럽트 핸들러가 실행된다.
        - 인터럽트 핸들러도 Kernel 상의 코드이다.
            - Kernel의 데이터인 변수를 수정(count—)하고, 인터럽트 처리가 끝나면 원래 상태로 되돌린다.
        - 이미 load한 상태여서 count++하기 때문에 count는 원래 값이 됨.
        
        그래서 중요한 작업 중에는 원래 작업이 끝나기 전까지는 interrupt를 disable 시키는 방식을 사용하는데, 무조건 막으면 여러가지 문제가 발생할 수 있기 때문에 어떻게 처리하는지 등 내용들을 다루고자 한다.
        
    - Process가 시스템 콜 하여 Kernel mode로 수행 중인데 Context Switching 이 일어나는 경우
        - 이를 해결하기 위해 Kernel 모드에서 수행 중일 때는 CPU를 preempt 하지 않게 한다.
    - Multiprocessor 에서 shared memory 내의 kernel data
        - Multiprocessor 에서는 위 2가지 케이스에서의 해결책으로는 해결이 안된다.
        - 해결하기 위해서는 데이터 접근 시 Lock / UnLock 을 걸어야 한다.
        1. 개별 데이터에 접근할 때 마다 Lock 거는 방법
            
            → 해당 데이터가 아니면 여러 프로세서가 접근할 수 있게!
            
        2. 커널에 한번에 하나의 CPU 만 접근할 수 있게 하는 방법
            
            → 너무 비효율적이다.
            
        
    
    <aside>
    🔖 결국, Shared data에  Concurrent access는 데이터의 inconsistency를 발생시킬 수 있고,
    
    이를 해결하기 위해(consistency 유지하기 위해) Cooperating process 간 실행 순서(orderly execution) 을 정해주는 메커니즘이 필요하다.
    → Concurrent process는 동기화(Synchronize) 되어야 한다.
    
    </aside>
    
<br>

**Critical-Section (임계 구역) Problem**

- 여러 프로세스가 공유 데이터를 동시에 사용하고자 하는 경우
- Critical Section 은 공유데이터에 접근하는 코드 영역
- 특정 프로세스가 Critical Section 에 있는 경우 다른 모든 프로세스는 여기에 접근 불가능해야 한다.
    - 디른 프로세스들이 Critical Section 전에 멈춰있어야 한다는 문제가 존재함
    - 이를 해결하기 위한 다양한 알고리즘 존재 → 다음 장에서 다룰 것