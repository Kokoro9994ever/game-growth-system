# 本机交付验证

验证日期：2026-11-03（北京时间）。

- JDK：Temurin 21.0.13。
- Maven：3.10.0，构建过程明确设置 JAVA_HOME 为 JDK 21。
- 数据库：实际 MySQL 8.4.13，game_growth_system 已初始化；应用使用专用最小权限账户。
- 容器：实际 Apache Tomcat 9.0.123，javax.servlet 4.0。
- 最终源码构建：2026-11-03 18:33:26 完成 `mvn clean package`，BUILD SUCCESS。
- 单元测试：5 项，失败 0，错误 0。
- HTTP 集成验证：40 项全部通过，测试临时数据已清理，保留 4 个角色、5 项属性、3 套方案与 15 条规则。
- 浏览器实际登录后检查工作台和计算器，控制台无 error/warn；截图见 screenshots/。
- 浏览器核对晨曦剑士 Lv.10：HP 2280、ATK 161.5、DEF 114、SPD 110.8、CRIT 6.35%。

运行地址：http://localhost:8088/game-growth-system/

演示账户：admin / Admin@123。

## IDEA 操作记录与限制

已定位本机 `C:\Users\user\Desktop\IntelliJ IDEA Community Edition 2025.2.1\IntelliJ IDEA Community Edition 2025.2.1\bin\idea64.exe`，使用 PowerShell `Start-Process` 并传入项目目录发出打开请求；已确认 IDEA 进程存在。

当前工具会话无法可靠激活/操作 IDEA 桌面窗口，未能确认项目在 IDEA 中成功导入，因此不声称已经通过 IDEA 图形界面运行或检查。按要求的替代方式提供 Maven WAR 结构、JDK 21 设置说明和 `.run/` 下两个 IDEA Maven 运行配置。实际构建、部署和功能验证通过本机 Maven、Tomcat、MySQL 与浏览器完成。

## 交付文件

- 完整源码：当前项目目录或同级 game-growth-system-source.zip。
- WAR：target/game-growth-system.war（同级输出目录另有一份便于取用）。
- 数据库：sql/init.sql。
- 操作说明：README.md。
- 构建证据：build.log。
- 联调证据：verification.log。

源码 ZIP 包排除了本机数据库密码、运行目录和构建缓存。取走源码 ZIP 到其他机器后，请按 README 初始化数据库并配置连接。
