# Polar Bookshop

A personal, repository-per-component implementation inspired by the Polar Bookshop system in [Cloud Native Spring in Action][book].

This organization contains independently versioned services, shared runtime configuration, and environment deployment definitions.\
It is a hands-on learning project, not an official implementation or a direct fork of the book source code.

> [!NOTE]
> For the chapter-by-chapter learning workspace, see [fResult/cloud-native-spring-in-action][learning-repo].

## System repositories

The responsibilities below describe the intended role of each component in the Polar Bookshop architecture.\
They are a guide derived from the book's final project—not a promise to reproduce its code, design, dependencies, or release versions exactly.

|            Repository            | Responsibility                                                                                                              |  Status   |
|:--------------------------------:|-----------------------------------------------------------------------------------------------------------------------------|:---------:|
|   [catalog-service]<br>(Java)    | REST API for managing the book catalog, with PostgreSQL persistence.                                                        | Available |
|   [order-service]<br>(Kotlin)    | Reactive API for placing book orders, storing them in PostgreSQL, and tracking their status.                                | Available |
|    [config-service]<br>(Java)    | Configuration server that serves application configuration from Git.                                                        | Available |
|          [config-repo]           | Version-controlled, externalized application configuration consumed by Config Service.                                      | Available |
|        [polar-deployment]        | Environment setup and deployment configuration, including local and Kubernetes resources.                                   | Available |
|    `edge-service`<br>(Kotlin)    | API gateway for routing requests, authentication, rate limiting, and circuit breakers.                                      |  Planned  |
| `dispatcher-service`<br>(Kotlin) | Processes accepted-order events and publishes dispatch notifications through RabbitMQ.                                      |  Planned  |
|     `polar-ui`<br>(Angular)      | Frontend for browsing and managing books, and placing and viewing orders.                                                   |  Planned  |
|         `quote-service`          | Reactive REST API for retrieving book quotes, including random quotes by genre.                                             |  Planned  |
|         `quote-function`         | API gateway for routing requests and cross-cutting concerns, including authentication, rate limiting, and circuit breakers. |  Planned  |

## Target architecture

The diagram shows the intended target topology for this organization.\
It is inspired by the final Polar Bookshop project from the book and the repositories published by the [PolarBookshop reference organization][polarbookshop-reference], but it represents this project's own architectural direction—not a reproduction of the reference implementation.

Components, integrations, technologies, and operational decisions may intentionally be added, omitted, or designed differently as the learning project evolves.\
For example, this target architecture includes shared externalized configuration for services beyond those configured that way in the reference implementation.

It includes both available and planned repositories; it does not imply that every component is already deployed or that this implementation will mirror the reference code, design, dependencies, or release versions exactly.

`polar-deployment` provides the runtime environment around the services—for example, Docker Compose, Kubernetes resources, backing services, and delivery configuration.\
Each application repository owns its source code, tests, container build, and CI workflow

