# websocket

**czc**

## 一、WebSocket 后端

位于ws包中

###  1、实体类

#### (1). 通信信息类	`ChatMessage<T>`

```java
@Data
public class ChatMessage<T> {
    /**
     * 消息类型
     */
    private MessageTypeEnum type;
    /**
     * 消息发送者
     */
    private String sender;
    /**
     * 消息接收者
     */
    private Set<String> receivers;
    
    private T data;
}
```

**MessageTypeEnum** : 指此通信信息的类型 即pk中每一个阶段的不同信息类型 eg匹配中 详见下文

**sender** : 指此通信信息的发出方 这些响应信息是由谁来发出的

**Set<Sting> receivers** : 指此通信信息的接收方 这些信息是由谁来接受 这个谁可以是单个人也可以是一群人

**T data** :  指此信息中包含的实际数据 由 **MessageTypeEnum** 来确认信息的类型，**data** 表示信息的数据



#### (2). 消息类型枚举类   `MessageTypeEnum`

```java
/**
 * 消息类型

 */
public enum MessageTypeEnum {
    /**
     * 匹配对手
     */
    MATCH_USER,
    /**
     * 游戏开始
     */
    PLAY_GAME,
    /**
     * 游戏结束
     */
    GAME_OVER,
}
```

此枚举类的三个信息分别代表在游戏的不同阶段中，三个不同的状态（不同的状态下发出不同的信息）



#### (3). 回答情况 `AnswerSituation`

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Component
public class AnswerSituation {
    private ArrayList<String> selfAnswerSituations;
}
```

此实体类代表着用户pk答题的情况 每一个用户 在每一局游戏中 每一个用户 都会有一个自己的 **AnswerSituation**



#### (4). 单局游戏回答情况 `RoomAnswerSituation`

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class RoomAnswerSituation {
    private String userId;
    private String receiver;
    private AnswerSituation selfAnswerSituations;
    private AnswerSituation opponentAnswerSituations;
}
```

其中一种响应信息的类型**(GAME_OVER)**所对应的**T data**数据，包含每一局游戏中双方的回答情况，以及对方和自己的id标识（便于前端判断）



#### (5). 用户的信息 `UserMatchInfo`

```java
@Data
public class UserMatchInfo {
    private String userId;
    private Integer score;
}
```

包含用户的id及分数 在响应信息类型为 **(TO_PLAY)** 时 传输对方用户以及自己的分数信息

​									在响应信息类型为**(MATCH_USER)**时 初始化双方的分数（总分）及id信息



#### (6). 游戏对局信息 **`GameMatchInfo`**

```java
@Data
public class GameMatchInfo {
    // TODO UserMatchInfo 后期按需扩充dan等属性
    private UserMatchInfo selfInfo;
    private UserMatchInfo opponentInfo;
    private List<Question> questions;
//    用户名
    private String selfUsername;
    private String opponentUsername;
//    头像
    private String selfPicAvatar;
    private String opponentPicAvatar;
}
```

在匹配到对手之后 发送的**chatMessage**中的 **T data** 数据类型

包含在这一轮游戏中 自己以及对方的数据信息 

此处包含**双方 id  用户名  头像   问题** 以及初始化后的用户总分信息（0）

可进一步优化封装为 **双方游戏信息类 + 问题信息类**



#### (7). 分数选项的反馈信息 `ScoreSelectedInfo`

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ScoreSelectedInfo {
    private UserMatchInfo userMatchInfo;
    private String userSelectedAnswer;
}
```

包含用户信息 UserMatchInfo 及 **userSelectedAnswer** 

顾名思义 其为用户每一题所选择的答案 此程序中记录的是问题的答案 （数组中的具体值）

可优化改变换成记录数组对应索引etc



### 2、异常处理类

#### (1). 错误码定义的统一接口

```java
public interface IServerError {

    /**
     * 返回错误码
     *
     * @return 错误码
     */
    Integer getErrorCode();

    /**
     * 返回错误详细信息
     *
     * @return 错误详细信息
     */
    String getErrorDesc();
}
```



#### (2). 运行时错误

```java
public enum GameServerError implements IServerError {

    /**
     * 枚举型错误码
     */
    WEBSOCKET_ADD_USER_FAILED(4018, "用户进入匹配模式失败"),
    MESSAGE_TYPE_ERROR(4019, "websocket 消息类型错误"),
    ;

    private final Integer errorCode;
    private final String errorDesc;

    GameServerError(Integer errorCode, String errorDesc) {
        this.errorCode = errorCode;
        this.errorDesc = errorDesc;
    }

    @Override
    public Integer getErrorCode() {
        return errorCode;
    }

    @Override
    public String getErrorDesc() {
        return errorDesc;
    }
}
```

枚举错误类型及错误码



#### (3). 运行时异常

```java
public class GameServerException extends RuntimeException {

    private Integer code;

    private String message;

    public GameServerException(GameServerError error) {
        super(error.getErrorDesc());
        this.code = error.getErrorCode();
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }
}
```

统一规范抛出的异常及其错误类型



### 3、游戏状态枚举类

```java
public enum EnumRedisKey {

    /**
     * userOnline 在线状态
     */
    USER_STATUS,
    /**
     * userOnline 匹配信息
     */
    USER_MATCH_INFO,
    /**
     * 房间
     */
    ROOM;

    public String getKey() {
        return this.name();
    }
}
```

使用枚举类规定游戏中玩家的状态 并且使用redis进行存储



### 4、ws主类

此部分代码分为多板块

此类完整代码见附

#### (1). 注入的相关类

```java
private Session session;

private String userId;

static MatchCacheUtil matchCacheUtil;
// 修改 查看用户的在线状态 客户端存储状态工具类

