# StudyRxSwift
Study RxSwift with KxCoding

### Hello, RxSwift
#### 1/98 Hello, RxSwift
- RxSwift를 공부하는 순서
<br>1. Swift Language
<br>2. Functional Programming, Protocol Oriented Programming
<br>3. RxSwift

- RxSwift의 장점
<br> 단순하고 직관적인 코드를 작성할 수 있다

- RxSwift 공부하는 법
<br> 처음부터 RxSwift가 무엇인지 명확히 이해하는 것은 불가능하다.
<br> 처음부터 이해하려고 하면 지친다. 코드를 통해 작은 것부터 하나씩 배워 나가라.
<br> 처음부터 이해되는 것은 말이 안 되니 이해 안 되는 부분은 그냥 건너 뛰어라.


#### 2/98 Hello Reactive Programming
- RxSwift로 코드를 작성하면 값이나 상태의 변화에 따라서 새로운 결과를 도출하는 코드를 비교적 쉽게 작성할 수 있다
  - 반응형 프로그래밍(Reactive Programming)
  
### Key Concepts
#### 3/98 Observable and Obervers #1
- 가장 중요한 Observable
<br> Observable은 Observable Sequence 또는 Sequence라고 부른다
<br> Observable은 Event를 전달한다
<br> Observer는 Observable을 감시하고 있다가(Subscribe) 전달되는 Event를 처리한다
<br> Observer를 구독자(Subscriber)라고도 한다
- Observable은 세 가지 Event를 전달한다(Next, Error, Completed)
  - Observable에서 발생한 새 Event는 Next Event를 통해서 구독자로 전달된다
  - Event에 값이 포함되어 있다면 Next와 함께 전달된다
  - RxSwift에서는 이를 Emission(방출, 배출)이라고 한다
  - Observable에서 error가 발생하면 Error event가 작동되고, 정상적으로 작동되면 Completed가 작동된다
  - Error event나 Completed event는 Emission이라고 하지 않고 Notification이라고 부른다
  - Observable의 Life cycle은 Completed에서 종료된다
  
- Observable을 생성하는 두 가지 방법
  - (1) Create 연산자를 통해서 Observable의 동작을 직접 구현하는 방법
    - Create 연산자는 Observable Protocol type에 선언되어 있는 Type 메소드이다
    - RxSwift에서는 이런 메소드들을 연산자라고 부른다
    - Create 연산자는 하나의 클로저를 파라미터로 받는다 (Observer를 받아서 Disposable을 리턴한다)
    - Disposable은 메모리 정리에 필요한 객체이다.
<br>
<pre>
<code>
Observable<Int>.create { (observer) -> Disposable in
  observer.on(.next(0))
  observer.onNext(1)
  
  observer.onCompleted()
  
  return Disposables.create()
}
</code>
</pre>

  - (2) Create가 아닌 다른 연산자를 사용하는 방법
    - 미리 정의된 규칙에 따라 이벤트를 전달하는 from 연산자
    - from 연산자를 활용해서 create 연산자로 만든 Observable과 동일한 이벤트를 전달하는 Observable을 생성한다
    - from 연산자는 파라미터로 전달한 배열에 있는 요소를 순서대로 방출하고 Completed 이벤트를 전달하는 Observable을 생성한다
    - 단순히 순서대로 방출되는 Observable을 생성할 때는 create 연산자로 직접 구현하는 것보다 from을 활용하는 것이 좋다
    - Observable이 생성된 상태일 뿐 방출되거나 event가 전달되지 않는다
    - Observable은 event가 어떤 순서로 전달되어야 하는지 정의할 뿐이다
    - 구현했던 클로저가 실행되거나 from 연산자로 만든 Observable에서 event가 전달되는 시점은 Observer가 Observable을 구독하는 시점이다
    - 이 시점에 Next event를 통해서 두 개의 정수가 순서대로 방출되고 이어서 completed event가 전달된다
<br>
<pre>
<code>
Observable.from([0, 1])
</code>
</pre>

#### 4/98 Observable and Obervers #2
- 실제로 event가 전달되는 시점은 observer가 구독을 시작하는 시점
  - Observer는 Observable에서 전달되는 event를 처리한다
  - 이것을 구독한다고 표현한다
  - 그래서 observer를 구독자라고 표현하기도 한다
- observer가 구독을 시작하는 방법
  - Observable에서 subscribe 메소드를 호출하는 것
  - subscribe 메소드는 Observable과 observer를 연결한다
  - 두 요소를 연결해야 이벤트가 전달되므로 Rx에서 가장 기초적이고 필수적이다
  
<br>
<pre>
<code>
let o1 = Observable<Int>.create { (observer) -> Disposable in
   observer.on(.next(0))
   observer.onNext(1)
   
   observer.onCompleted()
   
   return Disposables.create()
}

// 첫번째 방법
o1.subscribe { 
  print($0)
  if let elem = $0.element {
      print(elem)
  }
}
--> 출력결과
next(0)
0
next(1)
1
completed

// 두번째 방법
o1.subscribe(onNext: { elem in
   print(elem)
})
--> 출력결과
0
1

</code>
</pre>


- observer는 동시에 2개 이상의 event를 처리하지 않는다
  - Observable은 observer가 하나의 event를 처리한 후에 이어지는 event를 전달한다
  - 여러 event를 동시에 전달하지 않는다


#### 5/98 Disposables

<pre>
<code>
Observable.from([1, 2, 3])
  .subsribe(onNext: { elem in
    print("Next", elem)
  }, onError: { error in
    print("Error", error)
  }, onCompleted: {
    print("Completed")
  }, onDisposed: {
    print("Disposed")
  })
출력결과
  --> 
  Next 1
  Next 2
  Next 3
  Completed
  Disposed
  
</code>
</pre>

- Dispose는 Observable이 전달하는 event는 아니다
- 파라미터로 클로저를 전달하면 Observable과 관련된 모든 리소스가 제거된 후에 호출된다
- Observable이 Completed나 Error event로 해제되었다면 리소스도 해제된다
- 그럼에도 되도록 해제할 때는 Disposable을 사용하는 것이 좋다
- dispose()를 직접 호출하기보다는 disposeBag을 사용하는 것이 좋다
- dispose() 메소드를 직접 호출하면 completed event가 전달되지 않으므로, 직접 사용하지 않는 것이 좋다
- 만약 특정 시점에 해지하고 싶다면 take until 메소드를 사용하는 것이 좋다


#### 6/98 Operators

- Observable과 관련된 여러 메소드들을 연산자라고 부른다
- 연산자의 특징
  - 연산자들은 Observable 상에서 동작하고, 새로운 Observable을 리턴한다
  - Observable을 리턴하기 때문에 두 개 이상의 연산자를 연달아 호출할 수 있다
  - 연산자는 보통 subscribe 메소드 앞에 추가한다
  - 그래야 구독자로 전달된 최종 데이터가 우리가 원하는 데이터가 된다
  - 연산자를 호출할 때는 호출 순서에 유념해야 한다 -> 호출 순서에 따라 다른 결과가 나올 수 있다

<pre>
<code>
let bag = DisposeBag()
Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9])
    .take(5) // take 연산자는 소스 옵저버블이 방출하는 요소 중에서 파라미터로 지정한 수만큼 방출하는 새로운 옵저버블을 생성한다 -> 처음 5개의 요소만 전달한다
    .filter { $0.isMultiple(of: 2) } // 짝수만 구독자로 전달 -> 1~5 사이의 짝수인 2와 4만 전달됨
    .subscribe { print($0) }
    .disposed(by: bag)
--> 출력결과
next(2)
next(4)
Completed
</code>
</pre>

### Subjects
#### 7/98 Subject Overview
- Subject를 이해하기 위해서는 Observer와 Observable에 대해 이해해야 한다
- Observable은 이벤트를 전달한다
- Observer는 Observable을 구독하고 전달되는 이벤트를 처리한다
- Observable은 Observer와 달리 다른 Observable을 구독하지 못한다
- 마찬가지로 Observer는 다른 Observer로 이벤트를 전달하지 못한다
- 반면 Subject는 다른 Observable로부터 이벤트를 받아서 구독자로 전달할 수 있다
- 다시 말해 Subject는 Observable인 동시에 Observer이다
- RxSwift는 네 가지 Subject를 제공한다
  - PublishSubject: Subject로 전달되는 새로운 이벤트를 구독자로 전달한다
  - BehaviorSubject: 생성시점의 시작 이벤트를 지정한다 그리고 Subject로 전달되는 이벤트 중에서 가장 마지막에 전달된 최신 이벤트를 저장해 두었다가 새로운 구독자에게 최신 이벤트를 전달한다
  - ReplaySubject: 하나 이상의 최신 이벤트를 버퍼에 저장한다. Observer가 구독을 시작하면 버퍼에 있는 모든 이벤트를 전달한다
  - AsyncSubject: Subject로 Completed 이벤트가 전달되는 시점에 마지막으로 전달된 Next 이벤트를 구독자로 전달한다
- RxSwift는 Subject를 래핑하고 있는 두 가지 Relay를 제공한다 -> 이전 버전에서 제공되던 Variable이 Relay로 대체 됐다
  - PublishRelay: PublishSubject를 래핑한 것
  - BehaviorRelay: BehaviorSubject를 래핑한 것
  - Relay는 일반적인 Subject와 달리 Next 이벤트만 받고 나머지 Completed 이벤트와 Error 이벤트는 받지 않는다
  - 주로 종료 없이 계속 전달되는 이벤트 시퀀스를 처리할 때 활용한다
 
 
#### 8/98 Publish Subject
- PublishSubject는 Subject로 전달되는 이벤트를 Observer에게 전달하는 가장 기본적인 형태의 Subject이다
- Subject는 Observable인 동시에 Observer이다
  - 다른 Source로부터 이벤트를 전달받을 수 있고, 다른 Observer로 이벤트를 전달할 수 있다
  - Observable에서 Observer로 Next 이벤트를 전달할 때 Observer로 onNext 메소드를 호출하고, 파라미터로 요소를 전달한다
  - Subject 역시 Observer이기 때문에 onNext를 호출할 수 있다
- PublishSubject는 구독 이후에 전달되는 새로운 이벤트만 구독자로 전달한다
- PublishSubject는 이벤트가 전달되면 즉시 구독자에게 전달한다. 그래서 Subject가 최초로 생성되는 시점과 첫 번째 구독이 시작되는 시점 사이에 전달되는 이벤트는 그냥 사라진다.
  - 이벤트가 사라지는 것이 문제가 된다면 Replay Subject를 사용하거나 Hold Observable을 사용한다


#### 9/98 Behavior Subject
- Behavior Subject는 Publish Subject와 유사한 방식으로 동작한다
  - Subject로 전달된 이벤트를 구독자로 전달하는 것은 동일하다
  - 하지만 Subject를 생성하는 방식에 차이가 있다
    - Behavior Subject를 생성할 때는 Publish Subject와 다르게 하나의 값을 전달한다
    - 숫자를 전달하는 이유는 타입 파라미터가 Int로 선언되었기 때문
    - 또 다른 차이는 Subject를 구독할 때 나타난다
    - Publish Subject는 내부에 이벤트가 저장되지 않은 상태로 생성된다
    - 그래서 Subject로 이벤트가 전달되기 전까지 구독자로 이벤트가 전달되지 않는다
    - Behavior Subject를 생성하면 내부에 Next 이벤트가 생성되고, 생성자로 전달한 값이 저장된다
    - 새로운 구독자가 추가되면 저장되어 있던 Next 이벤트가 바로 전달된다
    - 다시 Behavior Subject로 Next 이벤트를 전달하면 Observer로 Next 이벤트가 전달된다
    - 이 시점에 새로운 Observer가 추가되면 가장 최신 Next 이벤트를 Observer로 전달한다
    
    
#### 10/98 Replay Subject
- Behavior Subject는 가장 최근 Next 이벤트 하나를 저장했다가 새로운 구독자로 전달한다
  - 최신 이벤트를 제외한 나머지 모든 이벤트는 사라진다
- 두 개 이상의 이벤트를 저장해두고 새로운 구독자로 전달하고 싶다면 Replay Subject를 사용한다
- Replay Subject는 create 메소드로 생성한다
  - Subject를 생성할 때 buffer의 크기를 지정하는데(bufferSize), bufferSize를 3으로 하면 세 개의 이벤트를 저장하는 buffer가 생성된다
  - 이 때 전달되는 것은 가장 마지막에서부터 세 개의 이벤트를 전달한다
- Replay Subject는 지정된 buffer 크기만큼 최신 이벤트를 저장하고 새로운 구독자에게 전달한다
- buffer는 메모리에 저장되기 때문에 항상 메모리 사용량에 신경 써야 한다
- 필요 이상으로 큰 buffer를 쓰는 것은 피해야 한다
- Replay Subject는 종료 여부에 관계 없이 항상 buffer에 저장돼 있는 이벤트를 새로운 구독자에게 전달한다


#### 11/98 Async Subject
- Async Subject는 이전의 Subject들과 이벤트를 전달하는 시점에 차이가 있다
- Publish Subject, Behavior Subject, Replay Subject는 Subject로 이벤트가 전달되면 즉시 구독자에게 전달한다
- 반면 Async Subject는 Subject로 Completed 이벤트가 전달되기 전까지 어떤 이벤트도 구독자로 전달하지 않는다
- Completed 이벤트가 전달되면 그 시점에 가장 최근에 전달된 넥스트 이벤트 하나를 구독자에게 전달한다
- Async Subject는 Completed 이벤트가 전달된 시점을 기준으로 가장 최근에 전달된 하나의 Next 이벤트를 구독자에게 전달한다
- 만약 Async Subject로 전달된 Next 이벤트가 없다면 그냥 Completed 이벤트만 전달하고 종료한다
- Error 이벤트가 전달된 경우에는 Next 이벤트가 구독자에게 전달되지 않고, Error 이벤트만 전달된다


#### 12/98 Relays (0216 여기까지)
- RxSwift는 Publish Relay와 Behavior Relay를 제공한다
- Relay는 Subject와 유사한 특징을 가지고 있고, 내부에 Subject를 래핑하고 있다
- Publish Relay는 Publish Subject를 래핑하고 있고, Behavior Relay는 Behavior Subject를 래핑하고 있다
- Relay는 Subject와 마찬가지로 다른 Source로부터 이벤트를 받아서 구독자에게 전달한다
- 가장 큰 차이는 Next 이벤트만 전달한다는 것이다
- Completed 이벤트와 Error 이벤트는 전달 받지도 않고, 전달 하지도 않는다
- 그래서 Subject와 달리 종료되지 않는다
- 구독자가 Dispose 되기 전까지 계속 이벤트를 처리한다
- 그래서 주로 UI 이벤트 처리에 활용된다
- Relay는 RxSwift 프레임워크가 아닌 RxCocoa 프레임워크를 통해 제공된다
- Subject에서는 onNext를 사용하지만, Relay에서 Next이벤트를 전달할 때는 accept 메소드를 사용한다
- accept 메소드를 호출하고 값을 전달하면 구독자에게 Next 이벤트가 전달된다
- Behavior Relay는 Behavior Subject와 마찬가지로 하나의 값을 생성자로 전달한다
- Behavior Relay는 value라는 속성을 제공한다
  - Behavior Relay가 저장하고 있는 Next 이벤트에 접근해서 여기에 저장되어 있는 값을 리턴한다 
  - 이 속성은 읽기 전용이고, 이 안에 있는 값을 바꿀 수는 없다
  - 값을 바꾸고 싶다면, accept 메소드를 통해 새로운 넥스트 이벤트를 전달해야 한다



