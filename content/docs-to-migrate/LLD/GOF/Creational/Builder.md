---
weight: 400
---
# GOF Design Patterns : Builder

![](/images/Screenshot%202025-02-13%20080533.png)

## Introduction

Let's say we want an HttpRequest object, which requires many parameters and configurations. Also, based on what the user builds \(GetRequest / PostRequest\), the parameters also differ. So, it is not a good decision to let the user build this object themselves \(we can make constructors private to avoid this\). Instead, we can guide the user through a structured process using a class \(Builder Class\). The pattern we are discussing is called the Builder Design Pattern, a creational design pattern that helps users build complex objects using a builder class.

## Example

Let's try to understand the concept through a coding example where we build Get and Post Requests.

```java
abstract class HttpRequest {
    protected final String url;
    protected final Map<String, String> headers;
    protected final int timeout;

    protected HttpRequest(String url, Map<String, String> headers, int timeout) {
        this.url = url;
        this.headers = headers;
        this.timeout = timeout;
    }

    public abstract void send();
}
```

First, we created an abstract HttpRequest class:

- With common attributes that Get/Post requests may have.
- The constructor is protected to ensure it is only accessible through child classes.
- The abstract send method forces child classes to implement it for their specific use case.

Now, let's create a child class of HttpRequest called GetRequest as follows:
```java
class GetRequest extends HttpRequest {

    // NOTE 1
    private GetRequest(String url, Map<String, String> headers, int timeout) {
        super(url, headers, timeout);
    }

    // NOTE 2
    @Override
    public void send() {
        System.out.println("Sending GET request to " + url);
        System.out.println("Headers: " + headers);
        System.out.println("Timeout: " + timeout + "ms");
    }

    // NOTE 3
    public static class Builder {
        private String url;
        private Map<String, String> headers = new HashMap<>();
        private int timeout = 5000; // default

        // NOTE 4
        public Builder setUrl(String url) {
            this.url = url;
            return this;
        }

        public Builder addHeader(String key, String value) {
            this.headers.put(key, value);
            return this;
        }

        public Builder setTimeout(int timeout) {
            this.timeout = timeout;
            return this;
        }

        // NOTE 5
        public GetRequest build() {
            if (url == null || url.isEmpty()) {
                throw new IllegalStateException("URL is required for GET request");
            }
            return new GetRequest(url, headers, timeout);
        }
    }
}
```
- Note 1: We created the GetRequest class and made the constructor private to prevent direct instantiation without using the Builder.
- Note 2: We overrode and implemented the send method in a way that uniquely handles GET requests.
- Note 3: We created a static nested class for Builder and copied all the arguments from the outer class to the Builder class.
- Note 4: We provided setter methods that users can use to set values as preferred.
- Note 5: Finally, we have a build\(\) method, where we validate parameters before creating a new object.

Now, let's create a child class of HttpRequest called PostRequest as follows:
```java
class PostRequest extends HttpRequest {
    private final String body; // A parameter specific to POST

    // NOTE 1
    private PostRequest(String url, Map<String, String> headers, int timeout, String body) {
        super(url, headers, timeout);
        this.body = body;
    }

    // NOTE 2
    @Override
    public void send() {
        System.out.println("Sending POST request to " + url);
        System.out.println("Headers: " + headers);
        System.out.println("Timeout: " + timeout + "ms");
        System.out.println("Body: " + body);
    }

    // NOTE 3
    public static class Builder {
        private String url;
        private Map<String, String> headers = new HashMap<>();
        private int timeout = 5000; // Default timeout
        private String body;

        // NOTE 4
        public Builder setUrl(String url) {
            this.url = url;
            return this;
        }

        public Builder addHeader(String key, String value) {
            this.headers.put(key, value);
            return this;
        }

        public Builder setTimeout(int timeout) {
            this.timeout = timeout;
            return this;
        }

        public Builder setBody(String body) {
            this.body = body;
            return this;
        }

        // NOTE 5
        public PostRequest build() {
            if (url == null || url.isEmpty()) {
                throw new IllegalStateException("URL is required for POST request");
            }
            return new PostRequest(url, headers, timeout, body);
        }
    }
}
```
- Note 1: We created the PostRequest class and made the constructor private to prevent direct instantiation without using the Builder.
- Note 2: We overrode and implemented the send method in a way that uniquely handles POST requests.
- Note 3: We created a static nested class for Builder and copied all the arguments from the outer class to the Builder class.
- Note 4: We provided setter methods that users can use to set values as preferred.
- Note 5: Finally, we have a build\(\) method, where we validate parameters before creating a new object.

Now, let's create HttpRequestBuilder:
```java
class HttpRequestBuilder {
    public static GetRequest.Builder createGet() {
        return new GetRequest.Builder();
    }

    public static PostRequest.Builder createPost() {
        return new PostRequest.Builder();
    }
}
```
Clients can use this to select available builders and proceed to create objects.
Now, let's put everything together:
```java
public class BuilderPatternExample {
    public static void main(String[] args) {
        // Create a GET request
        GetRequest getRequest = HttpRequestBuilder.createGet()
                .setUrl("https://api.example.com/data")
                .addHeader("Authorization", "Bearer token123")
                .setTimeout(3000)
                .build();

        getRequest.send();

        // Create a POST request
        PostRequest postRequest = HttpRequestBuilder.createPost()
                .setUrl("https://api.example.com/submit")
                .addHeader("Content-Type", "application/json")
                .setTimeout(5000)
                .setBody("{ \"name\": \"Test User\", \"age\": 30 }")
                .build();

        postRequest.send();
    }
}
```
## Main Benefits
- The user is guided through the process, ensuring that complex objects are created correctly.
- It encapsulates and hides internal logic, making things simpler since the user does not need to worry about what happens internally while creating the object.

Yeah, that's how the Builder Pattern works\!\
Happy Coding 🙌