static Lock lock = new ReentrantLock();
static Condition matchCond = lock.newCondition();
// 锁 防止并发异常情况

static QuestionService questionService;
// 题目业务类 用于在题库中随机获取题目

static CompetitionService competitionService;
// 比赛业务类 用于获取用户段位信息等 pk相关的功能信息

static UserService userService;
// 用户业务类 用于获取用户个人信息

static AnswerSituationUtil answerSituationUtil;
// 存储用户单轮答案信息 工具类


@Autowired
public void setQuestionService(QuestionService questionService) {
    ChatWebsocket.questionService = questionService;
}

@Autowired
public void setQuestionService(CompetitionService competitionService) {
    ChatWebsocket.competitionService = competitionService;
}

@Autowired
public void setQuestionService(UserService userService) {
    ChatWebsocket.userService = userService;
}

@Autowired
public void setMatchCacheUtil(MatchCacheUtil matchCacheUtil) {
    ChatWebsocket.matchCacheUtil = matchCacheUtil;
}

@Autowired
public void setAnswerSituationUtil(AnswerSituationUtil answerSituationUtil) {
    ChatWebsocket.answerSituationUtil = answerSituationUtil;
}

// 类单例模式注入相关业务工具类
```

详细工具类见下一部分



#### (2). 主要架构

```java
@OnOpen
public void onOpen(@PathParam("userId") String userId, Session session) {
    System.out.println(session);
    log.info("ChatWebsocket open 有新连接加入 userId: {}", userId);
    this.userId = userId;
    this.session = session;
    matchCacheUtil.addClient(userId, this);

    log.info("ChatWebsocket open 连接建立完成 userId: {}", userId);
}
```

**onOpen:** 建立ws连接时调用 使用工具类存储当前客户端的 **id和webSocket对象** 入redis中 便于后期调用

---

```java
@OnError
public void onError(Session session, Throwable error) {

    log.error("ChatWebsocket onError 发生了错误 userId: {}, errorMessage: {}", userId, error.getMessage());

    matchCacheUtil.removeClinet(userId);
    matchCacheUtil.removeUserOnlineStatus(userId);
    matchCacheUtil.removeUserFromRoom(userId);
    matchCacheUtil.removeUserMatchInfo(userId);

    log.info("ChatWebsocket onError 连接断开完成 userId: {}", userId);
}

@OnClose
public void onClose() {
    log.info("ChatWebsocket onClose 连接断开 userId: {}", userId);

    matchCacheUtil.removeClinet(userId);
    matchCacheUtil.removeUserOnlineStatus(userId);
    matchCacheUtil.removeUserFromRoom(userId);
    matchCacheUtil.removeUserMatchInfo(userId);

    log.info("ChatWebsocket onClose 连接断开完成 userId: {}", userId);
}
```

**OnError:** 遇到异常时退出并调用 将用户信息经过工具类 从redis中移除

**OnClose:** 同理 断开连接时调用 正常使用时机为 **用户pk结束**

---

```java
@OnMessage
public void onMessage(String message, Session session) {

    log.info("ChatWebsocket onMessage userId: {}, 来自客户端的消息 message: {}", userId, message);

    JSONObject jsonObject = JSON.parseObject(message);
    MessageTypeEnum type = jsonObject.getObject("type", MessageTypeEnum.class);

    log.info("ChatWebsocket onMessage userId: {}, 来自客户端的消息类型 type: {}", userId, type);

    if (type == MessageTypeEnum.MATCH_USER) {
        matchUser(jsonObject);
    } else if (type == MessageTypeEnum.PLAY_GAME) {
        toPlay(jsonObject);
    } else if (type == MessageTypeEnum.GAME_OVER) {
        gameover(jsonObject);
    } else {
        throw new GameServerException(GameServerError.WEBSOCKET_ADD_USER_FAILED);
    }

    log.info("ChatWebsocket onMessage userId: {} 消息接收结束", userId);
}
```

**OnMessage:** 收到客户端信息时调用 **ws实现功能关键部分！**

此处将整个游戏过程分割为三部分 分别为 **匹配玩家  游戏中  游戏结束**

#### (3). 实现逻辑方法

##### **1). 群发消息 `sendMessageAll`**

```java
/**
 * 群发消息
 */
private void sendMessageAll(MessageReply<?> messageReply) {

    log.info("ChatWebsocket sendMessageAll 消息群发开始 userId: {}, messageReply: {}", userId, JSON.toJSONString(messageReply));

    Set<String> receivers = messageReply.getChatMessage().getReceivers();
    for (String receiver : receivers) {
        ChatWebsocket client = matchCacheUtil.getClient(receiver);
        client.session.getAsyncRemote().sendText(JSON.toJSONString(messageReply));
    }

    log.info("ChatWebsocket sendMessageAll 消息群发结束 userId: {}", userId);
}
```

此方法根据形参 确定信息的发出者 并从工具类中 **获取对方的webSocket对象** 异步发送到对方的客户端中

##### **2). 用户随机匹配对手 `matchUser`**

```java
    /**
     * 用户随机匹配对手
     */
    @SneakyThrows
