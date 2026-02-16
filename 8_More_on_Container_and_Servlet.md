# Spring MVC Request Flow
This application follows the standard Spring Boot web request lifecycle using an embedded Apache Tomcat servlet container.

```mermaid
graph LR
    subgraph Client_Side [Client]
        A[Browser/Client]
    end

    subgraph Tomcat_Container [Tomcat Web Server]
        B{DispatcherServlet}
    end

    subgraph Spring_Application_Context [Spring Boot App]
        C[Handler Mapping]
        D[Controller]
        E[View Resolver]
    end

    %% Flow of Request
    A -- "1. HTTP Request" --> B
    B -- "2. Interrogates" --> C
    C -- "3. Returns Handler" --> B
    B -- "4. Invokes Method" --> D
    D -- "5. Returns View Name/Data" --> B
    B -- "6. Resolves View" --> E
    E -- "7. Renders Template" --> B
    B -- "8. HTTP Response" --> A

    %% Styling
    style B fill:#f9f,stroke:#333,stroke-width:2px
    style D fill:#bbf,stroke:#333,stroke-width:2px
    style Spring_Application_Context fill:#f5f5f5,stroke:#666,stroke-dasharray: 5 5
```

### 1. Request Initiation
The client (browser or API tool) sends an HTTP Request (GET, POST, etc.) to the server.

### 2. Front Controller (DispatcherServlet)
Tomcat receives the request and passes it to the DispatcherServlet. This acts as the Front Controller, centralizing all incoming traffic to manage the flow of the application.

### 3. Handler Mapping
The DispatcherServlet consults the HandlerMapping. It looks at the URL path and HTTP method to determine which specific @Controller and method (handler) should process the request.

### 4. Controller Execution
The DispatcherServlet dispatches the request to the identified Controller.

The Controller executes the business logic.

It returns a ModelAndView object (containing the data model and the logical view name).

### 5. View Resolution
The ViewResolver takes the logical view name (e.g., "dashboard") and resolves it to a physical resource (e.g., /WEB-INF/views/dashboard.jsp or a Thymeleaf template).

### 6. View Rendering
The DispatcherServlet renders the view by merging the model data with the template. The final HTTP Response is then sent back through Tomcat.

### 7. Client Presentation
The browser receives the response and renders the HTML/JSON data for the user.
