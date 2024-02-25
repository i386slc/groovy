# Обзор операций I/O в Groovy

## 1. Введение

Хотя в [Groovy](https://www.baeldung.com/groovy-language) мы можем работать с вводом-выводом так же, как и в Java, **Groovy расширяет функциональность ввода-вывода Java с помощью ряда вспомогательных методов**.

В этом уроке мы рассмотрим чтение и запись файлов, обход файловых систем и сериализацию данных и объектов с помощью методов расширения File Groovy.

Там, где это возможно, мы будем давать ссылки на наши соответствующие статьи по Java для удобства сравнения с эквивалентом Java.

## 2. Чтение файлов

Groovy добавляет удобную функциональность для [чтения файлов](https://www.baeldung.com/groovy-file-read) в виде методов eachLine, методов получения BufferedReaders и InputStreams, а также способов получения всех данных файла с помощью одной строки кода.

Java 7 и Java 8 имеют аналогичную поддержку [чтения файлов на Java](https://www.baeldung.com/reading-file-in-java).

### 2.1. Чтение с eachLine

Имея дело с текстовыми файлами, нам часто приходится читать каждую строку и обрабатывать ее. **Groovy предоставляет удобное расширение java.io.File с помощью метода eachLine**:

```groovy
def lines = []

new File('src/main/resources/ioInput.txt').eachLine { line ->
    lines.add(line)
}
```

Замыкание, предоставляемое eachLine, также имеет полезный необязательный номер строки. Давайте использовать номер строки, чтобы получить только определенные строки из файла:

```groovy
def lineNoRange = 2..4
def lines = []

new File('src/main/resources/ioInput.txt').eachLine { line, lineNo ->
    if (lineNoRange.contains(lineNo)) {
        lines.add(line)
    }
}
```

**По умолчанию нумерация строк начинается с единицы**. Мы можем предоставить значение, которое будет использоваться в качестве первого номера строки, передав его в качестве первого параметра методу eachLine.

Давайте начнем номера строк с нуля:

```groovy
new File('src/main/resources/ioInput.txt').eachLine(0, { line, lineNo ->
    if (lineNoRange.contains(lineNo)) {
        lines.add(line)
    }
})
```

Если в eachLine выдается исключение, **Groovy гарантирует закрытие файлового ресурса**. Очень похоже на try-with-resources или на try-finally в Java.

### 2.2. Чтение с Reader

Мы также можем легко получить BufferedReader из объекта Groovy File. Мы можем использовать withReader, чтобы получить BufferedReader для файлового объекта и передать его замыканию:

```groovy
def actualCount = 0
new File('src/main/resources/ioInput.txt').withReader { reader ->
    while(reader.readLine()) {
        actualCount++
    }
}
```

Как и в случае сeachLine, метод withReader автоматически закроет ресурс при возникновении исключения.

Иногда нам может потребоваться доступность объекта BufferedReader. Например, мы могли бы запланировать вызов метода, который принимает его в качестве параметра. Для этого мы можем использовать метод newReader:

```groovy
def outputPath = 'src/main/resources/ioOut.txt'
def reader = new File('src/main/resources/ioInput.txt').newReader()
new File(outputPath).append(reader)
reader.close()
```

В отличие от других методов, которые мы рассматривали до сих пор, **мы несем ответственность за закрытие ресурса BufferedReader, когда мы получаем BufferedReader таким образом**.

### 2.3. Чтение с помощью InputStreams

Подобно withReader и newReader, **Groovy также предоставляет методы для удобной работы с входными потоками inputStreams**. Хотя мы можем читать текст с помощью InputStreams, и Groovy даже добавляет для этого функциональность, InputStreams чаще всего **используется для двоичных данных.**

Давайте воспользуемся withInputStream, чтобы передать входной поток inputStream замыканию и прочитать байты:

```groovy
byte[] data = []
new File("src/main/resources/binaryExample.jpg").withInputStream { stream ->
    data = stream.getBytes()
}
```

Если нам нужен объект InputStream, мы можем получить его с помощью newInputStream:

```groovy
def outputPath = 'src/main/resources/binaryOut.jpg'
def is = new File('src/main/resources/binaryExample.jpg').newInputStream()
new File(outputPath).append(is)
is.close()
```

Как и в случае с BufferedReader, нам нужно самостоятельно закрыть наш ресурс InputStream, когда мы используем newInputStream, но не при использовании withInputStream.

### 2.4. Другие способы чтения

Давайте закончим тему чтения, рассмотрев несколько методов, которые Groovy позволяет получить все данные файла в одном операторе.

Если мы хотим, чтобы строки нашего файла были в списке List, мы можем использовать collect с итератором it, который он передает в замыкание:

```groovy
def actualList = new File('src/main/resources/ioInput.txt').collect {it}
```

Чтобы поместить строки нашего файла в массив строк String, мы можем использовать как `String[]`:

```groovy
def actualArray = new File('src/main/resources/ioInput.txt') as String[]
```

Для коротких файлов мы можем получить все содержимое в строке String, используя текст text:

```groovy
def actualString = new File('src/main/resources/ioInput.txt').text
```

А при работе с двоичными файлами есть метод bytes:

```groovy
def contents = new File('src/main/resources/binaryExample.jpg').bytes
```

## 3. Запись файлов

Прежде чем мы начнем [записывать в файлы](https://www.baeldung.com/java-write-to-file), давайте настроим текст, который мы будем выводить:

```groovy
def outputLines = [
    'Line one of output example',
    'Line two of output example',
    'Line three of output example'
]
```

### 3.1. Пишем с Writer

Как и при чтении файла, **мы можем легко получить BufferedWriter из объекта File**.

Давайте воспользуемся withWriter, чтобы получить BufferedWriter и передать его замыканию:

```groovy
def outputFileName = 'src/main/resources/ioOutput.txt'
new File(outputFileName).withWriter { writer ->
    outputLines.each { line ->
        writer.writeLine line
    }
}
```

Использование withWriter закроет ресурс в случае возникновения исключения.

В Groovy также есть метод получения объекта BufferedWriter. Давайте получим BufferedWriter, используя newWriter:

```groovy
def outputFileName = 'src/main/resources/ioOutput.txt'
def writer = new File(outputFileName).newWriter()
outputLines.forEach {line ->
    writer.writeLine line
}
writer.flush()
writer.close()
```

Мы несем ответственность за очистку и закрытие нашего объекта BufferedWriter, когда мы используем newWriter.

### 3.2. Запись с использованием потоков вывода Output Streams

Если мы записываем двоичные данные, **мы можем получить OutputStream, используя withOutputStream или newOutputStream**.

Давайте запишем несколько байтов в файл, используя withOutputStream:

```groovy
byte[] outBytes = [44, 88, 22]
new File(outputFileName).withOutputStream { stream ->
    stream.write(outBytes)
}
```

Давайте получим объект OutputStream с помощью newOutputStream и используем его для записи нескольких байтов:

```groovy
byte[] outBytes = [44, 88, 22]
def os = new File(outputFileName).newOutputStream()
os.write(outBytes)
os.close()
```

Как и в случае с InputStream, BufferedReader и BufferedWriter, мы сами несем ответственность за закрытие OutputStream при использовании newOutputStream.

### 3.3. Запись с помощью оператора <<

Поскольку запись текста в файлы очень распространена, оператор `<<` напрямую обеспечивает эту функцию.

Давайте воспользуемся оператором `<<`, чтобы написать несколько простых строк текста:

```groovy
def ln = System.getProperty('line.separator')
def outputFileName = 'src/main/resources/ioOutput.txt'
new File(outputFileName) << "Line one of output example${ln}" + 
  "Line two of output example${ln}Line three of output example"
```

### 3.4. Запись двоичных данных с помощью Bytes

**Ранее в статье мы видели, что мы можем получить все байты из двоичного файла, просто обратившись к полю bytes.**

Запишем двоичные данные таким же образом:

```groovy
def outputFileName = 'src/main/resources/ioBinaryOutput.bin'
def outputFile = new File(outputFileName)
byte[] outBytes = [44, 88, 22]
outputFile.bytes = outBytes
```

## 4. Обход файловых деревьев

**Groovy также предоставляет нам простые способы работы с деревьями файлов**. В этом разделе мы собираемся сделать это с eachFile, eachDir и их вариантами, а также методом traverse.

### 4.1. Обход файлов с помощью eachFile

Давайте перечислим все файлы и каталоги в каталоге, используя eachFile:

```groovy
new File('src/main/resources').eachFile { file ->
    println file.name
}
```

Другой распространенный сценарий работы с файлами — необходимость фильтровать файлы по имени. Давайте перечислим только файлы, которые начинаются с `"io"` и заканчиваются на `".txt"`, используя eachFileMatch и регулярное выражение:

```groovy
new File('src/main/resources').eachFileMatch(~/io.*\.txt/) { file ->
    println file.name
}
```

**Методы eachFile и eachFileMatch отображают только содержимое каталога верхнего уровня**. Groovy также позволяет нам ограничить то, что возвращают методы eachFile, передавая этим методам FileType. Возможные варианты: ANY, FILES и DIRECTORIES.

Давайте рекурсивно перечислим все файлы, используя eachFileRecurse и предоставив ему FileType FILES:

```groovy
new File('src/main').eachFileRecurse(FileType.FILES) { file ->
    println "$file.parent $file.name"
}
```

Методы eachFile выдают исключение IllegalArgumentException, если мы предоставляем им путь к файлу вместо каталога.

**Groovy также предоставляет методы eachDir для работы только с каталогами**. Мы можем использовать eachDir и его варианты для достижения того же, что и использование eachFile с типом FileTypes DIRECTORIES.

Давайте рекурсивно перечислим каталоги с помощью eachFileRecurse:

```groovy
new File('src/main').eachFileRecurse(FileType.DIRECTORIES) { file ->
    println "$file.parent $file.name"
}
```

Теперь давайте сделаем то же самое с eachDirRecurse:

```groovy
new File('src/main').eachDirRecurse { dir ->
    println "$dir.parent $dir.name"
}
```

### 4.2. Листинг файлов с помощью Traverse

**Для** [**более сложных случаев обхода каталогов**](https://www.baeldung.com/java-nio2-file-visitor) **мы можем использовать метод traverse**. Он функционирует аналогично eachFileRecurse, но предоставляет возможность возвращать объекты FileVisitResult для управления обработкой.

Давайте воспользуемся traverse для нашего каталога src/main и пропустим обработку дерева в каталоге groovy:

```groovy
new File('src/main').traverse { file ->
   if (file.directory && file.name == 'groovy') {
        FileVisitResult.SKIP_SUBTREE
    } else {
        println "$file.parent - $file.name"
    }
}
```

## 5. Работа с данными и объектами

### 5.2. Сериализация примитивов

В Java мы можем использовать DataInputStream и DataOutputStream для сериализации примитивных полей данных. Groovy также добавляет сюда полезные расширения.

Давайте настроим некоторые примитивные данные:

```groovy
String message = 'This is a serialized string'
int length = message.length()
boolean valid = true
```

Теперь давайте сериализуем наши данные в файл, используя withDataOutputStream:

```groovy
new File('src/main/resources/ioData.txt').withDataOutputStream { out ->
    out.writeUTF(message)
    out.writeInt(length)
    out.writeBoolean(valid)
}
```

И прочитайте его, используя withDataInputStream:

```groovy
String loadedMessage = ""
int loadedLength
boolean loadedValid

new File('src/main/resources/ioData.txt').withDataInputStream { is ->
    loadedMessage = is.readUTF()
    loadedLength = is.readInt()
    loadedValid = is.readBoolean()
}
```

Подобно другим методам with\*, withDataOutputStream и withDataInputStream передают поток замыканию и обеспечивают его правильное закрытие.

### 5.2. Сериализация объектов

**Groovy также опирается на Java ObjectInputStream и ObjectOutputStream, что позволяет нам легко сериализовать объекты, реализующие Serializable**.

Давайте сначала определим класс, реализующий Serializable:

```groovy
class Task implements Serializable {
    String description
    Date startDate
    Date dueDate
    int status
}
```

Теперь давайте создадим экземпляр Task, который мы сможем сериализовать в файл:

```groovy
Task task = new Task(description:'Take out the trash', startDate:new Date(), status:0)
```

Имея в руках наш объект Task, давайте сериализуем его в файл с помощью withObjectOutputStream:

```groovy
new File('src/main/resources/ioSerializedObject.txt').withObjectOutputStream { out ->
    out.writeObject(task)
}
```

Наконец, давайте снова прочитаем нашу задачу Task, используя withObjectInputStream:

```groovy
Task taskRead

new File('src/main/resources/ioSerializedObject.txt').withObjectInputStream { is ->
    taskRead = is.readObject()
}
```

Используемые нами методы withObjectOutputStream и withObjectInputStream передают поток замыканию и обрабатывают закрытие ресурсов соответствующим образом, как и в случае с другими методами with\*.

## 6. Заключение

В этой статье мы рассмотрели функциональные возможности, которые Groovy добавляет к существующим классам файлового ввода-вывода Java. Мы использовали эту функциональность для чтения и записи файлов, работы со структурами каталогов и сериализации данных и объектов.

Мы затронули лишь некоторые вспомогательные методы, поэтому стоит покопаться в [документации Groovy](http://docs.groovy-lang.org/docs/next/html/groovy-jdk/java/io/package-summary.html), чтобы узнать, что еще он добавляет к функциональности ввода-вывода Java.

Пример кода доступен [на GitHub](https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy).
