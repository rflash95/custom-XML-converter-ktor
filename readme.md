## Custom Converter In Ktor

1. Implement the interface ContentConverter
2. Implement the method convertForSend
3. Implement the method convertForReceive

This is an example we make the XmlConverter
```kotlin
/**  
 * Custom XML converter for [ContentNegotiation] feature 
 * */  
class XmlConverter() : ContentConverter {  
  
  
 override suspend fun convertForReceive(context: PipelineContext<ApplicationReceiveRequest, ApplicationCall>): Any? {  
    val request = context.subject  
    val channel = request.value as? ByteReadChannel ?: return null  
    val reader = channel.toInputStream().reader(context.call.request.contentCharset() ?: Charsets.UTF_8)  
    val type = request.type  
    val xmlMapper = XmlMapper()  
    val xml = reader  
    val result: Any? = xmlMapper.readValue(xml, type.javaObjectType)  
    return result   
    }  
  
    override suspend fun convertForSend(  
        context: PipelineContext<Any, ApplicationCall>,  
        contentType: ContentType,  
        value: Any  
    ): Any? {  
        val xmlMapper = XmlMapper()  
        val xml = xmlMapper.writeValueAsString(value)  
        return TextContent(xml, contentType.withCharset(context.call.suitableCharset()))  
    }  
}
```

4. Install the feature
```kotlin
install(ContentNegotiation) {  
  gson {  
 }  
 //register(the content type, the content type converter)  
  register(ContentType.Application.Xml, XmlConverter())  
}
```
5.  Uses the "Accept" header to find the matching send converter
- XML
```vim
curl \  
-H "Accept: application/xml" \  
"http://localhost:8080/spaceship"
```
Response :
```xml
<SpaceShip><name>Mike</name><fuel>88</fuel></SpaceShip>
```  

- JSON
```vim
curl \  
-H "Accept: application/json" \  
"http://localhost:8080/spaceship"
```  
Response :
```json
{"name":"Mike","fuel":88}
```
---

=Thank You :)

