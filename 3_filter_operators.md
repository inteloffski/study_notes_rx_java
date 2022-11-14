[Оглавление](README.md)

# 3. Фильтрующие операторы

В RxJava существует большое количество фильтрующих операторов, которые позволяют реализовать самую разнообразную логику. Рассмотрим следующие операторы:

1. *debounce()*
2. *distinct()*
3. *elementAt()*
4. *filter()*
5. *ignoreElement()*
6. *skip()*
7. *take()*

## 3.1 *debounce()*

_Описание:_

Испускает элемент из Observable только в том случае, если прошел заданный промежуток времени внутри оператора debounce() без испускания.
Выполняется на computation scheduler.

_Пример использования:_
```
with(viewBinding) {
            commonView.cleanButtonVisibility(true)
            commonView.setOnClearListener {}
            commonView
                .editText.apply {
                    requestFocus()
                    showSoftwareKeyboard()
                }
                .textChangeEvents()
                .debounce(500L, MILLISECONDS)
                .observeOn(schedulers.main())
                .subscribe(presenter::something)
                .disposeOnDestroy()
}
```

## 3.2 *distinct()*

_Описание:_

Оставляет в последовательности только уникальные элементы

_Пример использования:_
```
Observable.just(10,20,30,20,10).distinct()

//output
10 20 30
```

## 3.3 *elementAt()*

_Описание:_

Этот оператор испускает только один элемент, испускаемый Observable. Мы можем указать позицию, которую нам нужно испустить, с помощью оператора elementAt. Например, elementAt(0) выдаст первый элемент в списке. Функция elementAt() в качестве аргумента принимает номер индекса последовательности

_Пример использования:_
```
Observable.just(1,2,3,4,5).elementAt(2)

//output
3
```

## 3.4 *filter()*

_Описание:_

Этот оператор испускает только те элементы, которые из условия предиката возвращают true

_Пример использования:_
```
Observable.just(1,2,3,4,5).filter { it % 2 == 0 }
```

## 3.5 *ignoreElement()*

_Описание:_

Возвращает Completable, который игнорирует success value, если приходит error,  то возвращается error

_Пример использования:_
```
sessionDataSource
            .fetchClientId()
            .flatMap { clientId ->
                api.applyTransfer(clientId)
            }
            .ignoreElement()
```

## 3.6 *skip()*

_Описание:_

Пропускает первые заданные n элементов.

_Пример использования:_
```
Observable.just(1,2,3,4,5).skip(2)

//output
3,4,5
```

## 3.7 *take()*

_Описание:_

Берет только первые заданные n элементов.

_Пример использования:_
```
Observable.just(1,2,3,4,5).take(2)

//output
1,2
```
