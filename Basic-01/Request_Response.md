**1. Request-Response format
**Request = what the client sends to the server. It has: **

- **URL** — the address, e.g. http://localhost:8088/students
- **HTTP Method** — what you want to do: GET (fetch data), POST (create data), PUT/PATCH (update data), DELETE (remove data)
- **Headers** — extra metadata, like "I'm sending JSON" or "here's my login token"
- **Body** — actual data you're sending (only for POST/PUT usually), e.g. {"name": "Rahul", "age": 22}

**Response** = what the server sends back. It has: 

- **Status code** — a number telling you what happened: 200 (OK, success), 201 (created), 404 (not found), 500 (server crashed/error)
- **Headers** — metadata about the response

• • **Body** — the actual data you asked for (or an error message)


**2. Servlet and Servlet Container**

**The problem**: A raw HTTP request coming over the network is just text/bytes flowing over a socket. Someone needs to catch that raw data, parse it into something Java code can actually work with (extract the URL, method, headers, body), and later take whatever Java produces and convert it back into raw bytes to send over the network.

**Servlet** = a Java class with a defined contract (it implements the Servlet interface, usually via HttpServlet) that knows how to handle one specific job: "given a request, do something, produce a response." It has methods like doGet(), doPost() that you'd override to write your logic — this is literally how Java web apps were written before frameworks like Spring existed.

**Servlet Container** = the engine that manages the _lifecycle_ of servlets. It does the boring-but-essential plumbing:

- Listens on a network port for incoming HTTP connections
- Parses raw HTTP into a Java-friendly object (HttpServletRequest)
- Creates a matching HttpServletResponse object to be filled in
- Figures out _which_ servlet should handle this particular request
- Calls that servlet's method, passing in the request/response objects

• • Takes whatever the servlet wrote into the response object and sends it back over the network as raw HTTP

**Where Spring fits in**: In Spring Boot, you never write a raw Servlet yourself. Spring provides ONE big servlet called DispatcherServlet that's registered with the container. Every single request — no matter which of your @RestController methods it's meant for — goes to this one DispatcherServlet first. It then looks at the URL and method, figures out "oh, this matches @GetMapping("/students") in StudentController," and calls your method for you. So your @GetMapping method is just a convenient wrapper — underneath, it's still all servlet machinery doing the actual request handling.

**3. localhost:8088 — IP address and port

**IP address — "which computer?"  
**Port — "which program on that computer?"**

**4. Tomcat server  
Tomcat is one actual, real, downloadable piece of software that implements that Servlet and Servlet Container concept  
it embeds Tomcat directly inside your application. When you build your project, Tomcat's code gets bundled right into your runnable .jar file. So when you just run your main() method:

I. Your Spring Boot app starts
II. It automatically starts an embedded Tomcat instance internally
III. Tomcat starts listening on the configured port (default 8080, or your custom 8088)
IV. Your app is now reachable at localhost:8088 — with zero manual server setup