### Create Operators
#### 13/98 just, of, from (0217 여기부터)
- just는 하나의 항목을 방출하는 Observable을 생성한다
<pre>
<code>
let disposeBag = DisposeBag()
let element = "smile"

Observable.just(element)
  .subscribe { event in print(event) }
  .disposed(by: disposeBag)
==> 출력결과
next(smile)
completed

Observable.just([1, 2, 3])
  .subscribe { event in print(event) }
  .disposed(by: disposeBag)
==> 출력결과
next([1, 2, 3])
completed
</code>
</pre>
- just는 ObservableType 프로토콜에 Type 메소드로 선언되어 있다
- 파라미터로 하나의 요소를 받아서 Observable을 리턴한다
- from 연산자와 자주 혼동하게 되는데, just로 생성한 Observable은 파라미터로 전달한 요소를 그대로 방출한다는 사실을 기억하라
- 만약 두 개 이상의 요소를 방출할 Observable을 만들어야 한다면 just로는 불가능하고, of 연산자를 사용해야 한다
- of의 경우 가변 파라미터로 설정되어 있어서 여러 개의 값을 동시에 전달할 수 있다
<pre>
<code>
let disposeBag = DisposeBag()
let apple = "Apple"
let orange = "Orange"
let kiwi = "Kiwi"

Observable.of(apple, orange, kiwi)
  .subscribe { element in print(element) }
  .disposed(by: disposeBag)
==> 출력결과
next(Apple)
next(Orange)
next(Kiwi)
completed

Observable.of([1, 2], [3, 4], [5, 6])
  .subscribe { element in print(element) }
  .disposed(by: disposeBag)
==> 출력결과
next([1, 2])
next([3, 4])
next([5, 6])
completed
</code>
</pre>
- of 역시 ObservableType 프로토콜의 Type 메소드로 선언되어 있다
- 방출할 요소를 원하는 수만큼 전달할 수 있다
- 배열에 저장된 요소를 하나씩 방출하고 싶다면 from 연산자를 사용하면 된다
- from 역시 ObserableType 프로토콜의 Type 메소드로 선언되어 있다
- 첫 번째 파라미터로 배열을 받고, 리턴형은 배열이 아니라 배열에 포함된 요소이다
- 배열에 포함된 요소를 하나씩 순서대로 방출한다
- sequence 형식을 전달할 수도 있다
<pre>
<code>
let disposeBag = DisposeBag()
let fruits = ["Apple", "Kiwi", "Mango"]
Observable.from(fruits)
  .subscribe { element in print(element) }
  .disposed(by: disposeBag)
==> 출력결과
next("Apple")
next("Kiwi")
next("Mango")
completed
</code>
</pre>
- 하나의 요소를 방출하는 Observable을 생성할 때는 just 연산자를 사용한다
- 두 개 이상의 요소를 방출하는 Observable을 생성할 때는 of 연산자를 사용한다
- just와 of 연산자는 항목을 그대로 방출하기 때문에, 배열을 전달하면 배열이 방출된다
- 배열에 저장된 요소를 하나씩 방출하는 Observable이 필요하다면 from 연산자를 사용한다


#### 14/98 range, generate
- 정수를 지정된 수만큼 방출하는 Observable을 생성하기 위해서 range 연산자와 generate 연산자를 사용한다
- 첫 번째 파라미터에는 시작할 정수를 입력한다(실수를 입력하면 컴파일 에러가 발생한다)
- 두 번째 파라미터에는 방출할 정수의 수를 전달한다
<pre>
<code>
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 5)
  .subsribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
next(1)
next(2)
next(3)
next(4)
next(5)
completed
</code>
</pre>
- range 연산자는 시작 값에서 1씩 증가하는 squence를 생성한다
- 증가되는 크기를 바꾸거나 감소하는 sequence를 생성하는 것은 불가능하다
- 이걸 가능하게 하는 것이 generate 연산자이다
- generate 연산자는 총 네 개의 파라미터를 받는다(initialState, condition, scheduler, iterate)
- 첫 번째 파라미터인 initialState는 시작값을 전달한다. 즉, 가장 먼저 방출되는 값이 들어간다
- 두 번째 파라미터인 condition에 true를 리턴하는 경우에만 요소가 방출된다, false를 리턴하면 completed 이벤트를 전달하고 바로 종료한다
- 세 번째는 일단 무시
- 네 번째 파라미터인 iterate에는 값을 바꾸는 코드를 전달한다. 보통 값을 증가시키거나 감소시키는 코드를 전달한다
<pre>
<code>
let disposeBag = DisposeBag()

// iterate로 전달하는 식($0+2)은 값을 2씩 증가시키는 것을 의미
Observable.generate(initialState: 0, condition: { $0 <= 10 } , iterate: { $0 + 2 })
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
next(0)
next(2)
next(4)
next(6)
next(8)
next(10)
completed

let red = "RED"
let blue = "BLUE"

// 두 번째 파라미터 안의 { $0.count < 5 }는 문자열의 길이가 5보다 작을 때 true를 리턴함을 의미한다
// 세 번째 파라미터 안의 { $0.count.isMultiple(of:2) ? $0 + red : $0 + blue }는 현재 문자열 뒤에 다른 색의 문자열을 추가하도록 구현함을 의미
Observable.generate(initialState: red, condition: { $0.count < 5 }, iterate: { $0.count.isMultiple(of:2) ? $0 + red : $0 + blue })
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
next(RED)
next(RED,BLUE)
next(RED,BLUE,RED)
next(RED,BLUE,RED,BLUE)
completed
</code>
</pre>
- generate 연산자의 경우 range 연산자와 다르게 파라미터 형식이 정수로 제한되지 않는다


#### 15/98 repeatElement
- 동일한 요소를 반복적으로 방출하는 Observable을 생성할 수 있도록 해주는 연산자가 repeatElement이다
- repeatElement는 ObservableType 프로토콜의 Type 메소드로 선언되어 있다
- 첫 번째 파라미터로 요소를 전달하면 이 요소를 반복적으로 방출하는 Observable을 리턴한다
- 반복적의 의미는 설명에 나와 있는 대로(infinitely) 무한적으로 방출하는 것을 의미한다
<pre>
<code>
let disposeBag = DisposeBag()
let element = "Heart"
Observable.repeatElement(element)
  .subscribe { print($0) }
==> 출력결과
next(Heart)
next(Heart)
next(Heart)
... 무한반복

Observable.repeatElement(element)
  .take(4)
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
next("Heart")
next("Heart")
next("Heart")
next("Heart")
completed
</code>
</pre>
- repeatElement 연산자를 사용할 때는 방출되는 횟수를 제한해주는 것이 아주 중요하다
- take 연산자를 사용해서 방출 횟수를 지정할 수 있다

#### 16/98 deferred
- deferred의 뜻은 '연기'임
- deferred 연산자를 활용하면 특정 조건에 맞춰 Observable을 생성할 수 있다
- deferred 연산자는 Observable을 리턴하는 클로저를 파라미터로 받는다
<pre>
<code>
let disposeBag = DisposeBag()
let animals = ["Dog", "Cat", "Fox"]
let fruits = ["Apple", "Mango", "Kiwi"]
var flag = true

let factory: Observable<String> = Observable.deferred {
  flag.toggle() // flag의 상태를 뒤집는 역할, 기존에 flag가 true였기 때문에 toggle 이후에는 false가 된다
  // flag에 true가 저장되어 있다면 animals 배열에 있는 문자열을 방출하는 Observable을 리턴
  if flag {
    return Observable.from(animals) // from 연산자를 사용하면 배열 내 요소들을 순차적으로 방출할 수 있다
  } else { // flag에 false가 저장되어 있다면 fruits 배열에 있는 문자열을 방출하는 Observable을 리턴
    return Observable.from(fruits) 
  }
}

factory
  .subscribe { print($0) }
  .dispose(by: disposeBag)
// 한 번 더 호출해서 flag 값을 다시 true로 변경
factory
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next("Apple")
next("Mango")
next("Kiwi")
completed
next("Dog")
next("Cat")
next("Fox")
completed
</code>
</pre>

#### 17/98 create
- Observable이 동작하는 방식을 직접 구현하고 싶다면 create 연산자를 사용해야 한다
<pre>
<code>
let disposeBag = DisposeBag()
enum MyError: Error {
  case error
}

Observable<String>.create { (observer) -> Disposable in
  guard let url = URL(string: "https://www.apple.com" 
    else { 
    // 여기서의 옵저버는 클로저로 전달된 파라미터를 의미함
    observer.onError(MyError.error)
    return Disposables.create() // Disposable이 아니라 Disposables로 써야 한다
  }
  
  guard let html = try? String(contentsOf: url, encoding: .utf8) else {
    observer.onError(MyError.error)
    return Disposables.create()
  }
  // 문자열이 정상적으로 저장되었다면 Observer로 전달한다 -> 문자열을 방출한다
  observer.onNext(html) // 요소를 방출할 때는 onNext를 사용한다
  observer.onCompleted() // observer로 completed 이벤트가 전달된다
  
  return Disposables.create() // 마지막으로 Disposable을 생성해서 리턴해주면 모든 리소스가 정리되고 Observable이 정상적으로 종료된다
}
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
(html 내용이 출력됨)
</code>
</pre>
- Observable을 종료하기 위해서는 onError 또는 onCompleted 메소드를 반드시 호출해야 한다
- 둘 중 하나라도 호출하면 Observable이 종료되기 때문에, 그 이후에 onNext를 호출하면 요소가 방출되지 않는다
- onNext를 호출하려면 onCompleted() 메소드 또는 onError() 전에 호출해야 한다



#### 18/98 empty, error(2020/02/18 여기까지)
- 두 연산자가 생성한 Observable은 next 이벤트를 전달하지 않는다는 공통점이 있다
- 둘 다 어떠한 요소도 방출하지 않는다
- empty 연산자는 completed 이벤트를 전달하는 Observable을 생성한다 
- 요소를 방출하지 않기 때문에 요소의 형식은 중요하지 않다
- empty 연산자는 파라미터가 없다
- observer가 아무런 동작 없이 종료돼야 할 때 사용한다
<pre>
<code>
let disposeBag = DisposeBag()

Observable<Void>.empty()
    .subscribe { print($0) }
    .disposed(by: disposeBag)
==> 출력결과
completed
</code>
</pre>

- error 연산자는 error 이벤트를 전달하고 종료한다
- error 연산자는 파라미터로 error를 받는다
<pre>
<code>
let disposeBag = DisposeBag()
enum MyError: Error {
  case error
}

Observable<Void>.error(MyError.error)
    .subscribe { print($0) }
    .disposed(by: disposeBag)
==> 출력결과
error(error)
</code>
</pre>

### Filtering Operators
#### 19/98 ignoreElementsOperator (2020/02/19 여기부터)
- ignoreElements는 Observable이 방출하는 next 이벤트를 필터링하고 completed 이벤트와 error 이벤트만 구독자로 전달한다
- ignoreElements는 파라미터를 받지 않는다
- 리턴형은 Completable인데, Completable은 트레이츠라고 부르는 특별한 Observable이다
- Completable은 completed 또는 error 이벤트만 전달하고 next 이벤트는 무시한다
- 주로 작업의 성공과 실패에만 관심이 있을 때 사용한다
<pre>
<code>
let disposeBag = DisposeBag()
let fruits = ["Apple", "Pear", "Grape"]

Observable.from(fruits)
  .ignoreElements()
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
completed
</code>
</pre>

#### 20/98 elementAt Operator
- elementAt은 특정 인덱스에 위치한 요소를 제한적으로 방출한다
- elementAt은 정수 인덱스를 파라미터로 받아서 Observable을 리턴한다
- 해당 인덱스의 요소를 방출하고 completed 이벤트를 전달받는다
<pre>
<code>
let disposeBag = DisposeBag()
let fruits = ["Apple", "Pear", "Grape"]

Observable.from(fruits)
  .elementAt(1)
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next(Pear)
completed
</code>
</pre>


#### 21/98 filter Operator
- filter 연산자는 클로저를 파라미터로 받는다
- 클로저는 predicate로 사용된다 (predicate는 '서술하다', '단언하다', '주장하다', '속성'의 뜻을 가짐)
- true를 리턴하는 요소가 연산자가 리턴하는 Observable에 포함된다
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .filter { $0.isMultiple(of: 2) }
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next(2)
next(4)
next(6)
next(8)
next(10)
completed
</code>
</pre>


#### 22/98 skip, skipWhile, skipUntil Operator
- skip 연산자를 통해 특정 요소를 무시할 수 있다
- skip 연산자는 정수를 파라미터로 받는다
- Observable이 방출하는 요소 중에서 지정된 수만큼 무시한 다음에 이후에 방출되는 요소만 구독자로 전달한다
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .skip(3) // 인덱스가 아니라 count로 사용되는 것이다 -> 인덱스였으면 5부터 출력됐을 것이다
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next(4)
next(5)
next(6)
next(7)
next(8)
next(9)
next(10)
completed
</code>
</pre>

- skipWhile은 클로저를 파라미터로 받는다 
- 이 클로저는 filter 연산자와 마찬가지로 predicate로 사용되고, 클로저에서 true를 리턴하는 동안 방출되는 요소를 무시한다
- 클로저에서 false를 리턴하면 그때부터 요소를 방출하고, 이후에는 조건에 관계 없이 모든 요소를 방출한다
- 연산자는 방출되는 요소를 포함한 Observable을 리턴한다

<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .skipWhile { !$0.isMultiple(of: 2) } // skipWhile 연산자에 2가 전달되면 false가 리턴되고 이때부터 이어지는 모든 요소가 방출된다(구독자로 전달된다)
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next(2)
next(3)
next(4)
next(5)
next(6)
next(7)
next(8)
next(9)
next(10)
completed
</code>
</pre>

- skipUntil 연산자는 Observable 타입을 파라미터로 받는다
- 다른 Observable을 파라미터로 받고, 이 Observable이 next 이벤트를 전달하기 전까지, 원본 Observable이 전달하는 이벤트를 무시한다
- 이런 특징 때문에 파라미터로 전달되는 Observable을 trigger라고 부르기도 한다

<pre>
<code>
let disposeBag = DisposeBag()

let subject = PublishSubject<Int>()
let trigger = PublishSubject<Int>()

subject.skipUntil(trigger)
  .subscribe { print($0) }
  .dispose(by: disposeBag)
  
subject.onNext(1)
// 아직 trigger가 요소를 방출한적이 없기 때문에 subject가 방출한 요소는 구독자로 전달되지 않는다

trigger.onNext(0)
// 이번에는 trigger에서 요소를 방출하고 있다. 그런데 subject가 이전에 방출했던 요소는 여전히 구독자로 전달되지 않는다
// skipUntil은 trigger가 요소를 방출한 이후부터 원본 Observable에서 방출되는 요소들을 구독자로 전달한다

subject.onNext(2)

==> 출력결과
next(2)
completed
</code>
</pre>



#### 23/98 take, takeWhile, takeUntil, takeLast Operator
- take 연산자를 통해 요소의 방출 조건을 다양하게 구성할 수 있다
- take 연산자는 네 가지 형태가 있다(take, takeWhile, takeUntil, takeLast)
- take 연산자는 정수를 파라미터로 받아서 해당 숫자만큼만 요소를 방출한다
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .take(5)
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
==> 출력결과
next(1)
next(2)
next(3)
next(4)
next(5)
completed
</code>
</pre>

- takeWhile 연산자는 클로저를 파라미터로 받아서 predicate로 사용한다
- true를 리턴하면 구독자에게 전달된다 -> 요소를 방출한다
- 연산자가 리턴하는 Observable에는 최종적으로 조건을 만족하는 요소들만 포함된다

<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .takeWhile { !$0.isMultiple(of: 2) }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
==> 출력결과
next(1) // 이후에도 홀수가 방출되지만 구독자로는 전달되지 않는다 (takeWhile 연산자는 클로저가 false를 리턴하면 더 이상 요소를 방출하지 않는다)
completed
</code>
</pre>

- takeUntil 연산자
- takeUntil 연산자는 ObservableType을 파라미터로 받는다
- Observable을 파라미터로 받는다는 의미이다
- 파라미터로 전달한 Observable에서 next 이벤트를 전달하기 전까지 원본 Observable에서 next 이벤트를 전달한다
- trigger 상수에 저장된 Observable을 파라미터로 전달하는 예시
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

let subject = PublishSubject<Int>()
let trigger = PublishSubject<Int>()

subject.takeUntil(trigger)
  .subscribe { print($0) }
  .dispose(by: disposeBag)

subject.onNext(1)
subject.onNext(2)

trigger.onNext(0) // 이때부터 요소가 방출되지 않는다

subject.onNext(3)
==> 출력결과
next(1)
next(2)
completed
</code>
</pre>

- takeLast 연산자는 정수를 파라미터로 받아서 Observable을 리턴한다
- 리턴되는 Observable에는 원본 Observable이 방출한 요소 중에서 마지막으로 방출한 n개의 요소가 포함된다
- takeLast 연산자의 가장 중요한 점은 구독자로 전달되는 시점이 딜레이 된다는 점이다

<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

let subject = PublishSubject<Int>()
subject.takeLast(2)
    .subscribe { print($0) }
    .dispose(by: disposeBag)

numbers.forEach { subject.onNext($0) }

subject.onNext(11) // 끝에 두 요소가 10, 11로 바뀐다
subject.onCompleted()

==> 출력결과
next(10)
next(11)
completed
</code>
</pre>


#### 24/98 single Operator
- single 연산자는 원본 Observable에서 첫 번째 요소만 방출하거나, 조건과 일치하는 첫 번째 요소만 방출한다
- 두 개 이상의 요소가 방출되는 경우 error가 발생한다
- single 연산자는 단 하나의 요소만 방출해야 정상적으로 종료된다
- 원본 Observable이 아무 요소도 방출하지 않거나, 두 개 이상의 요소를 방출하는 경우 error가 발생한다
- single 연산자는 아무 파라미터가 없는 연산자와, predicate를 파라미터로 받은 연산자 두 경우가 있다
- single 연산자는 새로운 요소를 방출하면 구독자에게 바로 전달된다
- 다른 요소가 방출될 수도 있으므로 single 연산자가 리턴하는 Observable은 원본 Observable에서 completed 이벤트가 전달할 때까지 대기한다
- completed 이벤트가 전달되는 시점에 하나의 요소만 방출된 시점이라면 구독자에게 completed 이벤트가 전달되고, 그 사이에 다른 요소가 방출되었다면 구독자에게는 error 이벤트가 전달된다. 이와 같은 방식을 통해 하나의 요소만 방출되는 것을 보장받을 수 있다.
<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.just(1)
  .single()
  .subscribe { print($0) }
  .dispose(by: disposeBag)
 
Observable.form(numbers)
  .single()
  .subscribe { print($0) }
  .disposed(by: disposeBag)

Observable.from(numbers)
  .single { $0 == 3 }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
let subject = PublishSubject<Int>()
  
subject.single()
  .subscribe { print($0) }
  .disposed(by: disposeBag)

subject.onNext(100)

==> 출력결과
next(1)
completed
next(1)
error(Sequence contains more than one element.)
next(3)
completed
completed
next(100)
</code>
</pre>


#### 25/98 distinctUntilChange Operator
- distinctUntilChange 연산자는 동일한 항목이 연속적으로 방출되지 않도록 필터링 해준다
- 이 연산자는 파라미터가 없다.
- 원본 Observable에서 전달되는 두 개의 요소를 순서대로 비교한 다음에 이전 요소와 동일하면 방출하지 않는다
- 두 개의 요소를 비교할 때는 비교 연산자를 활용해 비교한다

<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 1, 3, 2, 2, 3, 1, 5, 5, 7, 7, 7]

