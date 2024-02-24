# Работа JSON в Groovy

## 1. Введение

В этой статье мы опишем и посмотрим примеры работы с JSON в приложении Groovy.

Прежде всего, чтобы запустить примеры из этой статьи, нам нужно настроить `pom.xml`:

```xml
<build>
    <plugins>
        // ...
        <plugin>
            <groupId>org.codehaus.gmavenplus</groupId>
            <artifactId>gmavenplus-plugin</artifactId>
            <version>2.1.0</version>
        </plugin>
    </plugins>
</build>
<dependencies>
    // ...
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-all</artifactId>
        <version>3.0.15</version>
    </dependency>
</dependencies>
```

Самый последний плагин Maven можно найти [здесь](https://mvnrepository.com/artifact/org.codehaus.gmavenplus/gmavenplus-plugin), а последнюю версию groovy-all — [здесь](https://mvnrepository.com/artifact/org.codehaus.groovy/groovy-all).

## 2. Анализ объектов Groovy в JSON

Преобразовать объекты в JSON в Groovy довольно просто. Предположим, у нас есть класс Account:

```groovy
class Account {
    String id
    BigDecimal value
    Date createdAt
}
```

Чтобы преобразовать экземпляр этого класса в строку JSON, нам нужно использовать класс JsonOutput и вызвать статический метод toJson():

```groovy
Account account = new Account(
    id: '123', 
    value: 15.6,
    createdAt: new SimpleDateFormat('MM/dd/yyyy').parse('01/01/2018')
) 
println JsonOutput.toJson(account)
```

В результате мы получим распарсенную строку JSON:

```json
{"value":15.6,"createdAt":"2018-01-01T02:00:00+0000","id":"123"}
```

### 2.1. Настройка вывода JSON

Как мы видим, вывод даты — это не то, что мы хотели. Для этой цели, начиная с версии 2.5, в пакет groovy.json входит специальный набор инструментов.

С помощью класса JsonGenerator мы можем определить параметры вывода JSON:

```groovy
JsonGenerator generator = new JsonGenerator.Options()
  .dateFormat('MM/dd/yyyy')
  .excludeFieldsByName('value')
  .build()

println generator.toJson(account)
```

В результате мы получим отформатированный JSON без исключенного нами поля значения и с отформатированной датой:

```json
{"createdAt":"01/01/2018","id":"123"}
```

### 2.2. Форматирование вывода JSON

С помощью приведенных выше методов мы увидели, что выходные данные JSON всегда представляют собой одну строку, и это может привести к путанице, если придется иметь дело с более сложным объектом.

Однако мы можем отформатировать наш вывод, используя метод PrettyPrint:

```groovy
String json = generator.toJson(account)
println JsonOutput.prettyPrint(json)
```

И мы получаем отформатированный JSON ниже:

```json
{
    "value": 15.6,
    "createdAt": "01/01/2018",
    "id": "123"
}
```

## 3. Преобразование JSON в Groovy-объекты

Мы собираемся использовать класс JsonSlurper в Groovy для преобразования JSON в объекты.

Кроме того, с JsonSlurper у нас есть множество перегруженных методов анализа и несколько конкретных методов, таких как parseText, parseFile и другие.

Мы будем использовать parseText для анализа строки в классе Account:

```groovy
def jsonSlurper = new JsonSlurper()

def account = jsonSlurper.parseText('{"id":"123", "value":15.6 }') as Account
```

В приведенном выше коде у нас есть метод, который получает строку JSON String и возвращает объект Account, который может быть любым объектом Groovy.

Кроме того, мы можем проанализировать строку JSON String в карте Map, вызывая ее без какого-либо приведения, а с помощью динамической типизации Groovy мы можем получить то же самое, что и объект.

### 3.1. Анализ ввода JSON

Реализация парсера по умолчанию для JsonSlurper — JsonParserType.CHAR\_BUFFER, но в некоторых случаях нам придется столкнуться с проблемой синтаксического анализа.

Давайте рассмотрим пример: учитывая строку JSON String  со свойством даты, JsonSlurper не сможет правильно создать объект, поскольку попытается проанализировать дату как строку String:

```groovy
def jsonSlurper = new JsonSlurper()
def account 
  = jsonSlurper.parseText('{"id":"123","createdAt":"2018-01-01T02:00:00+0000"}') as Account
```

В результате приведенный выше код вернет объект Account со всеми свойствами, содержащими нулевые значения.

Чтобы решить эту проблему, мы можем использовать JsonParserType.INDEX\_OVERLAY.

В результате он будет стараться изо всех сил избегать создания массивов символов или строк String:

```groovy
def jsonSlurper = new JsonSlurper(type: JsonParserType.INDEX_OVERLAY)
def account 
  = jsonSlurper.parseText('{"id":"123","createdAt":"2018-01-01T02:00:00+0000"}') as Account
```

Теперь приведенный выше код вернет созданный соответствующим образом экземпляр учетной записи.

### 3.2. Варианты парсера

Кроме того, внутри JsonParserType у нас есть несколько других реализаций:

* JsonParserType.LAX позволит более спокойно анализировать JSON с комментариями, без строк кавычек и т. д.
* JsonParserType.CHARACTER\_SOURCE используется для анализа больших файлов.

## 4. Заключение

Мы рассмотрели большую часть обработки JSON в приложении Groovy на нескольких простых примерах.

Для получения дополнительной информации о классах пакета groovy.json можно просмотреть [документацию Groovy](http://groovy-lang.org/json.html).

Проверьте исходный код классов, используемых в этой статье, а также некоторые модульные тесты в нашем [репозитории GitHub](https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy).
