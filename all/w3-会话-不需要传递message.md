```mermaid
flowchart TD
    A([开始]) --> B{根据conversation_id \n 判断是否是新会话}
    B -- 是 --> C[生成conversation_id,并插入t_conversations表]
    C --> D
    B -- 否 --> D{是否包含引用,option=2}
    D -- 是 --> E[设置LLM Server接口入参 \n 1.option=2 \n 2.message_id=赋值为对应的message_id]
    D -- 否 --> F{是否是重新生成回复,option=1}
    F -- 是 --> G[设置LLM Server接口入参 \n 1.option=1 \n 2.message_id=赋值为对应的message_id]
    F -- 否 --> I
     
    E & G --> I{是否是组件生成chat_role=1}
    I -- 否 --> K[设置LLM Server接口入参ai_role=0为基础聊天]
    I -- 是 --> J[设置LLM Server接口入参ai_role=1为应用生成专家]
    

    J --> L[组装LLM Server接口组件生成的其它参数]    
    K --> M[组装LLM Server接口基础聊天的其它参数]    
    M --> N[调用LLM Server,获取响应]    
    N --> O[获取message_id,转发流式响应给前端]
    O --> P[构建message 保存到t_messages表]
    P --> R([结束])


    L --> S[调用LLM Server,获取响应]
    S --> T[接收响应,获取源码以及其它回复数据]
    T --> U[生成component_id,调用module generator上传源码接口]
    U --> V[根据component_id,调用module generator打包接口]
    V --> W[轮询调用module generator提供打包状态查询功能接口获取打包结果]
    W --> X{是否打包成功}
    X -- 是 --> Y[按照模板,组装成功响应结果,流式返回给前端]
    X -- 否 --> Z[按照模板,组装失败响应结果,流式返回给前端]

    Y & Z --> Result[1.构建message,将结果保存到t_messages \n 2.构建component数据,将结果保存到t_components]
    Result --> R
```