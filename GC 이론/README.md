 취업준비를 하면서 내가 제대로 보지 못했던 자바의 '진짜 모습'을 보고자 노력하고 있다. 그래서 첫번째로, GC에대해서 살펴 볼 것이다.

자바를 공부하면서 흥미로웠던 것은 GC였다. 하지만, 학교내의 프로젝트에서는 GC를 보기 힘들다. 이번기회를 통해 GC를 튜닝하는 방법까지 공부하면서 구현할 예정이다. 

<br> 

**가비지 컬렉션(GC)** 

말그대로 '쓰레기 수집가'이다. 하지만 그냥 수집가가 아니다, GC는 프로그램의 성능과도 연결될 수 있다. 그만큼 GC는 자바의 필수적인 요소이다. 

<br>

**Stop-the-world**

<br>

<br>

이 용어의 뜻은 나는 이렇게 이해했다. 모두 멈추고 어둠의 지배자인 GC의 활동(?).. 이해하기 위해서 이렇게 생각했었다.  GC를 진행하는 쓰레드를 제외하고 모든 쓰레드가 멈춘후, GC가 작업을 완료하면 중단한 작업들을 다시 시작한다, 라는 용어이다. 생각해보면 멈춘시간이 짧을 수록 좋은 프로그램이 아닐까 생각든다. 그렇기때문에 GC튜닝과정에서는 이 시간을 줄이는 것이다.

<br>

**자바는 명시적으로 프로그램을 지정하여 해제 하지 않는다**

<br>

그렇다. 그러지 않는다, 명시적으로 해제하지 않고 가비지 컬렉터가 필요없는 객체들을 찾아 치우는 작업을 한다. 하고 싶다면 null로 지정하거나, system.gc()를 호출하는데 후자는 절대로 하면 안된다.(성능에 무리)

<br>

<br>

**weak generational hypothesis**

가비지 컬렉터는 이 가설을 바탕으로 만들어 졌는데 , 

- 대부분의 객체는 금방 접근 불가능상태가 된다.
- 오래된 객체에서 젊은 객체로의 참조는 아주 적다.

이 가설을 전제조건으로 가비지 콜렉터가 만들어졌다.

**2개의 공간 Young and Old**



![스크린샷 2017-11-01 오후 3.54.00](/Users/kingop/Desktop/스크린샷 2017-11-01 오후 3.54.00.png)

위 가설의 장점을 살리기 위해 2개의 물리적 공간으로 나뉘는데 , 

<br>

**Young**과 **Old**이다.

<br>

<br>

<u>Young 영역</u> 

-  새롭게 생성한 객체들이 이곳에 저장된다.
-  객체가 접근 불가능한 상태이므로 생성과 소멸의 과정을 거친다.
-  소멸의 과정에서 *Minor GC*가 발생한다. 

<u>Order 영역</u>

- 접근불가 구역이 아니다. 그래서 Young영역에서 살아남은 객체가 이곳으로 복사된다.
- 위의 사항에 대한 특징상 Young영역보다 크기가 커야된다.
- Young영역보다는 GC횟수가 비교적 적다.
- 소멸의 과정에서 *Major GC(Full GC)*가 발생한다.



```
      
추가적으로 perm영역이 있다. java 8에서는 사라졌다고 들었다, 아니 대체되었다는 표현이 정확하겠다. 이 영역에서는 객체or억류된 문자열이 저장된다. Old영역에서 살아남은 객체가 영원히 남아있는 것은 아니다. GC가 발생할 수 있고 Major GC라한다.
```

<br>

**Old영역에 있는 객체가 Young 영역의 객체를 참조하는 경우의 처리**

**카드 테이블**을 이용한다. 카드테이블의 크기는 512Byte이다. Old영역에 있는 객체가 Young영역의 객체를 참조할때마다 카드테이블에 표시를 해준다. 따라서 Young의 GC가 실행되면 Old영역이 아닌 카드테이블을 이용하여 처리한다. write barrier를 사용하며 관리하고 minor gc빠르게 해준다. 약간의 오버헤드의 위험이 있지만, 전반적인 GC시간은 줄어든다. 

<br>

<br>

**Young generation**

<br>

제일 먼재 객체가 생성되어 저장되는 곳이다 , 3개의 영역으로 나뉘는데

<br>

*Eden 과 두개의 Survivor이다.*

<br>

- 새로 생성된 객체들은 Eden에 저장된다.
- Eden에서 GC가 발생하고, 살아남은 객체들은 Survivor로 이동한다.
- 또 다시 Eden에서 GC가 발생할때, 이미 객체를 가지고 있는 Suvivor로 이동한다.
- Survivor이 가득차게 되면 그중에 살아남은 객체들을 다른 Survivor로 이동시킨다. 
- 그렇게 되면 기존에 Eden에서 살아남은 객체들이 있던 Survivor은 Empty상태가 된다. 
- 과정을 반복하면서 최종적으로 살아남은 객체들은 **Old generation** 으로 이동한다. 



