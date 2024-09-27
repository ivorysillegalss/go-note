MINIOB

启动的时候创建线程池。将Libevent事件监听包装为任务(std::bind())。放入线程池中执行（在程序运行整个过程中，此线程池中的线程都会执行此监听任务）。线程池的配置类似Java中的CachedThreadPool。

启动事件监听机制后，建立网络连接（sock or TCP）

包装Communicator操作对象 将事件信息（event） 线程池对象都放到 一个回调函数包装对象中 存放在map当中 以Communicator为键，包装对象为值，

Communicator对象的主要作用是，初始化连接，并读取事件。

```c++
/**
 * @brief SQL请求的处理器
 * @ingroup SQL
 */
class SqlTaskHandler
{
public:
  SqlTaskHandler()          = default;
  virtual ~SqlTaskHandler() = default;

  /**
   * @brief 指定连接上有数据可读时就读取消息然后处理
   * @details 步骤包含接收请求、处理请求，然后返回应答
   * @param communicator 连接对象
   * @return RC 如果返回失败，就要断开连接
   */
  RC handle_event(Communicator *communicator);

  RC handle_sql(SQLStageEvent *sql_event);

private:
  SessionStage    session_stage_;      /// 会话阶段
  QueryCacheStage query_cache_stage_;  /// 查询缓存阶段
  ParseStage      parse_stage_;        /// 解析阶段。将SQL解析成语法树 ParsedSqlNode
  ResolveStage    resolve_stage_;      /// 解析阶段。将语法树解析成Stmt(statement)
  OptimizeStage optimize_stage_;  /// 优化阶段。将语句优化成执行计划，包含规则优化和物理优化
  ExecuteStage  execute_stage_;   /// 执行阶段
};
```

```c++
/**
 * @brief 表示一个SQL请求
 *
 */
class SessionEvent
{
public:
  SessionEvent(Communicator *client);
  virtual ~SessionEvent();

  Communicator *get_communicator() const;
  Session      *session() const;

  void set_query(const string &query) { query_ = query; }

  const string &query() const { return query_; }
  SqlResult    *sql_result() { return &sql_result_; }
  SqlDebug     &sql_debug() { return sql_debug_; }

private:
  Communicator *communicator_ = nullptr;  ///< 与客户端通讯的对象
  SqlResult     sql_result_;              ///< SQL执行结果
  SqlDebug      sql_debug_;               ///< SQL调试信息
  string        query_;                   ///< SQL语句
};
```

```c++
  // 记录当前的操作
  // 1. 将当前的会话记录到线程变量中(当前线程)
  Session::set_current_session(sev->session());
  // 通过sessionEvent获取到当前的session 绑定当前的session和正在处理的request
  sev->session()->set_current_request(sev);
  SQLStageE vent sql_event(sev, sql);
```

注释存疑

当前理解 

SessionEvent是以会话为单位的。每一次处理请求的时候都会将当前Session和正在处理的Request绑定在一起，以单次请求为单位进行更新。

SqlTaskHandler是处理sql的工厂，整个sql的处理从头到底说白了也是责任链，而SqlTaskHandler则是启动这个责任链的工厂。

Communicator连接的信息，包括当前事件的信息。从一开始连接的初始化，之后读取事件，在对事件信息进行前置检查无误后，将其中的信息组装成为SessionEvent。

SqlStageEvent记录在执行sql语句每个阶段时的不同数据，类似SessionEvent，但个人理解颗粒度不同。前者是准确记录在每一个执行阶段时的数据，后者则是只记录在整个sql执行时的状态，不会记录具体的错误数据等。

unorder_map
