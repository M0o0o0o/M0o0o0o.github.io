---
title:  "Spring Rest Docs란?"
categories: 스프링
toc: true
toc_sticky: true
toc_label: "Contents"
---

### Generaing Documentation Snippets

- Spring REST Docs uses Spring MVC’s test framework to make requests to the service that you are documenting.

### Setting up Your JUnit 5 Test

- When using JUnit 5, the first step is generating documentation snippets is to apply the RestDocumentationExtension to your test class. The following example shows how to do so :

```java
@ExtendWith(RestDocumentationExtension.class)
public class JUnit5ExampleTests { } 

@ExtendWith({RestDocumentationExtension.class, SpringExtension.class})
public class JUnit5ExampleTests { }
```

- Next, you must provide a @BeforeEach method to configure MockMVC.

```java
private MockMvc mockMvc;

@BeforeEach
void setUp(WebApplicationContext webApplicationContext, RestDocumentationContextProvider restDocumentation) {
	this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
			.apply(documentationConfiguration(restDocumentation)) (1)
			.build();
}
```

### Invoking the RESTful Service

```java
this.mockMvc.perform(get("/").accept(MediaType.APPLICATION_JSON)) (1)
		.andExpect(status().isOk()) (2)
		.andDo(document("index")); (3)
```

1. Invoke the root(/) of the service and indicate that an application/json response is required
2. Assert that the service produced the expected response.
3. Document the call to the service, writing the snippets into a directory named index (which is located beneath the configured output directory). The snippets are written by a **RestDocumentationResultHandler**

### Request and Response Fields

- To provide more detailed documentation of a request or response payload, support for documenting the payload’s fields is provided.

```java
this.mockMvc.perform(get("/user/5").accept(MediaType.APPLICATION_JSON)).andExpect(status().isOk())
		.andDo(document("index", responseFields((1)
				fieldWithPath("contact.email").description("The user's email address"), (2)
				fieldWithPath("contact.name").description("The user's name")))); (3)
```

> To document a request , you can use requestfields.
>

**When documenting fields, the test fails if an undocumented field is found in the payload, similarly, the test also fails if a documented filed is not found in the payload and the field has not been marked as optional.**

---

## Fields in JSON Payloads

### JSON Field Paths

JSON field paths use either dot notation or bracket notation.

### JSON Field Types

When a field is documented, Spring REST Docs tries to determine its type by examining the payload. Seven different types are supported.

![스크린샷 2022-12-26 오후 3.45.57.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/42bd1c1f-c6a4-4701-bd6b-7fa8d8145e1d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-12-26_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.45.57.png)

```java
.andDo(document("index", responseFields(fieldWithPath("contact.email").type(JsonFieldType.STRING) (1)
		.description("The user's email address"))));
```

---

### Query Parameter

You can document a request’s query parameters by using ‘queryParameters’.

```java
this.mockMvc.perform(get("/users?page=2&per_page=100")) 
		.andExpect(status().isOk()).andDo(document("users", queryParameters(
				parameterWithName("page").description("The page to retrieve"), 
				parameterWithName("per_page").description("Entries per page") 
		)));
```

### Form parameters

you can document a request’s form parameters by using ‘formParameters’.

```java
this.mockMvc.perform(post("/users").param("username", "Tester")) (1)
		.andExpect(status().isCreated()).andDo(document("create-user", formParameters((2)
				parameterWithName("username").description("The user's username") (3)
		)));
```

### Path Parameters

you can document a request’s path parameters by using ‘pathParameters’.

```java
this.mockMvc.perform(get("/locations/{latitude}/{longitude}", 51.5072, 0.1275)) (1)
		.andExpect(status().isOk()).andDo(document("locations", pathParameters((2)
				parameterWithName("latitude").description("The location's latitude"), (3)
				parameterWithName("longitude").description("The location's longitude") (4)
		)));
```
