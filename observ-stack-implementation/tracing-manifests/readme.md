## 🕵️‍♂️ What is Jaeger?
- Jaeger is an open-source, end-to-end distributed tracing system used for monitoring and troubleshooting microservices-based architectures. It helps developers understand how requests flow through a complex system, by tracing the path a request takes and measuring how long each step in that path takes.

## ❓ Why Use Jaeger?
- In modern applications, especially microservices architectures, a single user request can touch multiple services. When something goes wrong, it’s challenging to pinpoint the source of the problem. Jaeger helps by:

- 🐢 **Identifying bottlenecks**: See where your application spends most of its time.
- 🔍 **Finding root causes of errors**: Trace errors back to their source.
- ⚡ **Optimizing performance**: Understand and improve the latency of services.


## 📚 Core Concepts of Jaeger

- 🛤️ **Trace**: A trace represents the journey of a request as it travels through various services. Think of it as a detailed map that shows every stop a request makes in your system.
- 📏 **Span**: Each trace is made up of multiple spans. A span is a single operation within a trace, such as an API call or a database query. It has a start time and a duration.
- 🏷️ **Tags**: Tags are key-value pairs that provide additional context about a span. For example, a tag might indicate the HTTP method used (GET, POST) or the status code returned.
- 📝 **Logs**: Logs in a span provide details about what’s happening during that operation. They can capture events like errors or important checkpoints.
- 🔗 **Context Propagation**: For Jaeger to trace requests across services, it needs to propagate context. This means each service in the call chain passes along the trace information to the next service.

# 🏠 Architecture
![Project Architecture](images/architecture.gif)