두 개의 Survivor중 반드시 하나는 비어있어야한다. 둘다 아무것도 없거나, 둘다 있으면 시스템은 정상이 아니다.

<br>

빠른 메모리 할당을 위한 기술이 있다. 

<br>

**bump-the-pointer**

Eden의 가장 높은(top)부분에는 마지막 객체가 존재한다. bump-the-pointer는 새로운 객체가 생성되면 

이 객체가 Eden에 들어가도 되는지 검사를 한다. 그리고 검사를 통과한 객체는 Top위치로 가게 된다. 

이러한 이유로 객체를 생성할때 마지막에 추가된 객체만 점검하면된다. 때문에 메모리 할당이 매우 빠르게 된다.

<br>

*하지만..!*

<br>

멀티 쓰레드 상황에서는 Thread-safe를 위해 여러 스레드에서 Eden영역을 사용할때에는 롹이 발생한다. 

그렇게 되면 성능은 떨어질 가능성이 있다. (..lock contention)

<br>

**Thread-Local-Allocation-Buffers(TLABs)**

각 스레드마다 Eden영역의 작은 덩어리를 나눠준다. 그렇기 때문에 각 스레드는 자신의 TLABs에만 접근하면 된다.

이렇게 롹이 발생할 가능성을 없앨 수 있다. 

<br>

<br>

**Old 영역의 GC**

<br>

*Old Generation*에 객체가 가득차면 실행하게된다.

<br>

**<Mark - sweep - compaction >**

 Old영역에서 사용하는 알고리즘이다. 

**Mark** : Old영역에 살아있는 객체를 식별한다. 

**Sweep** : 힙의 앞부분부터 확인하여 살아있는것만 남긴다. 

**Compaction** : 앞부분부터 채워서 객체가 존재하는 구역과 존재하지 않는 구역을 나눔 



# Serial GC 

-  단일스레드 
- 적은 메모리와 cpu 코어 개수가 적을때 사용 
- 잘안쓰임
- -XX:+UseSerialGC 로 활성화 

<br>

<br>

<br>

# Parallel GC(=Throughput GC)

- 쓰렏가 여러개 존재 그래서 Serial GC보다 빠르다.
- 메모리가 충분할때 사용해야한다. 
- -XX:+UseParallelGC 로 활성화

<br>

<br>

<br>

#Parallel Old GC

- Parallel GC와 비교하여 Old영역의 GC만 다름

- **mark-summary-Compaction** 단계를 거침

- 별도로 살아있는 객체를 식별한다는 점에서 더 복잡해짐

- -XX:+UseParallelOldGc 로 활성화

  <br>

  <Br>

  <br>

# CMS GC(=Low latency GC)

- 초기 Intial Mask 에서는 가장 가까운 객체를 먼저 찾는다. 
- Concurrent Mask 단계에서는 확인한 객체가 참조 하고 있는 객체들을 따라가면서 확인을 한다. 
- 다른 스레드가 실행되고 있어도 동시 진행이 가능하다.
- Remark에서는 Concurrent Mask 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다. 
- Councurrent sweep에서는 쓰레기(객체)들을 정리한다.
- Stop-the-world가 매우 짧다는 특징을 가지고 있다.
- 모든 어플리케이션이 빠른 응답속도를 요구할때 사용된다.
- 메모리와 CPU사용량이 많이 요구된다. 
- Compaction단계를 제공하지 않는다. 
- 위의 단점으로 인하여 조각난 메모리, 즉 메모리 파편화가 심해진다면 Serial GC와 다를 것이 없다.
- Compaction작업이 수행에대한 정보가 필요하다. 
- -XX:+UseConcMarkSweepGC 로 활성화

<br>

<br>

<br>

# G1 GC

- CMS GC의 단점을 극복하기 위해
- 여러 영역(Region)들을 나누어서 활용한다.
- Young 영역은 멀티쓰레드로 해결
- Old영역은 백그라운드에 있는 쓰레드가 정리  
- 각 영역에 객체들을 할당하고 GC를 실행한다. 
- 꽉차게 되면 다른영역에 객체를 할당하고 GC를 실행 
- 사용 중인 객체만 다른 영역에 복사 ( 참조 없는 것들은 BYE)
- CMS에서 발생하는 조각난 메모리의 증가 ( 메모리 파편화)에 대한 문제를 해결 할 수 있다. 