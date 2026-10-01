# 🍃 Cloud Native Spring in Action (Learning Repository)

<a href="https://www.manning.com/books/cloud-native-spring-in-action"><img src="/book-cover.png" alt="The book cover of 'Cloud Native Spring in Action' by Thomas Vitale" align="left" height="200px" /></a>

This GitHub Organization is a personal learning workspace for following along with the book [Cloud Native Spring in Action - With Spring Boot and Kubernetes](https://www.manning.com/books/cloud-native-spring-in-action) written by [Thomas Vitale](https://www.thomasvitale.com/) and published by Manning Publications.

<br clear="left">
<br>

> [!NOTE]
> The official source code for the book is available at [ThomasVitale/cloud-native-spring-in-action](https://github.com/ThomasVitale/cloud-native-spring-in-action).
> 
> This project is **not a direct fork** of the original repository.\
> Instead, it was created from scratch to learn and practice the concepts hands-on.\
> There might be some deviations from the book, such as using newer technologies like **Spring Boot 4**, exploring alternative approaches, or personal experimentation.

## Repositories

This organization hosts my implementation of Polar Bookshop from [Cloud Native Spring in Action](https://www.manning.com/books/cloud-native-spring-in-action).

The table includes existing repositories and planned additions based on the book's final project. **Existing** means the repository has been created; development continues as I work through the book.

| Repository           | Role                                                                                                                                   |  Status   |
|----------------------|----------------------------------------------------------------------------------------------------------------------------------------|:---------:|
| [polar-deployment]   | Environment setup and deployment configuration.<br>Currently contains Docker Compose and Kubernetes; GitOps configuration will follow. | Existing  |
| [config-service]     | Spring Cloud Config Server that serves application configuration from Git.                                                             | Existing  |
| [config-repo]        | Configuration YAML files served by Config Service.                                                                                     | Existing  |
| [catalog-service]    | REST API for managing the book catalog, with PostgreSQL persistence.                                                                   | Existing  |
| `order-service`      | Reactive API for placing book orders, storing them in PostgreSQL, and tracking their status.                                           | *Planned* |
| `edge-service`       | API gateway for routing requests, authentication, rate limiting, and circuit breakers.                                                 | *Planned* |
| `dispatcher-service` | Processes accepted-order events and publishes dispatch notifications through RabbitMQ.                                                 | *Planned* |
| `polar-ui`           | Angular frontend for browsing and managing books and placing and viewing orders.                                                       | *Planned* |
| `quote-service`      | Reactive REST API for retrieving book quotes, including random quotes by genre.                                                        | *Planned* |
| `quote-function`     | Spring Cloud Function implementation for retrieving book quotes.                                                                       | *Planned* |

> [!NOTE]
> *Planned repositories will be linked once they are created.*


## API Examples and Persistence Fields

The Chapter 5 Catalog Service uses the `Book` persistence record directly as the request and response body, without separate DTOs or DTO/entity mapping.\
Its primitive `int version` field must be supplied in POST/PUT requests with the current JSON configuration; omitting it results in HTTP 400 during deserialization.

HTTPie examples therefore include `version:=0` (a JSON number).\
For POST, zero represents a new entity.\
For an existing book, the current PUT implementation uses the version loaded from the database instead of the submitted value, so this request field does not provide client-side stale-update detection.\
See [Chapter 5's request-body notes]([Chapter05/README.md#why-post-and-put-include-version](https://github.com/fResult/cloud-native-spring-in-action/blob/main/Chapter05/README.md#why-post-and-put-include-version)) for details.

This applies to the current Chapter 5 implementation.\
Chapters 3 and 4 do not have a `version` field on `Book`.

## Prerequisites

Chapter after chapter, you'll build, containerize, and deploy cloud native applications.\
Along the journey, you will need the following software installed.

- Java 26+
  - OpenJDK: [Eclipse Temurin](https://adoptium.net)
  - GraalVM: [GraalVM](https://www.graalvm.org)
  - JDK Management: [SDKMAN](https://sdkman.io)
- Docker 29+
  - [Docker for Linux](https://docs.docker.com/engine/install/ubuntu/)
  - [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop)
  - [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop)
- Kubernetes 1.30+
  - [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
  - [minikube](https://minikube.sigs.k8s.io/docs/)
- Other
  - [HTTPie](https://httpie.org/)

## Gradle and Maven

The code samples in the book use Gradle as the build tool.\
Should you prefer Maven, here's a table mapping Gradle commands to Maven so that you can easily follow along.

| Gradle                     | Maven                                        | 
|----------------------------|----------------------------------------------|
| `./gradlew clean`          | `./mvnw clean`                               |
| `./gradlew build`          | `./mvnw install`                             |
| `./gradlew test`           | `./mvnw test`                                |
| `./gradlew bootJar`        | `./mvnw spring-boot:repackage`               |
| `./gradlew bootRun`        | `./mvnw spring-boot:run`                     |
| `./gradlew bootBuildImage` | `./mvnw spring-boot:build-image -DskipTests` |

## Guides, Tools and Tips

- [Configuring IntelliJ IDEA](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/configuring-intellij-idea.md)
- [Configuring Visual Studio Code](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/configuring-visual-studio-code.md)
- [Knative Platform Cloud Installation](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/knative-platform-cloud-installation.md)
- [Minikube configuration behind a proxy](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/minikube-configuration-behind-a-proxy.md)
- [Observability setup on Kubernetes](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/grafana-observability-stack)
- [Replacing Kubeval with Kubeconform](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/replacing-kubeval-with-kubeconform.md)
- [Replacing Octant](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/replacing-octant.md)
- [Setting up a Kubernetes cluster for Polar Bookshop on Azure](#)
- [Setting up a Kubernetes cluster for Polar Bookshop on DigitalOcean](#)
- [Testing RabbitMQ with Testcontainers](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/testing-rabbitmq-with-testcontainers.md)
- [Working with macOS on Apple Silicon](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/working-with-macos-on-apple-silicon.md)
- [Working with macOS on Intel](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/working-with-macos-on-intel.md)
- [Working with Windows](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Guides/working-with-windows.md)

## Source Code by Chapter

| Chapter                                          | Starting point                                                                                         | Intermediate version                                                                                                 | Final version                                                                                      | My version             |
|--------------------------------------------------|--------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|--------------------------|
| 1. Introduction to cloud native                  | -                                                                                                      | -                                                                                                                    | -                                                                                                  | -                       |
| 2. Cloud native patterns and technologies        | [02-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter02/02-begin) | -                                                                                                                    | [02-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter02/02-end) | -                       |
| 3. Getting started with cloud native development | [03-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter03/03-begin) | -                                                                                                                    | [03-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter03/03-end) | [Chapter 3][chapter-03] |
| 4. Externalized configuration management         | [04-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter04/04-begin) | -                                                                                                                    | [04-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter04/04-end) | [Chapter 4][chapter-04] |
| 5. Persisting and managing data in the cloud     | [05-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter05/05-begin) | [05-intermediate](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter05/05-intermediate) | [05-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter05/05-end) | [Chapter 5][chapter-05] |
| 6. Containerizing Spring Boot                    | [06-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter06/06-begin) | -                                                                                                                    | [06-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter06/06-end) | [Chatper 6][chapter-06] |
| 7. Kubernetes fundamentals for Spring Boot       | [07-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter07/07-begin) | -                                                                                                                    | [07-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter07/07-end) | [Chatper 7][chapter-07] |
| 8. Reactive Spring: Resilience and scalability   | [08-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter08/08-begin) | -                                                                                                                    | [08-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter08/08-end) |                         |
| 9. API gateway and circuit breakers              | [09-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter09/09-begin) | -                                                                                                                    | [09-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter09/09-end) |                         |
| 10. Event-driven applications and functions      | [10-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter10/10-begin) | [10-intermediate](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter10/10-intermediate) | [10-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter10/10-end) |                         |
| 11. Security: Authentication and SPA             | [11-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter11/11-begin) | -                                                                                                                    | [11-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter11/11-end) |                         |
| 12. Security: Authorization and auditing         | [12-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter12/12-begin) | -                                                                                                                    | [12-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter12/12-end) |                         |
| 13. Observability and monitoring                 | [13-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter13/13-begin) | -                                                                                                                    | [13-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter13/13-end) |                         |
| 14. Configuration and secrets management         | [14-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter14/14-begin) | -                                                                                                                    | [14-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter14/14-end) |                         |
| 15. Continuous delivery and GitOps               | [15-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter15/15-begin) | -                                                                                                                    | [15-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter15/15-end) |                         |
| 16. Serverless, GraalVM and Knative              | [16-begin](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter16/16-begin) | -                                                                                                                    | [10-end](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/Chapter16/16-end) |                         |

## Polar Bookshop

The final project developed throughout the book is available [here](https://github.com/ThomasVitale/cloud-native-spring-in-action/tree/main/PolarBookshop).

You can find the source code for the Angular frontend [here](https://github.com/PolarBookshop/polar-ui/tree/v1).

## Book Forum

Feel free to submit questions, feedback, or errata to the forum dedicated to "Cloud Native Spring in Action": https://livebook.manning.com/book/cloud-native-spring-in-action/.

## Contact the Author

You are very welcome to contact me for questions, feedback, or suggestions.\
Feel free to reach out to me on [Twitter](https://twitter.com/vitalethomas), [LinkedIn](https://www.linkedin.com/in/vitalethomas), [Mastodon](https://mastodon.online/@thomasvitale), [BlueSky](https://bsky.app/profile/thomasvitale.com) or here on [GitHub](https://github.com/ThomasVitale/).

<!-- Repositories -->
[polar-deployment]: https://github.com/fResult-PolarBookshop/polar-deployment
[config-repo]: https://github.com/fResult-PolarBookshop/config-repo
[config-service]: https://github.com/fResult-PolarBookshop/config-service
[catalog-service]: https://github.com/fResult-PolarBookshop/catalog-service

<!-- Chapters -->
[chapter-03]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter03
[chapter-04]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter04
[chapter-05]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter05
[chapter-06]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter06
[chapter-07]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter07
[chapter-08]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter08
[chapter-09]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter09
[chapter-10]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter10
[chapter-11]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter11
[chapter-12]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter12
[chapter-13]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter13
[chapter-14]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter14
[chapter-15]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter15
[chapter-16]: https://github.com/fResult/cloud-native-spring-in-action/tree/main/Chapter16
