```mermaid
flowchart TD
    Start([用户发起请求]) --> Auth{认证校验}
    Auth -- 失败 --> AuthError[返回认证错误]
    Auth -- 成功 --> CheckLock{检查请求锁}
    
    CheckLock -- 已锁定 --> LockError[返回重复请求错误:<br/>您有正在进行的对话]
    CheckLock -- 未锁定 --> SetLock[设置Redis请求锁]
    
    SetLock --> ValidateInput{基础参数校验}
    ValidateInput -- 失败 --> InputError[返回参数错误]
    ValidateInput -- 成功 --> CallLLM[调用LLM接口]
    
    CallLLM --> LLMResponse{LLM响应检查}
    LLMResponse -- 超时/错误 --> HandleError[错误处理]
    LLMResponse -- 成功 --> PrepareRes[准备返回数据]
    
    HandleError --> ReleaseLock[释放Redis锁]
    PrepareRes --> ReleaseLock
    
    ReleaseLock --> Response[返回响应给用户]
    Response --> End([结束])

    subgraph 错误处理过程
        HandleError --> LogError[记录错误日志]
        LogError --> PrepareError[准备错误响应]
    end
    
    subgraph Redis锁管理
        CheckLock --> GetLock[GET ]
        SetLock --> SetRedis[SET ]
        ReleaseLock --> DelLock[DEL chat_lock:{userId}]
    end
```