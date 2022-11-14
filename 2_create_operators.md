[Оглавление](README.md)

# 2. Операторы создания цепочек
В RxJava существует большое количество операторов, которые создаю цепочки. Мы рассмотрим только основные:

1. *fromCallable()*
2. *defer()*
3. *fromIterable()*
4. *just()*
5. *fromFuture()*
6. *interval()*
7. *timer()*

Рассмотрим каждый из операторов подробнее, чтобы понять какой лучше использовать.


## 2.1 *just()*

_Описание:_

Создает Observable, который выдает конкретный элемент. 
Также есть и другие варианты оператора just, который может принимать в качестве аргумента более одного элемента.

_Пример использования:_
```
fun proxy(str: String): Observable<String> = Observable.just(str)
```

## 2.2 *fromCallable()*

_Описание:_

Функция fromCallable() возвращает Observable, когда observer подписывается на него, вызывает указанную функцию, а затем выдает значение, возвращаемое этой функцией. Это позволяет отложить выполнение указанной функции до тех пор, пока observer не подпишется на
ObservableSource. Другими словами это делает функцию “ленивой”.
Также функция fromCallable() умеет заворачивать тип в реактивную обертку Observable.

_Пример использования:_
```
fun proxy(str: String): Observable<String> = Observable.fromCallable { str }
```

## 2.3 *defer()*

_Описание:_

Вызывает Callable для каждого отдельного SingleObserver, чтобы вернуть фактический SingleSource, на который нужно подписаться. Важное отличие от fromCallable, что он не умеет оборачивать в реактивную обертку.

_Пример использования:_
```
fun proxy(str: String) : Observable<String> = Observable.defer { Observable.just(str) }
```
## 2.4 *fromIterable()*

_Описание:_

Преобразовывает последовательность в реактивный стрим(ObservableSource), который выдает элементы в последовательность

_Пример использования:_

```
Пример № 1
fun createIterable(urls: List<String>): Observable<String> = Observable.fromIterable(urls)
```

```
Пример №2
fun createIterable(urls: List<String>): Observable<String> =
        Observable.fromIterable(urls)
            .flatMapSingle { url ->
                Single.just(url)
            }
    }
```

## 2.5 *fromFuture()*

_Описание:_

Преобразовывает из Future в реактивный стрим.

_Пример использования:_
```
Observable.fromFuture(future, Schedulers.io())
          .doOnSubscribe(disposable -> future.run())
          .subscribeOn(Schedulers.io())
          .subscribe(someConsumer())
```

## 2.6 *interval()*

_Описание:_

Оператор Interval возвращает Observable, который испускает бесконечную последовательность целых чисел с постоянным интервалом времени, между каждым событием onNext().
Оператор Interval по умолчанию запускается на computation scheduler, но планировщик можно задать и самостоятельно.

_Пример использования:_
```
Observable.interval(0, 2, TimeUnit.SECONDS)
        .flatMap {
            return@flatMap Observable.create<String> { emitter ->
                Log.d("IntervalExample", "Create")
                emitter.onNext("onNext")
                emitter.onComplete()
            }
        }
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe {
            Log.d("IntervalExample", it)
        }
```

## 2.7 *timer()*

_Описание:_

Возвращает экземпляр Completable, который запускает событие onComplete по истечении заданной задержки.
Оператор timer по умолчанию запускается на computation scheduler, но планировщик можно задать и самостоятельно

_Пример использования:_
```
Completable.timer(1, SECONDS).andThen(verifyIdentityAndPassword())
```




