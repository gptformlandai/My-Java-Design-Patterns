# Microservices Self-Registration Pattern

Category: Microservices and Distributed Systems Patterns  
MAANG interview meter: Medium  
Software usage meter: Medium  
Repository module: [github-repo/microservices-self-registration](../../github-repo/microservices-self-registration)

## How to Study This Page

Use this page in three passes:

1. First pass: understand why dynamic services must announce where they are.
2. Second pass: trace startup, registry registration, heartbeat, discovery, and deregistration.
3. Third pass: compare self-registration with third-party registration, DNS discovery, and Kubernetes services.

By the end, you should be able to say:

> Self-Registration lets a service instance register itself with a service registry so other services can discover it dynamically.

## 1. Technical Definition

Microservices Self-Registration is a service discovery pattern where each service instance registers its network location and health status with a registry when it starts.

Core idea:

- Service instance starts.
- It registers host, port, name, and metadata.
- It sends heartbeats to prove it is alive.
- Other services query the registry or use clients that query it.
- Dead instances are removed from discovery.

### 30-Second Interview Answer

I would use Self-Registration when service instances are dynamic and their addresses cannot be hardcoded. Each instance registers with a registry like Eureka, Consul, or etcd and sends heartbeats. Clients discover healthy instances through the registry. The trade-off is that service code now depends on registration behavior, so health checks, deregistration, and stale entries must be handled carefully.

## 2. Layman and Easy to Understand Definition

Self-Registration is like a hotel guest writing their room number at the front desk after check-in.

Anyone who needs to find the guest asks the front desk instead of guessing room numbers.

In software:

- Service starts.
- Service tells registry its address.
- Other services ask registry where it is.
- Registry removes it when it is unhealthy.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

In cloud environments, service instances change often:

- Containers restart.
- Pods move nodes.
- Autoscaling adds instances.
- Deployments replace old versions.
- IP addresses change.

Hardcoding addresses does not work.

### 3.2 The Self-Registration Solution

Use a registry:

```text
service instance -> register -> service registry
client -> discover -> service registry
client -> call -> service instance
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Service instance | Running copy of a service. |
| Registry | Stores live service locations. |
| Registration | Startup announcement. |
| Heartbeat | Periodic health signal. |
| Discovery client | Finds healthy instances. |
| Deregistration | Removal when instance stops or fails. |

### 3.4 Lifecycle

1. Service starts.
2. Service determines host, port, and metadata.
3. Service registers with registry.
4. Service sends periodic heartbeats.
5. Client discovers service by name.
6. Registry stops returning unhealthy instances.
7. Service deregisters on graceful shutdown.

## 4. Java Coding Example

This example shows the core mechanics without a framework.

```java
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

record ServiceInstance(String name, String host, int port) {
}

class ServiceRegistry {
    private final Map<String, ServiceInstance> instances = new ConcurrentHashMap<>();

    void register(ServiceInstance instance) {
        instances.put(instance.name(), instance);
    }

    ServiceInstance discover(String serviceName) {
        return instances.get(serviceName);
    }

    void deregister(String serviceName) {
        instances.remove(serviceName);
    }
}

