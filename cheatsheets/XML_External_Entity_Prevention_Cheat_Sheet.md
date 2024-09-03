# Шпаргалка по предотвращению появления внешних сущностей в формате XML

## Вступление

Внедрение внешней сущности *XML* (XXE), которое теперь является частью [OWASP Top 10](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A4-XML_External_Entities_%28XXE%29)  через точку **A4**, представляет собой атаку на приложения, которые анализируют вводимые данные в формате XML. Ссылка на эту проблему содержится в идентификаторе [611](https://cwe.mitre.org/data/definitions/611.html) в разделе [Common Weakness Enumeration](https://cwe.mitre.org/index.html). Атака XXE происходит, когда ненадежный ввод XML со **ссылкой на внешнюю сущность обрабатывается слабо сконфигурированным анализатором XML**, и эта атака может быть использована для организации нескольких инцидентов, включая:

- Атака типа "Отказ в обслуживании" на систему
- Атака [Подделка запроса на стороне сервера](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery) (SSRF)
- Возможность сканирования портов с компьютера, на котором расположен анализатор
- Другие системные проблемы.

Эта шпаргалка поможет вам предотвратить эту уязвимость.

Для получения дополнительной информации о XXE, пожалуйста, посетите[XML External Entity (XXE)](https://en.wikipedia.org/wiki/XML_external_entity_attack).

## Общие рекомендации

**Самый безопасный способ предотвратить XXE - это всегда полностью отключить DTD (внешние сущности).** В зависимости от синтаксического анализатора метод должен быть похож на следующий:

``` java
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
```

Отключение [DTD](https://www.w3schools.com/xml/xml_dtd.asp)s также обеспечивает защиту синтаксического анализатора от атак типа "отказ в обслуживании" (DOS), таких как [Миллиард смеха](https://en.wikipedia.org/wiki/Billion_laughs_attack). **Если невозможно полностью отключить DTDS, то внешние объекты и объявления типов внешних документов должны быть отключены способом, специфичным для каждого синтаксического анализатора.**

**Ниже приведены подробные инструкции по предотвращению XXE для нескольких языков (C++, ColdFusion, Java, .NET, iOS, PHP, Python, правила Semgrep) и их широко используемых синтаксических анализаторов XML.**

## C/C++

### libxml2

В перечислении [xmlParserOption](http://xmlsoft.org/html/libxml-parser.html#xmlParserOption) не должны быть определены следующие параметры:

- `XML_PARSE_NOENT`: Разворачивает объекты и заменяет их заменяющим текстом
- `XML_PARSE_DTDLOAD`: Загрузите внешний DTD

Примечание:

Согласно [этому посту] (https://mail.gnome.org/archives/xml/2012-October/msg00045.html), начиная с libxml2 версии 2.9, XXE по умолчанию отключен, что зафиксировано следующим [исправлением](https://gitlab.gnome.org/GNOME/libxml2/commit/4629ee02ac649c27f9c0cf98ba017c6b5526070f).

Проверьте, используются ли следующие API-интерфейсы, и убедитесь, что в параметрах не определены `XML_PARSER_NO END` и `XML_PARSE_DTDLOAD`:

- `xmlCtxtReadDoc`
- `xmlCtxtReadFd`
- `xmlCtxtReadFile`
- `xmlCtxtReadIO`
- `xmlCtxtReadMemory`
- `xmlCtxtUseOptions`
- `xmlParseInNodeContext`
- `xmlReadDoc`
- `xmlReadFd`
- `xmlReadFile`
- `xmlReadIO`
- `xmlReadMemory`

### libxerces-c

Используйте `XercesDOMParser`, чтобы предотвратить XXE:

``` cpp
XercesDOMParser *parser = new XercesDOMParser;
parser->setCreateEntityReferenceNodes(true);
parser->setDisableDefaultEntityResolution(true);
```

Используйте SAXParser, сделайте это, чтобы предотвратить XXE:

``` cpp
SAXParser* parser = new SAXParser;
parser->setDisableDefaultEntityResolution(true);
```

Используйте SAX2XMLReader, сделайте это, чтобы предотвратить XXE:

``` cpp
SAX2XMLReader* reader = XMLReaderFactory::createXMLReader();
parser->setFeature(XMLUni::fgXercesDisableDefaultEntityResolution, true);
```

## ColdFusion

Согласно [этой записи в блоге] (https://hoyahaxa.blogspot.com/2022/11/on-coldfusion-xxe-and-other-xml-attacks.html), как в Adobe ColdFusion, так и в Lucee есть встроенные механизмы отключения поддержки внешних XML-объектов.

### Adobe ColdFusion

Начиная с ColdFusion 2018 Update 14 и ColdFusion 2021 Update 4, все встроенные функции ColdFusion, которые обрабатывают XML, имеют аргумент синтаксического анализа XML, который отключает поддержку внешних объектов XML. Поскольку не существует глобального параметра, отключающего внешние объекты, разработчики должны убедиться, что при каждом вызове функции XML используются правильные параметры безопасности.

Из [документации к XmlParse() function](https://helpx.adobe.com/coldfusion/cfml-reference/coldfusion-functions/functions-t-z/xmlparse.html) вы можете отключить XXE с помощью приведенного ниже кода:

```
<cfset parseroptions = structnew()>
<cfset parseroptions.ALLOWEXTERNALENTITIES = false>
<cfscript>
a = XmlParse("xml.xml", false, parseroptions);
writeDump(a);
</cfscript>
```

Вы можете использовать структуру "параметры синтаксического анализа", показанную выше, в качестве аргумента для защиты других функций, которые также обрабатывают XML, таких как:

```
XxmlSearch(xmldoc, xpath,parseroptions);

XmlTransform(xmldoc,xslt,parseroptions);

isXML(xmldoc,parseroptions);
```

### Lucee

Начиная с версии Lycee 5.3.4.51 и более поздних версий, вы можете отключить поддержку внешних объектов XML, добавив в свое приложение следующее.cfc:

```
this.xmlFeatures = {
     externalGeneralEntities: false,
     secure: true,
     disallowDoctypeDecl: true
};
```

Начиная с Lucene 5.4.2.10 и Lucee 6.0.0.514, поддержка внешних XML-объектов по умолчанию отключена.

## Java

**Поскольку в большинстве синтаксических анализаторов Java XML по умолчанию включен XXE, этот язык особенно уязвим для атак XXE, поэтому для безопасного использования этих синтаксических анализаторов необходимо явно отключить XXE.** В этом разделе описано, как отключить XXE в наиболее часто используемых синтаксических анализаторах Java XML.

### JAXP DocumentBuilderFactory, SAXParserFactory и DOM4J

Синтаксические анализаторы XML` DocumentBuilderFactory`, `SAXParserFactory` и `DOM4J` могут быть защищены от атак XXE с помощью тех же методов.

**Для краткости мы только покажем вам, как защитить синтаксический анализатор `DocumentBuilderFactory`. Дополнительные инструкции по защите этого синтаксического анализатора включены в пример кода**

 Метод JAXP `DocumentBuilderFactory` [setFeature](https://docs.oracle.com/javase/7/docs/api/javax/xml/parsers/DocumentBuilderFactory.html#setFeature(java.lang.String,%20boolean)) позволяет разработчику контролировать, какие функции XML-процессора, зависящие от конкретной реализации, включены или отключены.

Эти функции могут быть установлены либо на заводе-изготовителе, либо в базовом методе `XmlReader` [setFeature](https://docs.oracle.com/javase/7/docs/api/org/xml/sax/XMLReader.html#setFeature%28java.lang.String,%20boolean%29).

**Каждая реализация XML-процессора имеет свои собственные функции, которые определяют, как обрабатываются DTD и внешние объекты. Полностью отключив обработку DTD, можно предотвратить большинство атак XXE, хотя также необходимо отключить или убедиться, что функция XInclude не включена.**

**Начиная с JDK 6, флаг [FEATURE_SECURE_PROCESSING](https://docs.oracle.com/javase/6/docs/api/javax/xml/XMLConstants.html#FEATURE_SECURE_PROCESSING) может использоваться для указания реализации синтаксического анализатора для безопасной обработки XML**. Его поведение зависит от реализации. Это может помочь при исчерпании ресурсов, но не всегда может уменьшить расширение объекта. Более подробную информацию об этом флаге можно найти здесь [here](https://docs.oracle.com/en/java/javase/13/security/java-api-xml-processing-jaxp-security-guide.html#GUID-88B04BE2-35EF-4F61-B4FA-57A0E9102342).

Пример фрагмента кода с выделенным синтаксисом, использующего `SAXParserFactory`, смотрите [здесь](https://gist.github.com/asudhakar02/45e2e6fd8bcdfb4bc3b2).
Пример кода, полностью отключающего Dtd (doctypes).:

``` java
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException; // отслеживание неподдерживаемых функций
import javax.xml.XMLConstants;

...

DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
String FEATURE = null;
try {
    // Это ОСНОВНАЯ защита. Если DTD (doctype) запрещены,
    // почти все атаки на объекты XML предотвращаются.
    // Только Xerces 2 - http://xerces.apache.org/xerces2-j/features.html#disallow-doctype-decl
    FEATURE = "http://apache.org/xml/features/disallow-doctype-decl";
    dbf.setFeature(FEATURE, true);

    // и это тоже, согласно статье Тимоти Моргана 2014 года "XML Schema, DTD, and Entity Attacks".
    dbf.setXIncludeAware(false);

    // оставшаяся логика синтаксического анализатора
    ...
} catch (ParserConfigurationException e) {
    // Это должно перехватить сбой функции setFeature
    // ПРИМЕЧАНИЕ: Каждый вызов функции set() должен быть выполнен в отдельном режиме try/catch, иначе последующие вызовы будут пропущены.
    // Это важно только в том случае, если вы игнорируете ошибки поддержки нескольких провайдеров.
    logger.info("ParserConfigurationException was thrown. The feature '" + FEATURE
    + "' is not supported by your XML processor.");
    ...
} catch (SAXException e) {
    // В Apache это должно быть сброшено при отключении DOCTYPE
    logger.warning("A DOCTYPE was passed into the XML document");
    ...
} catch (IOException e) {
    // XXE, указывающий на несуществующий файл
    logger.error("IOException occurred, XXE may still possible: " + e.getMessage());
    ...
}

// Загрузите XML-файл или поток, используя независимый от XXE синтаксический анализатор...
DocumentBuilder safebuilder = dbf.newDocumentBuilder();
```

Если вы не можете полностью отключить DTDs:

``` java
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException; // отслеживание неподдерживаемых функций
import javax.xml.XMLConstants;

...

DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();

String[] featuresToDisable = {
    // Xerces 1 - http://xerces.apache.org/xerces-j/features.html#external-general-entities
    // Xerces 2 - http://xerces.apache.org/xerces2-j/features.html#external-general-entities
    // JDK7+ - http://xml.org/sax/features/external-general-entities
    //Эту функцию необходимо использовать вместе со следующей, иначе она точно не защитит вас от XXE
    "http://xml.org/sax/features/external-general-entities",

    // Xerces 1 - http://xerces.apache.org/xerces-j/features.html#external-parameter-entities
    // Xerces 2 - http://xerces.apache.org/xerces2-j/features.html#external-parameter-entities
    // JDK7+ - http://xml.org/sax/features/external-parameter-entities
    //Эту функцию необходимо использовать вместе с предыдущей, иначе она точно не защитит вас от XXE
    "http://xml.org/sax/features/external-parameter-entities",

    // Также отключите внешние DTD
    "http://apache.org/xml/features/nonvalidating/load-external-dtd"
}

for (String feature : featuresToDisable) {
    try {    
        dbf.setFeature(FEATURE, false); 
    } catch (ParserConfigurationException e) {
        // Это должно привести к сбою функции setFeature
        logger.info("ParserConfigurationException was thrown. The feature '" + feature
        + "' is probably not supported by your XML processor.");
        ...
    }
}

try {
    // Добавьте их в соответствии с документом Тимоти Моргана 2014 года: ""XML Schema, DTD, and Entity Attacks".
    dbf.setXIncludeAware(false);
    dbf.setExpandEntityReferences(false);
        
    // Как указано в документации, "Функция безопасной обработки (FSP)" является центральным механизмом, который
    // поможет вам защитить обработку XML. Он инструктирует обработчики XML, такие как анализаторы, валидаторы,
    // и преобразователи, пытаться безопасно обрабатывать XML, и FSP может использоваться как альтернатива
    // dbf.setExpandEntityReferences(false); для обеспечения некоторого безопасного уровня расширения сущности
    // Существует начиная с JDK6.
    dbf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);

    // И, согласно Тимоти Моргану: "Если по какой-либо причине поддержка встроенных типов документов является обязательным требованием, то
    // убедитесь, что настройки сущности отключены (как показано выше), и будьте осторожны с SSRF-атаками
    // (http://cwe.mitre.org/data/definitions/918.html) и отказом
    // часть сервисных атак (таких как billion laughs или декомпрессионные бомбы через "jar:") представляет собой риск."

    // оставшаяся логика синтаксического анализатора
    ...
} catch (ParserConfigurationException e) {
    // Это должно привести к сбою функции setFeature
    logger.info("ParserConfigurationException was thrown. The feature 'XMLConstants.FEATURE_SECURE_PROCESSING'"
    + " is probably not supported by your XML processor.");
    ...
} catch (SAXException e) {
    // В Apache это должно быть сброшено при отключении DOCTYPE
    logger.warning("A DOCTYPE was passed into the XML document");
    ...
} catch (IOException e) {
    // XXE, указывающий на несуществующий файл
    logger.error("IOException occurred, XXE may still possible: " + e.getMessage());
    ...
}

// Загрузите XML-файл или поток, используя независимый от XXE синтаксический анализатор...
DocumentBuilder safebuilder = dbf.newDocumentBuilder();
```

[Xerces 1](https://xerces.apache.org/xerces-j/) [Особенности](https://xerces.apache.org/xerces-j/features.html):

- Не включайте внешние объекты, установив для [этой функции](https://xerces.apache.org/xerces-j/features.html#external-general-entities) значение `false`.
- Не включайте объекты параметров, установив для [этой функции](https://xerces.apache.org/xerces-j/features.html#external-parameter-entities) значение `false`.
- Не включайте внешние DTD, установив для [этой функции](https://xerces.apache.org/xerces-j/features.html#load-external-dtd) значение `false`.

[Xerces 2](https://xerces.apache.org/xerces2-j/) [Особенности](https://xerces.apache.org/xerces2-j/features.html):

- Отключите встроенный DTD, установив для [этой функции](https://xerces.apache.org/xerces2-j/features.html#disallow-doctype-decl) значение `true`.
- Не включайте внешние объекты, установив для [этой функции](https://xerces.apache.org/xerces2-j/features.html#external-general-entities) значение `false`.
- Не включайте объекты параметров, установив для [этой функции](https://xerces.apache.org/xerces2-j/features.html#external-parameter-entities) значение `false`..
- Не включайте внешние DTD, установив для [этой функции](https://xerces.apache.org/xerces-j/features.html#load-external-dtd) значение `false`..

** Примечание:** Для вышеуказанных средств защиты требуется обновление Java 7 до версии 67, Java 8 до версии 20 или выше, поскольку средства защиты для `DocumentBuilderFactory` и SAXParserFactory не работают в более ранних версиях Java, согласно: [CVE-2014-6517](http://www.cvedetails.com/cve/CVE-2014-6517 /).

### XMLInputFactory (синтаксический анализатор StAX)

[StAX](http://en.wikipedia.org/wiki/StAX) синтаксические анализаторы, такие как [`XMLInputFactory`](http://docs.oracle.com/javase/7/docs/api/javax/xml/stream/XMLInputFactory.html), позволяют устанавливать различные свойства и возможности.

Чтобы защитить Java `XMLInputFactory` от XXE, полностью отключите DTDs (doctype):

``` java
// Это полностью отключает DTDS для этой factory
xmlInputFactory.setProperty(XMLInputFactory.SUPPORT_DTD, false);
```

or if you can't completely disable DTDs:

``` java
// Это приводит к возникновению исключения XMLStreamException при обращении к внешним DTD.
xmlInputFactory.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
// отключить внешние объекты
xmlInputFactory.setProperty("javax.xml.stream.isSupportingExternalEntities", false);
```

Параметр `XMLInputFactory.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");` не требуется, поскольку XMLInputFactory зависит от средства проверки правильности XML в соответствии со схемами. Проверьте конкретную конфигурацию в разделе [Validator](#Валидатор).

### Анализатор Oracle DOM

Следуйте рекомендациям [Oracle recommendation](https://docs.oracle.com/en/database/oracle/oracle-database/18/adxdk/security-considerations-oracle-xml-developers-kit.html#GUID-45303542-41DE-4455-93B3-854A826EF8BB), например:

``` java
    // Расширьте oracle.xml.parser.v2.XMLParser
    DOMParser domParser = new DOMParser();

    // Не расширяйте ссылки на объекты
    domParser.setAttribute(DOMParser.EXPAND_ENTITYREF, false);

    // объект dtd - это экземпляр oracle.xml.parser.v2.DTD
    domParser.setAttribute(DOMParser.DTD_OBJECT, dtdObj);

    // Не допускайте расширения объекта более чем на 11 уровней
    domParser.setAttribute(DOMParser.ENTITY_EXPANSION_DEPTH, 12);
```

### TransformerFactory

Чтобы защитить файл javax.xml.transform.TransformerFactory от XXE, выполните следующее:

``` java
TransformerFactory tf = TransformerFactory.newInstance();
tf.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
tf.setAttribute(XMLConstants.ACCESS_EXTERNAL_STYLESHEET, "");
```

### Validator

Чтобы защитить `javax.xml.validation.Средство проверки подлинности` от XXE, выполните следующее:

``` java
SchemaFactory factory = SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema");
factory.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
factory.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");
Schema schema = factory.newSchema();
Validator validator = schema.newValidator();
validator.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
validator.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");
```

### SchemaFactory

Чтобы защитить файл javax.xml.validation.SchemaFactory от XXE, выполните следующее:

``` java
SchemaFactory factory = SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema");
factory.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
factory.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");
Schema schema = factory.newSchema(Source);
```

### SAXTransformerFactory

Чтобы защитить javax.xml.transform.sax.SAXTransformerFactory от XXE, выполните следующее:

``` java
SAXTransformerFactory sf = SAXTransformerFactory.newInstance();
sf.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
sf.setAttribute(XMLConstants.ACCESS_EXTERNAL_STYLESHEET, "");
sf.newXMLFilter(Source);
```

**Примечание: Для использования следующих `XMLConstants` требуется JAXP 1.5, который был добавлен в Java в версиях 7u40 и Java 8:**

- `javax.xml.XMLConstants.ACCESS_EXTERNAL_DTD`
- `javax.xml.XMLConstants.ACCESS_EXTERNAL_SCHEMA`
- `javax.xml.XMLConstants.ACCESS_EXTERNAL_STYLESHEET`

### XMLReader

Чтобы защитить Java `org.xml.sax.XmlReader` от атаки XXE, выполните следующее:

``` java
XMLReader reader = XMLReaderFactory.createXMLReader();
reader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
// Это может и не быть строго обязательным, поскольку, согласно предыдущей строке, DTD вообще не должны быть разрешены.
reader.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
reader.setFeature("http://xml.org/sax/features/external-general-entities", false);
reader.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
```

### SAXReader

Чтобы защитить Java `org.dom4j.io.SAXReader` от атаки XXE, выполните следующее:

``` java
saxReader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
saxReader.setFeature("http://xml.org/sax/features/external-general-entities", false);
saxReader.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
```

Если в вашем коде отсутствуют все эти строки, вы можете быть уязвимы для атаки XXE.

### SAXBuilder

Чтобы защитить Java `org.jdom2.input.SAXBuilder` от атаки XXE, полностью отключите Dtd (doctypes):

``` java
SAXBuilder builder = new SAXBuilder();
builder.setFeature("http://apache.org/xml/features/disallow-doctype-decl",true);
Document doc = builder.build(new File(fileName));
```

В качестве альтернативы, если DTDS не могут быть полностью отключены, отключите внешние объекты и расширение объектов:

``` java
SAXBuilder builder = new SAXBuilder();
builder.setFeature("http://xml.org/sax/features/external-general-entities", false);
builder.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
builder.setExpandEntities(false);
Document doc = builder.build(new File(fileName));
```

### No-op EntityResolver

Для API-интерфейсов, которые используют `EntityResolver`, вы можете нейтрализовать способность синтаксического анализатора XML разрешать сущности, [указав отсутствие операции implementation](https://wiki.sei.cmu.edu/confluence/display/java/IDS17-J.+Prevent+XML+External+Entity+Attacks):

```java
public final class NoOpEntityResolver implements EntityResolver {
    public InputSource resolveEntity(String publicId, String systemId) {
        return new InputSource(new StringReader(""));
    }
}

// ...

xmlReader.setEntityResolver(new NoOpEntityResolver());
documentBuilder.setEntityResolver(new NoOpEntityResolver());
```

или еще проще:

```java
EntityResolver noop = (publicId, systemId) -> new InputSource(new StringReader(""));
xmlReader.setEntityResolver(noop);
documentBuilder.setEntityResolver(noop);
```

### JAXB Unmarshaller

**Так как `javax.xml.bind.Unmarshaller` анализирует XML, но не поддерживает никаких флагов для отключения XXE, вы должны сначала проанализировать ненадежный XML с помощью настраиваемого безопасного синтаксического анализатора, сгенерировать исходный объект в результате и передать исходный объект в Unmarshaller.** Например:

``` java
SAXParserFactory spf = SAXParserFactory.newInstance();

//Вариант 1: Это ОСНОВНАЯ защита от XXE
spf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
spf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
spf.setXIncludeAware(false);

//Вариант 2: Если отключить doctype невозможно
spf.setFeature("http://xml.org/sax/features/external-general-entities", false);
spf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
spf.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
spf.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
spf.setXIncludeAware(false);

//Выполните операцию unmarshall 
Source xmlSource = new SAXSource(spf.newSAXParser().getXMLReader(),
                                new InputSource(new StringReader(xml)));
JAXBContext jc = JAXBContext.newInstance(Object.class);
Unmarshaller um = jc.createUnmarshaller();
um.unmarshal(xmlSource);
```

### XPathExpression

**Поскольку "javax.xml.xpath.XPathExpression" не может быть безопасно сконфигурировано само по себе, ненадежные данные должны быть сначала проанализированы с помощью другого защищенного XML-анализатора.**

Например:

``` java
DocumentBuilderFactory df = DocumentBuilderFactory.newInstance();
df.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
df.setAttribute(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");
DocumentBuilder builder = df.newDocumentBuilder();
String result = new XPathExpression().evaluate( builder.parse(
                            new ByteArrayInputStream(xml.getBytes())) );
```

### java.beans.XMLDecoder

**Метод [readObject()](https://docs.oracle.com/javase/8/docs/api/java/beans/XMLDecoder.html#readObject--) в этом классе принципиально небезопасен.**

**Анализируемый им XML не только подчиняется требованиям XXE, но и может быть использован для создания любого объекта Java и [выполнения произвольного кода, как описано here](http://stackoverflow.com/questions/14307442/is-it-safe-to-use-xmldecoder-to-read-document-files).**

**И нет никакого способа сделать использование этого класса безопасным, кроме как доверять или должным образом проверять передаваемые в него входные данные.**

**Таким образом, мы настоятельно рекомендуем полностью отказаться от использования этого класса и заменить его безопасным или правильно настроенным синтаксическим анализатором XML, как описано в других разделах этой шпаргалки.**

### Другие синтаксические анализаторы XML

**Существует множество сторонних библиотек, которые анализируют XML либо напрямую, либо с помощью других библиотек. Пожалуйста, протестируйте и убедитесь, что их XML-анализатор по умолчанию защищен от XXE.** Если синтаксический анализатор по умолчанию небезопасен, найдите флаги, поддерживаемые синтаксическим анализатором, чтобы отключить все возможные включения внешних ресурсов, как в примерах, приведенных выше. Если нет доступа к внешнему управлению, убедитесь, что ненадежный контент сначала проходит через безопасный анализатор, а затем передается небезопасному стороннему анализатору, аналогично тому, как защищен Unmarshaller.

#### Уязвимости Spring Framework MVC/OXM XXE

**Некоторые уязвимости XXE были обнаружены в [Spring OXM](https://pivotal.io/security/cve-2013-4152) и [Spring MVC](https://pivotal.io/security/cve-2013-7315) . Следующие версии платформы Spring Framework уязвимы для XXE.:

- **3.0.0** до **3.2.3** (Spring OXM & Spring MVC)
- **4.0.0.M1** (Spring OXM)
- **4.0.0.M1-4.0.0.M2** (Spring MVC)

Были и другие проблемы, которые были исправлены позже, поэтому для полного устранения этих проблем Spring рекомендует вам перейти на Spring Framework версии 3.2.8+ или 4.0.2+.

Для Spring OXM это относится к использованию org.springframework.oxm.jaxb.Jaxb2Marshaller. **Обратите внимание, что в CVE для Spring OXM конкретно указано, что разработчик должен разобраться с двумя ситуациями синтаксического анализа XML, а за две другие отвечает Spring, и они были исправлены для устранения этого CVE.**

Вот что они говорят:

Разработчики должны справиться с двумя ситуациями:

- Для `DOMSource` XML уже был проанализирован пользовательским кодом, и этот код отвечает за защиту от XXE.
- Для `StAXSource` XMLStreamReader уже был создан пользовательским кодом, и этот код отвечает за защиту от XXE.

Проблема была устранена весной:

Для экземпляров SAXSource и StreamSource Spring по умолчанию обрабатывал внешние объекты, тем самым создавая эту уязвимость.

Вот пример использования StreamSource, который был уязвим, но теперь безопасен, если вы используете исправленную версию Spring OXM или Spring MVC:

``` java
import org.springframework.oxm.Jaxb2Marshaller;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;

Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
// Необходимо привести возвращаемый объект к любому типу, который вы отменяете
marshaller.unmarshal(new StreamSource(new StringReader(some_string_containing_XML));
```

Итак, для записи [Spring OXM CVE](https://pivotal.io/security/cve-2013-4152) вышеуказанное теперь безопасно. Но если вместо этого вы будете использовать DOMSource или StAXSource, вам придется настроить эти источники так, чтобы они были защищены от XXE.

#### Castor

**Castor - это фреймворк для привязки данных для Java. Он позволяет выполнять преобразование между объектами Java, XML и реляционными таблицами. Функции XML в Castor до версии 1.3.3 уязвимы для XXE и должны быть обновлены до последней версии.** Для получения дополнительной информации ознакомьтесь с официальной конфигурацией [XML file](https://castor-data-binding.github.io/castor/reference-guide/reference/xml/xml-properties.html)

## .NET

**Актуальная информация для внедрения XXE в .NET взята непосредственно из [веб-приложения модульных тестов Дина Флеминга].(https://github.com/deanf1/dotnet-security-unit-tests ), который охватывает все поддерживаемые в настоящее время синтаксические анализаторы .NET XML и содержит тестовые примеры, которые демонстрируют, когда они безопасны от внедрения XXE, а когда нет, но эти тесты проводятся только с внедрением из файла, а не с прямым DTD (используется при DoS-атаках).**

Для DoS-атак с использованием прямого DTD (таких как [атака Billion laughs](https://en.wikipedia.org/wiki/Billion_laughs_attack)) было создано [отдельное тестовое приложение от Джоша Гроссмана из Bounce Security](https://github.com/BounceSecurity/BillionLaughsTester), чтобы убедиться, что .NET >=4.5.2 защищен от этих атак.

Ранее эта информация была основана на некоторых старых статьях, которые могут быть неточными на 100%, включая:

- [Превосходная статья Джеймса Джардина о .NET XXE](https://www.jardinesoftware.net/2016/05/26/xxe-and-net/).
- [Руководство от Microsoft о том, как предотвратить отказ в обслуживании в формате XXE и XML в .NET](http://msdn.microsoft.com/en-us/magazine/ee335713.aspx).

### Обзор уровней безопасности синтаксического анализатора .NET

**Ниже приведен обзор всех поддерживаемых синтаксических анализаторов .NET XML и их уровней безопасности по умолчанию. Более подробная информация о каждом синтаксическом анализаторе приведена после этого списка.

**XDocument (преобразование в XML)

Этот синтаксический анализатор защищен от внешних объектов в .NET Framework версии 4.5.2 и защищен от Billion Laughs в версии 4.5.2 или выше, но неясно, защищен ли этот синтаксический анализатор от Billion Laughs до версии 4.5.2.

#### Уровни безопасности XmlDocument, XmlTextReader, XPathNavigator по умолчанию

Эти анализаторы уязвимы для атак внешних сущностей и Billion Laughs в версиях ниже версии 4.5.2, но защищены в версиях, равных или превосходящих 4.5.2.

#### Уровни безопасности XmlDictionaryReader, XmlNodeReader, XmlReader по умолчанию

Эти синтаксические анализаторы не уязвимы для атак внешних сущностей или Billion Laughs до или после версии 4.5.2. Кроме того, при версиях ≥4.5.2 или выше эти библиотеки даже не будут обрабатывать встроенный DTD по умолчанию. Даже если вы измените значение по умолчанию, чтобы разрешить обработку DTD, при попытке DoS все равно будет выдано исключение, как описано выше.

### ASP.NET

ASP.NET приложения ≥ .NET 4.5.2 также должны обеспечивать установку параметра `<HttpRuntime targetFramework="..." />` в их `Web.config` на значение ≥4.5.2, иначе они могут быть уязвимы независимо от фактической версии .NET версии. Отсутствие этого тега также приведет к небезопасному поведению по умолчанию.

В целях понимания приведенной выше таблицы, `Версия .NET Framework` для ASP.NET приложений - это либо .NET версия, с помощью которой было создано приложение, или `targetFramework` HttpRuntime (Web.config), **в зависимости от того, что меньше**.

Этот тег конфигурации не следует путать с аналогичным тегом конфигурации: `<compilation targetFramework="..." />` или сборками / проектами targetFramework, которых недостаточно для обеспечения безопасного поведения по умолчанию, как указано в приведенной выше таблице.

### LINQ к XML

**Оба объекта `XElement` и `XDocument` в библиотеке `System.Xml.Linq` по умолчанию защищены от внедрения XXE из внешнего файла и DoS-атак.** `XElement` анализирует только элементы внутри XML-файла, поэтому DTD полностью игнорируются. В `XDocument` есть XmlResolver [по умолчанию отключен](https://docs.microsoft.com/en-us/dotnet/standard/linq/linq-xml-security), поэтому он защищен от SSRF. Хотя DTDS [включены с помощью default](https://referencesource.microsoft.com/#System.Xml.Linq/System/Xml/Linq/XLinq.cs,71f4626a3d6f9bad), начиная с версий платформы ≥4.5.2, он ** не** уязвим для DoS, как отмечалось, но может быть уязвим в более ранних версиях платформы. Дополнительные сведения см. в [Руководстве корпорации Майкрософт по предотвращению отказа в обслуживании в формате XXE и XML в .NET](http://msdn.microsoft.com/en-us/magazine/ee335713.aspx)

### XmlDictionaryReader

**`System.Xml.XmlDictionaryReader` по умолчанию безопасен, так как при попытке проанализировать DTD компилятор выдает исключение, сообщающее, что "элементы CData недопустимы на верхнем уровне XML-документа". Он становится небезопасным, если создан с использованием другого небезопасного синтаксического анализатора XML.**

### XmlDocument

**До версии .NET Framework 4.5.2 "System.Xml.XmlDocument` по умолчанию был небезопасен. Объект `XmlDocument` содержит объект `XmlResolver`, для которого должно быть установлено значение null в версиях, предшествующих 4.5.2. В версиях 4.5.2 и выше для этого `XmlResolver` по умолчанию установлено значение null.**

В следующем примере показано, как это делается безопасным:

``` csharp
 static void LoadXML()
 {
   string xxePayload = "<!DOCTYPE doc [<!ENTITY win SYSTEM 'file:///C:/Users/testdata2.txt'>]>"
                     + "<doc>&win;</doc>";
   string xml = "<?xml version='1.0' ?>" + xxePayload;

   XmlDocument xmlDoc = new XmlDocument();
   // Установка этого значения в NULL отключает DTDs - по умолчанию оно НЕ равно null.
   xmlDoc.XmlResolver = null;
   xmlDoc.LoadXml(xml);
   Console.WriteLine(xmlDoc.InnerText);
   Console.ReadLine();
 }
```

**Для .NET Framework версии ≥4.5.2 это безопасно по умолчанию**.

`XmlDocument` может стать небезопасным, если вы создадите свой собственный ненулевой `XmlResolver` с настройками по умолчанию или небезопасными настройками. Если вам необходимо включить обработку DTD, инструкции о том, как это сделать безопасно, подробно описаны в [ссылка на статью MSDN](https://msdn.microsoft.com/en-us/magazine/ee335713.aspx).

### XmlNodeReader

Объекты `System.Xml.XmlNodeReader` по умолчанию безопасны и будут игнорировать DTD, даже если они созданы с помощью небезопасного синтаксического анализатора или заключены в другой небезопасный синтаксический анализатор.

### XmlReader

Объекты `System.Xml.XmlReader` по умолчанию безопасны.

По умолчанию для их свойства ProhibitDtd установлено значение false в .NET Framework версий 4.0 и более ранних, или для их свойства `DtdProcessing` установлено значение "Запретить" в .NET версий 4.0 и более поздних.

Кроме того, в версиях .NET 4.5.2 и более поздних версиях для параметра `XmlReaderSettings`, принадлежащего `XmlReader`, по умолчанию установлено значение `XmlResolver`, равное нулю, что обеспечивает дополнительный уровень безопасности.

Следовательно, объекты `XmlReader` станут небезопасными только в версии 4.5.2 и выше, если для свойства `DtdProcessing` задано значение Parse, а для параметра `XmlResolver` `XmlReaderSetting' установлено значение, отличное от нулевого значения XmlResolver, с настройками по умолчанию или небезопасными настройками. Если вам необходимо включить обработку DTD, инструкции о том, как это сделать безопасно, подробно описаны в [ссылка на статью MSDN](https://msdn.microsoft.com/en-us/magazine/ee335713.aspx).

### XmlTextReader

`System.Xml.XmlTextReader` по умолчанию **небезопасен** в версиях .NET Framework до версии 4.5.2. Вот как сделать его безопасным в различных версиях .NET:

#### Prior to .NET 4.0

В версиях .NET Framework до версии 4.0 поведение синтаксического анализа DTD для объектов `XmlReader`, таких как `XmlTextReader`, управляется логическим свойством `ProhibitDtd`, находящимся в классах `System.Xml.XmlReaderSettings` и `System.Xml.XmlTextReader`.

Установите для этих значений значение true, чтобы полностью отключить встроенные DTDS.

``` csharp
XmlTextReader reader = new XmlTextReader(stream);
// НЕОБХОДИМО, потому что значение по умолчанию равно FALSE!!
reader.ProhibitDtd = true;  
```

#### .NET 4.0 - .NET 4.5.2

**В .NET Framework версии 4.0 поведение при анализе DTD было изменено. Свойство `ProhibitDtd` было отменено в пользу нового свойства `DtdProcessing`.**

**Однако они не изменили настройки по умолчанию, поэтому "XmlTextReader" по умолчанию по-прежнему уязвим для XXE.**

**Установка для `DtdProcessing` значения `Запретить` приводит к тому, что среда выполнения выдает исключение, если в XML присутствует элемент `<!DOCTYPE>`.**

Чтобы установить это значение самостоятельно, оно выглядит следующим образом:

``` csharp
XmlTextReader reader = new XmlTextReader(stream);
//НЕОБХОДИМ, потому что значение по умолчанию - Parse!!
reader.DtdProcessing = DtdProcessing.Prohibit;  
```

В качестве альтернативы, вы можете установить для свойства `DtdProcessing` значение `Игнорировать`, которое не будет генерировать исключение при обнаружении элемента `<!DOCTYPE>`, а просто пропустит его и не обработает. Наконец, вы можете установить для `DtdProcessing` значение `Parse`, если вы действительно хотите разрешить и обрабатывать встроенные DTD.

#### .NET 4.5.2 и позднее

В .NET Framework версий 4.5.2 и выше для внутреннего `XmlResolver` в `XmlTextReader` по умолчанию установлено значение null, что заставляет `XmlTextReader` игнорировать DTD по умолчанию. `XmlTextReader` может стать небезопасным, если вы создадите свой собственный ненулевой "XmlResolver" с настройками по умолчанию или небезопасными настройками.

### XPathNavigator

`System.Xml.XPath.XPathNavigator` по умолчанию является небезопасным в версиях .NET Framework до версии 4.5.2.

Это связано с тем, что он реализует объекты `IXPathNavigable`, такие как `XmlDocument`, которые также небезопасны по умолчанию в версиях до 4.5.2.

Вы можете сделать `XPathNavigator` безопасным, предоставив ему безопасный синтаксический анализатор, такой как `XmlReader` (который безопасен по умолчанию) в конструкторе `XPathDocument`.

Вот пример:

``` csharp
XmlReader reader = XmlReader.Create("example.xml");
XPathDocument doc = new XPathDocument(reader);
XPathNavigator nav = doc.CreateNavigator();
string xml = nav.InnerXml.ToString();
```

Для .NET Framework версии ≥4.5.2 XPathNavigator **по умолчанию безопасен**.

### XslCompiledTransform

`System.Xml.Xsl.XslCompiledTransform` (XML-трансформатор) безопасен по умолчанию, если указанный им синтаксический анализатор безопасен.

По умолчанию это безопасно, потому что синтаксическим анализатором методов `Transform()` по умолчанию является `XmlReader`, который по умолчанию безопасен (согласно приведенному выше).

[Исходный код для этого метода приведен ниже here.](http://www.dotnetframework.org/default.aspx/4@0/4@0/DEVDIV_TFS/Dev10/Releases/RTMRel/ndp/fx/src/Xml/System/Xml/Xslt/XslCompiledTransform@cs/1305376/XslCompiledTransform@cs)

Некоторые из методов `Transform()` принимают `XmlReader` или `IXPathNavigable` (например, `XmlDocument`) в качестве входных данных, и если вы передадите небезопасный синтаксический анализатор XML, то `Transform` также будет небезопасным.

## iOS

### libxml2

**iOS включает в себя библиотеку C/C++ libxml2, описанную выше, так что это руководство применимо, если вы используете libxml2 напрямую.**

**Однако версия libxml2, предоставляемая через iOS6, предшествует версии 2.9 libxml2 (которая по умолчанию защищает от XXE).**

### NSXMLDocument

**iOS также предоставляет тип "NSXMLDocument", который построен поверх libxml2.**

**Однако `NSXMLDocument` предоставляет некоторые дополнительные средства защиты от XXE, которые недоступны непосредственно в libxml2.**

В соответствии с разделом "NSXMLDocument External Entity Restriction API" этого [page](https://developer.apple.com/library/archive/releasenotes/Foundation/RN-Foundation-iOS/Foundation_iOS5.html):

- iOS4 и более ранние версии: по умолчанию загружаются все внешние объекты.
- iOS5 и более поздние версии: загружаются только объекты, для которых не требуется доступ к сети. (что более безопасно)

**Однако, чтобы полностью отключить XXE в "NSXMLDocument" в любой версии iOS, вы просто указываете "NSXMLNodeLoadExternalEntitiesNever" при создании `NSXMLDocument`.**

## PHP

**При использовании синтаксического анализатора XML по умолчанию (основанного на libxml2), PHP 8.0 и более поздних версий [по умолчанию не используется XXE](https://www.php.net/manual/en/function.libxml-disable-entity-loader.php).**

**Для версий PHP до 8.0, согласно [документации по PHP](https://www.php.net/manual/en/function.libxml-set-external-entity-loader.php), при использовании синтаксического анализатора PHP XML по умолчанию должно быть установлено следующее, чтобы предотвратить XXE:**

``` php
libxml_set_external_entity_loader(null);
```

Описание того, как использовать это в PHP, представлено в хорошей статье [SensePost](https://www.sensepost.com/blog/2014/revisting-xxe-and-abusing-protocols/), описывающей классную уязвимость XXE на основе PHP, которая была исправлена в Facebook.

## Python

Официальная документация по Python 3 содержит раздел, посвященный [уязвимостям xml](https://docs.python.org/3/library/xml.html#xml-vulnerabilities). С 1 января 2020 года Python2 больше не поддерживается, однако на веб-сайте Python по-прежнему содержится [некоторая устаревшая документация](https://docs.Python.org/2/library/xml.html#xml-vulnerabilities).

В таблице ниже показано, какие различные модули синтаксического анализа XML в Python 3 уязвимы для определенных атак XXE.

| Attack Type               | sax        | etree      | minidom    | pulldom    | xmlrpc     |
|---------------------------|------------|------------|------------|------------|------------|
| Billion Laughs            | Уязвим     | Уязвим     | Уязвим     | Уязвим     | Уязвим     |
| Quadratic Blowup          | Уязвим     | Уязвим     | Уязвим     | Уязвим     | Уязвим     |
| External Entity Expansion | Безопасен  | Безопасен  | Безопасен  | Безопасен  | Безопасен  |
| DTD Retrieval             | Безопасен  | Безопасен  | Безопасен  | Безопасен  | Безопасен  |
| Decompression Bomb        | Безопасен  | Безопасен  | Безопасен  | Безопасен  | Уязвим     |

Для защиты вашего приложения от соответствующих атак существуют [два пакета](https://docs.python.org/3/library/xml.html#the-defusedxml-and-defusedexpat-packages), которые помогают вам очистить вводимые данные и защитить ваше приложение от DDoS-атак и удаленных атак.

## Правила Semgrep 

[Semgrep](https://semgrep.dev/) - это инструмент командной строки для автономного статического анализа. Используйте готовые или пользовательские правила для обеспечения соблюдения кода и стандартов безопасности в вашей кодовой базе.

### Java

Ниже приведены правила для различных синтаксических анализаторов XML в Java

#### Digester

Выявление уязвимости XXE в библиотеке `org.apache.commons.digester3.Digester"
С правилом можно ознакомиться здесь [https://semgrep.dev/s/salecharohit:xxe-Digester](https://semgrep.dev/s/salecharohit:xxe-Digester)

#### DocumentBuilderFactory

Определение уязвимости XXE в библиотеке javax.xml.parsers.DocumentBuilderFactory
Правило можно воспроизвести здесь [https://semgrep.dev/s/salecharohit:xxe-dbf](https://semgrep.dev/s/salecharohit:xxe-dbf)

#### SAXBuilder

Определение уязвимости XXE в библиотеке `org.jdom2.input.SAXBuilder`
Правило можно воспроизвести здесь [https://semgrep.dev/s/salecharohit:xxe-saxbuilder](https://semgrep.dev/s/salecharohit:xxe-saxbuilder)

#### SAXParserFactory

Выявление уязвимости XXE в библиотеке javax.xml.parsers.SAXParserFactory
Правило можно воспроизвести здесь [https://semgrep.dev/s/salecharohit:xxe-SAXParserFactory](https://semgrep.dev/s/salecharohit:xxe-SAXParserFactory)

#### SAXReader

Выявление уязвимости XXE в библиотеке org.dom4j.io.SAXReader
С правилом можно ознакомиться здесь [https://semgrep.dev/s/salecharohit:xxe-SAXReader](https://semgrep.dev/s/salecharohit:xxe-SAXReader)

#### XMLInputFactory

Определение уязвимости XXE в библиотеке javax.xml.stream.XMLInputFactory
Правило можно воспроизвести здесь [https://semgrep.dev/s/salecharohit:xxe-XMLInputFactory](https://semgrep.dev/s/salecharohit:xxe-XMLInputFactory)

#### XMLReader

Выявление уязвимости XXE в библиотеке org.xml.sax.XmlReader
Правило можно воспроизвести здесь [https://semgrep.dev/s/salecharohit:xxe-XMLReader](https://semgrep.dev/s/salecharohit:xxe-XMLReader)

## References

- [XXE от InfoSecInstitute](https://resources.infosecinstitute.com/identify-mitigate-xxe-vulnerabilities/)
- [OWASP Top 10-2017 A4: Внешние объекты в формате XML (XXE)](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A4-XML_External_Entities_%28XXE%29)
- [Статья Тимоти Моргана 2014 года: "XML-схемы, DTD и атаки на объекты"](https://vsecurity.com//download/papers/XMLDTDEntityAttacks.pdf)
- [Обнаружение ошибок XXE с помощью FindSecBugs](https://find-sec-bugs.github.io/bugs.htm#XXE_SAXPARSER)
- [Инструмент поиска ошибок XXEBUG](https://github.com/ssexxe/XXEBugFind)
- [Тестирование для XML Injection](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/07-Input_Validation_Testing/07-Testing_for_XML_Injection.html)
