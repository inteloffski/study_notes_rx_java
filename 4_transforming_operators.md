[Оглавление](README.md)

# 4. Трансформирующие операторы

В RxJava существует огромное количество трансформирующих операторов, которые позволяют преобразовывать поток данных.
Рассмотрим следующие операторы:

1. *flatMap()*
2. *map()*
3. *switchMap()*
4. *concatMap()*
5. *buffer()*
6. *scan()*
7. *groupBy()*

## 4.1 *flatMap()*

_Описание:_

Оператор FlatMap преобразует Observable, применяя указанную вами функцию к каждому элементу, испускаемому исходным
Observable, где эта функция возвращает Observable, который сам испускает элементы. Затем FlatMap объединяет выбросы этих
результирующих Observables, выдавая эти объединенные результаты как свою собственную последовательность.

_Пример использования:_

```
fun fetchData(): Single<String> =
        sessionDataSource
            .fetchId()
            .flatMap { clientId ->
                api
                    .requestTo(
                        Request(
                            clientId = clientId,
                            pattern = "",
                            parameters = emptyList()
                        )
                    )
                    .map { response -> response.id }
            }
```

## 4.2 *map()*

_Описание:_

Этот оператор преобразует элементы, испускаемые Observable, применяя функцию к каждому элементу. Оператор map()
позволяет нам изменить испускаемый элемент из Observable, а затем испускает измененный элемент. В отличие от flatMap()
он не возвращает новый Observable, он лишь меняет элементы текущего.

_Пример использования:_

```
fun fetchData(): Single<String> =
        sessionDataSource
            .fetchId()
            .flatMap { clientId ->
                api
                    .requestTo(
                        Request(
                            clientId = clientId,
                            pattern = "",
                            parameters = emptyList()
                        )
                    )
                    .map { response -> response.id }
            }
```

## 4.3 *switchMap()*

_Описание:_

Возвращает новый ObservableSource, применяя функцию, которую вы предоставляете, к каждому элементу, испускаемому
источником ObservableSource, возвращает ObservableSource, а затем испуская элементы, испускаемые последним из этих
ObservableSource. Оператор switchMap отписывается от предыдущего события, каждый раз, когда приходит новый. Другими
словами, switchMap подпишется только на последнее событие в Observable и отменит подписку от всех предыдущих

_Пример использования:_

```
fun EditText.search(performSearch: (String) -> Unit) =
    textChangeEvents()
        .map { event -> event.text.toString() }
        .doOnNext {
            setCompoundDrawableThemed(
                defaultLeft = getCompoundDrawableResources()[0],
                defaultRight = if (it.isEmpty()) 0 else R.drawable.ic_clear_text
            )
        }
        .scanWith({ "" to 0 }) { (oldStr, _), newStr ->
            newStr to abs(oldStr.length - newStr.length)
        }
        .debounceIf(
            { (_, lengthDiff) -> lengthDiff <= 1 },
            500L,
            MILLISECONDS,
            AndroidSchedulers.mainThread()
        )
        .map { (newStr, _) -> newStr }
        .distinctUntilChanged()
        .filter(String::isNotEmpty)
        .switchMapSingle { searchText ->
            Single.fromCallable {
                performSearch(searchText)
            }
        }
        
//switchMap{ } отличается от switchMapSingle{ } тем, что он ожидает Single внутри лямбды
```

## 4.4 *concatMap()*

_Описание:_

ConcatMap работает почти так же, как flatMap, но сохраняет порядок элементов. Но у concatMap есть один большой
недостаток: он ждет, пока каждый наблюдаемый объект завершит всю работу, пока не будет обработан следующий.

_Пример использования:_

```
private fun watchSession(id: String): Observable<Session> =
        sessionRepository
            .watchSession(id)
            .filter { id -> response.id == id }
            .concatMap { response ->
                val time =
                        response
                            .exchangeTime
                            .toOffsetDateTime()
                val stateChangeTime = when (response.isFavorite) {
                    true -> response.dateNew.toOffsetDateTime()
                    else -> response.dateClose.toOffsetDateTime()
                }            
                Observable
                    .intervalRange(0L, time, 0, 1, SECONDS)
                    .map { second -> time - second }
                    .map { stateTime ->
                        if (stateTime == 0L) {
                            Session(response.id, stateTime)
                        } else {
                            TradingSession(response.id, stateChangeTime)
                        }
                    }
            }
```

## 4.5 *buffer()*

_Описание:_

Собирает элементы из Observable в пакеты и выпускать эти пакеты.

_Пример использования:_

```
Observable.just(1,2,3,4)
          .buffer(2)

//output
onNext: 1, 2
onNext: 3, 4
```

## 4.6 *scan()*

_Описание:_

Оператор scan применит функцию к первому элементу, а затем испускает результат этой функции как собственное первое значение

_Пример использования:_

```
Observable.just(1,2,3,4)
          .scan { init: Int, acc: Int -> acc + 2}
//Output
1,4,5,6

// 0ая итерация 
init = 1
// 1ая итерация
init = 1, acc = 2
acc + 2 = 4
// 2ая итерация
init = 4, acc = 3
acc + 2 = 5
// 3ая итерация
init = 5, acc = 4
acc + 2 = 6
```

## 4.7 *groupBy()*

_Описание:_

Разделить Observable на набор Observable, каждый из которых испускает группу элементов из исходного Observable, организованную по ключу

_Пример использования:_

```
Observable.range(1,9)
          .groupBy { i-> i % 3 }
          .flatMap { t-> Observable.just(t.key)}
```