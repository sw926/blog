---
title: Spring Boot 发起Http请求
date: 2019-02-22 15:05:23
category: Java
tags: 
    - Spring Boot
---

在 SpringBoot 发起 http request 一般使用 RestTemplate，在 Android 开发中，使用最多的是 OkHttp，当然在 SpringBoot 中也可以使用，但是不能做到开箱即用，还要添加 Json 解析的 Adapter，不然 RestTemplate 方便。

RestTemplate 可以直接使用，我们可以使用 `@Configuration` 进行配置

``` java
@Configuration
public class RestTemplateConfig {

    @Bean
    @ConditionalOnMissingBean({ClientHttpRequestFactory.class})
    public ClientHttpRequestFactory requestFactory() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setConnectTimeout(15000);
        factory.setReadTimeout(15000);
        return factory;
    }


    @Bean
    @ConditionalOnMissingBean({RestTemplate.class})
    public RestTemplate nimRestTemplate(ClientHttpRequestFactory factory) {
        RestTemplate restTemplate = new RestTemplate(factory);
        return restTemplate;
    }
}
```

之后就可以直接发起请求：

``` java
@Service
public class TestService {


    @Resource
    private RestTemplate restTemplate;

    public UserResult test() {
        String url = "http://localhost:3000/test";
        return restTemplate.getForObject(url, UserResult.class);
    }

}
```

## 添加 Header

``` java
HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.MULTIPART_FORM_DATA);
HttpEntity<String> entity1 = new HttpEntity<>(headers)
```




