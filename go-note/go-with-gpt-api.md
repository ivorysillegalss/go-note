# Go调用ChatGPTapi及429解决方案

本文简要介绍如何通过go语言进行gpt的api调用

### 设置请求体

openAI的api服务器网址为`"https://api.openai.com/v1/completions"`



```go
// 构造请求
func encodeReq(reqMessage *ApiRequestMessage) (*http.Request, error) {

	// 构造请求体
	data := ApiRequestMessage{
		Model: reqMessage.Model,
		//Model:     "gpt-3.5-turbo-instruct", // 替换为当前可用的模型
		InputPrompt: reqMessage.InputPrompt,
		MaxToken:    reqMessage.MaxToken,
	}
	jsonData, _ := json.Marshal(data)

	//OpenAI-API服务器默认网址
	req, _ := http.NewRequest("POST", "https://api.openai.com/v1/completions", bytes.NewBuffer(jsonData))

	secretKey := "sk-此处为密钥"
	//organizationID := "org-a"
	//req.Header.Set("OpenAI-Organization", organizationID)
	//organizationID 为可选项 有时候需要加上才运行成功 截至commit 不加也行

	key := fmt.Sprintf("Bearer %s", secretKey)
	req.Header.Set("Authorization", key) // 请确保使用你自己的API密钥
	req.Header.Set("Content-Type", "application/json")

	return req, nil
}
```

