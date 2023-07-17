# Ch05

## 안정 해시 설계

수평적 규모 확장성을 달성하기 위해서는 요청 또는 데이터를 서버에 균등하게 나누는것이 중요.

안정해시는 이 목표를 달성하기 위해 보편적으로 사용하는 기술이다.

### 해시 키 재비치(rehash) 문제

 - N개의 서버가 있을 때, 부하를 균등하게 나누기 위해 해시 함수 사용
 - serverIndex = hash(key) % N(서버의 개수)
 - hash(key0) % 4 = 1인 경우, 클라이언트가 캐시에 보관된 데이터를 가져오기 위해 서버 1에 접속
 - 서버 풀(server pool) 크기가 고정되어 있고, 데이터 분포가 균등할 때 잘 동작한다.
 - 대규모 캐시 미스(cache miss) : 하지만 서버가 추가되거나 기존 서버가 삭제되면 문제가 생긴다. 
   - 1번 서버 장애 -> 1번 동작 중지 -> 서버 풀 크기 3 변경 -> 나머지 서버 인덱스 값들이 달라짐 

![image](https://github.com/Gonue/architecture-in-action/assets/109960034/4e221bb1-9d47-462d-a732-ffa73ecaa9b4)

### 안정해시

 - 해시 테이블 크기가 조정될 때 평균적으로 k/n개의 키만 재배치하는 해시 기술(k:키의 개수, n:슬롯 개수)

### 해시 공간과 해시 링

 - 해시 함수 f를 SHA-1을 사용한다고 가정할 경우
   - 함수의 출력 값 : x0, x1, x2, .. xn
   - 해시 공간 범위 : 0 ~ (2^160)-1
   - 해시 공간 표현

![image](https://github.com/Gonue/architecture-in-action/assets/109960034/06b7bc14-8c66-4004-9c45-249cd7092c4e)

 - 해시 링(hash ring) : 해시 공간의 양쪽을 구부려 접은 형태

![image](https://github.com/Gonue/architecture-in-action/assets/109960034/105f6bf2-cdfd-4669-ab5e-e66c6993340d)

### 해시 서버

 - 이 해시 함수 f를 사용해 우리가 정의한 키(서버 IP /이름 등)의 위치를 대응시키는 것이다.

### 서버 조회

 - 해시 함수를 사용해서 서버 4개를 해시 링 위에 해시 서버 배치 : s0, s1, s2, s3
 - 해시 링 어느 지점에 해시 키 배치 : k0, k1, k2, k3
 - 키가 저장되는 서버 : 해당 키 위치로부터, 시계 방향으로 링을 탐색해 나가면서 만나는 첫 번째 서버

![image](https://github.com/Gonue/architecture-in-action/assets/109960034/85a5c7ce-1a5e-419b-a956-e6e0ffb6a453)

### 서버 추가

 - 키 가운데 일부만 재배치
 - 서버 4추가 시, key0만 재배치
 - key0은 서버 4에 저장된다.

![image](https://github.com/Gonue/architecture-in-action/assets/109960034/fa8280ac-6434-4534-8496-2984bc42291c)

### 서버 제거

 - 하나의 서버가 제거되면 키 가운데 일부만 재배치
 - 서버1이 삭제되면, key1만 서버 2로 재배치

![image](https://github.com/Gonue/architecture-in-action/assets/109960034/8e5bdd44-5503-45d8-b33c-c55419d551f2)

### 기본 구현법 두가지 문제

 - 어떤 서버는 굉장히 큰 해시 공간을 할당 받는 상황이 생길 수 있다.
 - ex. s1 삭제 -> s2 파티션이 다른 파티션 대비 2배로 커짐
 - 가상 노드(virtual node) 또는 복제(replica) 사용 : 키의 균등 분포 문제 해결을 위해 사용한다.

### 가상 노드

 - 실제 노드 또는 서버를 가리키는 노드
 - 하나의 서버는 링 위에 여러 개 가상 노드를 가질 수 있다.
 - 키가 저장될 서버 : 키의 위치로부터 시계 방향으로 링을 탐색하다가 만나는 최초의 가상 노드
 - ex. k0은 가상 노드 s1_1가 나타내는 서버1에 키 저장

![image](https://github.com/Gonue/architecture-in-action/assets/109960034/48c5922b-1964-45aa-9170-a0dd24dc0b76)

![image](https://github.com/Gonue/architecture-in-action/assets/109960034/cccde9d7-1822-46ad-85a9-f8e218ce2ae3)

### 재배치할 키 결정

- 서버 추가 or 제거되면 데이터 일부를 재배치해야 한다.

### 마무리 - 안정 해시 이점

 - 서버 추가 or 삭제시 재배치되는 키의 수 최소화
 - 데이터의 균등한 분포 가능 -> 수평적 규모 확장성 용이
 - 핫스팟 키 문제 축소
