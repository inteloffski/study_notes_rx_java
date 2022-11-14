[Оглавление](README.md)

# 5. Комбинирующие операторы

В RxJava существует большое количество комбинирующих операторов, которые объединяют разные Observable. Рассмотрим следующие операторы:

1. *combineLatest()*
2. *merge()*
3. *zip()*
4. *startWith()*
5. *concat()*

## 5.1 *combineLatest()*

_Описание:_

Этот оператор используется, когда элемент испускается одним из двух Observable, а последний элемент, испускаемый каждым Observable, объединяется через указанную функцию, и результирующие элементы испускаются на основе результатов этой функции.

_Пример использования:_

```
// Example 1
val observable1: Observable<Long> = Observable.interval(100, TimeUnit.MILLISECONDS)
val observable2: Observable<Long> = Observable.interval(50, TimeUnit.MILLISECONDS)

Observable.combineLatest(observable1, observable2){ obs1, obs2 ->
"Refreshed Observable1: " + obs1 + " refreshed Observable2: " + obs2
}

//Output
Refreshed Observable1: 0 refreshed Observable2: 1
Refreshed Observable1: 0 refreshed Observable2: 2
Refreshed Observable1: 0 refreshed Observable2: 3
Refreshed Observable1: 1 refreshed Observable2: 3
Refreshed Observable1: 1 refreshed Observable2: 4

// Example 2

val sourceFirst = Observable.just(1, 2, 3, 4, 5, 6, 7)
val sourceSecond = Observable.just(11, 22, 33, 44, 55, 66, 77)

Observable.combineLatest(sourceFirst,sourceSecond) { s1, s2 ->
"" + s1 + s2
}.subscribe(
{ element ->
Log.d("onNext", element.toString())
}
)
//Output
18, 29, 40, 51, 62, 73, 84
```

## 5.2 *merge()*

_Описание:_

Объединить несколько Observable в один, объединив их выбросы

_Пример использования:_

```
val sourceFirst = Observable.just(1,2,3,4,5,6,7)
val sourceSecond = Observable.just(11,22,33,44,55,66,77)

Observable.merge(sourceFirst, sourceSecond)
          .subscribe({ element -> Log.d("onNext:", element.toString())})

//output
onNext: 1, onNext: 2, onNext: 3, onNext: 4, onNext: 5, onNext: 6, onNext: 7, onNext: 11, onNext: 22, onNext: 33, onNext: 44, onNext: 55, onNext: 66, onNext: 77
```


## 5.3 *zip()*

_Описание:_

Объединять выбросы нескольких Observable вместе с помощью указанной функции и испускать отдельные элементы для каждой комбинации на основе результатов этой функции.

_Пример использования:_

```
val sourceFirst = Observable.just(1, 2, 3, 4, 5, 6, 7)
val sourceSecond = Observable.just(11, 22, 33, 44, 55, 66, 77)

        Observable.zip(
            sourceFirst,
            sourceSecond
        ) { s1, s2 ->
            s1 + s2
        }
            .subscribe(
                { element ->
                    Log.d("onNext", element.toString())
                }
            )
//Output
12, 24, 36, 48, 60, 72, 84
```

## 5.4 *startWith()*

_Описание:_

Если вы хотите, чтобы Observable выдавал определенную последовательность элементов, прежде чем он начнет выдавать элементы, то примените оператор startWith()

_Пример использования:_

```
Observable.just(1,2,3,4)
            .startWith(100)
            .subscribe(
                {Log.d("onNext", it.toString())}
            )
//output
100,1,2,3,4
```

## 5.5 *concat()*

_Описание:_

Этот оператор объединяет выходные данные двух или более Observable в один Observable, не чередуя их, т. е. первый Observable завершает свою эмиссию до начала второго и так далее, если наблюдаемых больше.

_Пример использования:_

```
val obs1 = Observable.just(1, 2, 3, 4)
val obs2 = Observable.just(5, 6, 7, 8)

Observable.concat(obs1,obs2)
          .subscribe({Log.d("TAG", it.toString())})

// Разница между merge() и concat()заключается в том, что у merge()
// выходные данные могут переплетаться, а concat() перед обработкой новых эмиссий ожидается завершение предыдущих эмиссий.

```