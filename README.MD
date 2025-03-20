# Advance Programming Module 6: Concurrency

## Table of Contents
* [Milestone 1: Single-threaded Web Server Reflection](#milestone-1-single-threaded-web-server-reflection)

## Milestone 1: Single-threaded Web Server Reflection

This is a simple single-threaded web server implemented in Rust. The web server consists of two main functions:

1. `main()`: Sets up a TCP listener and handles incoming connections
2. `handle_connection()`: Processes each individual TCP stream (connection)

### TCP Listener Setup

```rust
let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
```

This binds the server to localhost (`127.0.0.1`) on port `7878`. The use of `unwrap()` ensures that any errors (e.g., if the port is in use) will cause a panic, which is fine for a simple implementation but should be handled more gracefully in production.

### Handling Connections

```rust
for stream in listener.incoming() { 
    let stream = stream.unwrap();
    handle_connection(stream);
}
```

This loop iterates over incoming connections and processes them one at a time. Since the server processes requests sequentially, it cannot handle multiple clients simultaneously, making it a single-threaded server.

### Processing HTTP Requests

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Request: {:#?}", http_request);
}
```

This function is responsible for reading and processing an incoming HTTP request. Here’s a breakdown of what it does:

1. **Creating a buffered reader:**
   ```rust
   let buf_reader = BufReader::new(&mut stream);
   ```
    - This wraps the TCP stream in a `BufReader`, which improves efficiency when reading data by allowing line-by-line processing.

2. **Reading and collecting HTTP request headers:**
   ```rust
   let http_request: Vec<_> = buf_reader
       .lines()
       .map(|result| result.unwrap())
       .take_while(|line| !line.is_empty())
       .collect();
   ```
    - `.lines()`: Reads the incoming TCP stream line by line.
    - `.map(|result| result.unwrap())`: Extracts the actual string content from the `Result<String, Error>` returned by `.lines()`. If an error occurs, `unwrap()` will panic.
    - `.take_while(|line| !line.is_empty())`: Continues collecting lines until it encounters an empty line, which marks the end of the HTTP headers.
    - `.collect()`: Stores the collected lines in a `Vec<String>` for further processing.

3. **Printing the request headers:**
   ```rust
   println!("Request: {:#?}", http_request);
   ```
    - This prints the parsed HTTP request headers in a nicely formatted way for debugging.

### Reflection & Learnings
- **Single-threaded Limitation**: Handling one request at a time means other clients must wait for the current request to finish before being processed.
- **Error Handling**: The code relies heavily on `unwrap()`, which isn't ideal for robustness. Proper error handling should be introduced.
- **Request Parsing**: The server captures HTTP headers but does not process different request types or send responses yet.


