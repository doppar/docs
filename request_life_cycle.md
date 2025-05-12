## Request Lifecycle
### Introduction
Understanding how a tool operates makes using it much easier and more intuitive. This principle applies just as much to application development as it does to any other tool in the real world. When you comprehend the inner workings of your development tools, you gain confidence and efficiency in using them.

This document aims to provide a high-level overview of how the Doppar framework functions. By familiarizing yourself with its core concepts, you’ll reduce the sense of mystery surrounding it and feel more empowered when building applications. Don’t worry if some of the terminology seems unfamiliar at first—focus on grasping the big picture. As you continue exploring the documentation, your understanding will naturally deepen over time.

### Here’s a Request Lifecycle Graph for the Doppar framework
```markdown
┌───────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                  REQUEST LIFECYCLE                                              │
├───────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                               │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐                  │
│  │  1. Entry Point │     │ 2. Bootstrapping│     │ 3. Middleware   │     │ 4. Routing &    │                  │
│  │  (index.php)    │────▶│   (Service      │────▶│   Stack         │────▶│   Controller    │                  │
│  │                 │     │   Providers)    │     │                 │     │   Execution     │                  │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘                  │
│         │                                                                                         │            │
│         │                                                                                         │            │
│         ▼                                                                                         ▼            │
│  ┌─────────────────┐                                                                 ┌─────────────────────────┐│
│  │ Web Server      │                                                                 │ Response Return &      ││
│  │ (Apache/Nginx)  │                                                                 │ Termination            ││
│  └─────────────────┘                                                                 └─────────────────────────┘│
│                                                                                                               │
│                                                                                                               │
│  ┌───────────────────────────────────────────────────────────────────────────────────────────────────────────┐│
│  │                                       MIDDLEWARE STACK (GLOBAL & ROUTE)                                   ││
│  └───────────────────────────────────────────────────────────────────────────────────────────────────────────┘│
│          ▲                                                                                         │          │
│          │                                                                                         │          │
│          └───────────────────────────────────────┐       ┌────────────────────────────────────────┘          │
│                                                  │       │                                                 │
│                                                  ▼       ▼                                                 │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐                  │
│  │ Request         │────▶│ Pre-Middleware   │────▶│ Route/Controller│────▶│ Post-Middleware │────▶ Response   │
│  │ (HTTP)          │     │ (Auth, CORS, etc)│     │ Processing      │     │ (Modify Response)│     (Send)     │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘                  │
│                                                                                                               │
└───────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

Let's see the unidirectional flow of a request in Doppar

```markdown
HTTP Request → index.php → Bootstrap → Middleware → Router → Controller → Response → HTTP Response
```
### Lifecycle Overview
#### First Steps

The entry point for all requests to a Doppar application is the public/index.php file. All requests are directed to this file by your web server (Apache / Nginx) configuration. The index.php file doesn't contain much code. Rather, it is a starting point for loading the rest of the framework.

The index.php file loads the Composer generated autoloader definition, and then retrieves an instance of the Doppar application from bootstrap/app.php. The first action taken by Doppar itself is to create an instance of the application service container.
#### Second Steps

At this stage, Doppar initializes its core configuration and loads the service providers along with their registered methods. Once the core setup is complete, Doppar proceeds to load and execute the boot methods of the registered service providers, ensuring that all necessary services are properly initialized and ready for use.
#### Thirds Steps

At this stage, Doppar generate global middleware stack. The method signature for the middleware's __invoke() method is quite simple: it receives a Request and Closer $next and send the Request and Response to the next Request. Feed it HTTP requests and it will return HTTP responses.
#### Final Steps

Once the application has been fully bootstrapped and all service providers have been registered, the request is passed to the router for processing. The router is responsible for directing the request to the appropriate route or controller while also executing any route-specific middleware.

Middleware acts as a powerful filtering mechanism for incoming HTTP requests, allowing the application to inspect, modify, or restrict access before reaching the intended destination. For instance, Doppar includes authentication middleware that verifies whether a user is logged in. If the user is not authenticated, they are redirected to the login screen; otherwise, the request proceeds as expected.

After the designated route or controller method processes the request and generates a response, the response begins its journey back through the middleware stack. This allows the application to inspect or modify the outgoing response before it reaches the user.

Finally, as the response completes its trip through the middleware, the __invoke method returns the response object, which then calls the send method. The send method delivers the response content to the user's web browser, marking the completion of Doppar’s request lifecycle.