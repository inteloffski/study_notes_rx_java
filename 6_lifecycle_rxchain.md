[Оглавление](README.md)

# 6. Жизненный цикл Rx-цепочек

Для того чтобы лучше понимать как работают операторы в Rx, необходимо понимать, жизненный цикл Rx-цепочек. У Rx-цепочек существует 4 этапа жизненного цикла:

1.  Assembly time - операторы создаются сверху вниз, в текущем потоке выполнения.
2.  Subscription time - подписка операторов. Выполняется снизу вверх. Каждый оператор подписывается на верхний.
3.  Emitting time(Runtime) - начинается, когда все операторы успешно подписались друг на друга. Выполняется сверху вниз
4.  Unsubscribe - вся цепь умирает. Выполнение всех операторов останавливается.

В основном все операторы выполняются во время Runtime, но некоторые операторы могут выполняться во время Subscription time. 
Поэтому какие-то операторы могут влиять на downstream, а какие-то операторы могут влиять на upstream. 
Под upstream и downstream, необходимо понимать следующее. Upstream - это все, что выше рассматриваемого оператора. Downstream - это все, что ниже рассматриваемого оператора.

Для наглядности рассмотрим несколько примеров таких цепочек:

```
Observable.just("Hey")
    .subscribeOn(Schedulers.io())
    .map(String::length)
    .subscribeOn(Schedulers.computation())
    .observeOn(AndroidSchedulers.mainThread())
    .doOnSubscribe { doAction() }
    .flatMap {
        doAction()
        Observable.timer(1, TimeUnit.SECONDS)
            .subscribeOn(Schedulers.single())
            .doOnSubscribe { doAction() }
    }.subscribe {
        doAction()
    }
```

## 6.1 *Assembly time*

Во-первых, Observable.just() создается со значением “Hey”, если мы посмотрим на исходных код, то увидим, что наше значение просто кэшируется в ObservableJust

```
@CheckReturnValue
@NonNull
@SchedulerSupport(SchedulerSupport.NONE)
public static <T> Observable<T> just(T item) {
   ObjectHelper.requireNonNull(item, "item is null");
   return RxJavaPlugins.onAssembly(new ObservableJust<T>(item));
}
```

Во-вторых, вызывается оператор subscribeOn() аналогично Observable.just(). Обратите внимание, что на данном этапе здесь происходит только создание операторов, без изменения потока. Основное отличие от предыдущего оператора в том, что этот поток хранит ссылку на верхний Observable:

```
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.CUSTOM)
public final Observable<T> subscribeOn(Scheduler scheduler) {
   ObjectHelper.requireNonNull(scheduler, "scheduler is null");
   return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
}
```

Все остальные операторы до subscribe() создаются таким же образом и не выполняют никаких действий. Все операторы, кроме первого хранят ссылку на предыдущий вызов.

```
1.  Observable.just("Hey")
    кэширует переданное значение

2.  subscribeOn(Schedulers.io())
    кэширует планировщик подписки

3.  map(String::length)
    кэширует лямбду

4.  subscribeOn(Schedulers.computation())
    кэширует планировщик подписки

5.  observeOn(AndroidSchedulers.mainThread())
    кэширует излучающий планировщик

6.  doOnSubscribe { doAction() }
    кэширует лямбду, которая будет выполняться при подписке

7. flatMap{ ... }
   кэширует лямбду, которая будет выполняться при подписке
```

## 6.2 *Subscription time*

Процесс подписки идет снизу вверх и начинается сразу после вызова подписки. На этом этапе некоторые операторы уже могут выполняться. Когда мы вызываем subscribe, подписка распространяется не на всю цепочку, а только на оператор flatMap, который находится выше. Затем flatMap подписывается на оператор, ссылку на который он сохранил при создании doOnSubscribe(). Вот почему операторы сохраняют ссылку на upstream.

