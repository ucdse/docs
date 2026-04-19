# Frontend

在本项目（**Dublin Bike Usage Prediction**）的前端工程化起步阶段，我最初对于如何选择React项目脚手架感到困惑——传统的`create-react-app`已经停止维护，而Vite等新型构建工具是否适用于生产级项目尚不确定。在与AI的讨论中，它帮我系统对比了`Vite`与`create-react-app`在构建速度、热更新机制以及React 19兼容性方面的差异，最终建议我通过`npm create vite@latest`命令搭建基于`React 19`加`TypeScript`的项目骨架。在此基础上，AI还指导我完成了React函数式组件与Hooks范式下的核心概念学习——从`useState`、`useEffect`的基本用法到`useContext`的全局状态传播，再到路由管理中`React Router v6`的`loader`与`action`声明式数据流，这些由AI逐步梳理的学习路径使我快速跨越了从类组件到函数式组件的思维转换。

在**地图交互页面**的开发过程中，多模态路线导航查询的实现一度成为瓶颈。我需要同时处理`google.maps.places.Autocomplete`的地点自动补全、步行—骑行—步行三段式多色折线的并发渲染，以及视口的智能缩放。AI协助我构建了基于Autocomplete的响应式循环，从TypeScript类型层面理清了前端状态管理的层次关系，并帮助我实现了三段不同颜色折线在同一条路线上的叠加渲染，同时为`fitBounds`方法封装了自动缩放逻辑，使地图视口能够智能地对所有途经点进行居中展示。

在**AI聊天页面**的开发中，我最初采用的是浏览器原生`EventSource`API来实现服务端推送流式响应，但很快发现`EventSource`不支持在请求头中携带JWT Token，这与项目中基于`axios`拦截器统一注入身份认证令牌的安全机制完全冲突。我向AI描述了这一兼容性困境后，它提供了一个深度重构方案：引入`@microsoft/fetch-event-source`库替换原生`EventSource`，在`api/chat.ts`中通过`fetchEventSource`重新构建网络层，使其能够在请求头中自定义注入`Authorization: Bearer`令牌，同时结合`getToken()`函数实现Token的动态刷新。这一方案完美地在保持身份认证的前提下实现了Server-Sent Events（SSE）逐字打字机式的流式输出效果。在此基础上，结合AI建议的本地状态拦截策略，我还实现了跨页面刷新后的会话状态持久化。

# Backend

在后端架构的设计与实现中，我首先面对的是Flask应用的入口启动设计。项目通过`run.py`作为开发环境的启动入口，调用`create_app()`创建应用实例后以`debug=True`模式运行；而生产部署则依赖`wsgi.py`作为WSGI服务器的接入点，同样调用`create_app()`但剥离了调试模式。AI帮我明确了这两层入口的职责分离——`run.py`面向开发时热重载，`wsgi.py`面向Gunicorn等生产服务器的标准调用规范。

在Flask应用工厂模式的搭建上，AI帮助我梳理了`create_app()`工厂函数的组织方式，指导我将扩展初始化（`SQLAlchemy`、`Flask-Migrate`、`Flask-Mail`、`JWT`等）集中在`app/__init__.py`中统一管理，使应用可以根据不同配置环境灵活创建实例。尤其是在集成核心**机器学习预测流程**与数据库的环节，我遭遇了严重的冷启动延迟问题——我最初的"懒加载"设计导致首次API请求时后端需要动态加载大量数据到模型中进行推理，响应时间极长。AI分析了我的部署环境后，提出了利用Flask App上下文生命周期在应用启动时执行"预热"的架构重构方案：在`prediction_service.py`中将模型和特征数据提前加载到内存中，通过`copy-on-write`机制在内存层面避免重复拷贝。优化后，后端能够在收到请求时瞬时将预测特征送入模型推理，并将结果无卡顿地返回给前端。

在**数据库表设计**方面，AI协助我完成了从需求到SQLAlchemy Model的完整建模。`Station`表以站号`number`为主键，包含`name`、`address`、`position`（经纬度）、`banking`（是否有支付终端）、`bonus`等字段；`Availability`表通过外键关联`station.number`，记录实时可用车辆数与空停车位数；`Weather`表存储天气观测数据；`User`表采用`email_hash`与`password_hash`的加密存储方案，并设置`verified`字段配合邮件验证流程；`Session`表和`ChatHistory`表则支撑了LangChain的`SQLChatMessageHistory`持久化对话机制。AI在每一个表的设计过程中都提示我注意外键约束、索引策略以及字段类型的选择，避免了我早期方案中因`station.number`字段类型不匹配而导致的外键关联失败。

在**数据库迁移**方案的选择上，我最初倾向于手动编写SQL脚本来管理schema变更，但AI帮我分析了`Flask-Migrate`（基于Alembic）的自动化迁移工作流后，我转而采用了这一方案。AI详细指导了我如何执行`flask db init`初始化迁移目录、`flask db migrate -m "description"`自动生成迁移脚本、`flask db upgrade`将变更应用到数据库，以及如何在`migrations/`目录中管理和审查自动生成的版本文件。这一选择使得团队在协同开发中避免了手动SQL带来的版本漂移问题。

在**LLM对话集成**方面，AI帮助我在`chat_service.py`中完成了LangChain与阿里云通义千问（`qwen-plus`）模型的对接，实现了流式SSE输出。同时，`JWT`认证模块的自定义Token生成与验证逻辑，以及`Flask-Mail`实现的邮箱验证流程，也都在AI的指导下完成——从异步邮件发送的配置到激活链接的安全签名，AI逐一协助我排除了开发中的各类障碍。

# Scraper

在数据采集器的设计与实现中，AI协助我构建了一套基于双线程守护进程的定时架构。`main_scraper.py`中采用`threading.Thread(daemon=True)`创建了两个常驻后台线程：站点线程每5分钟从`JCDecaux API`拉取Dublin自行车站实时数据，天气线程每1小时从`OpenWeatherMap API`获取都柏林天气观测信息。AI帮我理清了两个线程在共享数据库连接时的并发安全问题，指导我设计了合理的线程调度策略，避免了因API请求重叠而导致的资源竞争。

在**与数据库的通信**方面，Scraper作为独立于Flask应用的组件，需要自行维护数据库连接。AI指导我在`database.py`中搭建了独立的`SQLAlchemy`引擎，与Flask应用的`db`实例完全解耦，同时确保ORM模型定义（`models.py`）在两端保持一致。在会话管理上，AI帮助我实现了严格的`commit/rollback`异常处理流程——每次数据写入后确保显式提交，一旦发生异常则立即回滚并记录错误日志，避免了脏数据的写入。

在**容错与重试机制**方面，我最初对API请求失败的处理过于简单，仅进行了基本的异常捕获。AI指出在面对网络抖动和API速率限制时，这种设计会导致数据断层。它协助我为每个API调用添加了指数退避重试逻辑，结合对HTTP状态码的细粒度判断（区分可重试的5xx错误与不可重试的4xx客户端错误），使Scraper在面对短暂网络故障或API限流时能够自动恢复，保证了数据采集链路的持续稳定运行。