# 金蝶云星空对接 authentik实现 sso登录


## 编写 proxy 服务
端口开放为：8888

逻辑：
1. 获取authentik返回的请求头 X-authentik-username获取用户的用户名
2. 对接[金蝶第三方登录](https://vip.kingdee.com/article/9788?productLineId=1&lang=zh-CN)生成html5跳转链接，并进行302 跳转
```golang
username := r.Header.Get("X-authentik-username")
req.Username = username
l := logic.NewK3cloudLogic(r.Context(), svcCtx)
resp, err := l.K3cloud(&req)
if err != nil {
   httpx.ErrorCtx(r.Context(), w, err)
} else {
   w.Header().Set("Location", resp.Message)
   w.WriteHeader(http.StatusFound)
   http.RedirectHandler(resp.Message, http.StatusFound)
}
...

func (l *K3cloudLogic) K3cloud(req *types.Request) (resp *types.Response, err error) {
	//拼接
	userName := req.Username
	if userName == "" {
		return nil, errors.New("invalid param")
	}
	dbId := l.svcCtx.Config.K3Cloud.DbId
	appId := l.svcCtx.Config.K3Cloud.AppId
	appSecret := l.svcCtx.Config.K3Cloud.AppSecret
	timestamp := time.Now().Unix()
	arr := []string{dbId, userName, appId, appSecret, fmt.Sprintf("%d", timestamp)}
	sign := GetSignature(arr) // 签名
	l.Debugf("generate sign param:%+v result:%v", arr, sign)

	arg := domain.GenerateCloudSign{
		DBID:       dbId,
		Username:   userName,
		AppId:      appId,
		SignedData: sign,
		Timestamp:  fmt.Sprintf("%d", timestamp),
		LcId:       l.svcCtx.Config.K3Cloud.LcId,
		OriginType: "SimPas",
		EntryRole:  "",
		FormId:     "",
		FormType:   "",
		Pkid:       "",
		OtherArgs:  "",
	}

	argJson := arg.SerializeObject()
	argJsonBase64 := base64.StdEncoding.EncodeToString([]byte(argJson))
	html5Url := l.svcCtx.Config.K3Cloud.BaseUrl + argJsonBase64 // html5入口链接
	result := types.Response{}
	result.Message = html5Url
	l.Debugf("generate html5 url result:%v", html5Url)
	return &result, nil
}

func GetSignature(arr []string) string {
	// 将数组进行排序
	sort.Strings(arr)
	// 将数组拼接成一个字符串
	arrString := strings.Join(arr, "")
	// 对拼接后的字符串进行SHA1加密
	sha1Hash := sha1.New()
	sha1Hash.Write([]byte(arrString))
	sha1Bytes := sha1Hash.Sum(nil)
	// 将加密后的字节转换为十六进制字符串
	signature := hex.EncodeToString(sha1Bytes)
	return signature
}
```
## 配置 authentik proxy provider
详见 authentik proxy provider部分

配置proxy的nginx,10.2.192.4:8888/from/authentik地址是编写 proxy服务地址
```nginx
location / {
  # Put your proxy_pass to your application here, and all the other statements you'll need
  proxy_pass http://10.2.192.4:8888/from/authentik;
  proxy_set_header Host $host;

  ##############################
  # authentik-specific config
  ##############################
  auth_request     /outpost.goauthentik.io/auth/nginx;
  error_page       401 = @goauthentik_proxy_signin;
  auth_request_set $auth_cookie $upstream_http_set_cookie;
  add_header       Set-Cookie $auth_cookie;

  # translate headers from the outposts back to the actual upstream
  auth_request_set $authentik_username $upstream_http_x_authentik_username;
  auth_request_set $authentik_groups $upstream_http_x_authentik_groups;
  auth_request_set $authentik_email $upstream_http_x_authentik_email;
  auth_request_set $authentik_name $upstream_http_x_authentik_name;
  auth_request_set $authentik_uid $upstream_http_x_authentik_uid;

  proxy_set_header X-authentik-username $authentik_username;
  proxy_set_header X-authentik-groups $authentik_groups;
  proxy_set_header X-authentik-email $authentik_email;
  proxy_set_header X-authentik-name $authentik_name;
  proxy_set_header X-authentik-uid $authentik_uid;
}
```
