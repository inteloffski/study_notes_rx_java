[Оглавление](README.md)

# 9. Subject. Типы Subject

В RxJava кроме основных источников данных еще существуют subjects. Особенность subject заключается в том, что они выступают и как источник данных, и как подписчик. Subject - это интерфейс, который имеет 5 реализация:

1. BehaviorSubject
2. ReplaySubject
3. AsyncSubject
4. UnicastSubject
5. PublishSubject

Каждая из реализаций не имеет публичного конструктора. Чтобы создать экземпляр любой из реализаций необходимо пользоваться фабричным методом create().

### 1. PublishSubject

PublishSubject работает следующим образом, когда данные передаются в publushsubject, он их выдает всем подписчикам, которые подписаны на него в данный момент.

```
PublishSubject<Integer> subject = PublishSubject.create();
subject.onNext(0);
subject.subscribe(System.out::println);
subject.onNext(1);
subject.onNext(2);
subject.onNext(3);

// Output:
// 1
// 2
// 3
```

Стоит обратить внимание, что 0 не был напечатан, потому что на момент публикации 0 у PublishSubject не было ни одного подписчика.

### 2. ReplaySubject

ReplaySubject имеет специальную возможность кэшировать все поступившие в него данные. Таким образом, когда у него появиться новый подписчик, то все данные будут переданы в подписчика с самого начала.

```
ReplaySubject<Integer> s = ReplaySubject.create();
s.subscribe(v -> System.out.println("Early:" + v));
s.onNext(0);
s.onNext(1);
s.subscribe(v -> System.out.println("Late: " + v)); 
s.onNext(2);

// Output:
// Early:0
// Early:1
// Late: 0
// Late: 1
// Early:2
// Late: 2
```

Таким образом, мы видим, что все данные поступили обоим подписчикам. Иногда кэшировать все подряд не лучшая идея, поэтому с помощью двух методов, мы можем регулировать размер кэшированных данных и время жизни этого кэша. ReplaySubject.createWithSize() - задает размер кэша. ReplaySubject.createWithTime() - задает время жизни кэша.

### 3. BehaviorSubject

BehviorSubject является частно реализацией ReplaySubject, у которого размер буфера равен 1. То есть BehaviorSubject имеет возможность кэшировать только одно значение. Таким образом, BehaviorSubject возвращает последнее значение + одно закэшированое значение.

```
BehaviorSubject<Integer> s = BehaviorSubject.create();
s.onNext(0); 
s.onNext(1);
s.onNext(2);
s.subscribe(System.out::println);
s.onNext(3);

// Output:
// 2
// 3
```

BehaviorSubject можно задать значение по умоланию с помощью метода createDefault(). Начальное значение предоставляется для того, чтобы быть доступным еще до поступления данных. Так как роль BehaviorSubject – всегда иметь доступные данные, считается неправильным создавать его без начального значения, также как и завершать его.

### 4. AsyncSubject

AsyncSubject также хранит последнее значение. Разница в том, что он не выдает данные, пока последовательность не закончится, то есть пока не будет вызван метод onCompleted(). Его используют, когда нужно выдать единое значение и тут же завершиться.

```
AsyncSubject<Integer> s = AsyncSubject.create();
s.subscribe(v -> System.out.println(v));
s.onNext(0);
s.onNext(1);
s.onNext(2);
s.onCompleted();

// Output:
// 2
```

Обратите внимание, что если бы мы не вызвали s.onCompleted(), этот код ничего бы не напечатал.

### 5. UnicastSubject

UnicastSubject допускает только одного подписчика и отправляет все элементы независимо от времени подписки.

```
Observable<Integer> observable = 
	Observable.range(1, 5)
		  .subscribeOn(Schedulers.io());

UnicastSubject<Integer> pSubject = UnicastSubject.create();
observable.subscribe(pSubject);

pSubject.subscribe(it -> System.out.println("onNext: " + it));

// Output:
// onNext: 1
// onNext: 2
// onNext: 3
// onNext: 4
// onNext: 5
```