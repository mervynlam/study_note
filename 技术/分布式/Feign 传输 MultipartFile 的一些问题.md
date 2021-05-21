# Feign 传输 MultipartFile 的一些问题

## File 转 MultipartFile

`pom.xml`

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-mock -->
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-mock</artifactId>
<version>2.0.8</version>
</dependency>
```

```java
public static MultipartFile getMultipartFile(String fileName, File file) throws IOException {
    return new MockMultipartFile(fileName, file.getName(), ContentType.APPLICATION_OCTET_STREAM.toString(), new FileInputStream(file));
}
```

## 报错 Current request is not a multipart request、Content type '' not supported

- @PostMapping设置 consumes = MediaType.MULTIPART_FORM_DATA_VALUE
- 使用@RequestPart()，不能使用@RequestParam()

```java
@PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
ResultBody upload(@RequestPart(value = "file") MultipartFile file);
```

## 报错 Required request part ‘file’ is not present

`configuration`

```java
@Configuration
public class UploadFeignConfig {
    @Bean
    public Encoder multipartFormEncoder() {
        return new SpringFormEncoder(new SpringEncoder(new ObjectFactory<HttpMessageConverters>() {
            @Override
            public HttpMessageConverters getObject() throws BeansException {
                return new HttpMessageConverters(new RestTemplate().getMessageConverters());
            }
        }));
    }
}
```

`FeignClient`

```java
@FeignClient(value = FileConstants.FILE_SERVER, configuration= UploadFeignConfig.class)
public interface FileServiceClient extends IFileServiceClient {
    @Override
    @PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
	ResultBody upload(@RequestPart(value = "file") MultipartFile file);
}
```