Observable.from(numbers)
  .distinctUntilChanged()
  .subscribe { print($0) }
  .dispose(by: disposeBag)
==> 출력결과
next(1)
next(3)
next(2)
next(3)
next(1)
next(5)
next(7)
completed
</code>
</pre>

2020/02/19 여기까지

#### 26/98 debounce, throttle Operator (2020/02/20 여기부터)
- 두 연산자는 짧은 시간동안 반복적으로 방출되는 이벤트를 제어한다는 공통점이 있다
- 연산자로 전달하는 파라미터도 동일하다
- 하지만 연산의 결과는 완전히 다르다
- debounce 연산자는 두 개의 파라미터를 받는다
  - 첫 번째 파라미터(dueTime: RxTimeInterval)에는 시간을 전달한다
  - 이 시간은 연산자가 next 이벤트를 방출할지 결정하는 조건으로 사용된다
  - Observer가 next 이벤트를 방출한 다음 지정된 시간 동안 다른 next 이벤트를 방출하지 않는다면 해당 시점에 가장 마지막으로 방출한 next 이벤트를 구독자에게 전달한다
  - 반대로 지정된 시간 이내에 또 다른 next 이벤트를 방출했다면 타이머를 초기화한다 (이해하는 것이 중요한 부분)
  - 타이머를 초기화한 다음에 다시 지정된 시간 동안 대기한다
  - 이 시간 이내에 다른 이벤트가 방출되지 않는다면 마지막 이벤트를 방출하고 이벤트가 방출된다면 타이머를 다시 초기화한다
  - 두 번째 파라미터(scheduler: SchedulerType)에는 타이머를 실행할 스케줄러를 전달한다

<pre>
<code>
let disposeBag = DisposeBag()
let buttonTap = Observable<String>.create { observer in
  DispatchQueue.global().async {
    for i in 1...10 {
      observer.onNext("Tap \(i)")
      Thread.sleep(forTimeInterval: 0.3) // 0.3초 주기로 10번 방출
    }
    Thread.sleep(forTimeInterval: 1) // 1초 동안 쓰레드 중지
    
    for i in 11...20 {
      observer.onNext("Tap \(i)")
      Thread.sleep(forTimeInterval: 0.5) // 0.5초 주기로 10번 방출
    }
    
    observer.onCompleted()
  }
  
  return Disposables.create {
  
  }
}

buttonTap
  .debounce(.milliseconds(1000), scheduler: MainScheduler.instance) // 1초의 debounce
  .subscribe { print($0) }
  .disposed(by: disposeBag)  
==> 출력결과
next(Tap 10)
next(Tap 20)
completed
// debounce 연산자는 지정된 시간 동안 새로운 이벤트가 방출되지 않으면 가장 마지막에 방출된 이벤트를 구독자에게 전달한다 -> 타이머가 만료되기 전에 새로운 이벤트가 방출되었기 때문에 타이머를 초기화 함
</code>
</pre>


- throttle 연산자 (스로틀이라는 용어는 엔진의 힘이나 속도가 규제되는 구조라는 의미)
- throttle 연산자는 세 개의 파라미터(dueTime: RxTimeInterval, latest: Bool, scheduler: SchedulerType)를 받는다
- 기본값을 가진 두 번째 파라미터는 생략하는 경우가 많기 때문에 debounce와 파라미터가 동일하다고 생각해도 무방하다
- 첫 번째 파라미터에는 반복 주기를 전달하고, 세 번째 파라미터에는 스케줄러를 전달한다
- throttle은 지정된 주기 동안 하나의 이벤트만 구독자에게 전달한다
- 보통 두 번째 파라미터는 기본값을 사용하는데, 주기를 엄격하게 지킨다
- 두 번째 파라미터에 false를 부여하면, 반복 주기가 경과한 다음 가장 먼저 방출되는 이벤트를 구독자에게 전달한다

<pre>
<code>
let disposeBag = DisposeBag()
let buttonTap = Observable<String>.create { observer in
  DispatchQueue.global().async {
    for i in 1...10 {
      observer.onNext("Tap \(i)")
      Thread.sleep(forTimeInterval: 0.3)
    }
    Thread.sleep(forTimeInterval: 1)
    for i in 11...20 {
      observer.onNext("Tap \(i)")
      Thread.sleep(forTimeInterval: 0.5)
    }
    observer.onCompleted()
  }
  return Disposables.create()
}

buttonTap
  .throttle(.milliseconds(1000), scheduler: MainScheduler.instance)
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력 결과
next(Tap 1)
next(Tap 4)
next(Tap 7)
next(Tap 10)
next(Tap 11)
next(Tap 12)
next(Tap 14)
next(Tap 16)
next(Tap 18)
next(Tap 20)
completed
</code>
</pre>

- throttle 연산자는 next 이벤트를 지정된 주기마다 하나씩 구독자에게 전달한다
- 반면 debounce 연산자는 next 이벤트가 전달된 다음 지정된 시간이 경과하기까지 다른 이벤트가 전달되지 않는다면 마지막으로 방출된 이벤트를 구독자에게 전달한다
- 짧은 시간 동안 반복되는 tap 이벤트나 delegate 이벤트를 처리할 때는 throttle을 사용하고, debounce는 주로 검색 기능을 구현할 때 사용한다
- debounce를 활용해 사용자가 짧은 시간동안 연속적으로 타이핑을 할 때는 검색작업을 실행하지 않다가, 타이핑을 멈추면 검색을 실행한다
- toArray 연산자는 별도의 파라미터를 받지는 않는다
- 하나의 요소를 방출하거나 error 이벤트를 방출하는 ObservableType의 메소드이다
- 하나의 요소를 방출하고 바로 종료한다

### Tranforming Operators
#### 27/98 toArray Operator
- toArray 연산자는 Observable이 방출하는 모든 요소를 배열에 담은 다음, 이 배열을 방출하는 Observable을 생성한다

<pre>
<code>
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

let subject = PublishSubject<Int>()
  
subject
  .toArray()
  .subscribe { print($0) }
  .disposed(by: dispseBag)
  
subject.onNext(1)
subject.onNext(2)
subject.onCompleted()
==> 출력결과
success([1,2])
</code>
</pre>


#### 28/98 map Operator (2020/02/21 여기부터)
- map 연산자는 Observable 배출하는 항목을 대상으로 함수를 실행하고 결과를 새로운 Observable로 배출한다
- map 연산자를 사용하다보면 파라미터와 동일한 형식을 리턴해야 한다고 생각하는 경우가 많지만 그런 제약은 없다
- map 연산자는 Observable이 방출하는 요소들을 대상으로 클로저를 실행하고 그 결과를 구독자에게 전달한다
- 클로저로 전달되는 파라미터의 형식은 소스 Observable이 방출하는 요소와 동일하다
- 하지만 클로저가 리턴하는 값의 방식은 고정되어 있지 않으며, 원하는 형식으로 리턴할 수 있다.
 
<pre>
<code>
let disposeBag = DisposeBag()
let skills = ["Siwft", "SwiftUI", "RxSwift"]

Observable.from(skills)
//  .map { "Hello, \($0)"}
  .map { $0.count }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
==> 출력결과
// next(Hello, Swift)
// next(Hello, SwiftUI)
// next(Hello, RxSwift)
next(5)
next(7)
next(7)
completed
</code>
</pre>


#### 29/98 flatMap Operator
- flatMap 연산자는 모든 Observable이 방출하는 항목을 모아서 최종적으로 하나의 Observable을 리턴한다
- 개별 항목이 개별 Observable로 변환되었다가 다시 하나의 Observable로 합쳐지기 때문에 처음에는 이해하기 어렵다
- 실제 프로젝트에서 활용해보면 금방 이해 된다
- flatMap 연산자는 클로저를 파라미터로 받는데, BehaviorSubject를 원하는대로 변환한 다음 새로운 Observable을 리턴해야 한다
- flatMap이 내부적으로 여러 개의 Observable을 생성하지만, 최종적으로 모든 Observable이 하나의 Observable로 합쳐지고, 방출되는 항목들이 순서대로 구독자에게 전달된다
- flatMap 연산자는 원본 Observable이 방출하는 항목을 새로운 Observable로 변환한다. 새로운 Observable은 항목이 업데이트 될 때마다 새로운 항목을 방출한다
- 이렇게 생성된 모든 Observable은 최종적으로 하나의 Observable로 합쳐지고, 모든 항목들이 이 Observable을 통해서 구독자로 전달 된다
- 단순히 처음에 방출된 항목만 구독자로 전달되는 것이 아니라 업데이트된 최신 항목도 구독자로 전달된다
- 이 연산자는 네트워크 요청을 구현할 때 자주 활용한다
<pre>
<code>
let disposeBag = DisposeBag()

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()
  
subject
  .flatMap { $0.asObservable() } // subject를 observable로 변경시킨다
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
subject.onNext(a)
subject.onNext(b)

a.onNext(11)
b.onNext(22)

==> 출력결과
next(1)
next(2)
next(11)
next(22)

</code>
</pre>


#### 30/98 flatMapFirst, flatMapLatest Operator (2020/02/22 여기부터)
- flatMap 연산자에서 파생된 연산자들이다
- flatMap 연산자에 대한 이해가 우선돼야 한다
- flatMapFirst의 연산자, 리턴형은 flatMap과 동일하다
- 하지만 연산자가 리턴하는 Observable에는 처음에 변환된 Observable이 방출하는 항목만 포함된다

<pre>
<code>
let disposeBag = DisposeBag()

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()
  
subject
  .flatMapFirst { $0.asObservable() }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
subject.onNext(a)
subject.onNext(b)

a.onNext(11)
b.onNext(22)
b.onNext(222)
a.onNext(111)

==> 출력결과
next(1) 
next(11)
next(111) // a에 대한 내용만 출력된다

</code>
</pre>

- flatMapLatest는 원본 Observable이 방출하는 항목을 Observable로 변환하는 것은 동일하다
- 모든 Observable이 방출하는 항목을 하나로 병합하지 않는다. 대신 가장 최근의 항목을 방출한 Observable을 제외한 나머지는 모두 무시한다

<pre>
<code>
let disposeBag = DisposeBag()

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()
  
subject
  .flatMapLatest { $0.asObservable() }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
  
subject.onNext(a)
a.onNext(11)
subject.onNext(b)
b.onNext(22)
a.onNext(11)

