---
title: Asp.Net Core JWT Authentication
published: 2022-06-13
description: ''
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---

# 1 安装JWT工具包

> ### 管理NuGet包 —> 安装JWT 5.0.1版本
# 2 创建JWT_Code.cs工具类
        public class JWT_Code
        {
            public static string Key { get; set; } = "KuriyamaMirai";       //密钥
            public static string JWTEnCode(Dictionary<string, object> payloads, string key = null)
            {
                if (string.IsNullOrEmpty(key))
                    key = Key;
                var payload = payloads;    // 创建 payload 部分
                IJwtAlgorithm algorithm = new HMACSHA256Algorithm();    // 256位的HS加密算法
                IJsonSerializer serializer = new JsonNetSerializer();     // 进行序列化
                IBase64UrlEncoder urlEncoder = new JwtBase64UrlEncoder();   // 通过 base64 进行转译
                IJwtEncoder encoder = new JwtEncoder(algorithm, serializer, urlEncoder);// 实例化 JWT 加密器
                payload.Add("timeout", DateTime.Now.AddDays(1));    //设置时效性
                var token = encoder.Encode(payload, key);//加密
                return token;
            }

            public static Dictionary<string, object> JWTDeCode(string token, string key)
            {
                if (string.IsNullOrEmpty(key))
                    key = Key;
                try
                {
                    IJsonSerializer serializer = new JsonNetSerializer();
                    IDateTimeProvider dateTimeProvider = new UtcDateTimeProvider();
                    IJwtValidator jwtValidator = new JwtValidator(serializer, dateTimeProvider);
                    IBase64UrlEncoder base64UrlEncoder = new JwtBase64UrlEncoder();
                    IJwtDecoder jwtDecoder = new JwtDecoder(serializer, jwtValidator, base64UrlEncoder);
                    var json = jwtDecoder.Decode(token, key, verify: true); // 解密
                    var result = JsonConvert.DeserializeObject<Dictionary<string, object>>(json);//反序列化
                    if ((DateTime)result["timeout"] < DateTime.Now)
                        throw new Exception("tonken已过期");
                    result.Remove("timeout");   //去掉过期时间
                    return result;
                }
                catch (Exception)
                {
                    throw new Exception("验证失败");
                }
            }

            public static Dictionary<string, object> ValideLogined(HttpRequestHeaders headers)
            {
                if (headers.GetValues("tonken") == null || !headers.GetValues("tonken").Any())
                    throw new Exception("未登录");
                return JWTDeCode(headers.GetValues("tonken").First(), Key);
            }
        }
# 3 创建自定义过滤器
        public class MyAuthAttribute : Attribute, IAuthorizationFilter
        {
            public bool AllowMultiple { get; }

            public async Task<HttpResponseMessage> ExecuteAuthorizationFilterAsync(HttpActionContext actionContext, CancellationToken cancellationToken, Func<Task<HttpResponseMessage>> continuation)
            {
                //自定义过滤器
                IEnumerable<string> headers;
                if (actionContext.Request.Headers.TryGetValues("token", out headers))
                {   //如果获取到了 headers 里的 token
                    //取值
                    string Name = Comment.JWT_Code.JWTDeCode(headers.First(), null)["Name"].ToString();
                    string Id = Comment.JWT_Code.JWTDeCode(headers.First(), null)["Id"].ToString();
                    (actionContext.ControllerContext.Controller as ApiController).User = new Models.ApplicationUser(Name, Id);//存入 UserIdentity 类
                    return await continuation();//继续执行
                }
                return new HttpResponseMessage(System.Net.HttpStatusCode.Unauthorized);//return 401
            }
        }
# 4 创建 UserIdentity.cs 类 用于存放用户信息
        public class UserIdenetity : IIdentity
        {
            //实现 IIdentity 接口 使用构造函数赋值
            public UserIdenetity(string name, string id)
            {
                Name = name;
                Id = id;
            }
            public string Name { get; private set; }
            public string Id { get; private set; }
            public string AuthenticationType { get; private set; }
            public bool IsAuthenticated { get; private set; }
        }
# 5 创建 ApplictionUser.cs 类 用于调用UserIdentity.cs 类
         public class ApplicationUser : IPrincipal
        {
            //实现 IPrincipal 接口 调用 UserIdenetity 类
            public ApplicationUser(string name, string id)
            {
                Identity = new UserIdenetity(name, id);
            }
            public IIdentity Identity { get; private set; }

            public bool IsInRole(string role)
            {
                throw new NotImplementedException();
            }
        }
# 6 调用 JWT 工具类获取token

     Dictionary<string, object> payload = new Dictionary<string, object>();
     payload.Add("Name", u.UserName);
     payload.Add("Id", u.UserPassword);
     string token = Comment.JWT_Code.JWTEnCode(payload, null);
# 7 在需要进行身份验证的方法前加上自定义过滤器并获取存入信息

    [Route("valid"),MyAuth,HttpGet]
    public IHttpActionResult Valid()
    {
        Models.UserInfo user = new Models.UserInfo();
        user.UserName = ((Models.UserIdenetity)User.Identity).Name;
        user.UserPassword = ((Models.UserIdenetity)User.Identity).Id;
        return Ok(new Models.ResponseData() { Data = user });
    }
