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

