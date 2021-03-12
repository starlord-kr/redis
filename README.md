Redis 기초
==

**Redis의 주요 특징**
--
> Redis(Remote Dictionary Server) 는 오픈소스 NoSql로서 key-value 타입의 저장소로 주요 특징은 다음과 같습니다.   

* 영속성을 지원하는 인메모리 데이터 저장소다.
* 읽기 성능 증대를 위한 서버 측 복제를 지원한다.
* 쓰기 성능 증대를 위한 클라이언트 측 샤딩을 지원한다.
* ANSI C로 작성됐다. 따라서 ANSI C 컴파일러가 동작하는 곳이면 어디든 설치 및 실행이 가능하다.
* 레디스 클라이언트는 대부분의 언어로 포팅되어 있다.
* 전세계 많은 서비스에서 사용되고 있으며 성능적으로 검증된 솔루션이다.
* 문자열, 리스트, 해시, 셋, 정렬된 셋과 같은 다양한 데이터형을 지원한다. 메모리 저장소임에도 불구하고 많은 데이터형을 지원하므로 다양한 기능을 구현할 수 있다.

**레디스와 멤캐시드의 차이**
---

#### 멤캐시드
> 본질적으로 고성능 분산 메모리 객체 캐싱 시스템이지만, 원래는 동적 웹 서비스의 DB부하를 경감시키는 것이 목적이었다.  

#### 레디스
> 오픈소스이며 향상된 key-value  저장소다. 값으로 문자열, 리스트, 해시, 셋, 정렬된 셋을 포함할 수 있기 때문에 종종 데이터 구조 서버로 지칭된다.

**해시데이터**
---
```
해시 데이터는 문자열 필드와 값으로 이루어진 Map  구조로 되어 있다. 해시 데이터에는 2의 32승 -1 개의 필드와 값을 저장할 수 있는데,
숫자로 바꾸면 약 42억개가 넘는다. 또한 hgetall, hkeys, hvals를 제외한 모든 해시 명령의 시간복잡도는 O(1)이다. 해시 데이터는
키 하나가 여러 개의 필드-값 쌍으로 이루어진다. 이런 구조는 일반적인 프로그래밍 언어의 맵 자료구조와 동일하다.

해시 데이터의 키 목록 조회

모든 키의 목록과 모든 값의 목록을 조회할 때는 hgetall을 사용하면 된다. 키의 목록이나 값의 목록만을 조회하는 명령은 hkeys, hvals 이다.
hkeys 명령은 주어진 키에 저장된 모든 필드의 목록을 조회한다. hvals는 주어진 키에 저장된 모든 값에서 필드 이름을 제외한 값의 목록을 조회한다.
```

**해시 데이터의 특별한 유형 zipmap**
---
```
 레디스의 해시 데이터는 해시 데이터에 저장된 필드 수가 수백 개 정도일 때 매우 적은 양의 메모리에 많은 데이터를 저장할 수 있다.
이는 레디스가 메모리를 효율적으로 사용하기 위하여 내부적으로 몇 가지 특별한 저장구조를 사용하기 때문에 가능하다. 레디스는 데이터
저장에 구조체를 사용하는데, 이 구조체는 실제 데이터가 저장된 위치를 가르키는 포인터와 몇 가지 부가 정보로 구성된다. 원래 데이터에
부가 정보가 덧붙으면 자연스레 더 많은 메모리를 사용하게 되는데, 레디스는 메모리 공간을 절약하기 위해서 세 가지 내부 저장구조
(zipmap, ziplist, intset)를 사용한다. 그런데 이 내부 저장구조를 사용하게 되면 CPU를 더 사용하게 된다. 따라서 CPU와 메모리
공간의 적당한 트레이드 오프를 통해서 가장 효율적인 값을 찾아야 한다. 인스타그램의 개발자가 테스트한 결과, 백만 개의 키와 정수값을
문자열 데이터로 저장하는 데 필요한 메모리는 70MB인데,  zipmap을 사용한 해시 데이터로 저장하면 16MB만 소모된다.
이 zipmap은 레디스 2.6 버전부터 ziplist라는 구조로 대치된다.
```
#### **Set을 활용한 집합연산**   
>    sadd 명령어를 통해 트위터 팔로우/팔로워 구조를 저장하고, sinter 명령어를 통해 두개의 키 사이의 교집합(맞팔)을 구할 수 있다.

