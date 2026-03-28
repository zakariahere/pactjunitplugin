# Pact JUnit Plugin for Claude Code

A Claude Code plugin that generates production-ready **Pact contract tests for Java (JUnit 5)** — interactively or in one shot.

## What It Does

Contract testing with [Pact](https://docs.pact.io/) requires a lot of boilerplate: annotations, DSL chains, matchers, provider states, and Maven config. This plugin handles all of it. Give it an OpenAPI spec or describe your API, and it produces complete, compilable test classes you can drop straight into your project.

## Commands

| Command | Description |
|---|---|
| `/pact` | Interactive 4-phase workflow: discover → design interactions → generate → review |
| `/pact-consumer` | Quick consumer-side Pact test from a description or OpenAPI spec |
| `/pact-provider` | Quick provider verification test scaffold |
| `/pact-from-spec` | Parse an OpenAPI spec and generate all consumer tests in one shot |

## Installation

```bash
# Install from GitHub
claude plugin install zakariahere/pactjunitplugin
```

Or clone and install locally:

```bash
git clone https://github.com/zakariahere/pactjunitplugin
cd pactjunitplugin
claude plugin install .
```

## Quick Start

**One-shot from an OpenAPI spec:**
```
/pact-from-spec ./openapi.yaml
```

**Interactive workflow:**
```
/pact
```
The plugin walks you through 4 phases:
1. **Discovery** — consumer/provider names, endpoints, provider states, test framework
2. **Interaction Design** — request/response shapes, matchers, field types
3. **Generation** — complete Java test classes with correct imports and DSL
4. **Review** — verify the output and iterate

**Quick consumer test:**
```
/pact-consumer POST /orders returns 201 with order ID
```

**Quick provider verification:**
```
/pact-provider OrderService running on localhost:8080
```

## What Gets Generated

### Consumer Test (JUnit 5)

```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "OrderService")
class OrderConsumerPactTest {

    @Pact(consumer = "CheckoutUI")
    public RequestResponsePact createOrder(PactDslWithProvider builder) {
        return builder
            .given("products are in stock")
            .uponReceiving("a request to create an order")
                .method("POST")
                .path("/orders")
                .body(new PactDslJsonBody()
                    .stringType("productId", "abc-123")
                    .integerType("quantity", 2))
            .willRespondWith()
                .status(201)
                .body(new PactDslJsonBody()
                    .uuid("orderId")
                    .stringMatcher("status", "CREATED|PENDING", "CREATED"))
            .toPact();
    }

    @Test
    @PactTestFor(pactMethod = "createOrder")
    void testCreateOrder(MockServer mockServer) {
        // your test logic using mockServer.getUrl()
    }
}
```

### Provider Verification

```java
@Provider("OrderService")
@Consumer("CheckoutUI")
@PactFolder("pacts")
@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class OrderProviderPactTest {

    @LocalServerPort
    int port;

    @BeforeEach
    void setup(PactVerificationContext context) {
        context.setTarget(new HttpTestTarget("localhost", port));
    }

    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void verifyPact(PactVerificationContext context) {
        context.verifyInteraction();
    }

    @State("products are in stock")
    void productsInStock() {
        // seed test data
    }
}
```

## Reference Material

The plugin ships with detailed reference docs used during generation:

- `references/dsl/body-builders.md` — `PactDslJsonBody`, arrays, nested objects
- `references/dsl/matchers.md` — type, regex, date, UUID, equality matchers
- `references/dsl/interactions.md` — request/response patterns
- `references/openapi-mapping.md` — OpenAPI schema types → Pact DSL mapping
- `references/state-management.md` — provider state patterns

## Requirements

- [Claude Code](https://claude.ai/code) CLI
- Java project using `au.com.dius.pact` (`pact-jvm` 4.x)
- JUnit 5

## License

[Apache-2.0](LICENSE)