// 抛出异常注解
    private void matchUser(JSONObject jsonObject) {

        log.info("ChatWebsocket matchUser 用户随机匹配对手开始 message: {}, userId: {}", jsonObject.toJSONString(), userId);

        MessageReply<GameMatchInfo> messageReply = new MessageReply<>();
        ChatMessage<GameMatchInfo> result = new ChatMessage<>();
        result.setSender(userId);
        result.setType(MessageTypeEnum.MATCH_USER);

        lock.lock();
        try {
            // 设置用户状态为匹配中
//            TODO 修改工具类的类型 使他不只能存储id 也能存储玩家的段位
            matchCacheUtil.setUserInMatch(userId);
            matchCond.signal();
        } finally {
            lock.unlock();
        }

        // 创建一个异步线程任务，负责匹配其他同样处于匹配状态的其他用户
        Thread matchThread = new Thread(() -> {
            boolean flag = true;
            String receiver = null;
            while (flag) {
                // 获取除自己以外的其他待匹配用户
                lock.lock();
                try {
                    // 当前用户不处于待匹配状态 直接返回
                    if (matchCacheUtil.getUserOnlineStatus(userId).compareTo(StatusEnum.IN_GAME) == 0
//                            观察当前用户是否游戏中状态
                            || matchCacheUtil.getUserOnlineStatus(userId).compareTo(StatusEnum.GAME_OVER) == 0) {
//                        观察当前用户是否游戏结束 结算的状态
                        log.info("ChatWebsocket matchUser 当前用户 {} 已退出匹配", userId);
                        return;
                    }
                    // 当前用户取消匹配状态
                    if (matchCacheUtil.getUserOnlineStatus(userId).compareTo(StatusEnum.IDLE) == 0) {
                        // 当前用户取消匹配
                        messageReply.setCode(MessageCode.CANCEL_MATCH_ERROR.getCode());
                        messageReply.setDesc(MessageCode.CANCEL_MATCH_ERROR.getDesc());
//                        设定返回信息 将从匹配中的状态该为待匹配
                        Set<String> set = new HashSet<>();
                        set.add(userId);
                        result.setReceivers(set);
                        result.setType(MessageTypeEnum.CANCEL_MATCH);
                        messageReply.setChatMessage(result);
                        log.info("ChatWebsocket matchUser 当前用户 {} 已退出匹配", userId);
//                        发送返回信息
                        sendMessageAll(messageReply);
                        return;
                    }

//                    从room中获取对手的对象 这个receiver是对手的id
//                    可以在这里下手脚 加一个while循环 判断直至找到相同段位的用户为止
//                    TODO 直接在setUserMatchRoom那块 加入玩家的段位信息 需要改动工具类
                    String userDan = competitionService.showPlayersExtraDan(Integer.parseInt(userId));
                    while (true) {
                        receiver = matchCacheUtil.getUserInMatchRandom(userId);
//                        这里必须把判空放在上面 否则如果是队列中没有人在匹配 却给receiver调用了Integer.parseInt(receiver)方法 会报空指针
                        if (Objects.isNull(receiver))
                            break;
                        else if (userDan.equals(competitionService.showPlayersExtraDan(Integer.parseInt(receiver))))
                            break;
                    }
                    if (receiver != null) {
                        // 对手不处于待匹配状态
                        if (matchCacheUtil.getUserOnlineStatus(receiver).compareTo(StatusEnum.IN_MATCH) != 0) {
                            log.info("ChatWebsocket matchUser 当前用户 {}, 匹配对手 {} 已退出匹配状态", userId, receiver);
                        } else {
//                            进了这个else就表明用户已经匹配到状态正常的对手了
//                            设定对手的基本信息
                            matchCacheUtil.setUserInGame(userId);
                            matchCacheUtil.setUserInGame(receiver);
//                            将对手放入房间中 （指定唯一user）
                            matchCacheUtil.setUserInRoom(userId, receiver);
                            flag = false;
//                            匹配到了 令flag为false 跳出while循环
//                            此次匹配结束 进入开打状态
                        }
                    } else {
                        // 如果当前没有待匹配用户，进入等待队列
                        try {
                            log.info("ChatWebsocket matchUser 当前用户 {} 无对手可匹配", userId);
                            matchCond.await();
                        } catch (InterruptedException e) {
                            log.error("ChatWebsocket matchUser 匹配线程 {} 发生异常: {}",
                                    Thread.currentThread().getName(), e.getMessage());
                        }
                    }
                } finally {
                    lock.unlock();
                }
            }

            log.info("已找到玩家 双方分别为" + userId + "和" + receiver);

            UserMatchInfo senderInfo = new UserMatchInfo();
            UserMatchInfo receiverInfo = new UserMatchInfo();
            senderInfo.setUserId(userId);
            senderInfo.setScore(0);
            receiverInfo.setUserId(receiver);
            receiverInfo.setScore(0);
//            两个对象分别记录两个玩家的得分

            matchCacheUtil.setUserMatchInfo(userId, JSON.toJSONString(senderInfo));
            matchCacheUtil.setUserMatchInfo(receiver, JSON.toJSONString(receiverInfo));
//            初始化接下来pk中玩家的信息 (每一道题提交答案完成都会 刷新一次对应的得分)

            GameMatchInfo gameMatchInfo = new GameMatchInfo();

//            获取玩家段位 根据玩家段位来获取题目
            String dan = competitionService.showPlayersDan(Integer.parseInt(userId));
            List<Question> questions = questionService.getCompetitionQuestionsByDan(dan);

            gameMatchInfo.setQuestions(questions);
            gameMatchInfo.setSelfInfo(senderInfo);
            gameMatchInfo.setOpponentInfo(receiverInfo);
//            一次性获取所有题目
//            存入此次对战中的当前玩家 对方 题目
//            一个GameMatchInfo就代表一个玩家的对象

//            新增 存入此次pk中对方的用户名 头像
            UserWithValue userWithValue = userService.showUser(Integer.parseInt(userId));
            String username = userWithValue.getUser().getUsername();
            String userPic = userWithValue.getUserValue().getPic();
            gameMatchInfo.setSelfUsername(username);
            gameMatchInfo.setSelfPicAvatar(userPic);

            UserWithValue receiverValue = userService.showUser(Integer.parseInt(receiver));
            String receiverUsername = receiverValue.getUser().getUsername();
            String opponentPic = receiverValue.getUserValue().getPic();
            gameMatchInfo.setOpponentUsername(receiverUsername);
            gameMatchInfo.setOpponentPicAvatar(opponentPic);

            messageReply.setCode(MessageCode.SUCCESS.getCode());
            messageReply.setDesc(MessageCode.SUCCESS.getDesc());
//            确认返回信息的类型以及数据

            result.setData(gameMatchInfo);
            Set<String> set = new HashSet<>();
            set.add(userId);
            result.setReceivers(set);
            result.setType(MessageTypeEnum.MATCH_USER);
            messageReply.setChatMessage(result);
            sendMessageAll(messageReply);

//            自己的传给自己的 对面的传给对面的
            gameMatchInfo.setSelfInfo(senderInfo);
            gameMatchInfo.setOpponentInfo(receiverInfo);

            result.setData(gameMatchInfo);
            set.clear();
            set.add(receiver);
            result.setReceivers(set);
            messageReply.setChatMessage(result);

            sendMessageAll(messageReply);

            log.info("ChatWebsocket matchUser 用户随机匹配对手结束 messageReply: {}", JSON.toJSONString(messageReply));

        }, MATCH_TASK_NAME_PREFIX + userId);
        matchThread.start();
    }