```
Поведение на этапе подписки

Поведение на этапе подписки:

1. flatMap { ... }
    
    → Подписывается на upstream
    
2. doOnSubscribe { ... }
    
    → Выполняет код в лямбде
    
    → Подписывается на upstream
    
    → Поток выполнения: тот, где вызывается subscribe 
    
3. observeOn(AndroidSchedulers.mainThread())
    
    → Подписывается на upstream
    
4. subscribeOn(Schedulers.computation())
    
    → Изменяет планировщик на computation (предыдущий планировщик = планировщик подписки)
    
    → Подписывается на upstream
    
5. map(String::length)
    
    → Подписывается на upstream
    
    → Подписка на computation планировщике
    
6. subscribeOn(Schedulers.io())
    
    → Изменяет поток на IO (предыдущий поток = computation)
    
    → Подписывается на upstream

Из всех операторов были выполнены doOnSubscribe и два subscribeOn.
Планировщик исполнения изменялся дважды: сначала на computation, а затем на IO.
SubscribeOn будет выполняться столько раз, сколько он был вызыван.

Даже если вы напишете subscribeOn([Schedulers.io](http://schedulers.io/)()) 3 раза подряд, предыдущие 2 не будут проигнорированы.

Давайте посмотрим на следующий пример:

Observable.just("Hey")
     .doOnSubscribe { doAction() }
     .subscribeOn(Schedulers.io()) 
     .doOnSubscribe { doAction() }
     .subscribeOn(Schedulers.io()) 
     .doOnSubscribe { doAction() }
     .subscribeOn(Schedulers.io()) 
     .subscribe {     }
```

## 6.3 *Runtime*

Как только мы доходим до Observable у которого, нет ссылки на родительский поток, начинается процесс эмиссии. Он идет от корня Observable к самому нижнему оператору в иерархии.

```
Поведение цепочки:

1. Observsble.just("Hey")

   → Эмитить «Hey» на IO планировщике

2. subscribeOn(Schedulers.io())

   → Проксирует “Hey” по иерархии операторов ниже. Планировщик не меняется. IO - планировщик.

3. map(String::length)

   → Запускает код в лямбда-выражении на IO планировщике

4. subscribeOn(Schedulers.computation())

   → Проксирует значение по иерархии операторов ниже. Планировщик не меняется. IO - планировщик.

5. observeOn(AndroidSchedulers.mainThread())

   → Изменяет планировщик на mainThread, выдает значение.

   Важно помнить, что если мы уже находимся в основном потоке, элемент можно не эмитить сразу, потому что он в любом случае пройдет через`Handler/Looper/MessageQueue.`

6. doOnSubscribe { doAction() }

   → Проксирует значение по иерархии операторов ниже. Планировщик не меняется. mainThread - планировщик.

7. flatMap{...}

   Это сложно. Во-первых, давайте вспомним его содержимое:

   .flatMap {
      doAction()
      Observable.timer(1, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.single())
                .doOnSubscribe { doAction() }
}

Цепочка внутри flatMap пройдет все этапы creation, subscription, emission.
В этом случае отписки не будет, т.к. таймер продолжит выдавать значения.

Операторы timer/delay/interval исполняются на computation планировщике.
Этот шаг не очевиден, но влияет на downstream.

Поэтому помните, если вы комбинируете эти операторы с http/database запросами, измените на scheduler перед IO
выполнением этого запроса. В противном случае передайте планировщик подписки в качестве параметра непосредственно
в timer/delay/interval.
```

## 6.4 *Вывод*

Все операторы создаются сверху вниз и подписываются друг на друга по одному снизу вверх. Таким образом, они образуют `LinkedListObservable`. Эмиссия идет сверху вниз, начиная с корня.

- Есть такие операторы `startWith/doOnSubscribe`, которые выполняются в процессе подписки. Более того, они выполняются столько раз, сколько вы их написали.

- `subscribeOn`выполняется в процессе подписки и влияет как на верхний, так и на нижний потоки. Он также может менять планировщик исполнения столько раз, сколько вы его написали. Применяется тот, который находится ближе всего к вершине цепочки.

- `observeOn`влияет только на нижние потоки и выполняется во время эмиссии. Он меняет планировщик исполнения столько раз, сколько вы его написали.

- `flatMap`запускает цепочку только во время передачи данных корневой цепочки. В процессе подписки на корневой поток не выполняются никакие действия.

- Операторы `interval/delay/timer` выполняются на `computation` планировщике по умолчанию.