class GreetingService {
    void start(ServiceRegistry registry) {
        registry.register(new ServiceInstance("greeting-service", "localhost", 8081));
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `ServiceInstance` | Service name and network location. |
| `ServiceRegistry` | Central discovery store. |
| `register` | Instance announces itself. |
| `discover` | Client finds an instance by service name. |
| `deregister` | Stale instance is removed. |

### Java Usage

```java
ServiceRegistry registry = new ServiceRegistry();
new GreetingService().start(registry);

ServiceInstance instance = registry.discover("greeting-service");
System.out.println(instance.host() + ":" + instance.port());
```

## 5. Python Coding Example

```python
class ServiceRegistry:
    def __init__(self):
        self.instances = {}

    def register(self, name, host, port):
        self.instances[name] = {"host": host, "port": port}

    def discover(self, name):
        return self.instances.get(name)

    def deregister(self, name):
        self.instances.pop(name, None)


registry = ServiceRegistry()
registry.register("greeting-service", "localhost", 8081)
print(registry.discover("greeting-service"))
```

### Python Usage

Use this shape when explaining:

- Registry maps logical service names to live locations.
- Real systems add heartbeats and health checks.
- Discovery removes hardcoded addresses.

## 6. Where It Comes Handy in Real Life

- Autoscaled microservices.
- Service discovery in Spring Cloud.
- Containerized environments.
- Blue/green deployments.
- Client-side load balancing.
- Dynamic internal service calls.

## 7. Advantages Over Normal Code Without Pattern

Without Self-Registration:

```text
clients use hardcoded host and port
```

With Self-Registration:

```text
clients discover live instances by service name
```

Benefits:

- Supports dynamic scaling.
- Removes hardcoded addresses.
- Enables health-aware routing.
- Improves resilience during restarts.
- Works with client-side load balancing.

## 8. Where It Excels

- Instances come and go frequently.
- Service addresses change.
- Multiple instances serve one service name.
- Clients need service discovery.
- Registry infrastructure is already available.

## 9. Where It Fails

- Registry becomes unavailable.
- Instances fail to deregister cleanly.
- Heartbeats are too slow and stale entries remain.
- Service code should not own infrastructure registration.
- Platform already provides discovery more safely.

In Kubernetes, native services often replace app-level self-registration.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java | Spring Cloud Netflix Eureka, Spring Cloud Consul, Spring Cloud Zookeeper |
| Registries | Eureka, Consul, ZooKeeper, etcd |
| Kubernetes | Services, Endpoints, EndpointSlices |
| Clients | OpenFeign, Spring Cloud LoadBalancer, Ribbon legacy |
| Health | Spring Boot Actuator health endpoints |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Enables dynamic service discovery. | Adds dependency on registry. |
| Supports autoscaling. | Stale registrations can route traffic badly. |
| Removes hardcoded addresses. | Service code handles registration concerns. |
| Works with health-aware routing. | Registry must be highly available. |

## 12. Real-World Identification Example

Scenario: Context service must call greeting service, but greeting instances scale up and down.

Self-Registration fit:

- Greeting instances register with Eureka.
- Context service calls by logical name.
- Discovery client resolves a healthy instance.
- New instances become available automatically.

Without it:

- Context service needs hardcoded addresses.
- Scaling creates manual config work.
- Dead instances may still receive traffic.

## 13. MAANG Interview Triggers

Use Self-Registration when you hear:

- "How do services find each other?"
- "Instances are dynamic."
- "We cannot hardcode IP addresses."
- "How does autoscaling affect service discovery?"
- "How do unhealthy instances leave rotation?"

Strong answer keywords:

- registry
- heartbeat
- health check
- service discovery
- deregistration
- client-side load balancing
- stale instance
- metadata

## 14. Common Mistakes

### Mistake 1: No heartbeat expiration

- Why it is wrong: dead instances remain discoverable.
- Better approach: expire registrations after missed heartbeats.

### Mistake 2: Hardcoding discovered addresses forever

- Why it is wrong: instances can move or disappear.
- Better approach: refresh discovery cache and respect TTLs.

### Mistake 3: Ignoring graceful shutdown

- Why it is wrong: traffic can route to an instance that is terminating.
- Better approach: deregister and drain before shutdown.

### Mistake 4: Using app-level registry when platform discovery is enough

- Why it is wrong: duplicates infrastructure complexity.
- Better approach: use Kubernetes services or platform-native discovery when available.

## 15. Self-Registration vs Similar Patterns

| Pattern | Difference |
|---|---|
| Self-Registration | Service instance registers itself. |
| Third-Party Registration | Platform or sidecar registers the instance. |
| DNS Discovery | Name resolution returns service endpoints. |
| API Gateway | Client-facing entry point, not service registry. |
| Client-Side Load Balancing | Uses registry data to choose an instance. |

## 16. Self-Registration Design Checklist

- What registry stores service instances?
- What metadata is registered?
- How often are heartbeats sent?
- When does a registration expire?
- How does graceful shutdown deregister?
- How do clients cache discovery results?
- Is registry highly available?
- Does the platform already solve this?

## 17. Quick Revision Notes

- One-line summary: Self-Registration lets services publish their live locations to a registry.
- Three keywords: registry, heartbeat, discovery.
- Interview trap: forgetting stale instance cleanup.
- Memory trick: each instance writes itself into the phonebook.

## 18. Mini Exercise

Design self-registration for a notification service.

Answer these:

1. What service name is registered?
2. What host, port, and metadata are stored?
3. What health check drives registration?
4. What happens on shutdown?
5. How do clients choose an instance?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/microservices-self-registration/README.md](../../github-repo/microservices-self-registration/README.md)
- [github-repo/microservices-self-registration/eurekaserver/src/main/java/com/learning/eurekaserver/EurekaserverApplication.java](../../github-repo/microservices-self-registration/eurekaserver/src/main/java/com/learning/eurekaserver/EurekaserverApplication.java)
- [github-repo/microservices-self-registration/greetingservice/src/main/java/com/learning/greetingservice/GreetingserviceApplication.java](../../github-repo/microservices-self-registration/greetingservice/src/main/java/com/learning/greetingservice/GreetingserviceApplication.java)
- [github-repo/microservices-self-registration/greetingservice/src/main/java/com/learning/greetingservice/controller/GreetingsController.java](../../github-repo/microservices-self-registration/greetingservice/src/main/java/com/learning/greetingservice/controller/GreetingsController.java)
- [github-repo/microservices-self-registration/contextservice/src/main/java/com/learning/contextservice/ContextserviceApplication.java](../../github-repo/microservices-self-registration/contextservice/src/main/java/com/learning/contextservice/ContextserviceApplication.java)
- [github-repo/microservices-self-registration/contextservice/src/main/java/com/learning/contextservice/client/GreetingServiceClient.java](../../github-repo/microservices-self-registration/contextservice/src/main/java/com/learning/contextservice/client/GreetingServiceClient.java)
- [github-repo/microservices-self-registration/contextservice/src/main/java/com/learning/contextservice/controller/ContextController.java](../../github-repo/microservices-self-registration/contextservice/src/main/java/com/learning/contextservice/controller/ContextController.java)