```

基本思路为 改变玩家状态为匹配中 并且匹配相同状态 相同段位的对手 **(段位可按需增删)**

匹配的过程是异步多线程匹配 若没有相同状态的对手 则线程沉睡 直至有对手匹配为止 **(此处可以设置匹配超时时间进行优化)**

确认对手状态无误后 根据id获取双方信息 此轮pk中题目

最后通过 **sendMessageAll** 的方法将信息发送给自己与对方

执行步骤见代码注释

##### **3). 游戏中 `toPlay`**

```java
    /**
     * 游戏中
     */
    @SneakyThrows
    public void toPlay(JSONObject jsonObject) {
//        每一道题提交了都会重新执行一次这个方法
//        (由答题方 执行)
        log.info("ChatWebsocket toPlay 用户更新对局信息开始 userId: {}, message: {}", userId, jsonObject.toJSONString());

        MessageReply<ScoreSelectedInfo> messageReply = new MessageReply<>();
//        返回的信息类型

//        下面的这个思路就是 从房间中找到对方 并且发送自己的分数更新信息给他
        ChatMessage<ScoreSelectedInfo> result = new ChatMessage<>();
        result.setSender(userId);
        String receiver = matchCacheUtil.getUserFromRoom(userId);
//        从房间中找出对面的对手是谁 发信息给他
        Set<String> set = new HashSet<>();
        set.add(receiver);
        result.setReceivers(set);
        result.setType(MessageTypeEnum.PLAY_GAME);
//        设置消息的发送方 和 接收方 以及消息类型 (游戏中)

//        获取新的得分 并且重新赋值给当前的user   (当前的user就是得分的那个)
        UserMatchChoice userMatchChoice = jsonObject.getObject("data", UserMatchChoice.class);
        Integer newScore = userMatchChoice.getUserScore();
        String userSelectedAnswer = userMatchChoice.getUserSelectedAnswer();

//        获取answerSituation对象 此对象中是所有正在游戏中的用户的回答信息 暂时存在这里
        answerSituationUtil.addAnswer(userId, userSelectedAnswer);

        UserMatchInfo userMatchInfo = new UserMatchInfo();
        userMatchInfo.setUserId(userId);
        userMatchInfo.setScore(newScore);

//        setUserMatchInfo所改变的数据是 同一时刻所有对战的用户的信息
//        在这里set是根据当前用户的id
//        重新设置一下对应的用户对战信息
        matchCacheUtil.setUserMatchInfo(userId, JSON.toJSONString(userMatchInfo));

//        设置响应数据的类型
//        更新 同时发送对面所选的选项
        result.setData(new ScoreSelectedInfo(userMatchInfo, userSelectedAnswer));
        messageReply.setCode(MessageCode.SUCCESS.getCode());
        messageReply.setDesc(MessageCode.SUCCESS.getDesc());
        messageReply.setChatMessage(result);

//        返回包含当前用户的新信息的响应数据
        sendMessageAll(messageReply);

        log.info("ChatWebsocket toPlay 用户更新对局信息结束 userId: {}, userMatchInfo: {}", userId, JSON.toJSONString(userMatchInfo));
    }
