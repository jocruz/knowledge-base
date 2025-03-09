# Fetch API

## Fetch API Documentation

### Overview

The **Fetch API** is a modern JavaScript interface for making HTTP requests. It provides a **promise-based** approach, replacing `XMLHttpRequest`, and integrates well with **service workers**, **CORS (Cross-Origin Resource Sharing)**, and **streaming responses**. The Fetch API simplifies asynchronous data retrieval while offering fine-grained control over requests and responses.

#### Key Characteristics:

* **Promise-Based:** Fetch returns a `Promise` that resolves to a `Response` object.
* **Flexible Request Handling:** Supports custom headers, authentication, and streaming.
* **Better Error Handling:** Differentiates between network failures and HTTP errors.
* **Works in Window and Worker Contexts:** Supports `fetch()` in web workers.

***

### Basic Request

#### Example: Fetching JSON Data

```javascript
async function fetchData() {
    const url = "https://example.com/data.json";
    try {
        const response = await fetch(url);
        if (!response.ok) {
            throw new Error(`HTTP Error: ${response.status}`);
        }
        const jsonData = await response.json();
        console.log(jsonData);
    } catch (error) {
        console.error("Fetch error:", error);
    }
}
```

#### Explanation:

* `fetch(url)`: Sends a request to the given URL.
* `response.ok`: Checks if the HTTP status code is **2xx** (success).
* `.json()`: Parses the response body as JSON.
* `try...catch`: Handles network errors and unsuccessful responses.

***

### HTTP Methods and Request Options

Fetch supports all standard HTTP methods, including `GET`, `POST`, `PUT`, `DELETE`, and `PATCH`.

#### Example: Sending a `POST` Request

```javascript
fetch("https://example.com/api", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({ username: "JohnDoe" })
});
```

#### Available Options

| Option        | Description                                                                         | Default       |
| ------------- | ----------------------------------------------------------------------------------- | ------------- |
| `method`      | HTTP method (GET, POST, PUT, DELETE, etc.)                                          | `GET`         |
| `headers`     | Custom headers for the request                                                      | `{}`          |
| `body`        | Data sent with `POST` or `PUT` requests                                             | `null`        |
| `credentials` | Defines whether to send cookies with the request (`omit`, `same-origin`, `include`) | `same-origin` |
| `cache`       | Caching strategy (`default`, `no-store`, `reload`, `force-cache`, `only-if-cached`) | `default`     |

***

### Handling Responses

#### Checking the Response Status

```javascript
const response = await fetch(url);
if (!response.ok) {
    throw new Error(`HTTP Error: ${response.status}`);
}
```

#### Parsing the Response

```javascript
const jsonData = await response.json(); // Parses JSON data  
const textData = await response.text(); // Reads response as plain text  
const blobData = await response.blob(); // Handles binary data  
```

***

### Streaming Responses

Large responses can be processed as streams to avoid memory issues.

```javascript
const response = await fetch(url);
const reader = response.body.getReader();
const decoder = new TextDecoder("utf-8");

reader.read().then(function processText({ done, value }) {
    if (done) return;
    console.log(decoder.decode(value));  
    return reader.read().then(processText);
});
```

***

### Setting Custom Headers

#### Example: Using Headers in Fetch

```javascript
const response = await fetch("https://example.com", {
    headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer token123"
    }
});
```

#### Using `Headers` Object

```javascript
const headers = new Headers();
headers.append("Content-Type", "application/json");
headers.append("Authorization", "Bearer token123");

fetch("https://example.com", { headers });
```

***

### Handling Authentication

#### Sending Requests with Cookies

```javascript
fetch("https://example.com/secure", {
    credentials: "include"
});
```

#### Explanation:

* `credentials: "include"` allows **cross-origin requests with cookies**.
* Useful for **authenticated API calls** and **session-based authentication**.

***

### Canceling Requests

Use `AbortController` to cancel fetch requests before they complete.

```javascript
const controller = new AbortController();
fetch(url, { signal: controller.signal });

// Cancel request
controller.abort();
```

***

### Handling CORS (Cross-Origin Resource Sharing)

CORS policies define how browsers handle **cross-origin requests**.

#### CORS Request Modes

| Mode          | Description                                           |
| ------------- | ----------------------------------------------------- |
| `cors`        | Allows cross-origin requests if the server permits.   |
| `same-origin` | Blocks cross-origin requests.                         |
| `no-cors`     | Limits headers and methods, used for opaque requests. |

#### Example: Making a Cross-Origin Request

```javascript
fetch("https://external-api.com", {
    mode: "cors"
});
```

***

### Cloning Requests and Responses

Both `Request` and `Response` objects are **one-time use** but can be cloned.

```javascript
const request = new Request("https://example.com", { method: "POST" });
const clonedRequest = request.clone();
fetch(clonedRequest);
```

```javascript
const response = await fetch(url);
const clonedResponse = response.clone();
const jsonData = await clonedResponse.json();
```

***

### Error Handling Best Practices

1.  **Check the Response Status**

    ```javascript
    if (!response.ok) throw new Error(`HTTP Error: ${response.status}`);
    ```
2.  **Handle Network Failures**

    ```javascript
    try {
        const response = await fetch(url);
    } catch (error) {
        console.error("Network error:", error);
    }
    ```
3.  **Use Timeouts to Avoid Hanging Requests**

    ```javascript
    const controller = new AbortController();
    setTimeout(() => controller.abort(), 5000); // 5-second timeout
    fetch(url, { signal: controller.signal });
    ```

***

### Security Considerations

| Issue                          | Mitigation                                                           |
| ------------------------------ | -------------------------------------------------------------------- |
| **Leaking credentials**        | Use `credentials: "same-origin"` unless necessary.                   |
| **Cross-Site Scripting (XSS)** | Always sanitize user inputs before inserting into fetch requests.    |
| **Cross-Origin Data Exposure** | Implement CORS headers carefully to prevent unauthorized API access. |

***

### Summary

The Fetch API provides a powerful and flexible approach for making HTTP requests in JavaScript. It improves **error handling, authentication, and response streaming** compared to `XMLHttpRequest`. Proper use of **headers, credentials, CORS policies, and response handling** is critical for security and performance.

***

### Additional Resources

* [MDN Fetch API Documentation](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
* [Understanding CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
* [Using Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
