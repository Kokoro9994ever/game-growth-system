# 游戏角色数值与成长配置管理系统

Java 21 + javax.servlet 4 + JSP/JSTL + JDBC + Maven WAR + MySQL 8.4 + Tomcat 9。无 Spring、ORM 或前端框架。页面使用本地 CSS 构建 Bootstrap 风格的表单、表格、卡片与自适应布局，不依赖 CDN。

## 本机运行

访问 **http://localhost:8088/game-growth-system/**，演示管理员：`admin` / `Admin@123`。

```powershell
# 在项目根目录执行；脚本将 Maven 的 Java 环境设为本机 JDK 21
powershell -ExecutionPolicy Bypass -File .\scripts\build.ps1
powershell -ExecutionPolicy Bypass -File .\scripts\start.ps1
# 停止由 start.ps1 后台启动的本项目实例
powershell -ExecutionPolicy Bypass -File .\scripts\stop.ps1
```

启动脚本使用独立 `runtime/tomcat` 目录，不需要修改 Tomcat 安装目录；默认端口 8088。日志位于 `runtime/tomcat/logs/`。更换机器时可传入 `-JavaHome`、`-TomcatHome` 和 `-Port`。也可设置 `JAVA_HOME` / `CATALINA_HOME`。同一端口运行时请先停止再启动。

## IntelliJ IDEA 2025.2.1

用 IDEA **打开项目根目录或 pom.xml**，选择 Maven 导入，项目 SDK 与 Maven Runner JRE 都设为 JDK 21。

项目包含两个共享运行配置：

1. `01 - Maven clean package`：编译、单元测试并生成 WAR。
2. `02 - Run Tomcat 9 (8088)`：通过 Maven 的 `run-tomcat` profile 在 IDEA Run 窗口运行本机 Tomcat，运行前先完成打包。若 8088 已在运行，先执行 `scripts/stop.ps1`。

Community Edition 使用上述 Maven 配置运行 Tomcat。也可以在 IDEA Terminal 中执行项目脚本。运行配置中的本机 JDK 路径、启动脚本的默认 Tomcat 路径更换机器后需修改或通过环境变量覆盖。

如 IDEA 弹出 `.port` 文件系统锁错误，可运行 `powershell -ExecutionPolicy Bypass -File .\scripts\open-idea.ps1`。此脚本通过 `IDEA_PROPERTIES` 将 IDEA system/cache 目录切换至独立位置，保留 IDEA 配置与插件，不触碰原有锁节点；也可传入 `-IdeaExe` 指定 IDEA 安装路径。

## 数据库初始化和配置

初始化 SQL：`sql/init.sql`，包括 `admin_user`、`game_role`、`attribute_definition`、`role_attribute`、`growth_template`、`growth_rule` 六张表。脚本使用 `CREATE IF NOT EXISTS` 和 `INSERT IGNORE`，不会清空已有数据；不要将其当作数据库迁移工具。

在 MySQL 命令行登录，输入管理员密码后执行：

```text
mysql -u root -p --default-character-set=utf8mb4
mysql> source C:/实际项目目录/sql/init.sql;
```

使用有相应库权限的专用账户运行应用，例如由数据库管理员执行（请替换密码）：

```sql
CREATE USER 'game_growth_app'@'localhost' IDENTIFIED BY '替换为你的强密码';
GRANT SELECT, INSERT, UPDATE, DELETE ON game_growth_system.* TO 'game_growth_app'@'localhost';
```

复制 `config/db.example.properties` 为 `config/db.local.properties`，填写 URL、账户与密码。本次本机已完成初始化和专用账户配置。个人 root 密码不会写入项目；本机应用配置、运行目录、日志和构建目录均已加入 `.gitignore`。

配置加载优先级：`GAME_DB_URL`、`GAME_DB_USER`、`GAME_DB_PASSWORD` 环境变量覆盖本地属性。属性文件路径可通过 `-Dgame.config=绝对路径` 或 `GAME_CONFIG` 环境变量传入。随附启动脚本自动传入本项目本地配置路径。

