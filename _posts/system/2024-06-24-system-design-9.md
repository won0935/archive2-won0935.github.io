---
title: "S3와 유사한 객체 저장소"
date: '2024-06-24'
categories: [ 설계, 가상면접 사례로 배우는 대규모 시스템 설계기초 ]
tags: [ 면접 ]
---

# 저장소 시스템 101

## 블록 저장소

- HDD, SSD 하드디스크
- 유연하고 융통성이 높음

## 파일 저장소

- 블록 저장소 위에 구현됨
- 파일과 디렉터리 구조
- 보편적으로 사용

## 객체 저장소

- 실시간 갱신이 필요없는 데이터 보관에 초점(아카이브, 백업)
- 모든 데이터를 수평구조 객체에 보관
- 계층 디렉터리 X
- RESTful API 를 통한 접근
- 느림

![IMG_3468.heic](S3%E1%84%8B%E1%85%AA%20%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A2%E1%86%A8%E1%84%8E%E1%85%A6%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%89%E1%85%A9%20df1a921b522c49cdb27943ab11ae6f78/IMG_3468.heic)

## 용어 정리

- **버킷** : 객체를 보관하는 논리적 컨테이너. 이름은 유일해야 함
- **객체** : 버킷에 저장하는 개별 데이터
    - 데이터(혹은 페이로드) : 모든 유형 가능
    - 메타데이터 : 키 값 쌍의 집합
- **버전** : 한 객체의 여러 버전을 같은 버킷 안에 둘 수 있음. 덮어 쓴 객체를 복구할 수 있음
- **URI** : 객체 저장소는 버킷과 객체에 접근할 수 있도록 하는 RESTful API 제공
- **SLA** : 서비스 수준 협약
    - S3의 경우
        - 여러 가용성 구역에 걸쳐 99.9999999% 객체 내구성을 제공하도록 설계
        - 하나의 가용성 구역 전체가 소실되어도 복원 가능
        - 연간 99.9%의 가용성 제공

---

# 1. 문제 이해 및 설계 범위 확정

## 기능 요구사항

- 버킷생성
- 객체 업로드 및 다운로드
- 객체 버전관리
- 버킷 내 객체 목록 조회(s3 ls 명령어와 유사해야 함)

## 비기능 요구사항

- 데이터는 매년 100PB 증가
- 식스 나인(99.9999%) 수준의 데이터 내구성
- 포 나인(99.99%) 수준의 서비스 가용성
- 저장소 효율성 : 높은 수준의 안성성과 성능은 보증하되 저장소 비용은 최소로 해야함

## 대략적인 규모 추정

객체 저장소는 용량이나 IO 가 병목이 될 가능성이 큼

- 객체 용량 분포
    - 20% : ~ 1MB
    - 60% : 1MB ~ 64MB
    - 20 % : 64MB ~
- IO : 초당 100 ~ 150회 임의 데이터 탐색을 지원할 수 있다고 가정(100 ~ 150 IOPS)

### 저장소에 수용 가능한 객체의 수

- 각 유형별로 중앙값 계산
- 40% 의 공간 사용을 할 경우 아래와 같이 계산할 수 있음(약 6억 8천만 개)

$$
100PB = 100*1000*1000*1000MB = 10^11MB
$$

$$
10^11MB * 0.4/(0.2*0.5MB) + (0.6*32MB) + (0.2 * 200MB) = 680000000
$$

---

# 2. 개략적 설계안 제시 및 동의 구하기

## 객체 저장소의 특성

- **객체 불변성** : 객체 저장소의 객체들은 변경이 불가능. 삭제하고 새 버전으로 업로드는 가능
- **키-값 저장소** : URI = 키, 데이터 = 값
- **저장은 1회 읽기는 여러 번** : 95% 는 읽기 요청
- **소형 및 대형 객체 동시 지원** : 객체 저장소는 UNIX와 비슷, 메타데이터 저장소에는 데이터를 가져올 수 있는 ID만 저장됨

