# Кратко о работе с веб-службами

## 1. Обзор

В настоящее время мы видим множество способов предоставления доступа к данным приложения через Интернет.

Часто приложение использует веб-службу [SOAP](https://www.baeldung.com/spring-boot-soap-web-service) или [REST](https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration) для предоставления доступа к своим API. Однако следует учитывать и такие протоколы потоковой передачи, как RSS и Atom.

В этом кратком руководстве мы рассмотрим несколько удобных способов работы с веб-сервисами в [Groovy](https://www.baeldung.com/groovy-language) для каждого из этих протоколов.

## 2. Выполнение HTTP-запросов

Для начала давайте выполним простой HTTP-запрос GET, используя класс URL. Во время исследования мы будем использовать API-интерфейсы [Postman Echo](https://docs.postman-echo.com/?version=latest).

Сначала мы вызовем метод openConnection класса URL, а затем установим для requestMethod значение GET:

```groovy
def postmanGet = new URL('https://postman-echo.com/get')
def getConnection = postmanGet.openConnection()
getConnection.requestMethod = 'GET'
assert getConnection.responseCode == 200
```

Аналогичным образом мы можем выполнить запрос POST, установив для параметра requestMethod значение POST:

```groovy
def postmanPost = new URL('https://postman-echo.com/post')
def postConnection = postmanPost.openConnection()
postConnection.requestMethod = 'POST'
assert postConnection.responseCode == 200
```

Кроме того, мы можем передать параметры POST-запросу, используя [outputStream.withWriter](https://www.baeldung.com/groovy-io#writing):

```groovy
def form = "param1=This is request parameter."
postConnection.doOutput = true
def text
postConnection.with {
    outputStream.withWriter { outputStreamWriter ->
        outputStreamWriter << form
    }
    text = content.text
}
assert postConnection.responseCode == 200
```

Здесь Groovy с замыканием with выглядит весьма удобно и делает код чище.

Давайте воспользуемся JsonSlurper для анализа ответа String в JSON:

```groovy
JsonSlurper jsonSlurper = new JsonSlurper()
assert jsonSlurper.parseText(text)?.json.param1 == "This is request parameter."
```

## 3. RSS и Atom-каналы

[RSS](https://en.wikipedia.org/wiki/RSS) и канал [Atom](https://en.wikipedia.org/wiki/Atom\_\(Web\_standard\)) – это распространенные способы публикации контента, например новостей, блогов и технических форумов, в Интернете.

Кроме того, оба канала имеют формат XML. Поэтому мы можем использовать класс [XMLParser](https://www.baeldung.com/groovy-xml#xml-parser) Groovy для анализа содержимого.

Давайте прочитаем несколько главных новостей из новостей [Google News](https://news.google.com/), используя их RSS-канал:

```groovy
def rssFeed = new XmlParser()
    .parse("https://news.google.com/rss?hl=en-US&gl=US&ceid=US:en")
def stories = []
(0..4).each {
    def item = rssFeed.channel.item.get(it)
    stories << item.title.text()
}
assert stories.size() == 5
```

Точно так же мы можем читать каналы Atom. Однако из-за различий в спецификациях обоих протоколов мы будем по-разному получать доступ к контенту в каналах Atom:

```groovy
def atomFeed = new XmlParser()
    .parse("https://news.google.com/atom?hl=en-US&gl=US&ceid=US:en")
def stories = []
(0..4).each {
    def entry = atomFeed.entry.get(it)
    stories << entry.title.text()
}
assert stories.size() == 5
```

Кроме того, мы понимаем, что Groovy поддерживает все библиотеки Java, которые поддерживаются в Groovy. Таким образом, мы с уверенностью можем использовать [Rome API](https://www.baeldung.com/rome-rss) для чтения RSS-каналов.

## 4. SOAP-запрос и ответ

SOAP — один из самых популярных протоколов веб-служб, используемый приложениями для предоставления доступа к своим службам через Интернет.

Мы будем использовать библиотеку [groovy-wslite](https://github.com/jwagenleitner/groovy-wslite) для использования API-интерфейсов SOAP. Давайте добавим его последнюю [зависимость](https://mvnrepository.com/artifact/com.github.groovy-wslite/groovy-wslite) в наш pom.xml:

```xml
<dependency>
    <groupId>com.github.groovy-wslite</groupId>
    <artifactId>groovy-wslite</artifactId>
    <version>1.1.3</version>
</dependency>
```

Альтернативно мы можем добавить последнюю зависимость с помощью Gradle:

```gradle
compile group: 'com.github.groovy-wslite', name: 'groovy-wslite', version: '1.1.3'
```

Или если мы хотим написать сценарий Groovy. Мы можем добавить его напрямую, используя @Grab:

```groovy
@Grab(group='com.github.groovy-wslite', module='groovy-wslite', version='1.1.3')
```

**Библиотека groovy-wslite предоставляет класс SOAPClient для взаимодействия с API-интерфейсами SOAP**. В то же время у него есть класс SOAPMessageBuilder для создания сообщения запроса.

Давайте воспользуемся [службой SOAP преобразования чисел](http://www.dataaccess.com/webservicesserver/numberconversion.wso) с помощью SOAPClient:

```groovy
def url = "http://www.dataaccess.com/webservicesserver/numberconversion.wso"
def soapClient = new SOAPClient(url)
def message = new SOAPMessageBuilder().build({
    body {
        NumberToWords(xmlns: "http://www.dataaccess.com/webservicesserver/") {
            ubiNum(123)
        }
    }
})
def response = soapClient.send(message.toString());
def words = response.NumberToWordsResponse
assert words == "one hundred and twenty three "
```

## 5. REST-запрос и ответ

REST — еще один популярный архитектурный стиль, используемый для создания веб-сервисов. Кроме того, API предоставляются на основе методов HTTP, таких как GET, POST, PUT и DELETE.

### 5.2. GET

Мы будем использовать уже обсуждавшуюся библиотеку groovy-wslite для использования REST API. **Она предоставляет класс RESTClient для беспроблемного общения**.

Давайте сделаем GET-запрос к уже обсуждавшемуся API Postman:

```groovy
RESTClient client = new RESTClient("https://postman-echo.com")
def path = "/get"
def response
try {
    response = client.get(path: path)
    assert response.statusCode = 200
    assert response.json?.headers?.host == "postman-echo.com"
} catch (RESTClientException e) {
    assert e?.response?.statusCode != 200
}
```

### 5.2. POST

Теперь давайте отправим POST-запрос к API Postman. При этом параметры формы передадим в формате JSON:

```groovy
client.defaultAcceptHeader = ContentType.JSON
def path = "/post"
def params = ["foo":1,"bar":2]
def response = client.post(path: path) {
    type ContentType.JSON
    json params
}
assert response.json?.data == params
```

Здесь мы установили JSON в качестве заголовка принятия по умолчанию.

## 6. Аутентификация для веб-сервиса

С ростом количества веб-сервисов и приложений, взаимодействующих друг с другом, рекомендуется иметь безопасный веб-сервис.

В результате **важно сочетание HTTPS и механизма аутентификации, такого как Basic Auth и OAuth**.

Таким образом, приложение должно аутентифицировать себя при использовании API веб-службы.

### 6.1. Базовая аутентификация Basic Auth

Мы можем использовать уже обсуждавшийся класс RESTClient. Давайте воспользуемся классом HTTPBasicAuthorization с учетными данными для выполнения базовой аутентификации:

```groovy
def path = "/basic-auth"
client.authorization = new HTTPBasicAuthorization("postman", "password")
response = client.get(path: path)
assert response.statusCode == 200
assert response.json?.authenticated == true
```

В качестве альтернативы мы можем напрямую передать учетные данные (в кодировке Base64) в параметре заголовков:

```groovy
def response = client
.get(path: path, headers: ["Authorization": "Basic cG9zdG1hbjpwYXNzd29yZA=="])
```

### 6.2. OAuth 1.0

Аналогичным образом мы можем сделать запрос OAuth 1.0, передав параметры аутентификации, такие как ключ потребителя и секрет потребителя.

Однако, поскольку у нас нет встроенной поддержки OAuth 1.0, как для других механизмов, нам придется выполнить всю работу самостоятельно:

```groovy
def path = "/oauth1"
def params = [oauth_consumer_key: "RKCGzna7bv9YD57c", 
    oauth_signature_method: "HMAC-SHA1", 
    oauth_timestamp:1567089944, oauth_nonce: "URT7v4", oauth_version: 1.0, 
    oauth_signature: 'RGgR/ktDmclkM0ISWaFzebtlO0A=']
def response = new RESTClient("https://postman-echo.com")
    .get(path: path, query: params)
assert response.statusCode == 200
assert response.statusMessage == "OK"
assert response.json.status == "pass"
```

## 7. Заключение

В этом уроке мы рассмотрели несколько удобных способов **работы с веб-сервисами в Groovy**.

В то же время мы увидели простой способ чтения каналов RSS или Atom.

Как обычно, реализации кода [доступны на GitHub](https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2).
