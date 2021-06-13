# 使用RestTemplate进行Http请求

RestTemplate可以方便地进行HTTP请求。

## 1. 初始化RestTemplate对象

==RestTemplate 必须手动初始化，如果直接使用自动注入则为空。==

```java
@Configuration
public class RestConfiguration {

    @Autowired
    RestTemplateBuilder builder;

    @Bean
    public RestTemplate restTemplate(ResponseErrorHandler responseErrorHandler){
        RestTemplate template = builder.build();
        template.setErrorHandler(responseErrorHandler);
        return template;
    }

    @Bean
    public ResponseErrorHandler responseErrorHandler() {
        return new ResponseErrorHandler() {
            @Override
            public boolean hasError(ClientHttpResponse clientHttpResponse) throws IOException {
                // 这里表示不管返回什么异常都认为没有问题，否则返回的不是200的时候，都会抛出异常
                return false;
            }

            @Override
            public void handleError(ClientHttpResponse clientHttpResponse) throws IOException {

            }
        };
    }
}
```

## 2. 使用

### 2.1 注入对象

```java
@Autowired
private RestTemplate restTemplate;
```

### 2.2 代码中使用

可以使用restTemplate.postForEntity，restTemplate.postForObject进行请求，区别是使用restTemplate.postForEntity可以获取返回的HttpStatus Code等Http Response相关信息，而使用restTemplate.postForObject则只能返回对象。

还可以使用restTemplate.exchange进行请求，此处暂不讨论该用法。

==注意返回的json中的字段信息必须与POJO的字段定义一致==

```java
HttpHeaders headers = new HttpHeaders();
// default is MediaType.APPLICATION_FORM_URLENCODED
//headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);

// 放入请求参数
MultiValueMap<String, String> map= new LinkedMultiValueMap<>();
map.add("domain", domain);
map.add("username", username);
HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(map, headers);

ResponseEntity<User4Mongo> response = restTemplate.postForEntity( GET_USER_URL, request , User4Mongo.class );
/*User4Mongo object = restTemplate.postForObject(url, request, User4Mongo.class, map);
return object.toString();*/

logger.debug(response.toString());
if(response.getStatusCode().is2xxSuccessful()) {
    user = response.getBody();
}

return user;
```

## 3. 参考文档

详细的使用可以参考下列blogs

[Spring Boot 入门 - 进阶篇（4）- REST访问(RestTemplate)](https://rensanning.iteye.com/blog/2362105)

[Springboot — 用更优雅的方式发HTTP请求(RestTemplate详解)](https://www.cnblogs.com/javazhiyin/p/9851775.html)

[RestTemplate 使用中，异常报错处理](https://blog.csdn.net/houzidengyue/article/details/81907085)

https://www.cnblogs.com/huangminwen/p/6069072.html

https://www.cnblogs.com/linjiangxian/p/13097863.html
