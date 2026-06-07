# API Alerts • Java

[![Maven Central](https://img.shields.io/maven-central/v/com.apialerts/client)](https://central.sonatype.com/artifact/com.apialerts/client)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

[Maven Central](https://central.sonatype.com/artifact/com.apialerts/client) • [GitHub](https://github.com/apialerts) • [API Alerts](https://apialerts.com)

---

Hey Java developer 👋

There's no separate `apialerts-java` artifact. But you don't need one — the **[Kotlin Multiplatform SDK](https://github.com/apialerts/apialerts-kotlin)** works directly from Java, and it's on Maven Central.

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.apialerts</groupId>
    <artifactId>client</artifactId>
    <version>1.1.0</version>
</dependency>
```

```groovy
// build.gradle
implementation 'com.apialerts:client:1.1.0'
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

## Dependency injection (Spring)

Construct an injectable `ApiAlertsClient` with the `ApiAlertsJvm.client(...)` factory and register it as a bean:

```java
import com.apialerts.client.ApiAlertsClient;
import com.apialerts.client.ApiAlertsJvm;

@Bean
ApiAlertsClient apiAlerts() {
    return ApiAlertsJvm.client("your-api-key");
}
```

Then inject it and send:

```java
class DeployNotifier {
    private final ApiAlertsClient alerts;

    DeployNotifier(ApiAlertsClient alerts) { this.alerts = alerts; }

    void onDeploy() {
        // fire-and-forget
        alerts.send(new EventBuilder("Deploy complete").build(), null);
        // awaitable (sendAsync is a Kotlin suspend fn, so use the bridge)
        ApiAlertsJvm.sendFuture(alerts, new EventBuilder("Deploy complete").build());
    }
}
```

## The honest answer

The Java surface is small but complete: `EventBuilder`, static `send`, the `CompletableFuture` bridge, and an injectable client. Recommended paths:

- **Kotlin Multiplatform project** — use the SDK natively, it's what it's designed for
- **Mixed Java/Kotlin project** — add the dependency, everything above works cleanly
- **Pure Java backend** — the interop above works, or you can call the API directly with a single HTTP POST (it's really simple):

```bash
curl -X POST https://api.apialerts.com/event \
  -H "Authorization: Bearer your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"message": "Deploy complete"}'
```

The `CompletableFuture` API and an injectable client now ship on the Kotlin Multiplatform artifact, so a separate Java package isn't needed - this is the Java path.

## Links

- [Kotlin SDK](https://github.com/apialerts/apialerts-kotlin) — the real thing
- [API Reference](https://apialerts.com/docs) — direct HTTP if you prefer
- [Sign up](https://apialerts.com)
