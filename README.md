## 动物城管理系统（基于《疯狂动物城》的城市综合管理平台）

### 项目简介

**动物城管理系统**是一个基于《疯狂动物城》世界观的城市综合管理平台，模拟现实城市中的：
- 居民服务（公告、投诉、购票、购物等）
- 政府/管理员管理（员工管理、工单处理、数据统计）
- 商业经营（商品管理、订单管理、销售分析）

项目采用**前后端分离**架构，前端高度还原了一份原生 HTML+内联 CSS 的旧系统交互与视觉风格，同时在此基础上重构为 Vue3 组件化单页应用，后端提供完整的业务接口与权限控制。

- **后端**：Spring Boot 3.1.5 + MyBatis Plus + MySQL + JWT
- **前端**：Vue 3.3 + Vite + 原生 HTML & CSS + Pinia + Axios + ECharts

---

### 亮点

- **前后端分离 + 登录态与权限体系**
  - 使用 JWT 统一认证，基于角色（管理员 / 工作人员 / 居民）和员工就业状态（在职 / 休假 / 待审批）进行细粒度权限控制。
  - 自定义 Spring 拦截器 + 白名单路径（如商品图片、部分静态资源），解决图片 401 等实际问题。

- **现代化前端架构升级实践**
  - 基于 Vue3 组件化设计，将原有功能模块拆分为高内聚、可复用的组件单元，采用**纯原生标签结合内联样式**的方案，在保持原有操作习惯与界面风格的同时，提升代码可维护性。
  - 引入 Pinia 进行全局状态管理，实现用户信息、权限状态等数据的集中管理，支持**状态实时同步与更新**，有效应对员工状态变更、资料修改等业务场景，提升用户体验。

- **业务场景设计较完整**
  - **员工管理 & 休假流程**：支持管理员审批、员工自助结束休假，包含“待审批 / 在职 / 休假”等多状态流转，并在前端做了限制视图展示。
  - **居民服务**：公告发布与评论、投诉/工单处理、投票、车次查询与购票、商城购物与订单管理等。
  - **数据统计与可视化**：基于 ECharts 的销售统计与数据看板，支持动态加载和懒加载。

---

### 角色与核心功能

#### 管理员 / 工作人员
- **员工管理**
  - 员工入职审批、编辑信息、调整薪资、记录薪资历史、防止重复记录。
  - 就业状态管理：在职 / 休假 / 待审批，并控制不同状态下的前端可见页面。
- **休假管理**
  - 管理员发起结束休假的通知。
  - 员工自助结束休假时，后端直接将状态置为“在职”，不再需要管理员二次确认。
- **市政公告**
  - 发布 / 管理公告，支持紧急公告、过期时间配置。
  - 公告评论区展示与权限控制。
- **工单系统（投诉/建议）**
  - 查看居民提交的问题，回复并记录处理结果。
- **铁路 32106**
  - 新增车次（始发站、终点站、发车时间、到达时间、票价、座位数）。
  - 查看车次售票情况及购票名单（含姓名、动物种类、居住区域）。
- **居民管理**
  - 查看居民基础信息与部分行为数据。
- **商城管理**
  - 商品增删改查、上下架、图片上传（限制尺寸与大小）。
  - 可将商品库存调整为 0，前端/后端都做了健壮性处理。
- **订单管理 & 销售分析**
  - 订单发货、确认、状态流转。
  - 使用 ECharts 对销售额、订单量等进行可视化展示。

#### 居民
- **公告中心**：查看公告、对非紧急/已过期公告发表评论。
- **接诉即办**：提交工单/投诉，并查看处理结果和评价。
- **动物城之星**：为优秀员工/居民投票。
- **铁路 32106**
  - 按始发站、终点站、出行日期查询车次。
  - 购买车票、查看“我的车票”、退票。
- **动物商城**
  - 浏览商品、加入购物车、下单、查看订单、确认收货。
- **个人中心**
  - 修改个人信息和密码。
  - 实时刷新就业状态、个人资料等信息。

---

### 技术架构与实现细节

#### 前端：各技术的具体使用场景

- **Vue 3 (Composition API)**
  - 整个前端是一个单页应用（SPA），所有业务页面都在 `frontend-vue3/src/views/components/` 下拆分成独立组件，例如：
    - `Announcements.vue`：公告列表、详情、评论交互。
    - `Rail.vue`：车次查询、购票、车次新增、购票名单弹窗。
    - `Shop.vue` / `MallAdmin.vue`：前台商城与后台商品管理。
    - `WorkerSelf.vue` / `WorkerAdmin.vue`：员工自助中心与管理员端员工管理。
  - 使用 `setup()` + `ref/computed/onMounted` 组织状态和生命周期逻辑，避免 Options API 的臃肿写法。