## 手动部署 WAR

```powershell
$env:JAVA_HOME='C:\Program Files\Eclipse Adoptium\jdk-21.0.13.9-hotspot'
mvn clean package
```

生成 `target/game-growth-system.war`。复制至 Tomcat 9 的 `webapps`，并为 Tomcat 设置 `GAME_CONFIG` 为本地数据库配置的绝对路径。使用 JDK 21 启动 Tomcat。默认容器使用 8080 时，访问 `http://localhost:8080/game-growth-system/`。

## 功能与演示步骤

1. 登录，查看工作台的角色、属性、方案与规则数量。
2. 角色管理支持新增、编辑、删除和名称/编码搜索；在编辑中分配成长方案。
3. 属性字典维护 HP、ATK、DEF、SPD、CRIT 等定义；删除属性会级联移除关联基础值和成长规则，页面会请求确认。
4. 基础属性配置：选择角色，修改等级 1 数值，保存。
5. 成长方案：创建/编辑方案，进入「配置规则」填写各属性的每级成长值。保存采用单个事务，输入错误时完整回滚。
6. 成长计算器：选择晨曦剑士，目标等级输入 10，点击计算。
7. 退出登录后访问管理页会返回登录页。

晨曦剑士等级 10 的预期结果：

| 属性 | 基础 | 每级成长 | 最终 |
|---|---:|---:|---:|
| HP | 1200 | 120 | 2280 |
| ATK | 85 | 8.5 | 161.5 |
| DEF | 60 | 6 | 114 |
| SPD | 100 | 1.2 | 110.8 |
| CRIT | 5 | 0.15 | 6.35% |

公式为 `最终属性 = 基础属性 + 每级成长值 × (目标等级 - 1)`，使用 `BigDecimal` 计算，等级限定 1–10000，输入最多 4 位小数。缺少基础值/规则时取 0，未分配方案时每级成长为 0。CRIT 单位为百分点，不自动封顶。

删除方案会清除其规则并将对应角色设为「未分配」；删除角色会清除其基础属性。属性数值限定为非负。所有删除均使用 POST 和 CSRF 校验。

## 验证

2026-11-03 本机实际验证：JDK 21.0.13、MySQL 8.4.13、Tomcat 9.0.123；`mvn clean package` 显示 **BUILD SUCCESS**，5 项单元测试通过，40 项 HTTP 集成检查通过。详见 `build.log`、`verification.log` 和 `VERIFICATION.md`。演示截图位于 `screenshots/`。

```powershell
# 应用已经启动后执行；通过真实 HTTP 和 MySQL 验证，自动清理临时测试记录
powershell -ExecutionPolicy Bypass -File .\scripts\smoke-test.ps1
```

单元测试验证一级数值、小数精度、演示 HP、零成长和等级边界。HTTP 测试覆盖登录失败/成功、全部业务 JSP、角色/属性/方案 CRUD、规则、基础值、成长计算、事务回滚、搜索注入、HTML 转义、CSRF、级联删除、退出与访问控制。

## 代码结构

- `src/main/java/com/game/AppServlet.java`：请求处理、校验与页面数据。
- `Db.java`：JDBC 配置与 PreparedStatement 查询/更新，使用 try-with-resources。
- `SecurityFilter.java`：UTF-8、登录检查、CSRF、响应头。
- `GrowthCalculator.java`：纯 Java 线性计算，可单独测试。
- `src/main/webapp/WEB-INF/views/`：受保护的 JSP 页面。
- `src/main/webapp/assets/`：本地样式和删除确认脚本。

本项目定位为本地演示应用。演示账户使用加盐 SHA-256，不提供密码重置/多管理员管理；部署到公网前应更换演示密码、使用专用密码哈希算法与 HTTPS，并配置容器访问限制。
