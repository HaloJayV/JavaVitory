[TOC]

## 使用

* 依赖

  ```xmljava
   <dependencies>
          <!--swagger-->
          <dependency>
              <groupId>io.springfox</groupId>
              <artifactId>springfox-swagger2</artifactId>
              <scope>provided </scope>
          </dependency>
          <dependency>
              <groupId>io.springfox</groupId>
              <artifactId>springfox-swagger-ui</artifactId>
              <scope>provided </scope>
          </dependency>
      </dependencies>
  ```

* ```java
  @Configuration
  @EnableSwagger2
  public class SwaggerConfig {
  
      @Bean
      public Docket webApiConfig(){
  
          return new Docket(DocumentationType.SWAGGER_2)
                  .groupName("webApi")
                  .apiInfo(webApiInfo())
                  .select()
                  .paths(Predicates.not(PathSelectors.regex("/admin/.*")))
                  .paths(Predicates.not(PathSelectors.regex("/error.*")))
                  .build();
  
      }
      
      private ApiInfo webApiInfo(){
  
          return new ApiInfoBuilder()
                  .title("网站-课程中心API文档")
                  .description("本文档描述了课程中心微服务接口定义")
                  .version("1.0")
                  .contact(new Contact("Helen", "http://atguigu.com", "55317332@qq.com"))
                  .build();
      }
  }
  ```

  

* 其他module想要使用swagger，除了要在pom里引入改module依赖，还要在启动类扫描swagger配置类

  ```java
  @ComponentScan(basePackages = {"com.atguigu"})  // 扫描SwaggerConfig包括其他module的com.atguigu下的类
  ```

* 启动swagger： http://localhost:8001/swagger-ui.html

