## 개략적 설계안

![IMG_3469.heic](S3%E1%84%8B%E1%85%AA%20%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A2%E1%86%A8%E1%84%8E%E1%85%A6%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%89%E1%85%A9%20df1a921b522c49cdb27943ab11ae6f78/IMG_3469.heic)

- **로드밸런서**
- **API 서비스** : 인증, 권한 부여, 접근 제어 서비스
- **데이터 저장소** : 객체 ID(UUID)를 통해 접근
- **메타데이터 저장소**

## 업로드

![IMG_3470.heic](S3%E1%84%8B%E1%85%AA%20%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A2%E1%86%A8%E1%84%8E%E1%85%A6%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%89%E1%85%A9%20df1a921b522c49cdb27943ab11ae6f78/IMG_3470.heic)

1. 클라이언트는 PUT 버킷생성 요청, API 서비스로 전달
2. API 서비스는 IAM 에 사용자가 권한을 가졌는지 확인
3. API 서비스는 버킷 메타데이터 생성, 버킷이 만들어지면 메시지가 클라이언트에게 전달
4. 클라이언트는 PUT 객체생성 요청
5. API 서비스는 사용자 신원 및 권한 여부 확인
6. API 서비스는 데이터 저장소에 데이터를 보냄, 데이터 저장소는 object_id(UUID)를 반환
7. API 서비스는 메타데이터 저장소에 저장(object_name, object_id, bucket_id)

## 다운로드

![IMG_3471.heic](S3%E1%84%8B%E1%85%AA%20%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A2%E1%86%A8%E1%84%8E%E1%85%A6%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%89%E1%85%A9%20df1a921b522c49cdb27943ab11ae6f78/IMG_3471.heic)

1. 클라이언트는 GET 요청을 로드밸런서로 보냄, 로드밸런서는 API 서비스로 보냄
2. API 서비스는 IAM에 사용자가 권한을 가졌는지 확인
3. 객체의 object_id(UUID)를 메타데이터 저장소에서 가져옴
4. API 서비스는 object_id로 데이터 저장소에서 데이터를 가져옴
5. 결과 반환

---

# 3. 상세 설계

## 객체 목록 조회

---

## ETC

- NFS(Network File System)
    - 분산 파일시스템 프로토콜
    - 원격 노드의 저장소를 마치 local 저장소처럼 활용할 수 있음
    
    ### 동작 원리
    
    1. NFS 서버는 `nfsd` 데몬을 실행 
        1. 어떤 파일을 클라이언트에게 제공할지 `/etc/exports` 에 기재 
    2. Client는 `mount` 명령어를 활용해서 NFS 서버에 커넥트 
    3. Client는 서버의 (mounted 된) 파일 시스템을 확인할 수 있음 
    
