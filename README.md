# HTTPie-rs: A Rust-based HTTP CLI

HTTPie-rs is a command-line HTTP client inspired by the popular Python tool [HTTPie](https://httpie.io/). It aims to provide a user-friendly interface for making HTTP requests directly from your terminal, built with the performance and safety of Rust.

Currently, HTTPie-rs supports GET, POST, PUT, PATCH, and DELETE requests, along with custom headers, query parameters, JSON and form data submissions, pretty-printed output, offline request inspection, and more.

## Project Structure

The project is organized into several modules to separate concerns and enhance maintainability:

```mermaid
graph TD
    A[CLI Input] --> B(main.rs);
    B --> C{cli.rs - Clap Parsing};
    C --> D[processor.rs - Command Logic];
    D --> E{HTTP Request Construction (reqwest)};
    E --> F[HTTP Client (reqwest)];
    F --> G[Remote Server];
    G --> F;
    F --> H{Response Handling};
    H --> I[Output to Console];

    D --> T(types.rs - Shared Enums/Structs);
    D --> U(utils.rs - Helper Functions);
    D --> V(error.rs - Custom Error Types);

    subgraph Core Logic
        D
        E
        F
        H
    end

    subgraph Definitions & Utilities
        C
        T
        U
        V
    end
```

*   **`main.rs`**: Entry point of the application. Initializes the Reqwest client and dispatches commands to the processor.
*   **`cli.rs`**: Defines the command-line interface using `clap`, including all commands, options, and arguments.
*   **`processor.rs`**: Contains the core logic for processing parsed CLI arguments, constructing HTTP requests, executing them, and preparing the output.
*   **`types.rs`**: Holds shared data types and enums, such as `HttpMethod`.
*   **`error.rs`**: Defines custom error types for the application using `thiserror` for robust error handling.
*   **`utils.rs`**: Provides utility functions, such as parsers for headers and key-value pairs, and JSON formatting.

## Features

*   Support for GET, POST, PUT, PATCH, DELETE HTTP methods.
*   Custom HTTP headers.
*   URL query parameters.
*   JSON body construction (`field=value`, `field:=json_literal`).
*   Form data submission (`application/x-www-form-urlencoded`).
*   Raw request body via `--data`.
*   Automatic `Content-Type` detection for JSON and forms, or manual override.
*   Pretty-printed JSON output by default (using `jsonxf`).
*   Control over output formatting (`--pretty=none`).
*   Verbose mode (`-v`) to display response headers and status.
*   Offline mode (`--offline`) to print the request that would be sent without actually sending it.
*   Redirect following (`--follow`, `--max-redirects`).
*   Request timeouts (`--timeout`).

## Installation / Building

1.  **Clone the repository**:
    ```bash
    # If your project is hosted, replace with your repository URL, e.g.:
    git clone https://github.com/nehcuh/httpie
    cd httpie-rs 
    ```
2.  **Build the project**:
    ```bash
    cargo build --release
    ```
3.  The binary will be available at `httpie/target/release/httpie`. You can copy it to a directory in your `PATH`, e.g., `/usr/local/bin` or `~/.local/bin`.
    ```bash
    ./target/release/httpie get httpbin.org/get
    ```
    ```

## Usage

The basic syntax is:
`httpie [GLOBAL_OPTIONS] <COMMAND> [COMMAND_OPTIONS] URL [BODY_ITEMS...]`

**Global Options**:
*   `-v, --verbose`: Verbose output. Prints response status and headers.
*   `--offline`: Print the request that would be sent, then exit without sending.
*   `--pretty [all|colors|format|none|auto]`: Controls output formatting (default: `auto`). `auto` pretty-prints JSON.
*   `--style [auto|solarized|monokai|none]`: Output coloring style (default: `auto`). (Note: Full colorization based on themes is not yet implemented beyond basic JSON formatting).
*   `-F, --follow`: Follow redirects.
*   `--max-redirects <NUM>`: Maximum number of redirects to follow (default: 10).
*   `--timeout <SECONDS>`: Request timeout in seconds.

**Commands**:
*   `get`: Performs a GET request.
*   `post`: Performs a POST request.
*   `put`: Performs a PUT request.
*   `patch`: Performs a PATCH request.
*   `delete`: Performs a DELETE request.

**Command Options (for commands like GET, POST, PUT, etc.)**:
*   `-H, --header <KEY:VALUE>`: Add a custom HTTP header. Can be used multiple times.
*   `-q, --query <KEY=VALUE>`: Add a URL query parameter. Can be used multiple times.
*   `URL`: The request URL. Scheme can be omitted for `http://`.

**Body Options (for POST, PUT, PATCH)**:
*   `-f, --form`: Submits data as `application/x-www-form-urlencoded`. `BODY_ITEMS` should be `key=value`. This flag should come *after* the command (e.g., `post --form ...`).
*   `-j, --json`: Submits data as `application/json`. `BODY_ITEMS` are interpreted as JSON fields. This flag should come *after* the command (e.g., `post --json ...`).
    *   `key=string_value`  (e.g., `name=Alice`) -> `"name": "Alice"`
    *   `key:=json_literal` (e.g., `age:=30`, `active:=true`, `tags:='["rust","cli"]'`)
    *   If a single body item is a full JSON document, it's used as the raw JSON body.
*   `-d, --data <STRING>`: Send a raw request body string. Overrides `BODY_ITEMS`. Default `Content-Type` is `text/plain` unless otherwise specified by a header.
*   `BODY_ITEMS...`: Positional arguments treated as request body data. Interpretation depends on `--form`, `--json` flags, or `Content-Type` header. If using after options that take multiple values (like `-H`), use `--` to separate options from body items if ambiguity arises (e.g., `httpie post URL -H "Header: Val" -- body_item_that_looks_like_a_header`).

### Examples using httpbin.org
(Run from the `RustPractise` directory using `cargo run -p httpie -- ` or if `httpie` is in your PATH after building and installing)

1.  **Simple GET Request**:
    ```bash
    httpie get httpbin.org/get
    ```

2.  **GET Request with Query Parameters**:
    ```bash
    httpie get httpbin.org/get -q search=rust -q lang=en
    ```

3.  **GET Request with Custom Headers**:
    ```bash
    httpie get httpbin.org/headers -H "X-Api-Key:your_api_key" -H "Accept:application/json"
    ```

4.  **POST Form Data (Explicit `--form`)**:
    ```bash
    httpie post --form httpbin.org/post name=Rustacean project=httpie-rs
    ```

5.  **POST JSON Data (Field syntax, explicit `--json`)**:
    ```bash
    httpie post --json httpbin.org/post \
        name=httpie-rs \
        language=Rust \
        awesome:=true \
        version:=0.1 \
        features:='["cli", "http", "json"]'
    ```

6.  **POST JSON Data (Raw JSON with `--data` and `Content-Type` header)**:
    ```bash
    httpie post httpbin.org/post \
        -H "Content-Type: application/json" \
        --data '{"name": "httpie-rs", "raw_json": true}'
    ```

7.  **PUT Request with JSON Body**:
    ```bash
    httpie put --json httpbin.org/put id:=123 status=updated description="Item was modified"
    ```

8.  **DELETE Request**:
    ```bash
    httpie delete httpbin.org/delete
    ```

9.  **Verbose Output**:
    ```bash
    httpie -v get httpbin.org/get
    ```

10. **Offline Mode**:
    ```bash
    httpie --offline post httpbin.org/post name=Test value=Offline
    ```

## Contributing

Contributions are welcome! If you'd like to improve HTTPie-rs, here's how you can get started:

**1. Getting Started**
*   Ensure you have the latest stable Rust toolchain installed. See [rustup.rs](https://rustup.rs/).
*   Fork the repository (if applicable).
*   Clone your repository: `git clone <your-repo-url>`
*   Navigate to the workspace root: `cd RustPractise` (or your project's root).
*   Build the `httpie` package: `cargo build -p httpie`
*   Run tests for the `httpie` package: `cargo test -p httpie`

**2. Understanding the Code Structure**
Familiarize yourself with the modules within `RustPractise/httpie/src/` as described in the "Project Structure" section. Key modules for modifications are typically:
*   `src/cli.rs`: For adding or changing CLI commands, options, and arguments.
*   `src/processor.rs`: For implementing the logic associated with new/changed CLI features.
*   `src/types.rs`: For new shared data types.
*   `src/error.rs`: For new custom error types.
*   `tests/`: For adding integration tests for new features (`RustPractise/httpie/tests/`).

**3. Example: Adding a New HTTP Method (e.g., `HEAD`)**
1.  **Define the method**: Add `Head` to the `HttpMethod` enum in `src/types.rs`.
2.  **Add CLI Subcommand**: Add `Head(MethodSpecificOpts)` to the `CommandWithUrl` enum in `src/cli.rs`.
3.  **Handle in Processor**: Update the `match` statement in `process_command` within `src/processor.rs` to extract details for `CommandWithUrl::Head`.
4.  **Implement in Request Execution**: Ensure `HttpMethod::Head` is handled in `execute_actual_request` in `src/processor.rs` (e.g., `client.head(url.clone())`).
5.  **Add Tests**: Create new integration tests in `RustPractise/httpie/tests/cli_requests.rs`.

**4. Adding Other Features/Flags**
*   Define the new option/flag in the appropriate struct in `src/cli.rs`.
*   Update `src/processor.rs` to read and act upon the new option.
*   Add unit tests for any new isolated logic and integration tests.

**5. Code Style and Quality**
*   Please format your code: `cargo fmt --all` (from workspace root).
*   Check for lints: `cargo clippy -p httpie -- -D warnings` (to treat warnings as errors).
*   Write clear commit messages.

**6. Submitting Changes**
*   Create a new branch for your feature or bug fix.
*   Make your changes, commit them, and push.
*   Open a Pull Request.
*   Describe your changes clearly.
*   Ensure all tests pass.

## License
This project is licensed under the MIT License.