==> 출력결과
next(1) 
next(11)
next(2)
next(22) 
// a.onNext(11)은 구독자로 전달되지 않아 출력되지 않는다

</code>
</pre>

- flatMapLatest는 원본 Observable이 방출하는 요소를 새로운 Observable로 변환하고 가장 최근에 변환된 Observable이 방출하는 요소만 구독자에게 전달한다

#### 31/98 scan Operator
- scan 연산자는 accumulator function을 활용한다 (accumulator는 누산기, 축압기 등을 의미한다)
- 이 연산자는 기본값으로 연산을 시작하고, 원본 Observable이 방출하는 항목을 대상으로 변환을 실행한 다음 결과를 방출하는 하나의 Observable을 리턴한다
- 원본이 방출하는 항목의 수와 구독자로 전달되는 항목의 수가 동일하다
- 첫 번째 파라미터로 기본값을 전달하고, 두 번째 파라미터에는 클로저(두 개의 파라미터)를 전달한다
- 이 연산자는 작업 결과를 누적 시키면서 중간 결과와 최종 결과가 모두 필요한 경우에 사용한다

<pre>
<code>
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 10)
  .scan(0, accumulator: +)
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
next(1)
next(3) // 1 + 2
next(6) // 3 + 3
next(10) // 6 + 4
next(15) // 10 + 5
next(21) // 15 + 6
next(28) // 21 + 7
next(36) // 28 + 8
next(45) // 36 + 9
next(55) // 45 + 10
completed
</code>
</pre>


#### 32/98 buffer Operator
- buffer 연산자는 특정 주기 동안 옵저버블이 방출하는 항목을 수집하고 하나의 배열로 리턴한다
- RxSwift에서는 이런 동작을 컨트롤드 버퍼링이라고 한다
- 세 개의 파라미터(timeSpan: 항목을 수집할 시간(DispatchTimeInterval), count: 수집할 항목의 최대 숫자(Int), scheduler: SchedulerType)
- 연산자의 리턴형은 Type 파라미터가 배열로 선언되어 있다
- 지정된 시간동안 수집한 항목들을 배열에 담아서 리턴한다

<pre>
<code>
let disposeBag = DisposeBag()

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .buffer(timeSpan: .seconds(2), count: 3, scheduler: MainScheduler.instance)
  .take(5) // 5개만 방출
  .subscribe { print($0) }
  .disposed(by: disposerBag)
==> 출력 결과
next([0])
next([1, 2, 3])  
next([4, 5])
next([6, 7])
next([8, 9])
completed  
// Observable은 1초마다 항목을 방출하고 있고, buffer 연산자는 2초마다 세 개씩 수집하고 있다
// buffer 연산자는 첫 번째 파라미터로 전달한 timeSpan이 경과하면 수집된 항목들을 즉시 방출한다
// 두 번째 파라미터로 지정한 수만큼 수집되지 않았더라도 즉시 방출한다 (0만 출력되거나 1, 2, 3 세 개가 출력되는 경우)
</code>
</pre>



#### 33/98 window Operator
- window 연산자는 버퍼 연산자처럼 timeSpan과 maxCount를 지정해서 원본 Observable이 방출하는 항목들을 작은 단위의 Observable로 분해한다
- buffer와 달리 window 연산자는 수집된 항목을 방출하는 Observable을 리턴한다
- 리턴된 Observable이 무엇을 방출하고, 언제 방출하는지 이해하는 것이 중요하다
- 파라미터는 buffer와 똑같이 첫 번째로는 timeSpan, 두 번째로는 count, 세 번째로는 scheduler를 전달한다
- buffer 연산자와의 차이는 리턴형에 있다. buffer는 수집된 배열을 방출하는 Observable을 리턴, window 연산자는 Observable을 방출하는 Observable을 리턴
- Observable이 방출하는 Observable을 inner Observable이라고 한다
- inner Observable은 지정된 최대항목수만큼 방출하거나 지정된 시간이 경과하면 completed 이벤트를 전달하고 종료한다

<pre>
<code>
let disposeBag = DisposeBag()
Observable<Int>.interval(.seconds(1), scheduler: MainSchedulerType)
  .window(timeSpan: .seconds(5), count: 3, scheduler: MainScheduler.instance)
  .take(5)
  .subcribe { 
    print($0) 
    if let observable = $0.element {
      observable.subscribe { print(" inner: ", $0) }
    }
  }
  .disposed(by: disposeBag)
==> 출력결과
next(RxSwift.AddRef<Swift.Int>) // AddRef는 특별한 형태의 Observable이다. inner Observable이 바로 AddRef Observable이다. 구체적으로 공부할 필요는 없다. 구독할 수 있다는 걸 이해해라
  inner: next(0)
  inner: next(1)
  inner: next(2)
  inner: completed
next(RxSwift.AddRef<Swift.Int>)
  inner: next(0)
  inner: next(1)
  inner: next(2)
  inner: completed
next(RxSwift.AddRef<Swift.Int>)
  inner: next(3)
  inner: next(4)
  inner: next(5)
  inner: completed
next(RxSwift.AddRef<Swift.Int>)
  inner: next(6)
  inner: next(7)
  inner: next(8)
  inner: completed
next(RxSwift.AddRef<Swift.Int>)
  inner: next(9)
  inner: next(10)
  inner: next(11)
  inner: completed
next(RxSwift.AddRef<Swift.Int>)
  inner: next(12)
  inner: next(13)
  inner: next(14)
  inner: completed
completed
</code>
</pre>


#### 34/98 groupBy Operator
- groupBy 연산자는 Observable이 방출하는 요소를 원하는 기준으로 그루핑할 때 사용한다
- 파라미터로 클로저로 받고, 클로저는 요소를 파라미터로 받아서 키를 리턴한다
- 키의 형식은 hasable 프로토콜을 채용한 형식으로 한정되어 있다
- 연산자를 실행하면 클로저에서 동일한 값을 리턴하는 요소끼리 그룹으로 묶이고 그룹에 속한 요소들은 개별 Observable을 통해 방출된다
- 연산자가 리턴하는 Observable을 보면 TypeParameter가 grouped Observable로 선언되어 있다
- 방출하는 요소와 함께 키가 저장되어 있다


<pre>
<code>
let disposeBag = DisposerBag()
let words = ["Apple", "Banana", "Orange", "Book", "City", "Axe"]

Observable.from(words)
  .groupBy { $0.count }
  .subscribe(onNext: { groupedObservable in
    print("== \(groupedObservable.key)")
    groupedObservable.subscribe { print(" \($0)") }
  })
  .disposed(by: disposeBag)
==> 출력결과
// next(GroupedObservable<Int, String>(key: 5, source: RxSwift.(unkown context at $115767838) .GroupedObservableImpl<Swift.String>))
// next(GroupedObservable<Int, String>(key: 6, source: RxSwift.(unkown context at $115767838) .GroupedObservableImpl<Swift.String>))
// next(GroupedObservable<Int, String>(key: 4, source: RxSwift.(unkown context at $115767838) .GroupedObservableImpl<Swift.String>))
// next(GroupedObservable<Int, String>(key: 3, source: RxSwift.(unkown context at $115767838) .GroupedObservableImpl<Swift.String>)) 
== 5
  next(Apple)
== 6
  next(Banana)
  next(Orange)
== 4
  next(Book)
  next(City)
== 3
  next(Axe)
  completed
  completed
  completed
  completed

</code>
</pre>

- groupBy 연산자를 활용할 때는 보통 flatMap 연산자와 toArray 연산자를 활용해서 그루핑된 최종 결과를 하나의 배열로 방출하도록 구현한다

<pre>
<code>
let disposeBag = DisposerBag()
let words = ["Apple", "Banana", "Orange", "Book", "City", "Axe"]

