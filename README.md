# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [✔] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [✔] Commit: `Create Subscriber model struct.`
    -   [✔] Commit: `Create Notification model struct.`
    -   [✔] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [✔] Commit: `Implement add function in Subscriber repository.`
    -   [✔] Commit: `Implement list_all function in Subscriber repository.`
    -   [✔] Commit: `Implement delete function in Subscriber repository.`
    -   [✔] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [✔] Commit: `Create Notification service struct skeleton.`
    -   [✔] Commit: `Implement subscribe function in Notification service.`
    -   [✔] Commit: `Implement subscribe function in Notification controller.`
    -   [✔] Commit: `Implement unsubscribe function in Notification service.`
    -   [✔] Commit: `Implement unsubscribe function in Notification controller.`
    -   [✔] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [✔] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [✔] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [✔] Commit: `Implement publish function in Program service and Program controller.`
    -   [✔] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [✔] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber
is defined as an interface. Explain based on your understanding of Observer design patterns,
do we still need an interface (or trait in Rust) in this BambangShop case, or a single Model
struct is enough?

In Head First Design Patterns book, the Observer is an interface because the Publisher needs to call an update() method on local objects. Using an interface allows the Publisher to be connected, it doesn't care what class the observer is, as long as it implements update().

However, in Bambangshop, the Observers (Subscribers) aren't local objects sitting in the same memory space. They are completely separate web applications running on different ports. Our Publisher communicates over the network via HTTP POST requests. Therefore, we don't need a trait in Rust to handle different object types locally. We only need a simple data container (a struct) to hold the information required to make that web request.

2. id in Program and url in Subscriber is intended to be unique. Explain based on your
understanding, is using Vec (list) sufficient or using DashMap (map/dictionary) like we currently
use is necessary for this case?

By using Vec (list), it means that everytime we need to find the unique url, we need to iterate through the list one by one which will result in a compleixity of O(n). On th other hand, by using DashMap (map/dictionary), the lookup, insertion and deletion will all only take O(1) operations which is drastically faster. For this case, using DashMap will be better.

3. When programming using Rust, we are enforced by rigorous compiler constraints to make a
thread-safe program. In the case of the List of Subscribers (SUBSCRIBERS) static variable, we
used the DashMap external library for thread safe HashMap. Explain based on your
understanding of design patterns, do we still need DashMap or we can implement Singleton
pattern instead?

The Singleton pattern ensures that a class has only one instance and provides a global point of access to it. In our code, we are implementing a Singleton pattern to create a single, globally accessible SUBSCRIBERS database.

However, simply making something a Singleton does not make it thread-safe. If multiple requests try to add or remove subscribers from our Singleton at the exact same time, it will cause a data race. We need DashMap inside our Singleton because it acts like a standard HashMap wrapped in concurrent locking mechanisms. It ensures that multiple threads can read and write to our Singleton safely.

#### Reflection Publisher-2

1. In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”.
Model in MVC covers both data storage and business logic. Explain based on your
understanding of design principles, why we need to separate “Service” and “Repository” from
a Model?

If we let only Model to handle data structure, logic and accessing database, Model will be very big and complex. To resolve it, we apply the Single Responsibility Principle and Separation of Concerns so Model will only handle the data structure, repository can handle the interaction with database, service will handle the logic. With this, our code will be easier to be read and maintain.

2. What happens if we only use the Model? Explain your imagination on how the interactions
between each model (Program, Subscriber, Notification) affect the code complexity for
each model?

If every architecture revolves around Model, it will result in high complexity for the code. If we only used Models in the Bambangshop, the Product model would have to handle the entire workflow whenever a new product is created. It would need the internal logic to search through the entire database of subscribers, construct the Notification object itself, and manually execute HTTP POST requests to each subscriber's URL. This forces the Model to understand every networking details and the internal storage mechanisms of entirely different domains. If we ever wanted to change how notifications are delivered, we would have to rewrite the core Product model, which directly violates good software design principles.

3. Have you explored more about Postman? Tell us how this tool helps you to test your current
work. You might want to also list which features in Postman you are interested in or feel like it
is helpful to help your Group Project or any of your future software engineering projects.

Postman is a tool that lets us test REST API endpoints directly without needing to build a frontend first. For future collaborative work like Group Projects, its most helpful features include Collections and Workspaces for easily sharing API documentation with my teammate, Environment Variables for quickly switching between local and production servers without rewriting URLs, and Automated Tests to easily verify that our endpoints return the correct status codes and data structures.

#### Reflection Publisher-3

1. Observer Pattern has two variations: Push model (publisher pushes data to subscribers) and
Pull model (subscribers pull data from publisher). In this tutorial case, which variation of
Observer Pattern that we use?

In this tutorial, we are using the Push model. The Publisher actively pushes the data directly to the Subscribers via an HTTP POST request as soon as an event occurs.

2. What are the advantages and disadvantages of using the other variation of Observer Pattern
for this tutorial case? (example: if you answer Q1 with Push, then imagine if we used Pull)

If we used the Pull model, the Publisher would only notify the Subscribers that an event happened, and the Subscribers would then have to make a separate HTTP GET request back to the Publisher to fetch the actual product details.

Advantage: The Subscriber only pulls the exact data it needs. If the Publisher sends a massive payload but the Subscriber only needs the title, the Pull model saves bandwidth.

Disadvantage: It is highly inefficient for this specific REST API scenario. It requires two network requests instead of just one. This increases latency and server load on the Publisher, as it has to handle incoming requests immediately after sending out the pings.

3. Explain what will happen to the program if we decide to not use multi-threading in the
notification process.

If we don't use multi-threading, the Publisher would have to send the HTTP POST requests sequentially. It would wait for Subscriber A to receive the notification and respond before it attempts to send the notification to Subscriber B. If one subscriber's server is slow, offline, or times out, the entire notification process stop. Multi-threading allows the Publisher to send notifications concurrently without waiting, making the system significantly faster and more resilient.