**유닉스 타임스탬프**
---
```
 유닉스 타임스탬프는 유닉스에서 사용하는 시간으로서 1970년 1월1일 0시00분00초부터 지금까지 몇 초가 지났는지를 알려주는 값이다.
예를 들어 유닉스 타임스탬프값 '1234567890'은 국제 표준시 2009년 2월13일 31분30초를 나타낸다. 그러므로 'expireat test:key 0'
명령은 국제 표준시 1970년 1월1일 00시00분00초에 키가 만료된다. 한국은 GMT+9를 사용하므로 구해진 유닉스타임스탬프값에 9시간을 더하려고
하는 개발자가 있을지 모르겠다. 레디스는 입력된 타임스탬프값을 GMT값으로 인식하여 처리하므로 별도로 시간을 더할 필요가 없다.
```
> **키의 만료 처리**   
```
저장된 키에 대하여 만료시간을 설정할 수 있다(레디스가 멤캐시드와 같은 캐시 시스템과 비교되는 이유가 바로 이 기능 때문이다).  레디스가 제공하는 만료 처리 방법은 '지정된 시간에 만료'와 (expireat) '명령이 수행된 이후부터 일장 시간 이후의 만료' (expire)를 지원한다.
```

**레디스의 키와 값의 관계**
---
```
레디스에 키와 값을 저장하면 레디스 내부에서는 키를 위한 메모리 영역과 값을 위한 메모리 영역이 생성되는데, 키를 위한 메모리 영역에는 값이 저장된 메모리 영역의 포인터가 저장된다. 같은 키로 다른 데이터를 저장하는 명령이 입력되면 값을 저장하기 위한 새로운 메모리 영역이 생성되고 키에 저장된 포인터 정보가 새로운 값의 메모리 영역을 가리키게 되며, 이전에 저장되었던 값의 메모리는 사라지게 된다.
```
expireat, expire, ttl, persist(만료시간제거)

**대량의 데이터 입력**
---
```
-   레디스 클라이언트 라이브러리를 사용해서 하나씩 순차적으로 데이터를 입력하는 방법
-   레디스 파이프라인을 사용하여 한꺼번에 입력하는 방법
    cat redis\_data\_command.txt | ./redis\_cli --pipecat redis\_data\_protocol.txt | ./redis\_cli --pipelettuce 나 jedis 의 파이프라인 라이브러리
-   레디스 스냅샷 파일에 직접 기록하는 방법
```

**레디스의 성능**
---
레디스는 단일 스레드로 동작하기 때문에 하나의 레디스 인스턴스가 사용할 수 있는 최대 CPU의 개수는 하나다. 따라서 처리 성능을 높이기 위해서 CPU를 추가하는 것은 레디스의 성능 향상에 큰 도움이 되지 못한다. 그래서 스케일아웃처럼 동일한 사양의 시스템을 추가하는 방법을 사용해야한다. 스케일아웃을 하기 위해서 복제(Replication)와 쓰기 분산을 위한 샤딩(sharding)을 사용할 수 있다.

**레디스의 복제**
---

복제란 클라이언트가 어느 노드에 접근하더라도 동일한 데이터를 읽을 수 있도록 데이터를 각 노드에 복제하여 저장하는 것을 말한다. 복제를 구성할 때 하나의 마스터에 너무 많은 슬레이브를 구성하지 않도록 한다. 데이터복제를 위한 네트워크 사용률이 데이터 서비스를 위한 네트워크 사용률보다 커지게 되어 매우 느린 응답시간을 보이게 된다. (별도의 네트워크카드 설정)

레디스의 샤딩

**메모리 크기와 설정**
---

**redis.conf 파일의 maxmemory**  설정을 통해 레디스가 사용할 메모리 크기를 설정한다. 이 설정은 물리적인 메모리와 스왑공간을 고려하여 설정해야한다. 스왑공간은 메모리가 적을 경우 메모리의 2배, 메모리가 클 경우 1/2로 한다고 한다. 또한 **maxmemory** 설정에는 redisObject와 같은 구조체 정보도 모두 포함되니 실제 입력될 데이터의 크기는 설정한 것보다 작게 입력될 것이다.

  만약 레디스에 저장될 데이터의 크기를 산정하지 못하여 redis.conf의 maxmemory 설정에 값을 지정할 수 없게 됐다고 가정하자. 그러면 레디스에 데이터가 계속 추가되어 물리 메모리를 모두 사용하게 된다. 이때 운영체제는 애플리케이션이 시스템에서 사용 가능한 물리 메모리보다 더 많은 메모리를 사용하려고 할 때 스왑이라는 가상 메모리 공간을 하드디스크에 생성하여 사용하는데 이 영역을 스왑공간이라 부른다. 이때 스왑영역이 충분하지 않으면 운영체제는 메모리에서 동작 중인 프로세스를 제거하여 사용 가능한 메모리 영역을 확보하게 된다. (OOM - Out Of Memory killer 발생)

 여하튼 지정된 레디스의 메모리 크기보다 더 많은 데이터를 저장하려들면 레디스는 오류 메시지를 전송하고 쓰기 요청을 실패한다. 하지만 이 상황에서 읽기 요청은 정상적인 응답시간으로 서비스된다. 그러므로 응답시간이 중요한 서비스에서는 레디스가 사용할 메모리의 크기를 지정하는 것이 유리하다. 레디스에 저장된 데이터의 크기가 maxmemory 설정에 지정된 값을 초과하게 되면 레디스의 쓰기 연산이 실패하게 되고, 이에 레디스는 저장 가능한 메모리 영역을 확보하기 위하여 기존에 저장한 데이터를 지운다. 이와 같은 동작은 redis.conf의 **maxmemory-policy** 설정에 지정된 값에 따라서 달라지는데 자세한 내용은 아래와 같다.

