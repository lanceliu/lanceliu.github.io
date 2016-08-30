---
layout: post
title:  "Spring MVC整合Swagger"
date:   2016-08-30 09:50:52
categories: springmvc swagger
published: true
comments: true
thread: 20160830095055555
---
Spring MVC整合Swagger
---
API文档相信大家都有维护的经历，但如何构建一个容易阅读，维护，测试的API文档呢，找过一些资料，发现Swagger比较符合上面这三点。
[swagger](http://swagger.io/)包括库、UI、编辑器、代码生成器等很多部分.
  - UI, API列表显示和测试。
  - 编辑器：API文档定义，生成一个API的json定义文件。
  - 代码生成器：根据定义生成相应的代码。

上面这三项我们都不讲，主要讲Spring MVC和Swagger的集成。

## 1. Maven依赖
如果需要集成的项目，spring json消息转换器不是jackson的需要加特殊处理。
```xml
<repositories>
    <repository>
      <id>jcenter-snapshots</id>
      <name>jcenter</name>
      <url>https://jcenter.bintray.com/</url>
    </repository>
</repositories>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.5.0</version>
</dependency>
```
## 2. Swagger配置
```java
@Configuration
@PropertySource("classpath:swagger.properties")
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Yuntu API")
                .description("Move your app forward with the rest API")
                .license("")
                .licenseUrl("")
                .termsOfServiceUrl("")
                .version("1.0.0")
                .contact(new Contact("","", ""))
                .build();
    }

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.basePackage("org.yuntu.wap"))
                .build();
    }
}
```
```xml
<!-- Enables swgger ui-->
<mvc:resources mapping="/swagger-ui.html" location="classpath:/META-INF/resources/"/>
<mvc:resources mapping="/webjars/**" location="classpath:/META-INF/resources/webjars/"/>    <!-- Include a swagger configuration-->
```
## 3. 项目Controller配置
Swagger官方不推荐使用swagger-core包中的annotation来配置，尽量使用例如jackson annotation，Spring annotation等。

1. 入参映射为对象的参数描述
下面这种情况，本来是传入x-www-form-urlencoded类型参数， Swagger生成的参数类型却是application/json
```java
@RequestMapping(value = "/tel/{tel}", method = RequestMethod.GET)
public ResponseEntity mobileUniqueCheck(LoginNameRequestDTO loginNameRequest) {
    BaseResponseDTO baseResponse = accountv4Biz.mobileUniqueCheck(loginNameRequest);

    return baseResponse.createSuccessResponse();
}
```
修正后
```java
@RequestMapping(value = "/tel/{tel}", method = RequestMethod.GET)
@ApiImplicitParams({
        @ApiImplicitParam(name = "loginName", dataType = "string", paramType = "query",
                value = "Results "),
        @ApiImplicitParam(name = "authCodeType", dataType = "byte", paramType = "query",
                value = "Number of records per page.")
})
public ResponseEntity mobileUniqueCheck(@ApiIgnore LoginNameRequestDTO loginNameRequest) {
    BaseResponseDTO baseResponse = accountv4Biz.mobileUniqueCheck(loginNameRequest);

    return baseResponse.createSuccessResponse();
}
```
@ApiImplicitParams,@ApiImplicitParam 必须配合使用。当rest服务接收参数为一个资源对象时，需要这么配置， 同时要在参数处标注@ApiIgnore
2. API描述的分组
  - 一种分组方式，把API描述在不同的API页面中显示。在Swagger-UI.html 页面中右上角的下拉列表切换显示。
  ```java
  @Bean
  public Docket xxxx() {
    .......
  }
  ```
  - 第二种方式，在同一个页面中，通过tag的形式分组。@Api，@ApiOperation中都提供了对tag配置的属性。

## 4. [官方参考示例](https://github.com/springfox/springfox-demos)

## 5. 效果
输入 `your context/swagger-ui.html`
