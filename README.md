# API Alerts • Java

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

[API Alerts](https://apialerts.com) • [GitHub](https://github.com/apialerts)

---

Hey Java developer 👋

There's no separate `apialerts-java` artifact. But you don't need one — the **[Kotlin Multiplatform SDK](https://github.com/apialerts/apialerts-kotlin)** works directly from Java, and it's on Maven Central.

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.apialerts</groupId>
    <artifactId>client</artifactId>
    <version>2.0.0</version>
</dependency>
```

```groovy
// build.gradle
implementation 'com.apialerts:client:2.0.0'
```

## Usage from Java

Use `EventBuilder` for a refactor-safe, named-field experience:

```java
import com.apialerts.client.ApiAlerts;
import com.apialerts.client.EventBuilder;

ApiAlerts.configure("your-api-key", false);

// message only
ApiAlerts.send(new EventBuilder("Deploy complete").build());

// with optional fields
ApiAlerts.send(new EventBuilder("Deploy complete")
    .channel("releases")
    .event("ci.deploy")
    .title("Deployed")
    .tags(List.of("CI/CD", "Java"))
    .build());
```


## Awaitable sends from Java

Use `ApiAlertsJvm.sendFuture()` for a proper `CompletableFuture`-based API:

```java
import com.apialerts.client.ApiAlertsJvm;

ApiAlertsJvm.sendFuture(new EventBuilder("Deploy complete").build())
    .thenAccept(result ->
        System.out.println("Sent to " + result.getWorkspace() + " (" + result.getChannel() + ")")
    );

// Or block synchronously if needed
SendResult result = ApiAlertsJvm.sendFuture(new EventBuilder("Deploy complete").build()).get();
```

Yes, you still need a Kotlin dependency in your `pom.xml`. We still love you.

## The honest answer

If you're in a **pure Java** project without Kotlin, the interop is functional but not pretty. The recommended paths are:

- **Kotlin Multiplatform project** — use the SDK natively, it's what it's designed for
- **Mixed Java/Kotlin project** — add the dependency, the fire-and-forget `send()` works cleanly
- **Pure Java backend** — the interop above works, or you can call the API directly with a single HTTP POST (it's really simple):

```bash
curl -X POST https://api.apialerts.com/event \
  -H "Authorization: Bearer your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"message": "Deploy complete"}'
```

A dedicated Java SDK with `CompletableFuture`-based API is on the roadmap. Until then, the Kotlin SDK is your best bet.

## Links

- [Kotlin SDK](https://github.com/apialerts/apialerts-kotlin) — the real thing
- [API Reference](https://apialerts.com/docs) — direct HTTP if you prefer
- [Sign up](https://apialerts.com)