- Amazon S3 Strong Consistency
    
    ### S3의 특성
    
    - strong read-after-write consistency
    - 객체에 대한 new write, overwrite, delete 이후에 따라오는 읽기(GET, PUT, LIST)에 대해 강일관성을 보장함
    
    - Consistency가 없다면 어떤 불편함이 있을까……?
        
        ![Untitled](S3%E1%84%8B%E1%85%AA%20%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A2%E1%86%A8%E1%84%8E%E1%85%A6%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%89%E1%85%A9%20df1a921b522c49cdb27943ab11ae6f78/Untitled.png)
        
    
    ### Diving Deep on S3 Consistency
    
    - https://www.allthingsdistributed.com/2021/04/s3-strong-consistency.html (알고보니 아마존 최고기술책임자 블ㄹ그)
    - S3
        - 처음에는 단순 storage 용도면 충분했음
        - 시간이 지나자 data lake, analytics의 용도로도 사용하기 시작함
        
        ⇒ 다양한 기능 지원을 위한 신규 피처 중 하나로 strong consistency를 지원 
        
    - Consistency
        - AS-IS에서는 S3 metadata subsystem에 캐싱을 적용해두었음
            - 이 시스템에서는 write가 지극히 낮은 확률로 eventual consistency가 발생한다는 점
            
            > sometimes the API call would return an older version of an object that had not fully propagated throughout all the nodes in the system yet
            > 
            - 고객이 eventual consistency를 해결하기 위해 코드를 추가로 작성하거나 시스템을 개발함
                - 보통은 consistency를 위해 extra infrastructure을 추가하게됨
                - https://netflixtechblog.com/s3mper-consistency-in-the-cloud-b6a1076aa4f8
                
                ⇒ S3의 eventual consistency 문제를 해결하기 위한 솔루션 (일관성을 위해 DynamoDB를 활용)
                
    - Strong consistency를 위한 여정
        
        > **we wanted strong consistency with no additional cost, applied to every new and existing object, and with no performance or availability tradeoffs.**
        > 
        - strong consistency as default, FREE of charge, with no PERFORMACE implications
    - Eventual consistency가 발생하던 원인
        - S3 metadata를 저장하기 위한 subsystem에서 캐시를 사용
        - write와 read가 서로 다른 캐시를 사용하는 경우 eventual consistency가 발생
            
            ⇒ cache bypass를 생각해봤지만 performance cost is too high 
            
    
    ![Untitled](S3%E1%84%8B%E1%85%AA%20%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A2%E1%86%A8%E1%84%8E%E1%85%A6%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%89%E1%85%A9%20df1a921b522c49cdb27943ab11ae6f78/Untitled%201.png)
    
    - S3 metadata subsystem을 strong consistent하게 만들자!!
        - persistence tier에 새로운 복제 로직을 도입
            - at-least-once event notification delivery system
            - replication time control
        
        ⇒ per-object operation 순서를 알 수 있게해줌 
        
    - Witness라는 새로운 컴포넌트가 도입됨
        - cache의 object metadata의 stale 여부를 확인함
        - 읽기 요청이 들어왔을 때 read barrier 역할을 함
            - cache가 stale하지 않으면 cache 데이터로 응답
            - cache가 stale하다면 cache invalidate
    - Witness는 어떻게 구성된 시스템일까 ?
        - 가용성 OK
        - in-memory ⇒ state만 저장하면 되기 때문에 RAM으로 충분
        
- LinkedIn’s Scalable Geo-Distributed Object Store
    
    https://dprg.cs.uiuc.edu/data/files/2016/ambry.pdf
    
    ## Overview
    
    ![Untitled](S3%E1%84%8B%E1%85%AA%20%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%80%E1%85%A2%E1%86%A8%E1%84%8E%E1%85%A6%20%E1%84%8C%E1%85%A5%E1%84%8C%E1%85%A1%E1%86%BC%E1%84%89%E1%85%A9%20df1a921b522c49cdb27943ab11ae6f78/Untitled%202.png)
    
    - Partition
        - 데이터 덩어리
        - 특정 사이즈 이하에서는 read, write 모두 가능 (append only log)
        - 특정 사이즌 넘어가면 read만 가능 (immutable)
    - Cluster manager: partition state를 관리
    - Frontend: request routing (put, get, delete)
    - Datanode: 실제 데이터 저장
    
- Erasure Coding(EC)
    - 데이터를 안전하게 저장할 수 있는 방법 중 하나
        - RAID에 비해 overhead 가 낮음
    - 원리
        - 하나의 파일 또는 객체를 여러 data block으로 나누고 여기에 parity block을 추가함(for data recovery)
        - erasure coding algorithm을 활용해 parity를 계산함
        - data, parity는 다수의 저장소에 저장됨
        - parity + data 조합으로 기존 데이터를 복구할 수 있음
    - 5 + 2 enconding config란
        - 데이터를 5개로 나누고 2개의 parity를 추가
        - 총 7개의 디스크가 필요함 (5 + 2)
        - parity 저장하기 위한 용량은 전체 데이터에서 29%
        - 2 disk fail까지 견딜 수 있음
        
        ⇒ 데이터의 full replication 없이도 replication이 가능해짐