Observable.from(words)
//  .groupBy { $0.count }
  .groupBy { $0.first ?? Character(" ) }
  .flatMap { $0.toArray() }
  .subscribe { print($0) }
  .disposed(by: disposeBag)
==> 출력결과
// next(["Book", "City"])
// next(["Axe"])
// next(["Apple"])
// next(["Banana", "Orange"])
next(["Banana", "Book"])
next(["City"])
next(["Apple", "Axe"])
next(["Orange"])
completed
</code>
</pre>

// 2020/02/22 여기까지

// 2020/02/23 여기부터

### Combining Operators
#### 35/98 startWith Operator
- Observable squence 앞에 새로운 요소를 추가하는 startWith 연산자
- Observable이 요소를 방출하기 전에 다른 항목들을 앞 부분에 추가한다
- 주로 기본값이나 시작값을 지정할 때 활용한다
- 파라미터는 가변 파라미터이다 -> 파라미터로 전달하는 하나 이상의 값을 Observable sequence 앞 부분에 추가한다. 그 다음 새로운 Observable을 리턴한다
- last in, first out의 특성이 있다
<pre>
<code>
let bag = DisposeBag()
let numbers = [1, 2, 3, 4, 5]

Observable.from(numbers)
  .startWith(0)
  .startWith(-1, -2)
  .startWith(-3)
  .subscribe { print($0) }
  .disposed(by: bag)
  
--> 출력결과
next(-3)
next(-1)
next(-2)
next(0)
next(1)
next(2)
next(3)
next(4)
next(5)
completed
</code>
</pre>



#### 36/98 concat Operator
- 두 개의 Observable을 연결하는 concat 연산자
- concat 연산자는 Type 메소드와 Instance 메소드로 구성돼 있다
- Type 메소드로 구현된 concat 연산자는 파라미터로 전달된 콜렉션에 있는 모든 Observable을 순서대로 연결한 하나의 Observable을 리턴한다
- Instance 메소드로 구현된 concat 연산자는 대상 Observable이 completed 이벤트를 전달한 경우에 파라미터로 전달한 Observable을 연결한다
- 만약 error 이벤트가 전달된다면 Observable은 연결되지 않는다
- 대상 Observable이 방출하는 요소만 전달되고, error 이벤트가 전달된 다음에 종료된다 -> 이건 Type 메소드로 구현된 concat 연산자도 마찬가지
- concat 연산자는 두 Observable을 단순하게 연결한다 -> 연결된 모든 Observable이 방출하는 요소들이 방출 순서대로 정렬되지는 않는다
- 이전 Observable이 모든 요소들을 방출하고 completed 이벤트를 전달해야 이어진 Observable이 방출을 시작한다


<pre>
<code>
let bag = DisposeBag()
let fruits = Observable.from(["Apple", "Kiwi", "Peach"])
let animals = Observable.from(["Dog", "Mouse", "Monkey"])
Observable.concat([fruits, animals])
  .subscribe { print($0) }
  .disposed(by: bag)
  
animals.concat(animals)
  .subscribe { print($0) }
  .disposed(by: bag)
--> 출력결과
// next("Apple")
// next("Kiwi")
// next("Peach")
// next("Dog")
// next("Mouse")
// next("Monkey")
// completed // completed는 연결된 옵저버블이 모든 요소를 방출한 후에 전달된다
next("Dog")
next("Mouse")
next("Monkey")
next("Apple")
next("Kiwi")
next("Peach")
completed
</code>
</pre>


#### 37/98 merge Operator
- 여러 Observable이 방출하는 이벤트를 하나의 Observable에서 방출하도록 병합하는 merge 연산자
- concat 연산자와 혼동하기 쉽지만 동작 방식이 다르다
- concat은 하나의 Observable이 모든 요소를 방출하고 completed 이벤트를 전달하면 이어지는 Observable을 연결
- merge는 두 개 이상의 Observable을 병합하고 모든 Observable에서 방출하는 요소들을 순서대로 방출하는 Observable을 리턴한다
- merge는 두 개 이상의 Observable이 방출하는 요소들을 병합한 하나의 Observable을 리턴한다
- 단순히 뒤에 연결하는 것이 아니라 하나의 Observable로 합쳐준다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let oddNumbers = BehaviorSubject(value: 1)
let evenNumbers = BehaviorSubject(value: 2)
let negativeNumbers = BehaviorSubject(value: -1)

let source = Observable.of(oddNumbers, evenNumbers, negativeNumbers)

source
  .merge(maxConcureent: 2) // 이렇게 하면 병합 가능한 Observable의 수가 2로 제한된다
  .subscribe { print($0) }
  .disposed(by: bag)

oddNumbers.onNext(3)
evenNumbers.onNext(4)

evenNumbers.onNext(6)
oddNumbers.onNext(5)

// oddNumbers.onCompleted() // oddNumbers는 이벤트 못 받는다
// oddNumbers.onError(MyError.error)
// evenNumbers.onNext(8)
// evenNumbers.onCompleted() // evenNumbers도 이벤트 못 받는다

negativeNumbers.onNext(-2) // maxConcurrent가 2이기 때문에 negativeNumber는 병합되지 않는다
oddNumbers.onCompleted()
--> 출력결과
next(1)
next(2)
next(3)
next(4)
next(6)
next(5)
// error(error) -> 8 전에 error가 전달 되었으므로 종료됨
//next(8)
// completed
next(-2) // oddNumbers에서 completed 이벤트를 전달하면 병합대상에서 oddNumbers가 제외되고, queue에 저장되어 있던 negativeNumbers가 병합대상에 추가된다(maxConcurrent는 2니까)
</code>
</pre>



#### 38/98 combineLatest Operator
- 소스 Observable이 방출하는 최신 요소를 병합하는 combineLatest 연산자
- combine은 결합한다는 의미이다
- 소스 Observable을 결합한 다음 파라미터로 전달한 함수를 실행하고 결과를 방출하는 새로운 Observable을 리턴한다
- 핵심은 연산자가 리턴한 Observable이 언제 이벤트를 방출하는지 이해하는 것이다
- 두 개의 Observable과 클로저를 파라미터를 받는다
- Observable이 Next 이벤트를 통해 전달하는 요소들은 클로저 파라미터를 통해 클로저에 전달된다
- 이후 클로저는 실행 결과를 리턴하고 연산자는 최종적으로 이 결과를 방출하는 Observable을 리턴한다
- 다양한 오버로딩이 있는데, 클로저를 전달하지 않는 경우에는 리턴 타입이 달라진다
- 파라미터로 전달한 Observable이 방출하는 요소들을 하나의 tuple로 합친 다음, 이 tuple을 방출하는 Observable을 리턴한다
- Observable을 최대 여덟개까지 전달할 수 있는 연산자들이 선언되어 있다
- 파라미터의 수만 다르고 동작 방식은 동일하다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let greetings = PublishSubject<String>()
let languages = PublishSubject<String>()
  
Observable.combineLatest(greetings, languages) { lhs, rhs
  -> String in
  return "\(lhs), \(rhs)"
}
  .subscribe { print($0) }
  .disposed(by: bag)
  
greetings.onNext("Hi")
languages.onNext("World!")

greetings.onNext("Hello")
languages.onNext("RxSwift")

// greetings.onCompleted()
greetings.onError(MyError.error)
languages.onNext("SwiftUI")

languages.onCompleted()

--> 출력결과
next(Hi World!)
next(Hello World!)
next(Hello RxSwift)
error(error) // 소스 Observable 중 error가 하나라도 전달되면 그 즉시 구독자에게 error 이벤트를 전달하고 종료한다
// next(Hello SwiftUI)
// completed

</code>
</pre>


#### 39/98 zip Operator
- Indexed Sequencing을 구현하는 zip 연산자
- combineLatest와 비교해서 이해하면 쉽게 이해할 수 있다
- zip 연산자는 소스 Observable이 방출하는 요소를 결합한다
- Observable을 결합하고 클로저를 실행한 다음 이 결과를 방출하는 result Observable을 리턴한다
- 집 연산자는 클로저에게 중복되는 요소를 전달하지 않고, index가 일치하는 짝을 전달한다
- 첫 번째 요소는 첫 번째 요소와 결합하고, 두 번째 요소는 두 번째 요소와 결합한다
- 소스 Observable이 방출하는 요소들을 순서를 일치시켜 결합하는 것을 Indexed Sequencing이라고 한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let numbers = PublishSubject<Int>()
let strings = PublishSubjct<String>()
  
Observable.zip(numbers, strings) { "\($0) - \($1)"}
  .subscribe { print($0) }
  .disposed(by: bag)
  
numbers.onNext(1)
strings.onNext("one")

numbers.onNext(2)
strings.onNext("two")

// numbers.onCompleted()
numbers.onError(MyError.error)

strings.onNext("three") // 짝이 없으므로 출력되지 않는다
strings.onCompleted()
--> 출력결과  
next(1 - one)
next(2 - two)
error(error) // 소스 Observable 중에서 하나라도 error 이벤트를 전달하면 즉시 구독자에게 error 이벤트가 전달되고 종료된다
// completed
</code>
</pre>


#### 40/98 withLatestFrom Operator
- trigger Observable이 Next 이벤트를 방출하면 data Observable이 가장 최근에 방출한 Next 이벤트를 구독자에게 전달하는 withLastestFrom 연산자
- 연산자를 호출하는 Observable을 trigger Observable이라고 부르고
- 파라미터로 전달하는 Observable을 data Observable이라고 부른다
- trigger Observable이 Next 이벤트를 방출하면 data Observable이 가장 최근에 방출한 next 이벤트를 구독자에게 전달한다
- 회원가입 기능을 구현할 때 사용할 수 있다
- 이 연산자는 두 가지 형태로 사용한다
- 첫 번째는 data Observable과 클로저를 파라미터로 받는다
- 클로저에는 두 Observable이 방출하는 요소가 전달되고 여기에서 결과를 리턴한다
- 연산자가 최종적으로 리턴하는 Observable은 클로저가 리턴하는 결과를 방출한다
- 두 번째 형태는 trigger Observable에서 Next 이벤트를 전달하면 파라미터로 전달한 data Observable에서 가장 최근에 방출한 넥스트 이벤트를 가져온다
- 이벤트에 포함된 요소를 방출하는 Observable을 리턴한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let trigger = PublishSubject<Void>()
let data = PublishSubject<String>()

trigger.withLatestFrom(data)
  .subscribe { print($0) }
  .disposed(by: bag)
  
data.onNext("Hello")
trigger.onNext(())
trigger.onNext(())

// data.onCompleted()
// data.onError(MyError.error)
// trigger.onNext(())
trigger.onCompleted()

--> 출력결과
next(Hello)
next(Hello)
// error(error) // completed와 달리 error는 바로 전달된다
// next(Hello) // completed가 아닌 next가 전달 된다
completed


</code>
</pre>



#### 41/98 sample Operator
- trigger Observable이 Next 이벤트를 전달할 때마다 data Observable이 Next 이벤트를 방출하지만, 동일한 Next 이벤트를 반복해서 방출하지 않는 sample 연산자
- dataObservable.withLatestFrom(triggerObservable) 과 같은 형태로 사용한다 -> withLatestFrom 연산자와 반대다
- data Observable에서 연산자를 호출하고 trigger Observable을 파라미터로 전달한다
- trigger Observable에서 Next 이벤트를 전달할 때마다 data Observable이 최신 이벤트를 방출한다
- 동일한 Next 이벤트를 반복해서 방출하지 않는다


<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let trigger = PublishSubject<Void>()
let data = PublishSubject<String>()
  
data.sample(trigger)
  .subscribe { print($0) }
  .disposed(by: bag)

trigger.onNext(())
data.onNext("Hello")

trigger.onNext(())
trigger.onNext(()) // 한 번 더 해도 추가로 Next 이벤트가 방출되지 않는다

// data.onCompleted()
// trigger.onNext(())

data.onError(MyError.error)

--> 출력결과
next(Hello)
error(error) // error는 trigger Observable이 Next 이벤트를 방출하지 않더라도 즉시 구독자에게 전달된다
// completed // sample 연산자는 completed 이벤트를 그대로 전달한다


</code>
</pre>

#### 42/98 switchLatest Operator
- 가장 최근에 방출된 Observable을 구독하고, 이 Observable이 전달하는 이벤트를 구독자에게 전달하는 switchLatest 연산자
- 가장 최근 Observable이 방출하는 이벤트를 구독자에게 전달한다
- 어떤 Observable이 가장 최근 Observable인지 이해하는 것이 핵심이다
- 이 연산자는 파라미터가 없다
- 주로 Observable을 방출하는 Observable에서 사용된다
- 소스 Observable이 가장 최근에 방출한 Observable을 구독하고 여기에서 전달하는 Next 이벤트를 방출하는 새로운 Observable을 리턴한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let a = PublishSubject<String>()
let b = PublishSubject<String>()
  
let source = PublishSubject<Observable<String>>()
  
source
  .switchLatest()
  .subscribe { print($0) }
  .disposed(by: bag)
  
a.onNext("1")
b.onNext("b")

source.onNext(a) // 이게 a를 최신 Observable로 만들어준다

a.onNext("2")
b.onNext("b")

source.onNext(b) // 이제 b가 최신 Observable이 되었다

a.onNext("3")
b.onNext("c")

// a.onCompleted()
// b.onCompleted()

// source.onCompleted() // 이 때야 비로소 completed 이벤트가 전달된다

a.onError(MyError.error) // 이러면 구독자로 전달되지 않는다
b.onError(MyError.error) // 최신 Observable인 b는 error 이벤트를 받으면 즉시 구독자에게 전달 가능하다

--> 출력결과
next(2)
next(c)
// completed
error(error)
</code>
</pre>


#### 43/98 reduce Operator
- seed 값과 Observable이 방출하는 요소를 대상으로 클로저를 실행하고 최종 결과를 Observable로 방출하는 reduce 연산자
- scan 연산자와 비교하면 쉽게 이해할 수 있다
- reduce 연산자는 seed value와 accumulator 클로저를 파라미터로 받는다
- seed value와 소스 Observable이 방출하는 요소를 대상으로 클로저를 실행하고, result Observable을 통해 결과를 방출한다 (scan 연산자와 동일)
- accumulator 클로저의 실행 결과가 클로저로 다시 전달되는 것도 scan과 동일
- 하지만 reduce 연산자는 result Observable을 통해 최종 결과 하나만 방출한다. scan은 중간 과정까지 모두 방출한다
- 세 번째 파라미터 mapResult는 최종 결과를 다른 형식으로 바꾸고 싶을 때 사용한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let o = Observable.range(start: 1, count: 5)

print("== scan")

o.scan(0, accumulator: +)
  .subscribe { print($0) }
  .disposed(by: bag)
  
print("== reduce")

o.reduce(0, accumulator: +)
  .subsribe { print($0) }
  .disposed(by: bag)
  
--> 출력결과
== scan 
next(1) // 1
next(3) // 1 + 2
next(6) // 3 + 3
next(10) // 6 + 4
next(15) // 10 + 5
completed
== reduce
next(15) // 최종결과 하나만 출력된다(scan 연산자와의 가장 큰 차이)
completed

</code>
</pre>

// 20/02/23 여기까지

### Conditional Operators
#### 44/98 amb Operator
- 여러 Observable 중에서 가장 먼저 이벤트를 방출하는 Observable을 선택하는 amb 연산자
- 두 개 이상의 소스 Observable 중에서 가장 먼저 Next 이벤트를 전달하는 Observable을 구독하고 나머지는 무시한다
- 여러 서버로 요청을 전달하고 가장 빠른 응답을 처리하는 패턴을 구현할 수 있다
- amb 연산자는 하나의 Observable을 파라미터로 받는다
- 두 Observable 중에서 먼저 이벤트를 전달하는 Observable을 구독하고 이 Observable의 이벤트를 구독자에게 전달하는 새로운 Observable을 리턴한다
- 세 개 이상의 Observable을 대상으로 연산자를 사용해야 한다면 Type 메소드로 구현된 연산자를 사용하면 된다
- 모든 소스 Observable을 배열 형태로 전달한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let a = PublishSubject<String>()
let b = PublishSubject<String>()
let c = PublishSubject<String>()
  
// a.amb(b)
Observable.amb([a, b, c]) // 이렇게 하면 여러 소스 Observable을 받을 수 있다
  .subscribe { print($0) }
  .disposed(by: bag)
  
a.onNext("A") // 얘가 먼저 도착했으니까 얘만 전달된다
b.onNext("B")

b.onCompleted()
a.onCompleted()

--> 출력결과
next(A)
completed

</code>
</pre>


### Time-based Operators
#### 45/98 interval Operator
- 지정된 주기마다 정수를 방출하는 interval 연산자
- 첫 번째 파라미터로 반복 주기(RxTimeInterval -> Dispatch Time Interval과 같다)를 받고, 두 번째 파라미터로 정수를 방출할 scheduler를 받는다
- 연산자가 리턴하는 Observable은 지정된 주기마다 정수를 반복적으로 방출한다
- 종료 시점을 지정하지 않기 때문에 직접 dispose 하기 전까지 계속해서 방출한다
- 방출하는 정수의 형식은 Int로 한정되지 않는다. 다양한 정수 형식 지원이 가능하다
- 하지만 double이나 문자열은 사용 불가하다
- interval 연산자가 생성하는 Observable은 내부에 timer를 가지고 있다
- timer가 시작되는 시점은 생성 시점이 아니다. 구독자가 구독을 시작하는 시점이다
- Observable에서 새로운 구독자가 추가될 때마다 새로운 timer가 생성된다
- 새로운 구독이 추가되는 시점에 내부에 있는 timer가 시작된다는 점이 interval 연산자의 핵심이다


<pre>
<code>
let i = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)

let subscription1 = i.subscribe { print("1 >> \($0))}

DispatchQueue.main.asyncAfter(deadline: .now() + 5) { // 5초 뒤에 디스포즈 되도록 설정
  subscription1.disposed()
}

var subscription2: Disposable?

DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
  subscription2 = i.subscribe { print("2 >> \($0)")}
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
  subscription2?.dispose()
}
--> 출력결과
1 >> next(0)
1 >> next(1)
1 >> next(2)
2 >> next(0)
1 >> next(3)
2 >> next(1)
1 >> next(4)
2 >> next(2)
2 >> next(3)

</code>
</pre>

#### 46/98 timer Operator
- 지연 시간과 반복 주기를 지정해서 정수를 방출하는 timer 연산자
- interval과 마찬가지로 정수를 반복적으로 방출하는 Observable을 생성한다
- 하지만 지연 시간과 반복 주기를 설정할 수 있다
- interval과 마찬가지로 Type 메소드로 구현되어 있다
- 리턴되는 Observable이 방출하는 요소는 fixedWidthInteger로 제한되어 있다
- 세 개의 파라미터를 받는다
- 첫 번째 파라미터는 첫 번째 요소가 방출되기까지의 상대적인 시간이다(구독을 시작하고, 첫 번째 요소가 구독자에게 전달되는 시간) -> 1초를 전달하게 되면 구독 후 1초 뒤에 요소가 전달된다
- 두 번째 파라미터는 반복주기이다. 기본 값은 nil로 돼 있다. 값에 따라 타이머 연산자의 동작 방식이 달라진다
- 세 번째 파라미터는 타이머가 동작할 scheduler를 전달한다

<pre>
<code>
let bag = DisposeBag()

//Observable<Int>.timer(.seconds(1), scheduler: MainScheduler.instance) // 여기서 1초는 첫 번째 요소가 구독자에게 전달되는 상대적인 시간
//  .subscribe { print($0) }
//  .disposed(by: bag)
  
Observable<Int>.timer(.seconds(1), period: .milliseconds(500), scheduler: MainScheduler.instance) // 반복주기(period)가 0.5초로 설정됨
  .subscribe { print($0) }
  .disposed(by: bag)  
  
--> 출력결과
next(0)
next(1)
next(2)
next(3)
... // 0.5 초 간격으로 출력된다 -> 타이머를 중지하고 싶다면 직접 dispose

</code>
</pre>



#### 47/98 timeout Operator
- 지정된 시간 이내에 요소를 방출하지 않으면 error 이벤트를 전달하는 timeout 연산자
- timeout 연산자는 소스 Observable이 방출하는 모든 요소에 timeout 정책을 적용한다
- 첫 번째 파라미터로 timeout 시간을 전달하는데 이 시간 안에 Next 이벤트를 전달하지 않으면 error 이벤트를 전달하고 종료시킨다
- 에러 형식은 RxError.timeout이다
- 시간 내에 Next 이벤트를 전달하면 그대로 구독자에게 전달한다

<pre>
<code>
let bag = DisposeBag()
let subject = PublishSubject<Int>()

//subject.timeout(.seconds(3), scheduler: MainScheduler.instance) // 3초 이내에 새로운 이벤트가 전달되지 않으면 에러 이벤트가 전달된다
//  .subscribe { print($0) }
//  .disposed(by: bag)

subject.timeout(.seconds(3), other: Observable.just(0), scheduler: MainScheduler.instance) 
  .subscribe { print($0) }
  .disposed(by: bag)

// Observable<Int>.timer(.seconds(1), period: .seconds(1), scheduler: MainScheduler.intance) // 1초 만에 전달되므로 timeout(3초)에 문제가 생기지 않는다
// Observable<Int>.timer(.seconds(5), period: .seconds(1), scheduler: MainScheduler.intance) // 5초 만에 전달되므로 timeout(3초) 문제 발생
Observable<Int>.timer(.seconds(2), period: .seconds(5), scheduler: MainScheduler.intance) // 2초 만에 전달되고 5초 주기로 전달
  .subscribe(onNext: { subject.onNext($0) })
  .disposed(by: bag)
  
--> 출력결과
next(0) // 첫 번째 이벤트는 Subject가 전달한 이벤트
next(0) // 이건 5초 뒤에 전달되어 타임아웃이 발생하여 두번째 파라미터(other)로 Observable이 전달되는데, 이 Observable(Observable.just(0))로 구독대상이 변경된다 
completed
  
</code>
</pre>


#### 48/98 delay Operator
- Next 이벤트가 전달되는 시점과 구독이 시작되는 시점을 지연시키는 방법
- 첫 번째 파라미터에는 지연시킬 시간을 전달, 두 번째 파라미터에는 delay timer를 실행할 scheduler를 전달한다
- 연산자가 리턴하는 Observable은 원본 Observable과 동일한 형식이지만, Next 이벤트가 구독자에게 전달되는 시점이 첫 번째 파라미터에 전달된 시간만큼 지연된다
- error 이벤트는 지연 없이 즉시 전달된다
- delaySubscription은 구독 시점을 지연시킬 뿐 Next 이벤트가 전달되는 시점은 지연시키지 않는다

<pre>
<code>
let bag = DisposeBag()

func currentTimeString() -> String {
  let f = DateFormatter()
  f.dateFormat = "yyyy-MM-dd HH:mm:ss.SSS"
  return f.string(from: Date())
}

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .take(10)
  .debug()
  .delay(.seconds(5), scheduler: MainScheduler.instance) // 구독 시점을 5초 뒤로 지연
  .subscribe{ print(currentTimeString(), $0) }
  .disposed(by: bag)

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .take(10)
  .debug()
  .delaySubscription(.seconds(7), scheduler: MainScheduler.instance) // 7초 동안은 아무 로그도 출력되지 않는다 -> delay와 delaySubscription의 차이
  .subscribe{ print(currentTimeString(), $0) }
  .disposed(by: bag)
  
--> 출력결과
강의 참고!
</code>
</pre>



2020/02/25 여기부터
### Sharing Subscription
#### 49/98 Sharing Subscription
- 구독 공유를 통해서 불필요한 중복 작업을 피하는 방법
<pre>
<code>
let bag = DisposeBag()
let source = Observable<String>.create { observer in
  let url = URL(string: "https://kxcoding-study.azurewebsites.net/api/string")!
  let task = URLSession.shared.dataTask(with: url) { (data, response, error) in
    if let data = data, let html = String(data: data, encoding: .utf8) {
      observer.onNext(html)
    }
    observer.onCompleted()
  }
  task.resume()
  
  return Disposables.create {
    task.cancel()
  }
}
.debug()
.share() // share 연산자를 추가하면 모든 구독자가 구독을 공유하기 때문에 중복을 제거해준다

source.subscribe().disposed(by: bag)
source.subscribe().disposed(by: bag)
source.subscribe().disposed(by: bag)

--> 출력결과
// (중략) -> subcribed
// (중략) -> Event next("Hello")
// (중략) -> Event completed
// (중략) -> isDisposed
// (중략) -> subscribed
// (중략) -> subscribed
// (중략) -> Event next("Hello")
// (중략) -> Event completed
// (중략) -> isDisposed
// (중략) -> Event next("Hello")
// (중략) -> Event completed
// (중략) -> isDisposed
(중략) -> subcribed
(중략) -> Event next("Hello")
(중략) -> Event completed
(중략) -> isDisposed
</code>
</pre>

#### 50/98 multicast Operator
- multicast 연산자와 Connectable Observable
- multicast 연산자는 subject를 파라미터로 받는다
- 원본 Observable이 방출하는 이벤트는 구독자에게 전달되는 것이 아니라 이 subject로 전달된다
- subject는 전달받은 이벤트를 등록된 다수의 구독자에게 전달한다
- 기본적으로 unicast 방식으로 동작하는 Observable을 multicast 방식으로 바꿔준다
- 이를 위해 ConnectableObservable을 리턴한다
- 일반 Observable은 구독자가 추가되면 새로운 sequence가 시작된다(이벤트 방출 시작)
- ConnectableObservable은 sequence가 시작되는 시점이 다르다
- 구독자가 추가되어도 sequence는 시작되지 않고, connect 메소드를 호출하는 시점에 sequence가 시작된다
- 원본 Observable이 전달하는 이벤트는 구독자에게 바로 전달되는 것이 아니라 첫 번째 파라미터로 전달한 subject로 전달한다
- 전달받은 subject가 등록된 모든 구독자에게 이벤트를 전달한다
- 이를 통해 모든 구독자가 등록된 이후에 하나의 sequence가 시작되는 패턴을 구현할 수 있다
- ConnectableObservableAdapter는 원본 Observable과 subject를 연결해주는 특별한 클래스이다


<pre>
<code>
let bag = DisposeBag()
let subject = PublishSubject<Int>()

let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).multicast(subject) // multicast 연산자 사용
  
source
  .subscribe { print("A", $0} }
  .disposed(by: bag)
  
source
  .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
  .subscribe { print("B", $0) }
  .disposed(by: bag)
  
// multicast 연산자를 사용할 때는 connect() 연산자가 호출되어야만 이벤트 전달이 시작된다    
source.connect().disposed(by: bag) // connect()의 리턴형은 Disposable이므로 dispose를 통해 메모리를 정리할 수 있다
--> 출력결과
// A next(0)
// A next(1)
// A next(2)
// A next(3)
// B next(0) // delaySubscription에서 .seconds(3)이니까
// A next(4) // source에서 take(5)니까 5개 방출하고 끝남
// A completed
// B next(1)
// B next(2)
// B next(3)
// B next(4)
// B completed
A next(0)
A next(1)
A next(2)
B next(2) // 구독이 지연된 3초 동안 원본옵저버블이 전달한 두 번의 이벤트는 두 번째 구독자에게 전달되지 않았다
A next(3)
B next(3)
A next(4)
B next(4)
A completed
B completed


</code>
</pre>


#### 51/98 publish Operator
- PublishSubject를 활용해서 구독을 공유하는 publish 연산자
- Publish 연산자는 단순하다
- multicast 연산자를 호출하고 새로운 Publish Subject를 만들어서 파라미터로 전달한다
- 그 다음 multicast가 리턴하는 ConnectableObservable을 그대로 리턴한다
- multicast 연산자는 Observable을 공유하기 위해서 내부적으로 subject를 사용한다
- 파라미터로 Publish Subject를 전달한다면 직접 생성해서 전달하는 것보다 publish 연산자를 사용해서 활용하는 방법이 단순하고 좋다
- Publish Subject를 자동으로 생성해준다는 점을 제외하면 나머지는 multicast와 동일하다

<pre>
<code>
let bag = DisposeBag()

let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).publish() // publish 연산자 사용
  