```mermaid
%% Polar Bookshop — target topology, clarified from the reference implementation.
%% polar-ui serves SPA assets only; once loaded, its Angular code runs in the Browser.
%% Config Server reads Git-backed configuration and serves it to Config Clients.
%%{init: {'theme':'base','securityLevel':'loose','themeVariables':{'darkMode':true,'background':'#333333','primaryColor':'#FFFFFF','primaryTextColor':'#0F172A','primaryBorderColor':'#CBD5E1','secondaryColor':'#F8FAFC','secondaryTextColor':'#0F172A','tertiaryColor':'#F8FAFC','tertiaryTextColor':'#0F172A','lineColor':'#CBD5E1','textColor':'#F8FAFC','edgeLabelBackground':'#F8FAFC','nodeTextColor':'#0F172A','fontSize':'16px'},'themeCSS':'svg { background-color: #333333 !important; } .edgeLabel, .edgeLabel p { background-color: #F8FAFC !important; color: #0F172A !important; }'}}%%
flowchart TB
    browser[Browser]
    edge["[edge-service]<br/>API gateway / BFF"]
    ui["[polar-ui]<br/>Angular SPA assets"]

    %% The initial document/assets pass through the gateway; edge reverse-proxies polar-ui.
    browser browserToEdgeAssets@-->|GET /, JS, CSS, favicon| edge
    edge edgeToUi@-->|reverse proxy: SPA/static assets| ui

    %% After the SPA is loaded, the Browser—not polar-ui's server—makes these requests.
    browser browserToEdgeApi@-->|XHR/fetch: /books, /orders, /user<br/>OAuth2 login/logout| edge
    edge edgeToCatalog@-->|/books/**; TokenRelay| catalog["[catalog-service]"]
    edge edgeToOrder@-->|/orders/**; TokenRelay| order["[order-service]"]

    %% Internal synchronous call; it does not go back through the gateway.
    order orderToCatalog@-->|checks catalog directly| catalog

    catalog catalogToPostgres@-->|polardb_catalog| postgres[(PostgreSQL<br/>one instance; two logical databases)]
    order orderToPostgres@-->|polardb_order| postgres
    order orderToRabbitAccepted@-->|publishes order-accepted| rabbit[RabbitMQ]
    rabbit rabbitToDispatcher@-->|consumes order-accepted| dispatcher["[dispatcher-service]"]
    dispatcher dispatcherToRabbitDispatched@-->|publishes order-dispatched| rabbit
    rabbit rabbitToOrder@-->|consumes order-dispatched| order

    edge edgeToRedis@-->|session store / rate limiting| redis[(Redis)]
    edge edgeToKeycloak@-->|OAuth2 client / session| keycloak[Keycloak]
    catalog catalogToKeycloak@-->|JWT issuer/JWK discovery| keycloak
    order orderToKeycloak@-->|JWT issuer/JWK discovery| keycloak

    configRepo["[config-repo]<br/>application configuration"]
    config["[config-service]"]

    configRepo configRepoToConfig@-->|Git-backed configuration| config
    config configToCatalog@-->|serves externalized configuration| catalog
    config configToOrder@-.->|serves externalized configuration| order

    catalog catalogToTelemetry@--> telemetry[Observability stack]
    order orderToTelemetry@--> telemetry
    dispatcher dispatcherToTelemetry@--> telemetry
    edge edgeToTelemetry@--> telemetry
    config configToTelemetry@--> telemetry
    ui uiToTelemetry@-->|container logs| telemetry

    %% Independent Chapter 16 serverless examples.
    quote["[quote-service]"]
    quoteFunction["[quote-function]<br/>Spring Cloud Function"]
    knative[Knative Serving platform]

    quote quoteToKnative@-.->|knative/kservice.yml| knative
    quoteFunction quoteFunctionToKnative@-.->|knative/kservice.yml| knative

    deployment["[polar-deployment]"]

    deployment deploymentToEdge@-.->|deploys & provisions| edge
    deployment deploymentToUi@-.->|deploys & provisions| ui
    deployment deploymentToCatalog@-.->|deploys & provisions| catalog
    deployment deploymentToOrder@-.->|deploys & provisions| order
    deployment deploymentToDispatcher@-.->|deploys & provisions| dispatcher
    deployment deploymentToConfig@-.->|deploys & provisions| config
    deployment deploymentToPostgres@-.->|provisions| postgres
    deployment deploymentToRabbit@-.->|provisions| rabbit
    deployment deploymentToRedis@-.->|provisions| redis
    deployment deploymentToKeycloak@-.->|provisions| keycloak
    deployment deploymentToTelemetry@-.->|provisions| telemetry

    %% Node styles.
    classDef client fill:#E0F2FE,stroke:#0369A1,color:#0C4A6E,stroke-width:2px;
    classDef gateway fill:#EDE9FE,stroke:#6D28D9,color:#2E1065,stroke-width:2px;
    classDef service fill:#DBEAFE,stroke:#1D4ED8,color:#172554,stroke-width:2px;
    classDef serverless fill:#F3E8FF,stroke:#7E22CE,color:#3B0764,stroke-width:2px;
    classDef data fill:#DCFCE7,stroke:#15803D,color:#14532D,stroke-width:2px;
    classDef messaging fill:#FFEDD5,stroke:#C2410C,color:#7C2D12,stroke-width:2px;
    classDef identity fill:#FCE7F3,stroke:#BE185D,color:#831843,stroke-width:2px;
    classDef configuration fill:#FEF3C7,stroke:#B45309,color:#78350F,stroke-width:2px;
    classDef observability fill:#CFFAFE,stroke:#0E7490,color:#164E63,stroke-width:2px;
    classDef platform fill:#E2E8F0,stroke:#475569,color:#0F172A,stroke-width:2px;
    classDef deployment fill:#F1F5F9,stroke:#475569,color:#0F172A,stroke-width:2px;
    classDef planned stroke-dasharray:5 5;

    class browser client;
    class edge gateway;
    class ui,catalog,order,dispatcher service;
    class quote,quoteFunction serverless;
    class postgres,redis data;
    class rabbit messaging;
    class keycloak identity;
    class configRepo,config configuration;
    class telemetry observability;
    class knative platform;
    class deployment deployment;

    %% Planned components only. config-service is available, so it is intentionally excluded.
    class edge,ui,dispatcher,quote,quoteFunction planned;

    %% Edge styles, referenced by stable edge IDs rather than declaration indexes.
    classDef httpLink stroke:#60A5FA,color:#1E3A8A,stroke-width:2.5px;
    classDef internalLink stroke:#C4B5FD,color:#4C1D95,stroke-width:2.5px;
    classDef dataLink stroke:#4ADE80,color:#14532D,stroke-width:2.5px;
    classDef eventLink stroke:#FDBA74,color:#7C2D12,stroke-width:2.5px;
    classDef identityLink stroke:#F9A8D4,color:#831843,stroke-width:2.5px;
    classDef configurationLink stroke:#FCD34D,color:#78350F,stroke-width:2.5px;
    classDef telemetryLink stroke:#5EEAD4,color:#164E63,stroke-width:2.5px;
    classDef serverlessLink stroke:#C4B5FD,color:#3B0764,stroke-width:2.5px;
    classDef provisioningLink stroke:#CBD5E1,color:#0F172A,stroke-width:2px;

    class browserToEdgeAssets,edgeToUi,browserToEdgeApi,edgeToCatalog,edgeToOrder httpLink;
    class orderToCatalog internalLink;
    class catalogToPostgres,orderToPostgres,edgeToRedis dataLink;
    class orderToRabbitAccepted,rabbitToDispatcher,dispatcherToRabbitDispatched,rabbitToOrder eventLink;
    class edgeToKeycloak,catalogToKeycloak,orderToKeycloak identityLink;
    class configRepoToConfig,configToCatalog,configToOrder configurationLink;
    class catalogToTelemetry,orderToTelemetry,dispatcherToTelemetry,edgeToTelemetry,configToTelemetry,uiToTelemetry telemetryLink;
    class quoteToKnative,quoteFunctionToKnative serverlessLink;
    class deploymentToEdge,deploymentToUi,deploymentToCatalog,deploymentToOrder,deploymentToDispatcher,deploymentToConfig,deploymentToPostgres,deploymentToRabbit,deploymentToRedis,deploymentToKeycloak,deploymentToTelemetry provisioningLink;

    click deployment href "https://github.com/fResult-PolarBookshop/polar-deployment" "Open polar-deployment on GitHub" _blank
    %% click edge href "https://github.com/fResult-PolarBookshop/edge-service" "Open edge-service on GitHub" _blank
    %% click ui href "https://github.com/fResult-PolarBookshop/polar-ui" "Open polar-ui on GitHub" _blank
    click catalog href "https://github.com/fResult-PolarBookshop/catalog-service" "Open catalog-service on GitHub" _blank
    click order href "https://github.com/fResult-PolarBookshop/order-service" "Open order-service on GitHub" _blank
    %% click dispatcher href "https://github.com/fResult-PolarBookshop/dispatcher-service" "Open dispatcher-service on GitHub" _blank
    click config href "https://github.com/fResult-PolarBookshop/config-service" "Open config-service on GitHub" _blank
    click configRepo href "https://github.com/fResult-PolarBookshop/config-repo" "Open config-repo on GitHub" _blank
    %% click quote href "https://github.com/fResult-PolarBookshop/quote-service" "Open quote-service on GitHub" _blank
    %% click quoteFunction href "https://github.com/fResult-PolarBookshop/quote-function" "Open quote-function on GitHub" _blank
```