```

 pk中 每一道题答完后 客户端往服务器发起的请求类型

主要思路是 根据信息的发出方 从redis中的房间机制获取他的对手 **(在匹配的时候会将一轮对战中双方的id存入redis 可理解成放入房间 防止第三者客户端进入 对pk过程进行干扰)**  之后将答题情况发送给对方

##### **4). 游戏结束 `gameover`**

```java
    /**
     * 游戏结束
     */
    public void gameover(JSONObject jsonObject) {

        log.info("ChatWebsocket gameover 用户对局结束 userId: {}, message: {}", userId, jsonObject.toJSONString());

//        设置响应数据类型
        MessageReply<RoomAnswerSituation> messageReply = new MessageReply<>();

//        设置响应数据 改变玩家的状态
        ChatMessage<RoomAnswerSituation> result = new ChatMessage<>();
        result.setSender(userId);
        String receiver = matchCacheUtil.getUserFromRoom(userId);
        result.setType(MessageTypeEnum.GAME_OVER);
        lock.lock();
        try {
//            设定用户为游戏结束的状态
            matchCacheUtil.setUserGameover(userId);
            if (matchCacheUtil.getUserOnlineStatus(receiver).compareTo(StatusEnum.GAME_OVER) == 0) {
                messageReply.setCode(MessageCode.SUCCESS.getCode());
                messageReply.setDesc(MessageCode.SUCCESS.getDesc());

                //        记录赢了的玩家的ID
                Integer winnerId = jsonObject.getInteger("data");
                boolean isUpdate = competitionService.updateUserStar(winnerId);
                if (!isUpdate) {
                    messageReply.setCode(MessageCode.SYSTEM_ERROR.getCode());
                    messageReply.setDesc(MessageCode.SYSTEM_ERROR.getDesc());
                }

//                获取对战后的对战信息
                AnswerSituation selfAnswer = answerSituationUtil.getAnswer(userId);
                AnswerSituation opponentAnswer = answerSituationUtil.getAnswer(receiver);
                RoomAnswerSituation roomAnswerSituation = new RoomAnswerSituation(userId, receiver, selfAnswer, opponentAnswer);
                result.setData(roomAnswerSituation);

//                设置完结后的返回信息
                messageReply.setChatMessage(result);
                Set<String> set = new HashSet<>();
                set.add(receiver);
                result.setReceivers(set);
                sendMessageAll(messageReply);
//                屎山会出手 两边全部发
                set.clear();
                set.add(userId);
                result.setReceivers(set);
                sendMessageAll(messageReply);

//                移除属于游戏中的游戏信息
                matchCacheUtil.removeUserMatchInfo(userId);
                matchCacheUtil.removeUserFromRoom(userId);

//                移除属于这一次的游戏选择信息
                answerSituationUtil.removeAnswer(userId);
                answerSituationUtil.removeAnswer(receiver);
            }
        } finally {
            lock.unlock();
        }

        log.info("ChatWebsocket gameover 对局 [{} - {}] 结束", userId, receiver);
    }
```

代码中通过前端判断题目数组遍历完成 判断双方分数 发送gameover状态信息 

前端通过比较双方总分 将胜利用户的id发回给后端 即此状态信息中的 **T data**  

后端将其状态转换 并且将对应的双方**此轮对战信息 包括总分 获胜者 对方的回答情况** 发回给双方客户端

至此 一轮pk结束



### 5、配置类及工具类

#### (1). WebSocket配置类

```java
@Configuration
@EnableWebSocket
public class WebsocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter(){
        return new ServerEndpointExporter();
    }
}
```



#### **(2). AnswerSituation 用户答题情况存储工具类**

```java
@Component
//用户答题情况 工具类
public class AnswerSituationUtil {

    private static final Map<String, AnswerSituation> ANSWER_SITUATION = new HashMap<>();

//    新增答案
    public void addAnswer(String userId,String answer){
        boolean isScored = ANSWER_SITUATION.containsKey(userId);
        if (isScored)
            ANSWER_SITUATION.get(userId).getSelfAnswerSituations().add(answer);
        if (!isScored) {
            ArrayList<String> answers = new ArrayList<>();
            answers.add(answer);
            ANSWER_SITUATION.put(userId,new AnswerSituation(answers));
        }
    }

//    获取用户所有答案
    public AnswerSituation getAnswer(String userId){
        boolean isContain = ANSWER_SITUATION.containsKey(userId);
        if (!isContain)
            return null;
        return ANSWER_SITUATION.get(userId);
    }

//    移除答案
    public boolean removeAnswer(String userId){
        boolean isContain = ANSWER_SITUATION.containsKey(userId);
        if (!isContain)
            return false;
        ANSWER_SITUATION.remove(userId);
        return true;
    }
}
```

主要存储 记录 清除 用户每一轮答题的缓存

可优化存进 **redis** 中



#### (3). **MatchCacheUtil 存储用户在线状态及其客户端工具类**

```java
@Component
public class MatchCacheUtil {

    /**
     * 用户 userId 为 key，ChatWebsocket 为 value
     */
    private static final Map<String, ChatWebsocket> CLIENTS = new HashMap<>();

    /**
     * key 是标识存储用户在线状态的 EnumRedisKey，value 为 map 类型，其中用户 userId 为 key，用户在线状态 为 value
     */
    @Resource
    private RedisTemplate<String, Map<String, String>> redisTemplate;

    /**
     * 添加客户端
     */
    public void addClient(String userId, ChatWebsocket websocket) {
        CLIENTS.put(userId, websocket);
    }

    /**
     * 移除客户端
     */
    public void removeClinet(String userId) {
        CLIENTS.remove(userId);
    }

    /**
     * 获取客户端
     */
    public ChatWebsocket getClient(String userId) {
        return CLIENTS.get(userId);
    }

    /**
     * 移除用户在线状态
     */
    public void removeUserOnlineStatus(String userId) {
        redisTemplate.opsForHash().delete(EnumRedisKey.USER_STATUS.getKey(), userId);
    }

    /**
     * 获取用户在线状态
     */
    public StatusEnum getUserOnlineStatus(String userId) {
        Object status = redisTemplate.opsForHash().get(EnumRedisKey.USER_STATUS.getKey(), userId);
        if (status == null) {
            return null;
        }
        return StatusEnum.getStatusEnum(status.toString());
    }

    /**
     * 设置用户为 IDLE 状态
     */
    public void setUserIDLE(String userId) {
        removeUserOnlineStatus(userId);
        redisTemplate.opsForHash().put(EnumRedisKey.USER_STATUS.getKey(), userId, StatusEnum.IDLE.getValue());
    }

    /**
     * 设置用户为 IN_MATCH 状态
     */
    public void setUserInMatch(String userId) {
        removeUserOnlineStatus(userId);
        redisTemplate.opsForHash().put(EnumRedisKey.USER_STATUS.getKey(), userId, StatusEnum.IN_MATCH.getValue());
    }

