
# 开发规范指南
为保证代码质量、可维护性、安全性与可扩展性，请在开发过程中严格遵循以下规范。

## 一、基础信息与工作目录
- **工作区路径**：`D:\develop\code\PaiSmart`
- **代码作者**：11423
- **构建工具**：Maven
- **项目标识**：groupId为`com.yizhaoqi.smartpai`，artifactId为`SmartPAI`，版本`0.0.1-SNAPSHOT`

## 二、技术栈要求
- **主框架**：Spring Boot 3.4.2
- **语言版本**：JDK 17.0.19 / Java 17
- **前端框架**：Vue 3 + TypeScript + UnoCSS + Pinia
- **核心依赖**：
  | 分类         | 依赖说明                                                                 |
  |--------------|--------------------------------------------------------------------------|
  | 数据层       | `spring-boot-starter-data-jpa`、`spring-boot-starter-data-redis`、MySQL 8 驱动 |
  | 安全认证     | `spring-boot-starter-security`、JJWT 0.11.5                              |
  | 消息队列     | `spring-kafka` 3.2.1                                                    |
  | 文件存储     | MinIO SDK 8.5.12                                                         |
  | 文档解析     | Apache Tika 2.9.1、Commons IO 2.14.0、Commons Codec 1.16.1               |
  | AI能力       | Spring WebFlux（DeepSeek API调用）、阿里云OCR SDK 3.1.3、HanLP portable-1.8.6 |
  | 向量检索     | Elasticsearch Java Client 8.10.0                                         |
  | 支付能力     | 微信支付Java SDK 0.2.17                                                  |
  | 工具库       | Lombok 1.18.30、Jackson 2.15.2、Gson 2.10.1、SLF4J 2.0.16               |
  | 校验与测试   | `spring-boot-starter-validation`、`spring-boot-starter-test`、H2数据库   |

## 三、目录结构
```
PaiSmart
├── .qoder                          # 项目知识库配置
│   ├── quests
│   └── repowiki
│       └── zh
│           └── content             # 项目知识库文档（API、架构、部署等）
├── docs
│   └── databases                   # 数据库设计文档
├── frontend                        # 前端Vue3项目
│   ├── packages                    # 前端公共包（alova、axios、hooks、utils等）
│   ├── public                      # 静态资源
│   └── src
│       ├── assets                  # 图片、图标等资源
│       ├── components              # 组件库（advanced通用高级组件、common基础组件、custom业务组件）
│       ├── constants               # 前端常量
│       ├── enum                    # 前端枚举
│       ├── hooks                   # 自定义Hooks（business业务级、common通用级）
│       ├── layouts                 # 布局组件（base-layout基础布局、blank-layout空白布局、modules全局模块）
│       ├── locales                 # 国际化配置
│       ├── plugins                 # 前端插件
│       ├── router                  # 路由配置（elegant路由模式、guard路由守卫、routes业务路由）
│       ├── service                 # 接口封装（api接口定义、request请求拦截）
│       ├── store                   # Pinia状态管理（按模块划分：app、auth、chat、knowledge-base等）
│       ├── styles                  # 全局样式（css/scss）
│       ├── theme                   # 主题配置
│       ├── typings                 # TypeScript类型定义
│       ├── utils                   # 前端工具类
│       └── views                   # 业务视图（chat、knowledge-base、user、recharge等）
├── scripts                         # 运维、部署脚本
└── src                             # 后端Java项目
    ├── main
    │   ├── java
    │   │   └── com.yizhaoqi.smartpai
    │   │       ├── client           # 第三方SDK客户端封装（OCR、MinIO、DeepSeek、ES、微信支付等）
    │   │       ├── config           # 配置类（ES初始化、Kafka、Security、跨域等配置）
    │   │       ├── consumer         # Kafka消息消费者（文件解析、向量化等异步任务）
    │   │       ├── controller       # HTTP接口层
    │   │       ├── entity           # JPA数据库实体类
    │   │       ├── exception        # 全局异常处理、自定义异常
    │   │       ├── handler          # WebSocket消息处理器、其他业务处理器
    │   │       ├── model            # 通用请求/响应模型、参数封装对象
    │   │       ├── repository       # 数据访问层
    │   │       ├── service          # 业务逻辑层（接口与实现分离，实现类放在impl子包）
    │   │       ├── test             # 后端测试代码
    │   │       └── utils            # 后端工具类
    │   └── resources
    │       ├── es-mappings          # Elasticsearch索引映射文件
    │       ├── META-INF             # 元数据配置
    │       └── static               # 后端静态资源
    └── test                         # 后端集成测试
        ├── java
        └── resources
```

