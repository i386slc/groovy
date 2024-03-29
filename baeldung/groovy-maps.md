# Groovy Maps

## 1. Обзор

[Groovy ](https://www.baeldung.com/groovy-language)расширяет API [Map](https://www.baeldung.com/java-collections) в [Java](https://www.baeldung.com/category/java/), предоставляя методы для таких операций, как фильтрация, поиск и сортировка. Он также предоставляет множество сокращенных способов создания карт и управления ими.

В этом уроке мы рассмотрим способ работы с картами в Groovy.

## 2. Создание Groovy Map

Мы можем использовать синтаксис литерала карты `[k:v]` для создания карт. По сути, это позволяет нам создавать экземпляр карты и определять записи в одной строке.

Пустую карту можно создать с помощью:

```groovy
def emptyMap = [:]
```

Аналогично, можно создать экземпляр карты со значениями, используя:

```groovy
def map = [name: "Jerry", age: 42, city: "New York"]
```

Обратите внимание, что **ключи не заключены в кавычки**, и по умолчанию Groovy создает экземпляр [java.util.LinkedHashMap](https://www.baeldung.com/java-linked-hashmap). Мы можем переопределить это поведение по умолчанию, используя оператор as.

## 3. Добавление предметов

Начнем с определения карты:

```groovy
def map = [name:"Jerry"]
```

Мы можем добавить ключ в карту:

```groovy
map["age"] = 42
```

Однако есть еще один способ, более похожий на Javascript, — использование обозначения свойств (оператор точки):

```groovy
map.city = "New York"
```

Другими словами, Groovy поддерживает доступ к парам ключ-значение подобно бину.

Мы также можем использовать переменные вместо литералов в качестве ключей при добавлении новых элементов на карту:

```groovy
def hobbyLiteral = "hobby"
def hobbyMap = [(hobbyLiteral): "Singing"]
map.putAll(hobbyMap)
assertTrue(hobbyMap.hobby == "Singing")
assertTrue(hobbyMap[hobbyLiteral] == "Singing")
```

Во-первых, нам нужно создать новую переменную, в которой будет храниться ключевое _hobby_. Затем мы используем эту переменную, заключенную в круглые скобки, с синтаксисом литерала карты, чтобы создать другую карту.

## 4. Получение предметов

Литеральный синтаксис или обозначение свойств можно использовать для получения элементов из карты.

Для карты, определенной как:

```groovy
def map = [name: "Jerry", age: 42, city: "New York", hobby: "Singing"]
```

Мы можем получить значение, соответствующее ключу _name_:

```groovy
assertTrue(map["name"] == "Jerry")
```

или

```groovy
assertTrue(map.name == "Jerry")
```

## 5. Удаление элементов

Мы можем удалить любую запись с карты на основе ключа, используя метод `remove()`, но иногда нам может потребоваться удалить с карты несколько записей. Мы можем сделать это, используя метод `minus()`.

Метод `minus()` принимает карту _Map_ и возвращает новую карту _new Map_ после удаления всех записей данной карты из базовой карты:

```groovy
def map = [1:20, a:30, 2:42, 4:34, ba:67, 6:39, 7:49]

def minusMap = map.minus([2:42, 4:34]);
assertTrue(minusMap == [1:20, a:30, ba:67, 6:39, 7:49])
```

Далее мы также можем удалять записи на основе условия. Мы можем добиться этого, используя метод `removeAll()`:

```groovy
minusMap.removeAll{it -> it.key instanceof String}
assertTrue(minusMap == [1:20, 6:39, 7:49])
```

И наоборот, чтобы сохранить все записи, удовлетворяющие условию, мы можем использовать метод `retainAll()`:

```groovy
minusMap.retainAll{it -> it.value % 2 == 0}
assertTrue(minusMap == [1:20])
```

## 6. Итерация по записям

Мы можем [перебирать записи](https://www.baeldung.com/groovy-map-iterating), используя методы `each()` и `eachWithIndex()`.

Метод `each()` предоставляет неявные параметры, такие как entry, key и value, которые соответствуют текущей записи.

Метод `eachWithIndex()` также предоставляет индекс в дополнение к _Entry_. Оба метода принимают замыкание [Closure](https://www.baeldung.com/groovy-closures) в качестве аргумента.

В следующем примере мы пройдемся по каждой записи _Entry_. Замыкание _Closure_, передаваемое методу `each()`, получает пару ключ-значение из записи неявного параметра и печатает ее:

```groovy
map.each{entry -> println "$entry.key: $entry.value"}
```

Далее мы воспользуемся методом `eachWithIndex()` для печати текущего индекса вместе с другими значениями:

```groovy
map.eachWithIndex{entry, i -> println "$i $entry.key: $entry.value"}
```

Также можно попросить ключ _key_, значение _value_ и индекс предоставляться отдельно:

```groovy
map.eachWithIndex{key, value, i -> println "$i $key: $value"}
```

## 7. Фильтрация

Мы можем использовать методы `find()`, `findAll()` и `grep()` для фильтрации и поиска записей карты на основе ключей и значений. Начнем с определения карты для выполнения этих методов:

```groovy
def map = [name:"Jerry", age: 42, city: "New York", hobby:"Singing"]
```

Сначала мы рассмотрим метод `find()`, который принимает замыкание _Closure_ и возвращает первую запись _Entry_, соответствующую условию замыкания:

```groovy
assertTrue(map.find{it.value == "New York"}.key == "city")
```

Аналогично, `findAll()` также принимает замыкание, но возвращает карту Map со всеми парами ключ-значение, которые удовлетворяют условию замыкания:

```groovy
assertTrue(map.findAll{it.value == "New York"} == [city : "New York"])
```

Однако если мы предпочитаем использовать список _List_, мы можем использовать `grep()` вместо `findAll()`:

```groovy
map.grep{
    it.value == "New York"
}.each{
    it -> assertTrue(it.key == "city" && it.value == "New York")
}
```

Сначала мы использовали `grep()` для поиска записей со значением New York. Затем, чтобы продемонстрировать, что тип возвращаемого значения — _List_, мы пройдемся по результату `grep()`. Для каждой записи _Entry_ в списке, доступной в неявном параметре, мы проверяем, соответствует ли она ожидаемому результату.

Далее, чтобы выяснить, все ли элементы на карте удовлетворяют условию, мы можем использовать `every()`, которое возвращает логическое значение _Boolean_.

Давайте проверим, все ли значения на карте имеют тип String:

```groovy
assertTrue(map.every{it -> it.value instanceof String} == false)
```

Точно так же мы можем использовать `any()`, чтобы определить, соответствуют ли какие-либо элементы на карте условию:

```groovy
assertTrue(map.any{it -> it.value instanceof String} == true)
```

## 8. Трансформация и сборка коллекции

Иногда нам может потребоваться преобразовать записи в карте в новые значения. Используя методы `collect()` и `collectEntries()`, можно преобразовывать и собирать записи в коллекцию Collection или карту Map соответственно.

Давайте посмотрим на несколько примеров. Учитывая карту идентификаторов сотрудников и самих сотрудников:

```groovy
def map = [
  1: [name:"Jerry", age: 42, city: "New York"],
  2: [name:"Long", age: 25, city: "New York"],
  3: [name:"Dustin", age: 29, city: "New York"],
  4: [name:"Dustin", age: 34, city: "New York"]]
```

Мы можем собрать имена всех сотрудников в список с помощью функции `collect()`:

```groovy
def names = map.collect{entry -> entry.value.name}
assertTrue(names == ["Jerry", "Long", "Dustin", "Dustin"])
```

Затем, если нас интересует уникальный набор имен, мы можем указать коллекцию, передав объект Collection:

```groovy
def uniqueNames = map.collect([] as HashSet){entry -> entry.value.name}
assertTrue(uniqueNames == ["Jerry", "Long", "Dustin"] as Set)
```

Если мы хотим изменить имена сотрудников на карте со строчных на прописные, мы можем использовать collectEntries. Этот метод возвращает карту преобразованных значений:

```groovy
def idNames = map.collectEntries{key, value -> [key, value.name]}
assertTrue(idNames == [1:"Jerry", 2:"Long", 3:"Dustin", 4:"Dustin"])
```

Наконец, также **можно использовать методы collect в сочетании с методами find и findAll** для преобразования отфильтрованных результатов:

```groovy
def below30Names = map.findAll{it.value.age < 30}.collect{key, value -> value.name}
assertTrue(below30Names == ["Long", "Dustin"])
```

Здесь мы найдем всех сотрудников в возрасте от 20 до 30 лет и соберем их в карте.

## 9. Группировка

Иногда нам может потребоваться сгруппировать некоторые элементы карты в подкарты на основе условия.

Метод `groupBy()` возвращает карту карт, и каждая карта содержит пары ключ-значение, которые дают один и тот же результат для данного условия:

```groovy
def map = [1:20, 2: 40, 3: 11, 4: 93]
     
def subMap = map.groupBy{it.value % 2}
assertTrue(subMap == [0:[1:20, 2:40], 1:[3:11, 4:93]])
```

Другой способ создания подкарт — использование subMap(). Он отличается от groupBy() тем, что позволяет группировать только по ключам:

```groovy
def keySubMap = map.subMap([1,2])
assertTrue(keySubMap == [1:20, 2:40])
```

В этом случае записи для ключей 1 и 2 возвращаются в новой карте, а все остальные записи отбрасываются.

## 10. Сортировка

Обычно при сортировке нам может потребоваться отсортировать записи на карте по ключу или значению, или по тому и другому. Groovy предоставляет для этой цели метод sort().

Дана карта:

```groovy
def map = [ab:20, a: 40, cb: 11, ba: 93]
```

Если сортировку необходимо выполнить по ключу, мы будем использовать метод sort() без аргументов, который основан на естественном упорядочении:

```groovy
def naturallyOrderedMap = map.sort()
assertTrue([a:40, ab:20, ba:93, cb:11] == naturallyOrderedMap)
```

Или мы можем использовать метод sort(Comparator) для обеспечения логики сравнения:

```groovy
def compSortedMap = map.sort({k1, k2 -> k1 <=> k2} as Comparator)
assertTrue([a:40, ab:20, ba:93, cb:11] == compSortedMap)
```

Далее, чтобы выполнить сортировку по ключу, значению или по тому и другому, мы можем предоставить условие замыкания Closure в sort():

```groovy
def cloSortedMap = map.sort({it1, it2 -> it1.value <=> it1.value})
assertTrue([cb:11, ab:20, a:40, ba:93] == cloSortedMap)
```

## 11. Заключение

В этой статье мы узнали, как создавать карты Map в Groovy. Затем мы рассмотрели различные способы добавления, извлечения и удаления элементов из карты.

Наконец, мы рассмотрели готовые методы Groovy для выполнения общих операций, таких как фильтрация, поиск, преобразование и сортировка.

Как всегда, примеры, рассмотренные в этой статье, можно найти на GitHub.