    /**
     * 随机获取处于匹配状态的用户（除了指定用户外）
     */
    public String getUserInMatchRandom(String userId) {
        Optional<Map.Entry<Object, Object>> any = redisTemplate.opsForHash().entries(EnumRedisKey.USER_STATUS.getKey())
                .entrySet().stream().filter(entry -> entry.getValue().equals(StatusEnum.IN_MATCH.getValue()) && !entry.getKey().equals(userId))
                .findAny();
        return any.map(entry -> entry.getKey().toString()).orElse(null);
    }

    /**
     * 设置用户为 IN_GAME 状态
     */
    public void setUserInGame(String userId) {
        removeUserOnlineStatus(userId);
        redisTemplate.opsForHash().put(EnumRedisKey.USER_STATUS.getKey(), userId, StatusEnum.IN_GAME.getValue());
    }

    /**
     * 设置处于游戏中的用户在同一房间
     */
    public void setUserInRoom(String userId1, String userId2) {
        redisTemplate.opsForHash().put(EnumRedisKey.ROOM.getKey(), userId1, userId2);
        redisTemplate.opsForHash().put(EnumRedisKey.ROOM.getKey(), userId2, userId1);
    }

    /**
     * 从房间中移除用户
     */
    public void removeUserFromRoom(String userId) {
        redisTemplate.opsForHash().delete(EnumRedisKey.ROOM.getKey(), userId);
    }

    /**
     * 从房间中获取用户
     */
    public String getUserFromRoom(String userId) {
        return redisTemplate.opsForHash().get(EnumRedisKey.ROOM.getKey(), userId).toString();
    }

    /**
     * 设置处于游戏中的用户的对战信息
     */
    public void setUserMatchInfo(String userId, String userMatchInfo) {
        redisTemplate.opsForHash().put(EnumRedisKey.USER_MATCH_INFO.getKey(), userId, userMatchInfo);
    }

    /**
     * 移除处于游戏中的用户的对战信息
     */
    public void removeUserMatchInfo(String userId) {
        redisTemplate.opsForHash().delete(EnumRedisKey.USER_MATCH_INFO.getKey(), userId);
    }

    /**
     * 设置处于游戏中的用户的对战信息
     */
    public String getUserMatchInfo(String userId) {
        return redisTemplate.opsForHash().get(EnumRedisKey.USER_MATCH_INFO.getKey(), userId).toString();
    }

    /**
     * 设置用户为游戏结束状态
     */
    public synchronized void setUserGameover(String userId) {
        removeUserOnlineStatus(userId);
        redisTemplate.opsForHash().put(EnumRedisKey.USER_STATUS.getKey(), userId, StatusEnum.GAME_OVER.getValue());
    }
}
```

设置并改变用户的在线状态 **利用redis +  枚举类**

在每一轮游戏结束之后 将其移除缓存



#### (4). **MessageCode 响应码**

```java
@Getter
public enum MessageCode {

    /**
     * 响应码
     */
    SUCCESS(2000, "连接成功"),
    USER_IS_ONLINE(2001, "用户已存在"),
    CURRENT_USER_IS_INGAME(2002, "当前用户已在游戏中"),
    MESSAGE_ERROR(2003, "消息错误"),
    CANCEL_MATCH_ERROR(2004, "用户取消了匹配"),
    SYSTEM_ERROR(2005,"系统错误");

    private final Integer code;
    private final String desc;

    MessageCode(Integer code, String desc) {
        this.code = code;
        this.desc = desc;
    }
}
```

枚举类定义封装ws使用中会出现的错误



#### **(5). MessageTypeEnum 用户状态枚举类**

```java
public enum MessageTypeEnum {
    /**
     * 匹配对手
     */
    MATCH_USER,
    /**
     * 游戏开始
     */
    PLAY_GAME,
    /**
     * 游戏结束
     */
    GAME_OVER,
}
```

枚举类统一定义ws连接中**OnMessage**中会出现的消息类型

前后端根据判断消息类型 判断执行的逻辑 和方法执行的阶段



#### (6). StatusEnum 用户状态枚举类

```java
public enum StatusEnum {

    /**
     * 待匹配
     */
    IDLE,
    /**
     * 匹配中
     */
    IN_MATCH,
    /**
     * 游戏中
     */
    IN_GAME,
    /**
     * 游戏结束
     */
    GAME_OVER,
    ;

    public static StatusEnum getStatusEnum(String status) {
        switch (status) {
            case "IDLE":
                return IDLE;
            case "IN_MATCH":
                return IN_MATCH;
            case "IN_GAME":
                return IN_GAME;
            case "GAME_OVER":
                return GAME_OVER;
            default:
                throw new GameServerException(GameServerError.MESSAGE_TYPE_ERROR);
        }
    }