## 四、分层架构规范
| 层级          | 职责说明                                                                 | 开发约束与注意事项                                                                 |
|---------------|--------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| **Controller** | 处理HTTP请求与响应，定义API接口                                           | 不得直接访问数据库，必须通过Service层调用；统一返回封装后的响应对象，禁止直接返回Entity |
| **Service**    | 实现业务逻辑、事务管理与数据校验                                           | 必须通过Repository层访问数据库；优先返回DTO/模型对象，禁止直接返回Entity；接口与实现分离，实现类放在`impl`子包 |
| **Repository** | 数据库访问与持久化操作                                                     | 继承`JpaRepository`；复杂查询使用`@EntityGraph`避免N+1问题；ES查询统一封装在Repository |
| **Entity**     | 映射数据库表结构                                                           | 不得直接返回给前端，需转换为DTO/模型对象；包名统一为`entity`                      |
| **Consumer**   | Kafka异步消息消费处理                                                     | 仅处理非核心异步任务（文件解析、向量化等）；消费失败按策略重试，耗尽后自动进入死信主题；禁止执行阻塞性操作 |
| **Client**     | 第三方SDK封装                                                             | 统一封装所有第三方SDK调用逻辑，禁止在业务层直接调用原生SDK API；异常统一封装为项目自定义异常 |
| **Config**     | 配置类定义                                                                 | 统一管理Bean初始化、第三方配置、安全配置等；禁止硬编码配置信息，敏感配置必须通过环境变量注入 |
| **Handler**    | 消息/异常处理                                                             | WebSocket消息统一在handler包处理；全局异常统一封装返回，禁止直接抛出原生异常给前端 |

### 接口与实现分离
- 所有业务接口定义在`service`包下，具体实现类必须放在对应接口下的`impl`子包中，如`UserService`对应`UserServiceImpl`。

## 五、安全与性能规范
### 输入校验
- 所有接口入参必须使用`@Valid`与JSR-303校验注解（位于`jakarta.validation.constraints.*`），禁止跳过校验直接处理参数。
- 禁止手动拼接SQL/ES查询字符串，防止注入攻击。
- 文件上传必须校验文件类型、大小（单文件不超过50MB，总请求不超过100MB），禁止上传可执行文件等危险类型。

### 敏感信息管理
- 数据库密码、JWT密钥、第三方API密钥、微信支付密钥等所有敏感信息必须通过环境变量注入，禁止硬编码在代码、配置文件或提交到代码仓库。
- 日志中禁止打印敏感信息（密钥、用户密码、身份证号、支付信息等）。

### 事务与异步
- `@Transactional`注解仅用于Service层方法，禁止在Controller、Consumer、异步方法中使用。
- 非核心业务流程（文件解析、向量化、消息通知等）必须异步处理，禁止同步阻塞请求。

### 限流与配额
- 登录、注册、聊天、LLM、向量接口必须遵守配置的Redis限流规则，禁止超限提供服务。
- LLM、Embedding调用必须遵守用户配额限制，按配置的`usage-quota`规则统计token消耗。
- 异步消息必须配置重试策略，消费失败重试耗尽后自动进入死信主题`file-processing-dlt`，避免消息丢失。

### 支付与跨域安全
- 微信支付回调接口必须做签名校验，防止伪造请求，支付相关配置必须通过环境变量注入。
- WebSocket与HTTP接口必须校验Origin，符合配置的`security.allowed-origins`白名单规则，禁止未授权跨域访问。

## 六、代码风格规范
### 命名规范
| 类型               | 命名方式             | 示例                                  |
|--------------------|----------------------|---------------------------------------|
| 类名（普通类）     | UpperCamelCase       | `UserServiceImpl`                     |
| 类名（配置类）     | UpperCamelCase + Config | `RedisConfig`、`KafkaConfig`        |
| 类名（客户端）     | UpperCamelCase + Client | `MinioClient`、`DeepSeekClient`     |
| 类名（消费者）     | UpperCamelCase + Consumer | `FileProcessingConsumer`          |
| 类名（处理器）     | UpperCamelCase + Handler | `WebSocketMessageHandler`         |
| 类名（异常）       | UpperCamelCase + Exception | `BusinessException`              |
| 方法/变量          | lowerCamelCase       | `saveUser()`、`fileChunkSize`         |
| 常量               | UPPER_SNAKE_CASE     | `MAX_LOGIN_ATTEMPTS`、`CHUNK_SIZE`    |

### 注释规范
- 所有类、方法、字段必须添加**中文Javadoc**注释，说明用途、参数、返回值、异常说明。
- 复杂业务逻辑、特殊处理逻辑必须添加行内注释说明，禁止写无注释的复杂代码。