- **volatile-lru** : expire 명령을 사용하여 만료시간이 지정된 키 중에서 만료된 키를 대상으로 최근에 가장 적게 사용된 키를 제거한다.
- **allkey-lru** : 모든 키 중에서 최근에 가장 사용이 적은 키를 제거한다. 
- **noeviction** : 어떤 키도 제거하지 않는다.
    

#### **maxmemory-sample** 설정은 몇 개의 키를 조회하여 삭제할 키를 선정할지 지정한다.
스냅샷이나 AOF를 위해서 호출한 fork 함수는 부모 프로세스와 동일한 크기의 메모리를 사용하는 프로세스를 생성한다. 따라서 부모 프로세스가 사용하는 만큼의 메모리가 남아있지 않으면 fork 함수가 실패하게 된다.
linux /etc/sysctl.conf

#### **vm.overcommit\_memory** 0, 1, 2 값을 설정할 수 있으며 자세한 설명은 아래와 같다.
- 0 : 리눅스 시스템의 기본 설정값, 메모리 할당 요청인 malloc 함수의 요청이 들어오면 요청된 크기만큼의 물리적 메모리가 존재할 때에만 메모리를 할당한다.
- 1 : 메모리 할당 요청인 malloc 함수의 요청이 들어오면 남은 물리 메모리가 없더라도 성공을 응답한다. 단, 요청으로 입력된 크기의 스왑영역이 존재할 때에만 성공한다.
- 2 : 사용 중인 메모리 크기가 '스왑공간 크기 + vm.overcommit\_ratio \* 물리 메모리 크기' 이내일 때 메모리를 할당한다.

#### **maxclients**
레디스 인스턴스에 접속할 수 있는 클라이언트 수를 지정한다. 실제로는 지정된 값보다 약간 작은 값으로 설정된다. 예를 들어 리눅스에서 실행중인 32비트 레디스의 기본값은 1,024인데, 실제로는 레디스 프로세스가 사용하는 파일디스크립터개수(32개)를 뺀 값이 설정된다. 그리고 운영체제 자체에서 제한이 있을 수 있는데 ulimit 명령어로 확인할 수 있다. "ulimit -n" 을 통해 하나의 프로세스가 처리할 수 있는 최대 파일 디스크립터의 개수를 확인하고, /etc/security/limits.conf 파일에서 수정할 수 있다.

#### **hash-max-ziplist-entries**
> 해시 데이터에 ziplist 인코딩을 사용하여 저장할 데이터 개수를 지정한다. 여기서 지정된 개수보다 많은 데이터가 저장되면 레디스는 자동으로 hashtable 인코딩을 사용하여 저장한다.

#### **hash-max-ziplist-value**
> 해시 데이터에 ziplist 인코딩을 사용하여 저장할 데이터의 크기를 지정한다. 여기서 지정된 크기보다 큰 데이터가 저장되면 레디스는 자동으로 hashtable 인코딩을 사용하여 저장한다.

**레디스가 지원하는 데이터 인코딩**
---
* raw : 입력된 데이터를 변환하지 않고 그대로 저장하는 상태다.
* integer : 입력된 숫자 형식의 데이터를 숫자 데이터 타입으로 저장하고 있는 상태다. 주로 incr, incrby 명령으로 저장되는 데이터가 integer 인코딩을 사용한다.
* hashtable : raw 인코딩을 가진 데이터를 저장하고 이쓴 해시 테이블을 의미한다. 레디스의 기본 인코딩중의 하나로 전혀 압축되지 않는다.
* zipmap : 압축된 형태의 map 구조를 표현하는 데이터 인코딩이다. 레디스 2.6미만 버전까지만 사용되고 2.6 버전부터는 ziplist로 통합된다.
* linkedlist : raw 인코딩을 가진 데이터를 저장하고 있는 연결 리스트를 의미한다. 레디스의 기본 인코딩 중의 하나로 전혀 압축되지 않는다.
* ziplist : 압축된 형태의 데이터 인코딩이다. 해시, 셋, 정렬된 셋 데이터형이 압축된 형태의 인코딩을 의미한다.
* intset : 입력된 숫자 형식의 데이터를 숫자 데이터 타입으로 변환하여 셋 데이터에 저장된 상태다.
* skiplist :  raw 인코딩을 가진 데이터를 저장하고 있는 정렬된 셋 데이터의 인코딩을 의미한다.