source
  .subscribe { print("A", $0} }
  .disposed(by: bag)
  
source
  .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
  .subscribe { print("B", $0) }
  .disposed(by: bag)
  
source.connect().disposed(by: bag) // connect()의 리턴형은 Disposable이므로 dispose를 통해 메모리를 정리할 수 있다

--> 출력결과
A next(0)
A next(1)
A next(2)
B next(2)
A next(3)
B next(3)
A next(4)
B next(4)
A completed
B completed


</code>
</pre>

#### 52/98 replay Operator
- Connectable Observable에 버퍼를 추가하고 새로운 구독자에게 최근 이벤트를 전달하는 방법
- replay 연산자는 multicast 연산자를 호출한다
- replay Subject를 만들어서 파라미터로 전달한다
- multicast 연산자로 Publish Subject를 전달한다면 Publish 연산자를 사용하고, Replay Subject를 전달하면 replay 연산자를 사용한다
- 두 연산자 모두 multicast를 조금 더 쉽게 사용하도록 도와주는 유틸리티 연산자이다
- 보통은 파라미터를 통해 buffer의 크기를 지정하지만, buffer 크기에 제한이 없는 replayAll 연산자도 있다. 하지만 경우에 따라 메모리 사용량이 급격하게 증가하는 경우가 있어 가급적 사용하지 않는다
- replay 연산자를 사용할 때 buffer 크기를 지정하는 데 유의해야 한다. 필요 이상으로 크게 잡을 경우 메모리 문제가 발생할 가능성이 높기 때문이다
- 필요한 선에서 가장 작게 만들어야 하고, replayAll 연산자는 가급적 사용하지 말아야 한다

<pre>
<code>
let bag = DisposeBag()
// let subject = PublishSubject<Int>()
// let subject = ReplaySubject<Int>.create(bufferSize: 5) // 최대 5개의 이벤트를 버퍼에 저장
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).replay(5)

source
  .subscribe { print("A", $0) }
  .disposed(by: bag)
  
source
  .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
  .subscribe { print("B", $0) }
  .disposed(by: bag)
  
source.connect() 

--> 출력결과
// A next(0)
// A next(1)
// A next(2)
// B next(2)
// A next(3)
// B next(3)
// A next(4)
// B next(4)
// A completed
// B completed
A next(0)
A next(1)
B next(0)
B next(1)
A next(2)
B next(2)
A next(3)
B next(3)
A next(4)
B next(4)
A completed
B completed

</code>
</pre>

#### 53/98 refCount Operator
- refCount 연산자와 RefCount 옵저버블
- refCount 연산자는 ConnectableObservableType이다
- 일반 Observable에서는 사용할 수 없고, ConnectableObservable에서만 사용할 수 있다
- 파라미터는 없고, Observable을 리턴한다
- refCount는 ConnectableObservable을 통해 생성하는 특별한 Observable이다
- 앞으로 이 Observable을 refCountObservable이라 부르겠다
- refCountObservable은 내부에 ConnectableObservable을 유지하면서 새로운 구독자가 추가되는 시점에 자동으로 커넥트 메소드를 호출한다
- 구독자가 구독을 중지하고 더 이상 다른 구독자가 없다면 ConnectableObservable에서 sequence를 중지한다
- 새로운 구독자가 추가되면 다시 커넥트 메소드를 호출한다
- 이 때 ConnectableObservable에서는 새로운 시퀀스가 시작된다


<pre>
<code>
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).debug().publish().refCount() // publish 연산자로 옵저버블을 공유하고 있다

let observer1 = source
  .subscribe { print("A", $0)}
  
// source.connect() -> refCount는 내부적으로 connect를 호출하기 때문에 별도로 connect 연산자를 호출할 필요가 없다

DispatchQueue.main.asyncAfter(deadline: .now() + 3) { // 3초 뒤에 구독을 중지
  observer1.dispose()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) { // 7초 뒤에 구독을 시작했다가
  let observer2 = source.subscribe { print("B", $0) }
  
  DispatchQueue.main.asyncAfter(deadline: .now() + 3) { // 3초 뒤에 구독을 중지
    observer2.dispose()
  }
}


</code>
</pre>

#### 54/98 share Operator
- share 연산자를 활용해서 구독을 공유하는 방법
- share 연산자는 두 개의 파라미터를 받는다
- 첫 번째 파라미터(replay: Int = 0)는 replay buffer의 크기이다. 파라미터로 0을 전달하면 multicast를 호출할 때 Publish Subject를 전달한다
  - 0보다 큰 값을 전달한다면 replay Subject를 전달한다
  - 기본값이 0으로 선언되어 있기 때문에 다른 값을 전달하지 않는다면 새로운 구독자는 구독 이후에 방출되는 이벤트만 전달 받는다
  - multicast 연산자를 호출하니까 하나의 subject를 통해 sequence를 공유한다
- 두 번째 파라미터(scope: SubjectLifetimeScope = .whileConnected)는 이 subject의 수명을 결정한다
  - 기본값은 whileConnected로 선언되어 있다
  - 새로운 구독자가 추가되면(새로운 connection이 시작되면) 새로운 subject가 생성된다
  - 이후 connection이 종료되면 subject는 사라진다
  - connection마다 새로운 subject가 생성되기 때문에 connection은 다른 connection과 격리된다
  - 반대로 두 번째 파라미터에 forever를 전달하면 모든 connection이 하나의 subject를 공유한다

- share 연산자가 리턴하는 Observable은 refCount Observable이라는 점을 유념해라

<pre>
<code>
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).debug().share(replay: 5, scope: .forever) // 두 번째 파라미터에 forever를 넣으면 모든 구독자가 하나의 subject를 공유한다

let observer1 = source
  .subscribe { print("A", $0) }
  
let observer2 = source
  .delaySubscription(.seconds(3), scheduler: MainScheduler.instance) // 3초 뒤에 구독을 시작
  .subscribe { print("B", $0) }
  
DispatchQueue.main.asyncAfter(deadline: .now() + 5) { // 5초 뒤에 모든 구독을 중지 -> 내부의 ConnectableObservable 중지
  observer1.dispose()
  observer2.dispose()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) { // 새로운 sequence가 시작된다
  let observer3 = source.subscribe { print("C", $0) }
  
  DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
    observer3.dispose()
  }
}
  
--> 출력결과
// (중략) -> subscribed
// (중략) -> Event next(0)
// A next(0)
// (중략) -> Event next(1)
// A next(1)
// (중략) -> Event next(2)
// A next(2)
// (중략) -> Event next(3)
// A next(3)
// B next(3) // 3초 뒤에 구독을 시작 -> 이전의 세 개의 이벤트는 전달을 받지 못한다
// (중략) -> Event next(4)
// A next(4)
// B next(4)
// (중략) -> isDisposed
// (중략) -> subscribed // 이 로그가 출력되는 시점에 새로운 subject가 생성된다
// (중략) -> Event next(0) // 그래서 새로운 구독자가 처음 받는 Next 이벤트는 0
// C next(0) // share 연산자
// (중략) -> Event next(1)
// C next(1)
// (중략) -> Event next(2)
// C next(2)
// (중략) -> isDisposed

(중략) -> subscribed
(중략) -> Event next(0)
A next(0)
(중략) -> Event next(1)
A next(1)
(중략) -> Event next(2)
A next(2)
B next(0) // share 연산자의 첫 번째 파라미터(replay)가 5로 설정돼 있으므로 저장된 이벤트가 한꺼번에 전달됨
B next(1) // share 연산자의 첫 번째 파라미터(replay)가 5로 설정돼 있으므로 저장된 이벤트가 한꺼번에 전달됨
B next(2)
(중략) -> Event next(3)
A next(3)
B next(3)
(중략) -> Event next(4)
A next(4)
B next(4)
(중략) -> isDisposed // 여기에서 중지된 sequence가 다시 공유되는 것은 아니다
C next(0) // share 연산자의 첫 번째 파라미터(replay)가 5로 설정돼 있으므로 저장된 이벤트가 한꺼번에 전달됨
C next(1) // share 연산자의 첫 번째 파라미터(replay)가 5로 설정돼 있으므로 저장된 이벤트가 한꺼번에 전달됨
C next(2) // share 연산자의 첫 번째 파라미터(replay)가 5로 설정돼 있으므로 저장된 이벤트가 한꺼번에 전달됨
C next(3) // share 연산자의 첫 번째 파라미터(replay)가 5로 설정돼 있으므로 저장된 이벤트가 한꺼번에 전달됨
C next(4) // share 연산자의 첫 번째 파라미터(replay)가 5로 설정돼 있으므로 저장된 이벤트가 한꺼번에 전달됨
(중략) -> subscribed
(중략) -> Event next(0)
C next(0) // 이어지는 이벤트가 5가 아니라 0인 것은 sequence가 중지된 다음에 새로운 구독자가 추가되면 새로운 sequence가 시작되기 때문임
(중략) -> Event next(1)
C next(1)
(중략) -> Event next(2)
C next(2)
(중략) -> isDisposed
</code>
</pre>

// 2020/02/26 여기부터
### Scheduler
#### 55/98 Scheduler
- 코드를 원하는 스레드에서 실행하는 방법
- iOS 앱을 만들다가 스레드 처리가 필요하다면 GCD를 사용한다
- RxSwift에서는 GCD 대신 scheduler를 사용한다
- scheduler는 특정 코드가 실행되는 컨텍스트를 추상화한 것이다
- 컨텍스트는 로우레벨 스레드가 될 수도 있고, DispatchQueue나 OperationQueue가 될 수도 있다
- scheduler는 추상화된 컨텍스트이기 때문에 스레드와 1:1로 매칭되지 않는다
- 하나의 스레드에 두 개 이상의 개별 scheduler가 존재하거나, 하나의 scheduler가 두 개의 스레드에 걸쳐 있는 경우도 있다
- 이런 내부적인 내용까지 직접 알 필요는 없고, GCD와 유사하다는 점만 알면 된다
- UI를 업데이트 하는 경우에는 Main 스레드를 사용해야 한다
  - GCD에서는 Main Queue를 사용한다
  - RxSwift에서는 Main Scheduler를 사용한다