## Implementation approach

The book provides the architectural direction, tooling, and learning path.\
Each repository is created from scratch and can deliberately differ from the book when newer, more suitable, or more maintainable choices are available.

Examples of intentional differences may include:

- Current Java, Spring Boot, dependency, container, and GitHub Actions versions rather than the versions published with the book.
- Kotlin in selected services, alongside Java where appropriate.
- Immutable data structures and functional patterns, with [Vavr][vavr] as a primary library where appropriate.
- Alternative designs and implementation details while preserving the component's learning objective.

Repository documentation is the source of truth for the implementation, prerequisites, supported commands, APIs, and deployment instructions of that component.

## Related projects

- [Official book source code][official-source]
- [Polar Bookshop UI reference][polar-ui-reference]
- [My chapter-by-chapter learning repository][learning-repo]

[book]: https://www.manning.com/books/cloud-native-spring-in-action
[official-source]: https://github.com/ThomasVitale/cloud-native-spring-in-action
[learning-repo]: https://github.com/fResult/cloud-native-spring-in-action
[polar-ui-reference]: https://github.com/PolarBookshop/polar-ui/tree/v1
[polarbookshop-reference]: https://github.com/PolarBookshop
[catalog-service]: https://github.com/fResult-PolarBookshop/catalog-service
[order-service]: https://github.com/fResult-PolarBookshop/order-service
[config-service]: https://github.com/fResult-PolarBookshop/config-service
[config-repo]: https://github.com/fResult-PolarBookshop/config-repo
[polar-deployment]: https://github.com/fResult-PolarBookshop/polar-deployment
[vavr]: https://vavr.io/

---

<footer>
  <div align=center>
    <br><br>.<br><br>.<br><br>.<br><br>.<br><br>.<br><br>.<br><br>.<br><br>.<br><br>.<br><br>.<br><br>.<br><br>.<br><br>.<br><br>.<br><br>
  </div>

  <p align=center>
    [This Space Intentionally Left Blank]
  </p>

  <p align=center>
    The bottom of every page is padded so readers can maintain a consistent eyeline.
  </p>
</footer>
