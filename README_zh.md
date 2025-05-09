# HTTPie-rs: 基于 Rust 的 HTTP 命令行工具

HTTPie-rs 是一个命令行 HTTP 客户端，其灵感来源于广受欢迎的 Python 工具 [HTTPie](https://httpie.io/)。它旨在提供一个用户友好的界面，用于直接从终端发出 HTTP 请求，并利用 Rust 的高性能和安全性构建。

目前，HTTPie-rs 支持 GET、POST、PUT、PATCH 和 DELETE 请求，以及自定义头部、查询参数、JSON 和表单数据提交、格式化美观的输出、离线请求检查等功能。

## 项目结构

项目被组织成多个模块，以分离关注点并增强可维护性：

```mermaid
graph TD
    A[CLI 输入] --> B(main.rs);
    B --> C{cli.rs - Clap 参数解析};
    C --> D[processor.rs - 命令逻辑处理];
    D --> E{HTTP 请求构建 (reqwest)};
    E --> F[HTTP 客户端 (reqwest)];
    F --> G[远程服务器];
    G --> F;
    F --> H{响应处理};
    H --> I[输出到控制台];

    D --> T(types.rs - 共享枚举/结构体);
    D --> U(utils.rs - 辅助函数);
    D --> V(error.rs - 自定义错误类型);

    subgraph 核心逻辑
        D
        E
        F
        H
    end

    subgraph 定义与工具
        C
        T
        U
        V
    end
```

*   **`main.rs`**: 应用程序的入口点。初始化 Reqwest 客户端并将命令分派给处理器。
*   **`cli.rs`**: 使用 `clap` 定义命令行界面，包括所有命令、选项和参数。
*   **`processor.rs`**: 包含处理已解析的 CLI 参数、构建 HTTP 请求、执行请求以及准备输出的核心逻辑。
*   **`types.rs`**: 存放共享数据类型和枚举，例如 `HttpMethod`。
*   **`error.rs`**: 使用 `thiserror` 为应用程序定义自定义错误类型，以实现健壮的错误处理。
*   **`utils.rs`**: 提供实用功能，例如头部和键值对的解析器，以及 JSON 格式化。

## 功能特性

*   支持 GET, POST, PUT, PATCH, DELETE HTTP 方法。
*   自定义 HTTP 头部。
*   URL 查询参数。
*   JSON Body 构建 (`field=value`, `field:=json_literal`)。
*   表单数据提交 (`application/x-www-form-urlencoded`)。
*   通过 `--data` 发送原始请求体。
*   自动检测 JSON 和表单的 `Content-Type`，或手动覆盖。
*   默认情况下，JSON 输出会进行美化打印 (使用 `jsonxf`)。
*   控制输出格式 (`--pretty=none`)。
*   详细模式 (`-v`) 显示响应头部和状态。
*   离线模式 (`--offline`) 打印将要发送的请求，而不实际发送。
*   重定向跟随 (`--follow`, `--max-redirects`)。
*   请求超时 (`--timeout`)。

## 安装 / 构建

1.  **克隆仓库**:
    ```bash
    git clone https://github.com/nehcuh/httpie.git
    cd httpie
    ```
2.  **构建项目**:
    ```bash
    cargo build --release
    ```
3.  可执行文件将位于 `target/release/httpie`。您可以将其复制到 `PATH` 环境变量中的一个目录，例如 `/usr/local/bin` 或 `~/.local/bin`。
    ```bash
    ./target/release/httpie get httpbin.org/get
    ```

## 使用方法

基本语法是：
`httpie [全局选项] <命令> [命令选项] URL [BODY_ITEMS...]`

**全局选项**:
*   `-v, --verbose`: 详细输出。打印响应状态和头部。
*   `--offline`: 打印将要发送的请求，然后退出而不发送。
*   `--pretty [all|colors|format|none|auto]`: 控制输出格式 (默认: `auto`)。`auto` 会美化打印 JSON。
*   `--style [auto|solarized|monokai|none]`: 输出着色风格 (默认: `auto`)。(注意: 除了基本的 JSON 格式化外，基于主题的完整着色尚未实现)。
*   `-F, --follow`: 跟随重定向。
*   `--max-redirects <NUM>`: 最大重定向次数 (默认: 10)。
*   `--timeout <SECONDS>`: 请求超时秒数。

**命令**:
*   `get`: 执行 GET 请求。
*   `post`: 执行 POST 请求。
*   `put`: 执行 PUT 请求。
*   `patch`: 执行 PATCH 请求。
*   `delete`: 执行 DELETE 请求。

**命令选项 (用于 GET, POST, PUT 等命令)**:
*   `-H, --header <KEY:VALUE>`: 添加自定义 HTTP 头部。可多次使用。
*   `-q, --query <KEY=VALUE>`: 添加 URL 查询参数。可多次使用。
*   `URL`: 请求的 URL。对于 `http://`，可以省略协议方案。

**Body 选项 (用于 POST, PUT, PATCH)**:
*   `-f, --form`: 以 `application/x-www-form-urlencoded` 格式提交数据。`BODY_ITEMS` 应为 `key=value`。此标志应在命令之后 (例如 `post --form ...`)。
*   `-j, --json`: 以 `application/json` 格式提交数据。`BODY_ITEMS` 被解释为 JSON 字段。此标志应在命令之后 (例如 `post --json ...`)。
    *   `key=string_value`  (例如 `name=Alice`) -> `"name": "Alice"`
    *   `key:=json_literal` (例如 `age:=30`, `active:=true`, `tags:='["rust","cli"]'`)
    *   如果单个 body item 是一个完整的 JSON 文档，它将被用作原始 JSON body。
*   `-d, --data <STRING>`: 发送原始请求体字符串。覆盖 `BODY_ITEMS`。除非头部另有指定，默认 `Content-Type` 是 `text/plain`。
*   `BODY_ITEMS...`: 位置参数，被视为请求体数据。其解释取决于 `--form`、`--json` 标志或 `Content-Type` 头部。如果在可以接受多个值的选项 (如 `-H`) 之后使用，并且可能产生歧义，请使用 `--` 将选项与 body items 分开 (例如 `httpie post URL -H "Header: Val" -- body_item_that_looks_like_a_header`)。

### httpbin.org 示例
(从 `RustPractise` 目录使用 `cargo run -p httpie -- ` 运行，或者在构建和安装后如果 `httpie` 已在您的 PATH 中)

1.  **简单 GET 请求**:
    ```bash
    httpie get httpbin.org/get
    ```

2.  **带查询参数的 GET 请求**:
    ```bash
    httpie get httpbin.org/get -q search=rust -q lang=en
    ```

3.  **带自定义头部的 GET 请求**:
    ```bash
    httpie get httpbin.org/headers -H "X-Api-Key:your_api_key" -H "Accept:application/json"
    ```

4.  **POST 表单数据 (显式 `--form`)**:
    ```bash
    httpie post --form httpbin.org/post name=Rustacean project=httpie-rs
    ```

5.  **POST JSON 数据 (字段语法, 显式 `--json`)**:
    ```bash
    httpie post --json httpbin.org/post \
        name=httpie-rs \
        language=Rust \
        awesome:=true \
        version:=0.1 \
        features:='["cli", "http", "json"]'
    ```

6.  **POST JSON 数据 (原始 JSON, 使用 `--data` 和 `Content-Type` 头部)**:
    ```bash
    httpie post httpbin.org/post \
        -H "Content-Type: application/json" \
        --data '{"name": "httpie-rs", "raw_json": true}'
    ```

7.  **PUT 请求与 JSON Body**:
    ```bash
    httpie put --json httpbin.org/put id:=123 status=updated description="Item was modified"
    ```

8.  **DELETE 请求**:
    ```bash
    httpie delete httpbin.org/delete
    ```

9.  **详细输出**:
    ```bash
    httpie -v get httpbin.org/get
    ```

10. **离线模式**:
    ```bash
    httpie --offline post httpbin.org/post name=Test value=Offline
    ```

## 如何贡献

欢迎贡献！如果您想改进 HTTPie-rs，可以按照以下步骤开始：

**1. 开始之前**
*   确保您已安装最新的稳定版 Rust 工具链。请参阅 [rustup.rs](https://rustup.rs/)。
*   Fork 本仓库 (如果适用)。
*   克隆您的仓库：`git clone <your-repo-url>`
*   导航到工作区根目录：`cd RustPractise` (或您的项目根目录)。
*   构建 `httpie` 包：`cargo build -p httpie`
*   为 `httpie` 包运行测试：`cargo test -p httpie`

**2. 理解代码结构**
请熟悉 "项目结构" 部分描述的 `RustPractise/httpie/src/` 内的模块。进行修改时，关键模块通常是：
*   `src/cli.rs`: 用于添加或更改 CLI 命令、选项和参数。
*   `src/processor.rs`: 用于实现与新的/更改的 CLI 功能相关的逻辑。
*   `src/types.rs`: 用于新的共享数据类型。
*   `src/error.rs`: 用于新的自定义错误类型。
*   `tests/`: 用于为新功能添加集成测试 (`RustPractise/httpie/tests/`)。

**3. 示例：添加新的 HTTP 方法 (例如 `HEAD`)**
1.  **定义方法**: 在 `src/types.rs` 的 `HttpMethod` 枚举中添加 `Head`。
2.  **添加 CLI 子命令**: 在 `src/cli.rs` 的 `CommandWithUrl` 枚举中添加 `Head(MethodSpecificOpts)`。
3.  **在处理器中处理**: 更新 `src/processor.rs` 中 `process_command` 函数内的 `match` 语句，以提取 `CommandWithUrl::Head` 的详细信息。
4.  **在请求执行中实现**: 确保在 `src/processor.rs` 的 `execute_actual_request` 中处理 `HttpMethod::Head` (例如 `client.head(url.clone())`)。
5.  **添加测试**: 在 `RustPractise/httpie/tests/cli_requests.rs` 中创建新的集成测试。

**4. 添加其他功能/标志**
*   在 `src/cli.rs` 中适当的结构体中定义新的选项/标志。
*   更新 `src/processor.rs` 以读取并根据新选项执行操作。
*   为任何新的独立逻辑添加单元测试，并添加集成测试。

**5. 代码风格和质量**
*   请格式化您的代码：`cargo fmt --all` (从工作区根目录运行)。
*   检查 lint：`cargo clippy -p httpie -- -D warnings` (将警告视为错误)。
*   编写清晰简洁的提交消息。

**6. 提交更改**
*   为您的功能或错误修复创建一个新分支。
*   进行更改，提交它们，然后推送。
*   创建一个 Pull Request。
*   清楚地描述您的更改。
*   确保所有测试通过。

## 许可证
本项目根据 MIT 许可证授权。