- Network 요청이나 파일 처리 작업은 Main 스레드에서 사용하면 블로킹이 발생한다
  - GCD에서는 Global Queue를 사용한다
  - RxSwift에서는 Background Scheduler를 사용한다
- RxSwift는 GCD와 마찬가지로 다양한 기본 scheduler를 제공한다
- 내부적으로 GCD와 유사한 방식으로 동작하고, 실행할 작업을 스케줄링 한다.
- 스케줄링 방식에 따라 Serial Scheduler와 Concurrent Scheduler로 구분한다
- Serial Scheduler
  - CurrentThreadScheduler가 가장 기본적인 scheduler이다
  - Main 스레드와 연관된 scheduler는 MainScheduler이다. Main Queue처럼 UI를 업데이트할 때 사용한다
  - 작업을 실행할 DispatchQueue를 직접 지정하고 싶다면 SerialDispatchQueueScheduler나 ConcurrentDispatchQueueScheduler를 활용한다
  - 앞에서 사용한 Main Scheduler는 Serial DispatchQueue의 일종이다
  - 백그라운드 작업을 실행할 때는 DispatchQueue Scheduler를 사용한다
  - 실행 순서를 제외하거나 동시에 실행 가능한 작업 수를 제한하고 싶다면 OperationQueueScheduler를 사용한다. 이 scheduler는 DispatchQueue가 아닌 OperationQueue를 사용해서 생성한다
- 이 외에도 Unit Test에서 사용하는 TestScheduler가 있고, scheduler를 직접 구현할 수도 있다(Custom Scheduler)

<pre>
<code>
let bag = DisposeBag()

Observable.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
  .filter { num -> Bool in
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> filter")
    return num.isMultiple(of: 2)
  }
  .map { num -> Int in
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> map")
    return num * 2
  } // 여기까지만 코드가 작성되었다면 Observable이 어떤 요소를 방출하고, 어떻게 처리해야 하는지를 나타낼 뿐이다. Observable이 생성되고 연산자가 호출되는 시점은 구독이 시작되는 시점이다.
  .subscribe { // 구독이 시작되는 시점: Observable이 생성되고 연산자가 호출된다
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> map")
    print($0)
  }
  .disposed(by: bag)
  
--> 출력결과
...
Main Thread >> map
Main Thread >> map
next(16)
Main Thread >> filter
Main Thread >> map
completed
  
</code>
</pre>

- scheduler를 지정하는 코드가 없다면 기본 scheduler인 CurrentThreadScheduler가 사용된다
- 플레이그라운드가 실행되는 스레드는 Main 스레드이다
- RxSwift에서 scheduler를 지정할 때는 observeOn(_ :) 메소드와 subscribeOn(_ :) 메소드를 사용한다
- observeOn(_ :) 메소드는 연산자를 실행할 scheduler를 지정한다

<pre>
<code>
let bag = DisposeBag()
let backgroundScheduler = ConcurrentDispatchQueueScheduler(queue: DispatchQueue.global())


Observable.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
  .filter { num -> Bool in
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> filter")
    return num.isMultiple(of: 2)
  }
  .observeOn(backgroundScheduler) // observeOn(_ :) 메소드는 이어지는 연산자들이 작업을 실행할 scheduler를 지정한다. 그래서 뒤에 있는 맵은 백그라운드 scheduler에서 실행되지만, 앞에 있는 filter에는 영향을 주지 않는다
  .map { num -> Int in
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> map")
    return num * 2
  }
  .subscribe {
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> subscribe")
    print($0)
  }
  .disposed(by: bag)
  
--> 출력결과
Main Thread >> filter
Main Thread >> filter
Main Thread >> filter
Background Thread >> map
Background Thread >> subscribe
next(4)
Background Thread >> map
Background Thread >> subscribe
next(8)
Background Thread >> map
Background Thread >> subscribe
next(12)
Background Thread >> map
Background Thread >> subscribe
next(16)
Background Thread >> subscribe
completed
</code>
</pre>

- observeOn(_ :) 메소드로 지정한 scheduler는 다른 scheduler로 변경하기 전까지 계속 사용된다
- subscribeOn(_ :) 메소드는 구독을 시작하고 종료할 때 사용할 scheduler를 지정한다
- 구독을 시작하면 Observable에서 새로운 이벤트가 방출된다. 이벤트를 방출할 scheduler를 지정하는 것이다
- create 연산자로 구현한 코드 역시 이 메소드로 지정한 scheduler에서 실행된다
- 이 메소드를 사용하지 않는다면 subscribeOn(_ :) 메소드가 호출한 scheduler에서 새로운 sequence가 시작된다

<pre>
<code>
let bag = DisposeBag()
let backgroundScheduler = ConcurrentDispatchQueueScheduler(queue: DispatchQueue.global())

Observable.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
  .subscribeOn(MainScheduler.instance) // 메소드 이벤트 때문에 많이 혼동하게 되는데 subscribeOn(_ :) 메소드는 subscribe 메소드가 호출되는 scheduler를 지정하는 것이 아니다. 그리고 이어지는 연산자가 호출되는 scheduler를 지정하는 것도 아니다. Observable이 시작되는 시점에 어떤 scheduler를 사용할지 지정하는 것이다. 이 차이를 확실하게 구분해야 한다. 그리고 observeOn 메소드와 달리 호출 시점이 중요하지 않다.
  .filter { num -> Bool in
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> filter")
    return num.isMultiple(of: 2)
  }
  .map { num -> Int in
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> map")
    return num * 2
  } // Observable이 어떤 요소를 방출하고, 어떻게 처리해야 하는지를 나타낼 뿐이다. Observable이 생성되고 연산자가 호출되는 시점은 구독이 시작되는 시점이다.
  .observeOn(backgroundScheduler) // observeOn 메소드는 이어지는 연산자들이 작업을 실행할 scheduler를 지정한다. 그래서 뒤에 있는 map은 backgroundScheduler에서 실행되지만, 앞에 있는 filter에는 영향을 주지 않는다
  .subscribe {
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> subscribe")
    print($0)
  }
  .disposed(by: bag)
  
--> 출력결과
...
Background Thread >> map
Main Thread >> subscribe
next(4)
Main Thread >> subscribe
next(8)
Main Thread >> subscribe
next(12)
Main Thread >> subscribe
next(16)
Main Thread >> subscribe
completed

</code>
</pre>

- subscribeOn(_ :) 메소드는 Observable이 시작되는 scheduler를 지정한다
- observeOn(_ :) 메소드는 이어지는 연산자가 실행되는 scheduler를 지정한다


// 2020/02/26 여기까지

// 2020/02/28 여기부터
### Error Handling
#### 56/98 Error Handling
- Observable에서 전달한 error 이벤트가 구독자에게 전달되면 구독이 종료되고 더 이상 새로운 이벤트가 전달되지 않는다
- 더 이상 새로운 이벤트를 처리할 수 없게 된다
- RxSwift는 두 가지 방법으로 문제를 해결
- 첫 번째 방법은 error 이벤트가 전달되면 새로운 Observable을 리턴한다 - catchError 연산자
  - Observable이 전달하는 Next 이벤트와 completed 이벤트는 그대로 구독자에게 전달
  - 반면 error 이벤트가 전달되면 새로운 Observable을 구독자에게 전달
- 두 번째 방법은 error가 발생한 경우 Observable을 다시 구독하는 것 - retry 연산자
  - 에러가 발생하지 않을 때까지 무한정 재시도 하거나, 재시도 횟수를 제한할 수 있다


#### 57/98 catchError Operator
- catchError 이벤트는 Next 이벤트와 completed 이벤트는 구독자에게 그대로 전달하고, error 이벤트는 전달하지 않고 새로운 Observable이나 기본값을 전달
- 네트워크 요청을 구현할 때 많이 사용한다
- 올바른 응답을 받지 못한 상황에서 로컬 캐시를 사용하거나 기본값을 사용하도록 구현할 수 있다
- catchError 연산자는 클로저를 파라미터로 받는다
- error 이벤트는 클로저 파라미터로 전달되고, 클로저는 새로운 Observable을 리턴한다
- Observable이 방출하는 요소의 형식은 소스 Observable과 동일하다
- catchError Observable은 소스 Observable에서 error 이벤트가 전달되면 소스 Observable을 클로저가 전달하는 Observable로 교체한다
- 소스 Observable은 더 이상 다른 이벤트를 전달하지 못하지만, 교체된 Observable은 문제가 없기 때문에 계속해서 다른 이벤트를 전달할 수 있다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let subject = PublishSubject<Int>()
let recovery = PublishSubject<Int>()
  
subject
  .catchError { _ in recovery } // catchError 연산자가 recovery 연산자로 교체했다
  .subscribe { print($0) }
  .disposed(by: bag)
  
subject.onError(MyError.error)

subject.onNext(11) // subject는 recovery로 교체되었기 때문에 더 이상 아무 이벤트도 전달하지 못한다

recovery.onNext(22)
recovery.onCompleted()
--> 출력결과
next(22)
completed

</code>
</pre>

- Observable 대신 기본값을 리턴하는 catchErrorJustReturn 연산자
- catchErrorJustReturn 소스 Observable에서 error가 발생하면 파라미터로 전달한 기본값을 구독자에게 전달한다
- 파라미터의 형식은 항상 소스 Observable이 방출하는 요소의 형식과 같다
- error가 발생했을 때 사용할 수 있는 기본값이 있다면 catchErrorJustReturn 연산자를 사용한다
- 하지만 발생한 error 종류에 관계없이 항상 동일한 값이 리턴된다는 단점이 있다
- 나머지 경우에는 catchError 연산자를 사용한다
- 클로저를 통해 error 처리를 자유롭게 구현할 수 있다는 장점이 있다
- 작업을 처음부터 다시 하고 싶다면 retry 연산자를 사용하면 된다


<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let subject = PublishSubject<Int>()
  
subject
  .catchErrorJustReturn(-1)
  .subscribe { print($0) }
  .disposed(by: bag)
  
subject.onError(MyError.error)

--> 출력결과
next(-1)
completed // 더 이상 전달될 이벤트가 없기 때문에 completed 이벤트가 전달되고, 구독은 종료된다.
</code>
</pre>


#### 58/98 retry Operator
- retry 연산자는 Observable에서 error가 발생하면 Observable에 대한 구독을 해제하고 새로운 구독을 시작한다
- 새로운 구독이 시작되기 때문에 Observable Sequence는 처음부터 다시 시작된다
- Observable에서 error가 발생하지 않는다면 정상적으로 종료되고, error가 발생한다면 또다시 새로운 구독을 시작한다
- retry 연산자는 두 가지 형태가 있다
- 첫 번째처럼 파라미터 없이 호출하면, Observable이 정상적으로 완료될 때까지 계속해서 재시도 한다
- 만약 Observable에서 반복적으로 error가 발생하면, 그만큼 재시도 횟수가 늘어나면서 리소스가 낭비된다
- 심한 경우 무한 루프에 빠지거나 앱이 강제로 종료될 수 있다
- 따라서 파라미터 없이 호출하는 것은 가능한 피해야 한다
- 최대 재시도 횟수를 파라미터로 전달할 때는 항상 1을 더해서 전달해야 한다
- retry 연산자는 error가 발생한 즉시 재시도하기 때문에, 재시도 시점을 제어하는 것은 불가능하다
- 네트워크 요청에서 error가 발생했다면, 정상적인 응답을 받거나 최대 횟수에 도달할 때까지 계속해서 재시도 한다
- 만약 사용자가 재시도 버튼에만 재시도를 탭하고 싶다면 retryWhen을 사용해야 한다

<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

var attemps = 1

let source = Observable<Int>.create { observer in
  let currentAttempts = attempts
  print("#\(currentAttempts) START") // Sequence의 시작을 출력
  
  if attempts > 0 {
    observer.onError(MyError.error)
    attempts += 1
  }
  
  observer.onNext(1)
  observer.onNext(2)
  observer.onCompleted()
  
  return Disposable.create {
    print("#\(currentAttempts) END") // Sequence의 종료를 출력
  }
}

source
  .retry(7) // 6번 재시도 된다 -> 최대 재시도 횟수를 정할 때는 원하는 횟수 +1을 해줘야 한다
  .subscribe { print($0) }
  .disposed(by: bag)
  
--> 출력결과
#1 START
#1 END
#2 START
#2 END
#3 START
#3 END
#4 START
#4 END
#5 START
#5 END
#6 START
#6 END
#7 START
#7 END
error(error)

</code>
</pre>


- 재시도를 하고 싶을 때 호출하는 retryWhen 연산자
- retryWhen 연산자는 클로저를 파라미터로 받는다
- 클로저 파라미터에는 발생한 error를 방출하는 Observable이 전달된다
- 클로저는 triggerObservable을 리턴한다
- triggerObservable이 Next 이벤트를 전달하는 시점에 소스 Observable에서 새로운 구독을 시작한다 -> 작업을 재시도 한다


<pre>
<code>
let bag = DisposeBag()

enum MyError: Error {
  case error
}

var attemps = 1

let source = Observable<Int>.create { observer in
  let currentAttempts = attempts
  print("#\(currentAttempts) START") // Sequence의 시작을 출력
  
  if attempts < 3 {
    observer.onError(MyError.error)
    attempts += 1
  }
  
  observer.onNext(1)
  observer.onNext(2)
  observer.onCompleted()
  
  return Disposable.create {
    print("#\(currentAttempts) END") // Sequence의 종료를 출력
  }
}

let trigger = PublishSubject<Void>() // triggerSubject

source
  .retryWhen { _ in trigger }
  .subscribe { print($0) }
  .disposed(by: bag)
  
trigger.onNext(()) // triggerSubject로 Next 이벤트를 전달하면 소스 Observable에서 새로운 구독이 시작된다 -> attempts 값은 2
trigger.onNext(()) // attempts 값은 3
--> 출력결과
START #1
END #1
START #2
END #2
START #3
next(1)
next(2)
completed
END #3

</code>
</pre>



### RxCocoa Basics
#### 59/98 RxCocoa Overview
- RxCocoa는 Cocoa Framework에 Reactive의 장점을 더해주는 Library이다
- RxCocoa는 RxSwift를 기반으로하는 별도의 라이브러리이다
- Reactive는 RxSwift 라이브러리에 제네릭 구조체로 선언되어 있다
- 형식을 Reactive 방식으로 확장할 때 사용한다
- Reactive의 base 속성이 있는데, 확장할 형식의 인스턴스가 지정된다
- ReactiveCompatible의 역할은 기존 형식에 rx 속성을 추가한다 -> 네임스페이스를 통해서 제공된다
- 코드 마지막에 NSObject가 있는데 NSObject는 Cocoa Framework에 있는 모든 클래스가 상속하는 루트 클래스이기 때문에 결과적으로 모든 클래스에 rx라는 속성을 자동으로 추가한다는 의미이다
- UIButton + Rx에는 tap이라는 멤버가 ControlEvent 형식으로 선언되어 있다
- ControlEvent는 RxCocoa가 제공하는 trait이다
- trait는 UI 처리에 특화된 Observable이고, ControlEvent뿐만 아니라 드라이버, 시그널 같은 고유한 특성을 가진 trait가 제공된다
- tap은 특별한 Observable이라 구독할 수 있다
- 버튼에서 touchUpInside 이벤트가 전달될 때마다 구독자로 이벤트가 전달된다
- UILabel + Rx
- text 속성과 attributedText 속성이 선언되어 있다
- text 속성은 Binder 형식으로 돼 있다
- Binder는 인터페이스 binding에 사용되는 특별한 Observer이다 



