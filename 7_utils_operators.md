[Оглавление](README.md)

# 7. Утилитарные операторы

Помимо всех выше описанных операторов: создающих, фильтрующих, трансформирующих, комбинирующих. Есть группа операторов, которые были выделены в отдельную категорию - утилитарные операторы. К этой группе можно отнести такие операторы как:

1. *delay()*
2. *doOnNext() / doOnCompletable() / doOnEach() / doOnSubscribe()*
3. *observeOn()*
4. *subscribeOn()*
5. *timeInterval()*
6. *timeout()*

## 7.1 *delay()*

_Описание:_

Этот оператор сдвигает выбросы от Observable вперед во времени на определенную величину, т.е. модифицирует свой исходный Observable, делая паузу на определенный интервал времени перед испусканием каждого из элементов из Observable.
По умолчанию этот оператор запускается на computation планировщике.

_Пример использования:_

```
editText.textChangeEvents { text -> text.length == editText.textLength }
                .delay(50, MILLISECONDS, schedulers.main())
                .subscribe(::verify)
                .disposeOnDestroy()
```


## 7.2 *doOnNext() / doOnCompletable() / doOnEach() / doOnSubscribe()*

_Описание:_

Выполнить какое-то действие до какого-либо из событий, например, doOnNext() / doOnCompletable() / doOnEach() / doOnSubscribe()

_Пример использования:_

```
settingInteractor
            .fetchSetting()
            .mapList(SettingMapper::map)
            .observeOn(schedulers.main())
            .doOnSuccess(::onFetchSetting)
```

## 7.3 *observeOn()*

_Описание:_

Этот оператор переключает поток исполнения. Переключение происходит в момент жизненного цикла - Emission time.(сверху вниз)

_Пример использования:_

```
Observable.just("A", "BB", "CCC", "DDDD", "EEEEE", "FFFFFF") // UI
                .map(str -> str.length()) // UI
                .observeOn(Schedulers.computation()) //Changing the thread
                .map(length -> 2 * length)
                .subscribe()
```

## 7.4 *subscribeOn()*

_Описание:_

Этот оператор переключает поток исполнения. Переключение происходит в момент жизненного цикла - subscription time.(Cнизу вверх)

_Пример использования:_

```
Observable.just("A", "BB", "CCC", "DDDD", "EEEEE", "FFFFFF") // UI
                .map(str -> str.length()) // UI
                .subsctibeOn(Schedulers.computation())
                .map(length -> 2 * length)
                .subscribe()
```

## 7.5 *timeInterval()*

_Описание:_

Этот оператор преобразует Observable, испускающий элементы, в тот, который испускает количество времени, прошедшее между этими испусканиями, т.е. если нас больше интересует, сколько времени прошло с момента последнего элемента, а не абсолютный момент времени, когда элементы были выпущены, мы можем использовать timeInterval() метод.

_Пример использования:_

```
Observable.interval(100, TimeUnit.MILLISECONDS)
                .take(3)
                .timeInterval()
                .subscribe()
```

## 7.6 *timeout()*

_Описание:_

Этот оператор отражает исходный Observable, но выдает уведомление об ошибке, если в течение определенного периода времени отсутствуют какие-либо испускаемые элементы.

_Пример использования:_

```
Observable.just(1l, 2l, 3l, 4l, 5l, 6l)
          .timer(1, TimeUnit.SECONDS)
          .timeout(500, TimeUnit.MILLISECONDS)
          .subscribe()
```