    public String getValue() {
        return this.name();
    }
}
```

确认用户此时的状态 为 **匹配中 待匹配 游戏中 游戏结束**

- 注：**用户的状态 ≠ 消息类型**



## 二、WebSocket 前端

此处只介绍大致思路与相关js函数

### 0、记录用户id标识

````javaScript
let token = `${sessionStorage.getItem('sign1')}`

axios({
        method: 'GET',
        url: 'http://你说的对:3000/users',
        headers: {
            'token': `${token}`
        }
    })
    .then(res => { // 接口数据
        console.log(res.data);
        let { data } = res.data
        let { user } = data
        // 存储userId到 sessionStorage
        sessionStorage.setItem('userId', user.userId);
    })

.catch(error => { //  处理错误
    console.error(error);
});
````

将数据存储到**sessionStorage**中

### 1、大概流程

````javaScript
// 点击汇总页面的pk,开始链接到websocket
const inner3 = document.querySelector('.panel:nth-of-type(3) .inner')
console.log(inner3)
inner3.addEventListener('click', function() {
        let userId = sessionStorage.getItem('userId')
        console.log(userId)
        connect(userId)
    })
    // 点击开始匹配,进行匹配,并且进入到答题界面
const switchs = document.querySelector('.pk input')
switchs.addEventListener('click', function() {
    // 开始随机匹配
    let userId = sessionStorage.getItem('userId')
    console.log(userId)
    matchUser(userId)
    const fight = document.querySelector('.fight')
    const pk = document.querySelector('.pk')
    let timer = setTimeout(function() {
            pk.style.display = 'none'
            fight.style.display = 'block'
        }, 1000)
        // 渲染自己的信息
    userInfo()
    const questionBox = document.querySelector('.questions>div')
    questionBox.innerHTML = '正在寻找对手...'
})
````

其中调用的**connect**函数是重点 相关逻辑主要靠其实现

**matchUser**是匹配对手的函数

```javascript
function connect(userId){
var socketurl "ws://yourServerUrl:3000/competition/" + userId;
socket = new Websocket(socketurl);
//在此触发OnOpen
//打开事件
socket.onopen = function(){...};
//在此触发OnMessage
//获得消息事件
socket.onmessage = function(msg){...};
//在此触发onclose
//关闭事件
socket.onclose = function(){...};
//在此触发OnError
//发生了错误事件
socket.onerror = function(){...};
```

**connect**函数的大概架构如上

**onMessage**为主要逻辑实现 下一部分分析

**matchUser**内容见下一部分



### 2、OnOpen打开连接及OnError错误处理

```javascript
socket.onopen = function() {
    console.log("websocket 已打开 userId: " + userId);
};
socket.onerror = function() {
    console.log("websocket 发生了错误 userId : " + userId);
}
```

此处不多赘述错误处理和连接，可在对应函数处记录日志或心跳响应等



### 3、OnMessage响应信息及对应请求函数

````javascript
    // 在此触发OnMessage
    //获得消息事件
    socket.onmessage = function(msg) {
        // 此处编写得到消息之后的具体逻辑
        // 提交每一题之后 需要更新玩家的分数和
        // chatMessage就是真正的类 响应数据
        let chatMessage = msg.data;
        chatMessage = JSON.parse(chatMessage);
        let message = chatMessage.chatMessage
        let data = message.data;
        let type = message.type;
            // 以上均为通用代码
            // 正确答案的数组
        let rightAnswer = []
        var serverMsg = "收到服务端信息: " + msg.data;
        console.log(serverMsg);
    };
````

**onMessage** 表示收到服务器消息之后的逻辑 上方步骤将数据转换并赋值给相关变量



#### (1). MATCH_USER

前端发送开始匹配信号

````javascript
// 随机匹配
function matchUser(userId) {
    var chatMessage = {};
    var sender = userId;
    var type = "MATCH_USER";
    chatMessage.sender = sender;
    chatMessage.type = type;
    console.log("用户:" + sender + "开始匹配......");
    socket.send(JSON.stringify(chatMessage));
}
````

无需发送数据



````javaScript
            // 如果接收到的是匹配时 代表此时已经有用户匹配成功 后端发送题目以及 对应的用户信息
        if (type === "MATCH_USER") {
            let gameMatchInfo = message.data;
            console.log(gameMatchInfo)
                // gameMatchInfo对应chatMessage中的T data
            let questionList = gameMatchInfo.questions;
            console.log(questionList)
                // 获取到所有问题的题目及答案
                // questionList表示此次对战中的题库

            // 记录正确答案
            for (let i = 0; i < questionList.length; i++) {
                rightAnswer.push(questionList[i].rightAnswer)
            }
            console.log(rightAnswer)
            let userId = sessionStorage.getItem('userId')
                // 异步函数
            async function wait(questionList, rightAnswer, userId) {
                // 渲染题目到页面上
                await questionDisplay(questionList)
                choose(rightAnswer, userId)
                displayOver(questionList, rightAnswer)
            }
            // 调用异步函数
            wait(questionList, rightAnswer, userId)
                // 同时需要获取到对面的用户的信息
            let selfInfo = gameMatchInfo.selfInfo;

            // selfInfo是自己的信息
            // selfInfo则代表的是GameMatchInfo中selfInfo的属性
            let selfId = selfInfo.userId;
            if (selfId === userId) {
                selfUsername = gameMatchInfo.selfUsername;
                selfPicAvatar = gameMatchInfo.selfPicAvatar;
                let opponentInfo = gameMatchInfo.opponentInfo;
                // opponentInfo则是对面的信息
                // 获取到对面用户的id 头像 名字
                opponentUserId = opponentInfo.userId;
                opponentUsername = gameMatchInfo.opponentUsername;
                opponentPicAvatar = gameMatchInfo.opponentPicAvatar
                sessionStorage.setItem('opponentId', opponentUserId)

                // 渲染对手的信息
                opponentinfo(opponentPicAvatar, opponentUsername)
            } else {
                opponentUserId = selfId
                opponentUsername = gameMatchInfo.selfUsername;
                opponentPicAvatar = gameMatchInfo.selfPicAvatar;
                sessionStorage.setItem('opponentId', opponentUserId)

                    // 渲染对手的信息
                opponentinfo(opponentPicAvatar, opponentUsername)
            }

        }
````

这个阶段代表的是 匹配到用户了 未开始对局 初始化数据 **(双方头像账户名id + 题目信息)**

对应后端发送信息中的 **GameMatchInfo**



#### (2). PLAY_GAME

按照pk中的游戏进程 每一次交互产生在玩家确认题目答案并提交之后 服务器将新的分数变化告知客户端

即 提交答案

````javascript
function commitAnswer(score, userSelectedAnswerIndex, userId) {
    let chatMessage = {};
    let sender = userId;
    let type = "PLAY_GAME";

    // 如果答对了就更新分数 客户端向服务器发起请求
    // 如果没答对 也发送请求 此时也会执行代码 但是分数不会有变化
    // userScore是玩家的积分
    // 发送玩家的新得分以及自己的选项
    var data = {
        userScore: score,
        userSelectedAnswer: userSelectedAnswerIndex
    }
    chatMessage.sender = sender;
    chatMessage.data = data;
    chatMessage.type = type;
    console.log("用户:" + sender + "更新分数为" + data);
    console.log(chatMessage)
    socket.send(JSON.stringify(chatMessage));
    // 这里标记当前用户已经选择了
    isUserSelect = true;
}
````

此处需要发送**用户的总分**以及**用户的选择的答案**



````javaScript
        if (type === "PLAY_GAME") {
            let selfId = message.data.userMatchInfo.userId
            if (selfId === userId) {
                // 更新对方的分数
                const opponentScore = document.querySelector('.opponent-num')
                opponentnum = parseInt(opponentScore.innerHTML)
                let score = message.data.userMatchInfo.score
                opponentScore.innerHTML = score
                
            } else {
                option2 = true
                let opponentId = selfId
                const opponentScore = document.querySelector('.opponent-num')
                opponentnum = parseInt(opponentScore.innerHTML)
                let score = message.data.userMatchInfo.score
                opponentScore.innerHTML = score  
                
                if (option2 && option1) {
                    const questions = document.querySelector('.questions>div');
                    r++
                    questions.style.top = -520 * r + 'px'
                    
                    option1 = false
                    option2 = false
                    // 重置双方选或否
                }
            }

            // 这个data目前表示的是scoreSelectedInfo的匿名对象
            opponentSelectedAnswerIndex = data.userSelectedAnswerIndex;
            // 更新对面用户的分数
            opponentScore = data.userMatchInfo.score;
            isOpponentSelect = true;
            // 更新完分数了 这时候需要重新渲染出下一题 依次进行循环
        }
````

这个阶段代表的是 开始对局了之后所接受到的对方的信息

先进行对信息发送者 **sender** 的一个判断

本程序设计是在答题过程中 用户每回答一题 都将用户的回答情况广播给比赛的双方 所以在接收到状态为**对战中**的信息时  

首先要判断信息的发出方是谁 **(客户端执行)**    换而言之 客户端需要判断自己**是否此信息的发出方**  

如果不是 则说明对方答题情况有变化 渲染

如果是    则说明己方答题情况变化 渲染 (更建议将渲染己方的过程放置在己方答题之后即时进行)



####  (3). GAME_OVER

当题目遍历完了之后 由游戏中状态**PLAY_GAME**切换为**GAME_OVER**

````javascript
// 按照游戏流程 写完所有的题之后 需要到结算页面
// 游戏结束
// 结束的时候 要告诉服务器谁是获胜者 把获胜者的id传过去吧
function gameover(userId) {
    let chatMessage = {};
    let data = null;

    data = userId;
    var sender = userId;
    var type = "GAME_OVER";
    chatMessage.sender = sender;
    chatMessage.type = type;
    chatMessage.data = data;
    console.log("用户:" + sender + "结束游戏");
    socket.send(JSON.stringify(chatMessage));
    userScore = 0;
    opponentScore = 0;
}
````

可知这里的data是 获胜者的id

服务器将胜者告诉客户端 后台对其决定积分增减etc

````javaScript
        if (type === "GAME_OVER") {
            if (userId === data.receiver) {
                // 渲染结算页面的我的答案
                const userlist = document.querySelectorAll('.user-list li')
                    // 渲染对手的答案
                const opponentlist = document.querySelectorAll('.opponent-list li')
                for (let i = 0; i < userlist.length; i++) {
                    if (data.opponentAnswerSituations.selfAnswerSituations[i] !== rightAnswer[i]) {
                        userlist[i].backgroundColor = 'red'
                    } else {
                        userlist[i].backgroundColor = 'green'
                    }
                    if (data.selfAnswerSituations.selfAnswerSituations[i] !== rightAnswer[i]) {
                        opponentlist[i].backgroundColor = 'red'
                    } else {
                        opponentlist[i].backgroundColor = 'green'
                    }
                    userlist[i].innerHTML = data.opponentAnswerSituations.selfAnswerSituations[i]
                    opponentlist[i].innerHTML = data.selfAnswerSituations.selfAnswerSituations[i]
                }
            } else {
                // 渲染结算页面的我的答案
                const userlist = document.querySelectorAll('.user-list li')
                    // 渲染对手的答案
                const opponentlist = document.querySelectorAll('.opponent-list li')
                for (let i = 0; i < userlist.length; i++) {
                    opponentlist[i].innerHTML = data.opponentAnswerSituations.selfAnswerSituations[i]
                    userlist[i].innerHTML = data.selfAnswerSituations.selfAnswerSituations[i]
                }
            }

        }

````

本程序规定触发**gameover**状态是单个用户所决定的 当满足触发gameover的对应条件时 前端将gameover携带获胜者id返回后端**用户A**


后端以上方所返回**用户A**为**sender**的返回双方答题情况

前端通过判断当前用户id是否sender 来确认用户的身份 并将自己以及对方的答题情况保存并渲染



### 4、OnClose断开连接

````javascript
const turnon = document.querySelector('.turn-on')
turnon.addEventListener('click', function() {
    let userId = sessionStorage.getItem('userId')
        //关闭事件
    socket.close()
    socket.onclose = function() {
        console.log("websocket 已关闭 userId: " + userId);
    };
````

使用点击则关闭大法 至此断开websocket连接 一轮pk结束



### **issue**

可实现**再来一局**功能 大致思路为

在响应完gameover函数后 重新**matchUser**或直接**toPlay**

前者直接跳转为匹配中的状态 相当于退出重进

后者跳转为游戏中状态 需要刷新重新发送题库



将用户回答情况工具类结合redis进行优化