#### 60/98 Binding
- data를 UI에 표시하는 의미로 binding이 사용된다
- binding에는 data 생산자와 data 소비자가 있다
- data 생산자는 Observable이다
- 코드 레벨로 설명하면 ObservableType을 채용한 모든 형식이 생산자가 된다
- data 소비자는 label이나 imageView 같은 UI Component이다
- 생산자가 생산한 data는 소비자에게 전달되고, 소비자는 적절한 방식으로 data를 소비한다
- 예를 들어 label은 전달된 텍스트를 화면에 표시한다
- 반대로 소비자가 생산자에게 data나 이벤트를 전달하는 경우는 없다
- binder는 UI binding에 사용되는 특별한 Observer이다
- data 소비자의 역할을 수행한다
- Observer이기 때문에 binder로 새로운 값을 전달할 수 있지만, Observable이 아니기 때문에 구독자를 추가하는 것은 불가능하다
- binder는 error 이벤트를 받지 않는다
- 만약 error 이벤트를 전달하면 실행 모드에 따라 크래시가 발생하거나 error 메시지가 출력된다
- Observer에서 error 이벤트가 전달되면 Observable sequence가 종료된다
- Next 이벤트가 전달되지 않으면 binding 된 UI가 더 이상 업데이트 되지 않는다
- 이런 문제를 막기 위해서 error 이벤트를 받지 않는 것이다
- binding이 성공하면 UI가 업데이트 된다
- UI 코드는 메인 스레드에서 실행해야 한다는 것이 기본 중의 기본이다
- binder는 binding이 메인 스레드에서 실행되는 것을 보장한다
- UITextField + Rx 
  - text 속성이 ControlProperty<String?> 타입으로 선언되어 있다
  - ControlProperty 타입은 data를 특정 UI에 binding할 때 사용하는 특별한 Observable이다
  - 타입 파라미터가 옵셔널 스트링으로 선언되어 있다
  - text 속성이 변경될 때마다 Next 이벤트를 전달하고 여기에는 옵셔널 스트링 형식의 data가 저장되어 있다
- RxSwift에서 UI를 업데이트 하기 위해 메인 스레드를 사용해야 하는 경우, GCD의 DispatchQueue.main.async를 사용할 수도 있겠지만, RxSwift에서 제공하는 .observeOn(MainScheduler.instance)를 사용한다
- Binder 속성을 활용하면 DispatchQueue.main.async나 .observeOn(MainScheduler.instance)를 사용할 필요 없다. .bind(to: ObserverType)메소드를 사용하면 된다
- 예시 코드
<pre>
<code>

// valueField.rx.text
//  .observeOn(MainScheduler.instance) // GCD의 DispatchQueue.main.async를 사용하는 대신 RxSwift에서 제공하는 .observeOn(MainScheduler.instance)를 사용했다
//  .subscribe(onNext: { [weak self] str in
//    self?.valueLabel.text = str
//  })
//  .disposed(by: disposeBag)

valueField.rx.text
  .bind(to: value: valueLabel.rx.text) // bind 메소드를 사용하면 스레드에 신경 쓸 필요 없다
  .disposed(by: disposeBag)
</code>
</pre>


#### 61/98 RxCocoa Traits
- traits는 UI에 특화된 Observable이다.
- Observable이기 때문에 UI binding에서 data 생산자 역할을 수행한다
- binder와 반대라고 생각하면 쉽다
- RxCocoa는 네 가지 traits를 제공한다(ControlProperty, ControlEvent, Driver, Signal)
- traits는 모든 작업이 Main Scheduler(Main Thread)에서 실행된다
- 따라서 UI 업데이트 코드를 작성할 때 Scheduler를 직접 작성할 필요가 없다
- Observable sequence가 error 이벤트로 인해 종료되면 UI는 더 이상 업데이트되지않는다
- 하지만 traits는 error 이벤트를 전달하지 않는다. 그래서 이런 문제가 발생하는 경우는 없다
- UI가 항상 올바른 스레드에서 업데이트 되는 것을 보장한다
- Observable을 구독하면 기본적으로 새로운 sequence가 시작된다
- traits 역시 Observable이지만, 새로운 sequence가 시작되지는 않는다
- traits를 구독하는 모든 구독자는 동일한 sequence를 공유한다
- 일반 Observable에서 share 연산자를 사용한 것과 동일한 방식으로 동작한다
- UI관련 코드를 더 깔끔하게 쓰고 싶거나, binding이 잘못된 스레드에서 실행되는 것이 싫다면 subscribe 메소드가 아닌 traits를 써라
 

#### 62/98 Control Event, Control Property
- Cocoatouch framework가 제공하는 View에는 다양한 속성이 선언되어 있다
- rxcocoa는 익스텐션으로 뷰를 확장하고 동일한 이름을 가진 속성들을 추가한다
- 이런 속성들은 대부분 ControlProperty 형식으로 선언되어 있다
- ControlProperty는 제네릭 구조체로 선언되어 있고, ControlProtocolType 프로토콜을 채용하고 있다
- ControlPropertyType 프로토콜은 Observable Type과 Observer Type 프로토콜을 상속하고 있다
- ControlProperty는 특별한 Observable이면서, 동시에 특별한 Observer이다
- ControlProperty가 읽기 전용 속성을 확장했다면 Observable의 역할만 수행하고, 읽기 쓰기가 모두 가능하다면 Observer의 역할도 함께 수행한다
- ControlProperty의 특징
  - UI binding에 사용되므로 error 이벤트를 전달 하지도, 받지도 않는다
  - completed 이벤트는 컨트롤이 제거되기 직전에 전달된다
  - 모든 이벤트는 Main Scheduler에서 전달된다
  - ControlProperty는 sequence를 공유한다
  - 일반 Observable에서 share 연산자를 호출하고, replay 파라미터로 1을 전달한 것과 동일한 방식으로 동작한다
  - 새로운 구독자가 추가되면 가장 최근에 저장된 속성값이 바로 전달된다
  - UI 컨트롤을 상속한 컨트롤들은 다양한 이벤트를 전달한다
  - RxCocoa가 확장한 익스텐션에는 이벤트를 Observable로 wrapping한 속성이 추가되어 있다
  - 예를 들어 UIButton의 확장을 보면 tap이라는 속성이 선언되어 있다. 이 속성은 ControlEvent 형식으로 선언되어 있다
  - ControlEvent는 ControlEventType 프로토콜을 채용한 제네릭 타입이다
  - ControlEventType 프로토콜은 ObservableType 프로토콜을 상속하고 있다
  - ControlProperty와 달리 Observable의 역할은 수행하지만, Observer의 역할은 수행하지 못한다
  - 컨트롤 이벤트는 ControlProperty와 다수의 공통점을 가지고 있다
  - error 이벤트를 전달하지 않고 completed 이벤트는 컨트롤이 해제되기 직전에 전달된다
  - Main Scheduler에서 이벤트를 전달하는 것도 동일하다
  - 하지만 ControlProperty와 달리 가장 최근 이벤트를 replay 하지 않는다
  - 그래서 새로운 구독자는 구독 이후에 전달된 이벤트만 전달 받는다

- 예시 코드
<pre>
<code>
@IBOutlet weak var redComponentLabel: UILabel!
@IBOutlet weak var greenComponentLabel: UILabel!
@IBOutlet weak var blueComponentLabel: UILabel!

// Cocoatouch framework로 구현한 내용
@IBAction func sliderChanged(_ sender: Any) {
  let redComponent = CGFloat(redSlider.value) / 255
  let greenComponent = CGFloat(greenSlider.value) / 255
  let blueComponent = CGFloat(blueSlider.value) / 255
  
  let color = UIColor(red: redComponent, green: greenComponent, blue: blueComponent, alpha: 1.0)
  colorView.backgroundColor = color
  
  updateComponentLabel()
}

@IBAction func resetColor(_ sender: Any) {
  colorView.backgroundColor = UIColor.black
  
  redSlider.value = 0
  greenSlider.value = redSlider.value
  blueSlider.value = redSlider.value
  
  updateComponentLabel()
}

private func updateComponentLabel() {
  redComponentLabel.text = "\(Int(redSlider.value))"
  greenComponentLabel.text = "\(Int(greenSlider.value))"
  blueComponentLabel.text = "\(Int(blueSlider.value))"
}

// RxCocoa를 활용한 뷰 업데이트

func updateWithRxCocoa() {
  redSlider.rx.value
    .map { "\(Int($0))" }
    .bind(to: redComponentLabel.rx.text)
    .disposed(by: bag)
    
  greenSlider.rx.value
    .map { "\(Int($0))" }
    .bind(to: greenComponentLabel.rx.text)
    .disposed(by: bag)
    
  blueSlider.rx.value
    .map { "\(Int($0))" }
    .bind(to: blueComponentLabel.rx.text)
    .disposed(by: bag)
    
  // 색상 업데이트 하는 코드
  Observable.combineLatest([redSlider.rx.value, greenSlider.rx.value, blueSlider.rx.value])
    .map { UIColor(red: CGFloat($0[0]) / 255, green: CGFloat($0[1]) / 255, blue: CGFloat($0[2]) / 255, alpha: 1.0) }
    .bind(to: colorView.rx.backgroundColor)
    .disposed(by: bag)
    
  // 리셋하는 코드
  resetButton.rx.tap
    .subscribe(onNext: { [weak self] in
      self?.colorView.backgroundColor = UIColor.black
      
      self?.redSlider.value = 0
      self?.greenSlider.value = 0
      self?.blueSlider.value = 0
      
      self?.updateComponentLabel()
    })
    .disposed(by: bag)
    
}
</code>
</pre>


#### 63/98 Driver
- RxCocoa가 제공하는 traits 중에서 핵심은 driver이다
- driver는 data를 UI에 binding하는 직관적이고 효율적인 방법을 제공한다
- driver는 특별한 Observable이고, UI 처리에 특화된 몇 가지 특징을 가지고 있다
  - error 메시지를 전달하지 않으므로 오류로 인해 UI 업데이트가 중단되는 일이 없다
  - Scheduler를 강제로 변환하는 일이 없다면 항상 Main Scheduler에서 이벤트가 전달된다
  - driver는 side effect를 공유한다. 일반 Observable에서 share 연산자를 호출하고, 화면에 있는 파라미터를 전달한 것과 동일하게 동작한다
  - 모든 구독자가 sequence를 공유하고 새로운 구독이 시작되면 가장 최근에 전달된 이벤트가 즉시 전달된다
  - driver는 asDriver 메소드를 활용해서 사용한다. 이 때 기존에 존재하는 share() 메소드는 지워야 한다.

- 사용 예시
<pre>
<code>
// driver 사용 전
let result = inputField.rx.text
  .flatMapLatest {
    validateText($0)
      .observeOn(MainScheduler.instance)
      .catchErrorJustReturn(false)
  }
  .share()
  
result
  .map { $0 ? "Ok" : "Error" }
  .bind(to: resultLabel.rx.text)
  .disposed(by: bag)
  
result
  .map { $0 ? UIColor.blue : UIColor.red }
  .bind(to: resultLabel.rx.backgrounColor)
  .disposed(by: bag)
  
result
  .bind(to: sendButton.rx.isEnabled)
  .dispoed(by: bag)

// driver를 사용할 때
let result = inputField.rx.text.asDriver() // driver는 sequence를 공유하기 때문에 share 연산자는 필요 없다
  .flatMapLatest {
    validateText($0)
    .asDriver(onErrorJustReturn: false)
  }
  
result
  .map { $0 ? "Ok" : "Error" }
  .drive(resultLabel.rx.text)
  .disposed(by: bag)

result
  .map { $0 ? UIColor.blue : UIColor.red }
  .drive(resultLabel.rx.backgrounColor)
  .disposed(by: bag)
  
result
  .drive(sendButton.rx.isEnabled)
  .dispoed(by: bag)

</code>
</pre>



### RxCocoa Common Patterns
#### 64/98 Table View in RxCocoa
- Observable을 테이블뷰에 바인딩할 때는 items 메소드를 사용한다
- 코드예시
- Cocoatouch Framework 의 데이터 소스가 섞이게 되면 rx는 더 이상 동작하지 않는다
<pre>
<code>

var nameList = appleProducts.map { $0.name }
var productList = appleProducts

@IBOutlet weak var listTableView: UITableView!

let priceFormatter: NumberFomatter = {
  let f = NumberFormatter()
  f.numberStyle = NumberFormatter.Style.currency
  f.locale = Locale(identifier: "KO_kr")
  
  return f
}()

let bag = DisposeBag()

let nameObservable = Observable.of(appleProducts.map { $0.name })

let productObservable = Observable.of(appleProducts)

override func viewDidLoad() {
  super.viewDidLoad()
  
  // #1
  nameObservable.bind(to: listTableView.rx.items) { tableView, row, element in
    let cell = tableView.dequeueReusableCell(withIdentifier: "standardCell")!
    cell.textLabel?.text = element
    return cell
  }
  .disposed(by: bag)
  
  // #2
  nameObservable.bind(to: listTableView.rx.items(cellIdentifier: "standardCell")) { row, element, cell in
    cell.textLabel?.text = element
  }
  .disposed(by: bag)
  
  // #3
  productObservable.bind(to: listTableView.rx.items(cellIdentifier: "productCell", cellType: ProductTableViewCell.self)) { [weak self] row, element, cell, in
    cell.categoryLabel.text = element.category
    cell.productNameLabel.text = element.name
    cell.summaryLabel.text = element.summary
    cell.priceLabel.text = self?.priceFormatter.string(For: element.price)
  }
  .disposed(by: bag)
  
//  listTableView.rx.modelSelected(Product.self)
//    .subscribe(onNext: { product in
//      print(product.name)
//    })
//    .disposed(by: bag)
//    
//  listTableView.rx.itemSelected
//    .subscribe(onNext: {[weak self] indexPath in
//      self?.listTableView.deselectRow(at: indexPath, animated: true) // 선택 상태를 바로 제거 해준다
//    })
//    .disposed(by: bag)

  // 클릭하고 나면 다시 선택해제 되는 작업을 한꺼번에 하기
  Observable.zip(listTableView.rx.modelSelected(Product.self), listTableView.rx.itemSelected)
    .bind { [weak self] (product, indexPath) in
      self?.listTableView.deselectRow(at: indexPath, animated: true)
      print(product.name)
    }
    .disposed(by: bag)

  //listTableView.delegate = self
  listTableView.rx.setDelegate(self)
    .disposed(by: bag)
  
}
</code>
</pre>
