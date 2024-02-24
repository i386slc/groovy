# Кратко о итерации Map

## 1. Введение

В этом коротком руководстве мы рассмотрим способы перебора карты в Groovy с использованием стандартных функций языка, таких как each, eachWithIndex и цикл for-in.

## 2. Метод each

Предположим, у нас есть следующая карта:

```groovy
def map = [
    'FF0000' : 'Red',
    '00FF00' : 'Lime',
    '0000FF' : 'Blue',
    'FFFF00' : 'Yellow'
]
```

Мы можем перебирать карту, предоставляя методу each простое замыкание:

```groovy
map.each { println "Hex Code: $it.key = Color Name: $it.value" }
```

Мы также можем немного улучшить читабельность, дав имя входной переменной:

```groovy
map.each { entry -> println "Hex Code: $entry.key = Color Name: $entry.value" }
```

Или, если мы предпочитаем использовать ключ и значение отдельно, мы можем указать их отдельно в нашем замыкании:

```groovy
map.each { key, val ->
    println "Hex Code: $key = Color Name $val"
}
```

**В Groovy карты, созданные с помощью литеральных обозначений, упорядочены**. Мы можем ожидать, что наш вывод будет в том же порядке, который мы определили в нашей исходной карте.

## 3. Метод eachWithIndex

Иногда нам нужно знать индекс index во время итерации.

Например, предположим, что мы хотим сделать отступ для каждой второй строки на нашей карте. Чтобы сделать это в Groovy, мы будем использовать метод eachWithIndex с переменными entry и index:

```groovy
map.eachWithIndex { entry, index ->
    def indent = ((index == 0 || index % 2 == 0) ? "   " : "")
    println "$index Hex Code: $entry.key = Color Name: $entry.value"
}
```

## 4. Использование цикла For-in

С другой стороны, если наш вариант использования лучше подходит для императивного программирования, мы также можем использовать оператор for-in для перебора нашей карты:

```groovy
for (entry in map) {
    println "Hex Code: $entry.key = Color Name: $entry.value"
}
```

## 5. Заключение

В этом коротком руководстве мы узнали, как перебирать карту, используя методы Groovy each и eachWithIndex и цикл for-in.

Пример кода доступен на [GitHub](https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-collections).