### 类型命名规范（阿里巴巴风格）
| 后缀   | 用途说明                     | 示例         |
|--------|------------------------------|--------------|
| DTO    | 数据传输对象（前后端交互）   | `UserDTO`    |
| DO     | 数据库实体对象               | `UserDO`     |
| BO     | 业务逻辑封装对象             | `FileParseBO`|
| VO     | 视图展示对象（前端返回）     | `UserVO`     |
| Query  | 查询参数封装对象             | `UserQuery`  |
| Request| 请求参数对象                 | `ChatRequest`|
| Response| 响应结果对象                | `ChatResponse`|

### 实体类与工具类
- 实体类统一使用Lombok注解`@Data`、`@NoArgsConstructor`、`@AllArgsConstructor`，禁止手动编写getter/setter/构造方法。
- 工具类必须为`final`修饰，构造方法私有化，禁止实例化，所有方法为静态方法。

### 日志规范
- 统一使用`@Slf4j`注解生成日志对象，禁止使用`System.out.println`。
- 日志级别遵循配置：开发环境可开DEBUG，生产环境仅保留INFO及以上级别。
- 异常日志必须打印完整堆栈信息，方便排查问题；业务关键节点必须打印INFO级别日志。

## 七、通用开发规则
### 后端开发规则
1. **第三方SDK封装规则**：所有第三方SDK（阿里云OCR、MinIO、DeepSeek、Elasticsearch、微信支付、HanLP等）必须封装在`client`包下，提供统一的调用方法，异常统一封装为项目自定义异常，禁止在Controller/Service层直接调用原生SDK API。
2. **Kafka使用规则**：消息生产者在Service层发送，消费者统一放在`consumer`包下；消息体必须包含必要的上下文信息（如文件ID、用户ID）；消费失败按配置的重试策略重试，耗尽后自动进入死信主题`file-processing-dlt`，禁止在消费者中执行阻塞性IO操作。
3. **文件处理规则**：文件上传后先存储到MinIO，再发送Kafka消息异步完成解析、切块、向量化、存入ES的流程；文件切块遵循配置的`chunk-size=512`、`overlap-size=100`参数；PDF解析默认使用LiteParse，需OCR时调用阿里云OCR或本地Tesseract。
4. **AI调用规则**：调用DeepSeek LLM和Embedding API时，必须遵守配置的`usage-quota`配额限制与`rate-limit`限流规则；系统Prompt（`ai.prompt.rules`）优先级最高，禁止修改其核心规则；AI响应必须按规则标注来源信息。
5. **数据存储规则**：MySQL用于存储业务结构化数据，JPA实体类禁止直接返回前端，必须转换为DTO/模型对象；Elasticsearch用于存储文档向量与元数据，索引映射文件统一放在`src/main/resources/es-mappings`下，初始化逻辑在Config包统一处理。
6. **知识库初始化规则**：启动时按`knowledge.bootstrap`配置自动导入知识库文档，禁止手动修改初始化配置，避免覆盖已有数据。

### 前端开发规则
1. 前端API请求统一封装在`frontend/src/service`目录下，禁止硬编码接口地址，通过环境变量配置。
2. 组件按模块划分：通用组件放在`components/common`子包，业务组件放在`components/custom`子包，高级通用组件放在`components/advanced`子包。
3. 状态管理统一使用Pinia，按业务模块划分store，禁止滥用全局状态。
4. 路由统一在`router`目录下配置，路由守卫统一处理权限校验，未授权用户自动跳转登录页。

### 配置管理规则
- 所有配置优先从环境变量读取，本地开发可通过`.env`文件配置，禁止将敏感配置提交到代码仓库。
- 配置变更必须同步更新到项目知识库的部署文档，避免环境不一致。

### 测试规则
- 单元测试放在对应类的同级`test`目录下，集成测试放在`src/test`下。
- 测试用例必须覆盖核心业务逻辑，禁止提交无测试的核心代码。

## 八、编码原则总结
| 原则       | 说明                                                                 |
|------------|----------------------------------------------------------------------|
| **SOLID**  | 高内聚、低耦合，增强可维护性与可扩展性                               |
| **DRY**    | 避免重复代码，提高复用性                                             |
| **KISS**   | 保持代码简洁易懂，避免过度设计                                       |
| **YAGNI**  | 不实现当前不需要的功能，避免提前优化                                 |
| **OWASP**  | 防范常见安全漏洞，如SQL注入、XSS、越权访问等                         |
| **异步优先** | 非核心流程优先异步处理，提升系统吞吐量与响应速度                     |
| **可观测性** | 关键节点打印日志，异常统一封装，便于问题排查与链路追踪               |
