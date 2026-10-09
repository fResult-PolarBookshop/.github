# Polar Bookshop

A personal, repository-per-component implementation inspired by the Polar Bookshop system in [Cloud Native Spring in Action][book].

This organization contains independently versioned services, shared runtime configuration, and environment deployment definitions.\
It is a hands-on learning project, not an official implementation or a direct fork of the book source code.

> [!NOTE]
> For the chapter-by-chapter learning workspace, see [fResult/cloud-native-spring-in-action][learning-repo].

## System repositories

The responsibilities below describe the intended role of each component in the Polar Bookshop architecture.\
They are a guide derived from the book's final project—not a promise to reproduce its code, design, dependencies, or release versions exactly.

| Repository           | Responsibility                                                                                                              |  Status   |
|----------------------|-----------------------------------------------------------------------------------------------------------------------------|:---------:|
| [catalog-service]    | REST API for managing the book catalog, with PostgreSQL persistence.                                                        | Available |
| [order-service]      | Reactive API for placing book orders, storing them in PostgreSQL, and tracking their status.                                | Available |
| [config-service]     | Configuration server that serves application configuration from Git.                                                        | Available |
| [config-repo]        | Version-controlled, externalized application configuration consumed by Config Service.                                      | Available |
| [polar-deployment]   | Environment setup and deployment configuration, including local and Kubernetes resources.                                   | Available |
| `edge-service`       | API gateway for routing requests, authentication, rate limiting, and circuit breakers.                                      |  Planned  |
| `dispatcher-service` | Processes accepted-order events and publishes dispatch notifications through RabbitMQ.                                      |  Planned  |
| `polar-ui`           | Frontend for browsing and managing books, and placing and viewing orders.                                                   |  Planned  |
| `quote-service`      | Reactive REST API for retrieving book quotes, including random quotes by genre.                                             |  Planned  |
| `quote-function`     | API gateway for routing requests and cross-cutting concerns, including authentication, rate limiting, and circuit breakers. |  Planned  |

## Target architecture

The diagram shows the target topology for this organization.\
It is informed by the final Polar Bookshop project from the book and the repositories published by the [PolarBookshop reference organization][polarbookshop-reference].\
It includes both available and planned repositories; it does not imply that every component is already deployed or that this implementation will mirror the reference code, design, dependencies, or release versions exactly.

`polar-deployment` provides the runtime environment around the services—for example, Docker Compose, Kubernetes resources, backing services, and delivery configuration.\
Each application repository owns its source code, tests, container build, and CI workflow.

```mermaid
%% Polar Bookshop — target topology, clarified from the reference implementation.
%% polar-ui serves SPA assets only; once loaded, its Angular code runs in the Browser.
%%{init: {'theme':'base','securityLevel':'loose','themeVariables':{'darkMode':true,'background':'#333333','primaryColor':'#FFFFFF','primaryTextColor':'#0F172A','primaryBorderColor':'#CBD5E1','secondaryColor':'#F8FAFC','secondaryTextColor':'#0F172A','tertiaryColor':'#F8FAFC','tertiaryTextColor':'#0F172A','lineColor':'#CBD5E1','textColor':'#F8FAFC','edgeLabelBackground':'#F8FAFC','nodeTextColor':'#0F172A','fontSize':'16px'},'themeCSS':'svg { background-color: #333333 !important; } .edgeLabel, .edgeLabel p { background-color: #F8FAFC !important; color: #0F172A !important; }'}}%%
flowchart TB
    browser[Browser]
    edge[edge-service<br/>API gateway / BFF]
    ui[polar-ui<br/>Angular SPA assets]

    %% The initial document/assets pass through the gateway; edge reverse-proxies polar-ui.
    browser -->|GET /, JS, CSS, favicon| edge
    edge -->|reverse proxy: SPA/static assets| ui


    %% After the SPA is loaded, the Browser—not polar-ui's server—makes these requests.
    browser -->|XHR/fetch: /books, /orders, /user<br/>OAuth2 login/logout| edge
    edge -->|/books/**; TokenRelay| catalog[catalog-service]
    edge -->|/orders/**; TokenRelay| order[order-service]

    %% Internal synchronous call; it does not go back through the gateway.
    order -->|checks catalog directly| catalog

    catalog -->|polardb_catalog| postgres[(PostgreSQL<br/>one instance; two logical databases)]
    order -->|polardb_order| postgres
    order -->|publishes order-accepted| rabbit[RabbitMQ]
    rabbit -->|consumes order-accepted| dispatcher[dispatcher-service]
    dispatcher -->|publishes order-dispatched| rabbit
    rabbit -->|consumes order-dispatched| order

    edge -->|session store / rate limiting| redis[(Redis)]
    edge -->|OAuth2 client / session| keycloak[Keycloak]
    catalog -->|JWT issuer/JWK discovery| keycloak
    order -->|JWT issuer/JWK discovery| keycloak

    %% Target dependency only: reference application configs currently disable Config Client.
    configRepo[config-repo<br/>catalog configuration] -->|Git backend| config[config-service]
    config -.->|planned externalized config| catalog

    catalog --> telemetry[Observability stack]
    order --> telemetry
    dispatcher --> telemetry
    edge --> telemetry
    config --> telemetry
    ui -->|container logs| telemetry

    %% Independent Chapter 16 serverless examples.
    quote[quote-service] -. "knative/kservice.yml" .-> knative[Knative Serving platform]
    quoteFunction[quote-function<br/>Spring Cloud Function] -. "knative/kservice.yml" .-> knative

    deployment[polar-deployment] -. deploys & provisions .-> edge
    deployment -. deploys & provisions .-> ui
    deployment -. deploys & provisions .-> catalog
    deployment -. deploys & provisions .-> order
    deployment -. deploys & provisions .-> dispatcher
    deployment -. deploys & provisions .-> config
    deployment -. provisions .-> postgres
    deployment -. provisions .-> rabbit
    deployment -. provisions .-> redis
    deployment -. provisions .-> keycloak
    deployment -. provisions .-> telemetry

    %% Fixed #333333 canvas: consistent in light and dark mode.
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
    class config planned;

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

    %% Blue HTTP, purple internal service call, green data, orange events, pink identity,
    %% amber configuration, teal telemetry, purple serverless, and gray provisioning.
    linkStyle 0,1,2,3,4 stroke:#60A5FA,color:#1E3A8A,stroke-width:2.5px;
    linkStyle 5 stroke:#C4B5FD,color:#4C1D95,stroke-width:2.5px;
    linkStyle 6,7,12 stroke:#4ADE80,color:#14532D,stroke-width:2.5px;
    linkStyle 8,9,10,11 stroke:#FDBA74,color:#7C2D12,stroke-width:2.5px;
    linkStyle 13,14,15 stroke:#F9A8D4,color:#831843,stroke-width:2.5px;
    linkStyle 16,17 stroke:#FCD34D,color:#78350F,stroke-width:2.5px;
    linkStyle 18,19,20,21,22,23 stroke:#5EEAD4,color:#164E63,stroke-width:2.5px;
    linkStyle 24,25 stroke:#C4B5FD,color:#3B0764,stroke-width:2.5px;
    linkStyle 26,27,28,29,30,31,32,33,34,35,36 stroke:#CBD5E1,color:#0F172A,stroke-width:2px;
```


## Implementation approach

The book provides the architectural direction, tooling, and learning path.\
Each repository is created from scratch and can deliberately differ from the book when newer, more suitable, or more maintainable choices are available.

Examples of intentional differences may include:

- Current Java, Spring Boot, dependency, container, and GitHub Actions versions rather than the versions published with the book.
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
