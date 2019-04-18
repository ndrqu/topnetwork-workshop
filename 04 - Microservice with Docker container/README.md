# 04 - Microservice with Docker container

## Table of Contents

- [04 - Microservice with Docker container](#04---microservice-with-docker-container)
  - [Table of Contents](#table-of-contents)
    - [Prerequisites](#prerequisites)
    - [Spring-boot](#spring-boot)
      - [Init](#init)
      - [Model](#model)
      - [Repository](#repository)
      - [Controller and Service](#controller-and-service)
      - [Configuring Swagger](#configuring-swagger)
      - [Monitoring](#monitoring)
    - [Dockerization](#dockerization)
      - [Sample](#sample)

### Prerequisites

- Java SDK 1.8+
- Maven or gradle build tool

To efficently manage Java and build tools verions you can use [SDKMAN!](https://sdkman.io/).

### Spring-boot

**Scope:** deliver an read API for exposing data from [DocumentDB](https://aws.amazon.com/documentdb/).

In order to provide a consistent, usable and maintaneble microservice we need to provide, for our **MVP**, at least:

- Feature #capitan-obvious
  - Model
  - Controller
  - Service
  - Dockerfile spec
  - Logging config
  - ...
- Documentantion (ideally README.md in the root folder) with:
  - Brief description of the feature
  - Enumeration of the conventions in-place (or a reference if defined globally per company)
  - [Swagger](https://swagger.io/) or [OpenAPI](https://www.openapis.org) enpoints
  - Unit tests
  - ...

#### Init

Starting a Spring boot project is pretty easy, you have to options:

- Using [Sprint Initializr](https://start.spring.io/)
- Using spring-boot-cli available on [SDKMAN!](https://sdkman.io/)!

Make sure to select at least one dependencies for implementing Rest repositories.

#### Model

Our domain model: what we need to work with. For the sake of semplicity our model will be straightforward and will not need any [DTO](https://en.wikipedia.org/wiki/Data_transfer_object).

#### Repository

Is the _meta_ interface for accessing out Mongo Database.

Create a `ModelRepository.java` with:

```java
@Repository
public interface ModelRepository extends PagingAndSortingRepository<Model, String> {


    List<Model> findByField(String field);

}
```

Repositories are convention driven [see here](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation) for a detailed explanation.

#### Controller and Service

In a layered architecture the controller is responsible to transmit/receive data from the transport channel (ex. HTTP/S or WebSockets).
The service contains all the plumbings and the business logic needed.

To make the things easier, in this lab you will use our `Repository` directly in the `Controller`.

**Be aware:** this is not a best practice.

Create `ModelController.java`:

```java
@RestController
@RequestMapping("/api/endpoint")
public class ModelController {

    private ModelRepository modelRepository;

    @Autowired
    public ModelController(ModelRepository modelRepository) {
        this.modelRepository = modelRepository;
    }

    @GetMapping
    public List<Workshop> findAll() {
        return Lists.newArrayList(modelRepository.findAll());
    }
}
```

Exercises:

- try implementing a **GET** endpoint to search by authors.
- try implementing a **POST** endpoint to _save_ one `Model.java`.

#### Configuring Swagger

Swagger is a common lib for documenting our endpoints.

Add [Springfox](https://springfox.github.io/springfox/) dependency to your `pom.xml`:

```xml
<!-- Define a property ala <java.version /> -->
<springfox.version>2.9.2</springfox.version>

<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>${springfox.version}</version>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>${springfox.version}</version>
</dependency>
```

Create `SwaggerConfig.java` in the root dir:

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

  @Bean
  public Docket api() {
      return
        new Docket(DocumentationType.SWAGGER_2)
          .useDefaultResponseMessages(false)
          .select()
          .apis(RequestHandlerSelectors.any())
          .paths(PathSelectors.any())
          .build();
  }
}
```

You can now view the generated manifest on `/v2/api-docs` path and access the UI on `/swagger-ui.html` path.

Exercise:

- try exposing only the controllers you implemented (ex. those with `@RestController` annotation).

#### Monitoring

> Spring Boot Actuator module provides production-ready features like health checks, metrics, auditing etc. All of these features can be consumed over HTTP endpoints or JMX.

Update your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Modify `application.yml`:

```bash
info:
  app:
    name: Workshop 04
    description: Microservice with Docker container
    java:
      version: ${java.version}
management:
  endpoint:
    health:
      show-details: always
```

### Dockerization

#### Sample

```Docker
FROM image

WORKDIR /app

COPY . /app
RUN command

EXPOSE port

CMD do something on run
```

Exercise:

- Fill the gaps.