- **Pinia（状态管理）**
  - 在 `src/stores/user.js` 中维护用户登录态、角色、就业状态等信息。
  - 封装了 `refreshUser()` 方法，用于从 `/me` 接口拉取最新用户信息：
    - 在 `Main.vue` 的 `onMounted`、`window.onfocus`、`visibilitychange` 事件中统一调用，保证数据实时。
    - 在 `Settings.vue`（修改资料）、`WorkerSelf.vue`（员工结束休假）等操作成功后主动调用，避免“改完必须重新登录”的体验。

- **Vue Router**
  - 在 `src/router` 中基于角色划分不同路由访问权限。
  - 实现登录后按角色跳转到不同首页（管理员/工作人员/居民），并限制未登录用户访问需要鉴权的页面。

- **Axios**
  - 在 `src/utils/api.js`（或同类工具文件）中二次封装 Axios：
    - 在请求拦截器中自动注入 JWT Token。
    - 在响应拦截器中统一处理未登录/权限不足等错误码，做重定向或提示。
  - 所有组件通过该封装实例访问后端，如：
    - `/trains`、`/tickets/orders`、`/admin/shop/products`、`/shop/orders/my` 等。

- **ECharts**
  - 在 `Sales.vue` 等组件中进行数据可视化展示销售额、订单量。
  - 在 `src/utils/echarts.js` 中封装 `ensureEchartsLoaded()`：
    - 优先 `import('echarts')`（npm 包）。
    - 如果运行环境不支持，再从 `public/echarts.js` 动态加载 CDN 版，解决打包和静态资源路径问题。

- **Vite**
  - 作为前端构建工具，负责：
    - 本地开发热更新（HMR），提高调试效率。
    - 打包生成生产环境静态资源。
  - 通过 `vite.config.js` 配置别名（如 `@` 指向 `src`）和开发代理（转发 `/api` 到后端）。

#### 后端：各技术的具体使用场景

- **Spring Boot**
  - 作为整个后端的基础框架，负责：
    - 自动装配与启动入口（`ZootopiaApplication`）。
    - RESTful API 开发：`controller` 包下的各类控制器，比如：
      - `AuthController`：登录注册、返回 JWT 和用户信息。
      - `TicketController`：车次购票、订单查询。
      - `TrainController`：车次创建、查询。
      - `AdminController`：员工审批、薪资历史、管理员操作。
      - `LeaveRequestController`：休假申请、结束休假。
      - `Shop*Controller`：商城商品与订单管理。
    - 参数绑定、JSON 序列化（包含时间类型的格式化）。

- **Spring MVC 拦截器 & WebConfig**
  - 在 `config/WebConfig.java` 中注册了 `AuthInterceptor`：
    - **AuthInterceptor** 从请求头读取 JWT，解析出 `userId`、`role`、`employmentStatus` 等信息，放入 `request` 属性，供后续 Controller 使用。
    - 对部分路径放行（登录、注册、静态资源、商品图片、ECharts 脚本等），解决图片访问 401 和前端静态资源受限问题。

- **MyBatis Plus**
  - 在 `entity`、`mapper` 层大量使用：
    - `@TableName`、`@TableId`、`@TableField` 映射数据库字段（包括 `snake_case` 与 `camelCase` 的兼容）。
    - `Mapper` 接口继承 `BaseMapper`，自动生成常用 CRUD。
    - 在 `TrainController`、`TicketController`、`AdminController` 等中使用 `LambdaQueryWrapper` 构造条件查询（如按车次、按用户、按日期筛选订单和车次）。
  - `TicketOrder.java` 中通过 `@TableField(exist = false)` 增加了 `name`、`animalType`、`liveArea` 等“联表查询字段”，用于车次订单列表里直接展示用户信息，而不破坏原有表结构。

- **MySQL**
  - 作为持久化存储，`zootopia.sql` 定义了完整的表结构和初始数据：
    - 用户表、员工表、车次表、车票订单表、商品表、商城订单表、公告表、评论表、工单表等。
  - 合理设置外键和约束，保证：
    - 删除/清空数据时需要先处理依赖表（本项目中通过手动 `TRUNCATE` 顺序解决）。
    - 防止主键重复（重新导入 SQL 前先清空业务表）。

- **JWT（JSON Web Token）**
  - 在 `AuthController` 登录成功后生成 Token，前端持久化在 LocalStorage。
  - `AuthInterceptor` 验证 Token：
    - 校验签名与过期时间。
    - 将用户 ID、角色、状态写入 `HttpServletRequest` 属性，供业务 Controller 使用。
  - 所有需要鉴权的接口（如 `/trains` POST、`/tickets/orders`、`/admin/**`、`/shop/orders/my` 等）都依赖这个机制。

- **Lombok**
  - 在实体类上通过 `@Data` 等注解生成 getter/setter、`toString` 等样板代码，减轻实体维护成本。

- **事务管理**
  - 在 `TicketController.createOrder`、部分审批与薪资调整逻辑中使用 `@Transactional`，保证：
    - 下单与更新车次已售数量同时成功或同时回滚。
    - 审批、薪资历史变更与员工状态更新保持一致性。

---

### 核心业务与典型问题解决示例

- **车次新增与购票**
  - 前端通过 `datetime-local` 控件录入时间，后端在 `TrainController` 中兼容解析 `departTime` / `depart_time` 两种字段格式，并将 `yyyy-MM-ddTHH:mm` 转为 `LocalDateTime`。
  - 对出发/到达时间、价格、座位数做了完整校验，生成“行驶时长”字段并持久化。
  - 购票时校验车次是否存在、是否已发车、座位是否已满，并更新车次已售数量。

- **员工休假流程**
  - 员工主动结束休假：后端直接将状态更新为“在职”，不再生成“待管理员确认”的记录，前端立即刷新用户信息和主页面展示。
  - 管理员发起结束休假的通知：避免重复通知、避免旧记录影响新的休假周期。
  - 对“待审批”的员工，登录后前端统一展示“暂无权限，请等待管理员审批”，隐藏所有业务功能。

- **数据实时性**
  - 针对“必须重新登录页面状态才会变化”的问题，在多个组件（如 `Main.vue`、`WorkerSelf.vue`、`Settings.vue`）中统一调用 `userStore.refreshUser()`，在成功操作后主动刷新用户信息，提升体验。

- **数据库导入与运维问题**
  - 解决 PowerShell 下 `<` 重定向读取 SQL 的兼容问题，改为：`Get-Content zootopia.sql | mysql ...`。
  - 处理 MySQL 外键约束导致的 TRUNCATE 失败和主键重复问题，合理清理依赖表数据后再导入。

---

### 快速开始（本地运行）

#### 1. 数据库初始化

1. 创建数据库（例如 `zootopia`），再导入 `zootopia.sql`：

```bash
mysql -u root -p zootopia < zootopia.sql
```

或在 PowerShell 下使用：

```powershell
Get-Content zootopia.sql | mysql -uroot -p123456 zootopia
```

#### 2. 启动后端

1. 进入 `backend-ssm` 目录  
2. 修改 `src/main/resources/application.yml` 中的数据库连接配置  
3. 启动 Spring Boot：

```bash
mvn spring-boot:run
```

或在 IDE 中运行 `ZootopiaApplication.java`，默认端口：`http://localhost:3000`

#### 3. 启动前端

1. 进入 `frontend-vue3` 目录  
2. 安装依赖：

```bash
npm install
```

3. 启动开发服务器：

```bash
npm run dev
```

4. 在浏览器访问：`http://localhost:5173`

---

### 测试账号（示例）

- **管理员**
  - 登录名：`admin`
  - 密码：`123456`

其他角色（如居民、工作人员）可通过注册或初始化 SQL 中的示例数据获得。

---

### 项目结构概览

```
.
├── backend-ssm/                 # Spring Boot 后端
│   ├── src/main/java/com/zootopia
│   │   ├── controller/          # 控制层（Auth、公告、车次、商城、工单等）
│   │   ├── entity/              # 实体类（User、Worker、Train、TicketOrder、Product 等）
│   │   ├── mapper/              # MyBatis Plus Mapper
│   │   └── config/              # 拦截器、CORS、Web 配置
│   └── src/main/resources/      # 配置文件与 Mapper XML
├── frontend-vue3/               # Vue3 前端
│   ├── src/
│   │   ├── views/components/    # 功能组件（公告、铁路、商城、员工管理等）
│   │   ├── stores/              # Pinia 状态管理（用户等）
│   │   ├── utils/               # 通用工具（API 封装、ECharts 动态加载等）
│   │   └── router/              # 页面路由
│   └── vite.config.js           # Vite 配置
├── photo/                       # 部分图片资源
└── zootopia.sql                 # 数据库初始化脚本
```


