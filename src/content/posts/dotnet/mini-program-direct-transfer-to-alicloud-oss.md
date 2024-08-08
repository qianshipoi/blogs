---
title: MiniProgram Direct Transfer To Ali Cloud OSS
published: 2023-05-25
description: ''
image: ''
tags: ['Dotnet']
category: 'Develop'
draft: false
---

# NetCore服务器对接小程序直传阿里云OSS

>  问题描述：小程序只支持POST方式上传文件，阿里云生成上传链接只能为PUT上传。小程序得使用前端直传方式。本示例为服务器端签名直传并设置上传回调。

#####  PolicyToken.cs

```c#
internal class PolicyToken
{
	public string accessid { get; set; }
	public string policy { get; set; }
	public string signature { get; set; }
	public string dir { get; set; }
	public string host { get; set; }
	public string expire { get; set; }
	public string callback { get; set; }
}
```

##### CallbackParm.cs

```c#
internal class CallbackParam
{
	public string callbackUrl { get; set; }
	public string callbackBody { get; set; }
	public string callbackBodyType { get; set; }
}
```

#### 生成 `PolicyToken`

>**CallbackBody** 系统参数就不过多说明，详见上面的官方文档，主要说说自定义参数，在构造 Form 表单的参数时，参数名，可以任意命名，但是有两点要注意：
>
>- 自定义参数的格式，必须是 `${x:<占位符>}`，`x:` 一定不能少
>- 自定义参数的占位符，就是 `${x:<占位符>}` 部分，花括号内部的名称，必须全部小写，比如：可以是 `userName=${x:username}&Age=${x:age}` 也可以是 `username=${x:username}&age=${x:age}`，总之，注意占位符全部小写就行了，参数名，按照任意命名方式都可以，回调接口内注意读取就行了
>- 可以不用自定义占位符，直接对参数进行赋值，比如：`username=张三&age=20

```c#
        public string GetPolicyToken(string bucketName, string uploadDir, string callbackUrl, long expireTime)
        {
            //expireTime
            var expireDateTime = DateTime.Now.AddSeconds(expireTime);

            // example of policy
            //{
            //  "expiration": "2020-05-01T12:00:00.000Z",
            //  "conditions": [
            //    ["content-length-range", 0, 1048576000]
            //    ["starts-with", "$key", "user-dir-prefix/"]
            //  ]
            //}

            //policy

            var policyConds = new PolicyConditions();
            policyConds.AddConditionItem(PolicyConditions.CondContentLengthRange, 100, 1024 * 1024 * 400);
            // 指定key
            policyConds.AddConditionItem(MatchMode.Exact, PolicyConditions.CondKey, uploadDir);
            //policyConds.AddConditionItem(MatchMode.StartWith, PolicyConditions.CondKey, uploadDir);
            var policy = _ossClient.GeneratePostPolicy(expireDateTime, policyConds);

            var policy_base64 = EncodeBase64("utf-8", policy);

            var signature = ComputeSignature(_aliyunOSSConfig.AccessKeySecret, policy_base64);

            //callback
            var callback = new CallbackParam();
            callback.callbackUrl = callbackUrl;
            callback.callbackBody = "filename=${object}&size=${size}&mimeType=${mimeType}";
            callback.callbackBodyType = "application/x-www-form-urlencoded";

            var callback_string = JsonConvert.SerializeObject(callback);
            var callback_string_base64 = EncodeBase64("utf-8", callback_string);

            var policyToken = new PolicyToken();

            policyToken.accessid = _aliyunOSSConfig.AccessKeyId;
            policyToken.host = $"https://{bucketName}.{_aliyunOSSConfig.Host}";
            policyToken.policy = policy_base64;
            policyToken.signature = signature;
            policyToken.expire = ToUnixTime(expireDateTime);
            policyToken.callback = callback_string_base64;
            policyToken.dir = uploadDir;

            return JsonConvert.SerializeObject(policyToken);
        }

        private static string ToUnixTime(DateTime dtime)
        {
            const long ticksOf1970 = 621355968000000000;
            var expires = ((dtime.ToUniversalTime().Ticks - ticksOf1970) / 10000000L)
                .ToString(CultureInfo.InvariantCulture);

            return expires;
        }

        private static string ComputeSignature(string key, string data)
        {
            using (var algorithm = new HMACSHA1())
            {
                algorithm.Key = Encoding.UTF8.GetBytes(key.ToCharArray());
                return Convert.ToBase64String(
                    algorithm.ComputeHash(Encoding.UTF8.GetBytes(data.ToCharArray())));
            }
        }

        private static string EncodeBase64(string code_type, string code)
        {
            string encode = "";
            byte[] bytes = Encoding.GetEncoding(code_type).GetBytes(code);
            try
            {
                encode = Convert.ToBase64String(bytes);
            }
            catch
            {
                encode = code;
            }
            return encode;
        }
```

#### 回调验签

>验签步骤
>
>1. 获取 `Authoriaztion` 的值，进行 `Base64` 解码，得到 `byte[]`。
>2. 对 `x-oss-pub-key-url` 的值，进行 `Base64` 解码，得到公钥的 `url` 地址。
>3. 校验公钥 `url` 地址，防止伪造，公钥地址为：
>   - `http://gosspublic.alicdn.com/`
>   - `https://gosspublic.alicdn.com/`
>4. 根据公钥 `url` 地址获取公钥内容。
>5. 将 `<请求路径>?<参数>\n<body内容>` 或 `<请求路径>\n<body内容>` 拼接，计算 `MD5` 获取 `byte[]`（请求路径不包含 `Host`）。
>6. 基于 `RSA`，采用 `MD5` 模型进行验签。
>7.  将验证结果返回给OSS（OSS 仅接收 Json 格式的返回）。
>8. OSS 会将返回的内容直接返回给前端。

```c#
  public async Task<bool> VerifySignature()
        {
            // Get the Authorization Base64 from Request
            var request = _httpContextAccessor.HttpContext.Request;

            if (!request.Headers.TryGetValue("Authorization", out var authInfo)
                || StringValues.IsNullOrEmpty(authInfo))
            {
                return false;
            }

            // Decode the Authorization from Request
            var byteAuth = Convert.FromBase64String(authInfo);

            // Decode the URL of PublicKey
            if (!request.Headers.TryGetValue("x-oss-pub-key-url", out var tempPubKeyUrl)
                || StringValues.IsNullOrEmpty(tempPubKeyUrl))
            {
                return false;
            }

            var bytePubKeyUrl = Convert.FromBase64String(tempPubKeyUrl.ToString());
            var pubKeyUrl = Encoding.ASCII.GetString(bytePubKeyUrl);

            // 验证公钥域名
            if (!pubKeyUrl.StartsWith("http://gosspublic.alicdn.com/", StringComparison.OrdinalIgnoreCase)
                && !pubKeyUrl.StartsWith("https://gosspublic.alicdn.com/", StringComparison.OrdinalIgnoreCase))
            {
                return false;
            }

            // Get PublicKey from the URL
            ServicePointManager.ServerCertificateValidationCallback = new RemoteCertificateValidationCallback(ValidateServerCertificate);

            using var client = _httpClientFactory.CreateClient();
            var pubKey = await client.GetStringAsync(pubKeyUrl);

            var strPublicKeyContentBase64 = pubKey.Replace("-----BEGIN PUBLIC KEY-----\n", "").Replace("-----END PUBLIC KEY-----", "").Replace("\n", "");
            var strPublicKeyContentXML = RSAPublicKeyString2XML(strPublicKeyContentBase64);

            // Generate the New Authorization String according to the HttpRequest
            var httpURL = request.Path.ToString() + request.QueryString.ToString();
            // Read body
     	 var dictBody = new Dictionary<string, string>();
            if (request.HasFormContentType)
            {
                foreach (var item in request.Form)
                {
                    dictBody.Add(item.Key, item.Value);
                }
            }
            var httpBody = string.Join("&", dictBody.Select(per => $"{per.Key.UrlEncode(true)}={per.Value.UrlEncode(true)}"));
            // StreamReader stream = new StreamReader(_httpContextAccessor.HttpContext.Request.Body);
            // var httpBody = await stream.ReadToEndAsync();

            var strAuthSourceForMD5 = string.Empty;
            if (httpURL.Contains('?'))
            {
                var arrURL = httpURL.Split('?');
                strAuthSourceForMD5 = string.Format("{0}?{1}\n{2}", System.Web.HttpUtility.UrlDecode(arrURL[0]), arrURL[1], httpBody);
            }
            else
            {
                strAuthSourceForMD5 = string.Format("{0}\n{1}", System.Web.HttpUtility.UrlDecode(httpURL), httpBody);
            }

            // MD5 hash bytes from the New Authorization String
            var byteAuthMD5 = ByteMD5Encrypt32(strAuthSourceForMD5);

            // Verify Signature
            using var RSA = new RSACryptoServiceProvider();
            try
            {
                RSA.FromXmlString(strPublicKeyContentXML);
            }
            catch (ArgumentNullException e)
            {
                throw new ArgumentNullException(string.Format("VerifySignature Failed : RSADeformatter.VerifySignature get null argument : {0} .", e));
            }
            catch (CryptographicException e)
            {
                throw new CryptographicException(string.Format("VerifySignature Failed : RSA.FromXmlString Exception : {0} .", e));
            }
            RSAPKCS1SignatureDeformatter RSADeformatter = new RSAPKCS1SignatureDeformatter(RSA);
            RSADeformatter.SetHashAlgorithm("MD5");

            var bVerifyResult = false;
            try
            {
                bVerifyResult = RSADeformatter.VerifySignature(byteAuthMD5, byteAuth);
            }
            catch (ArgumentNullException e)
            {
                throw new ArgumentNullException(string.Format("VerifySignature Failed : RSADeformatter.VerifySignature get null argument : {0} .", e));
            }
            catch (CryptographicUnexpectedOperationException e)
            {
                throw new CryptographicUnexpectedOperationException(string.Format("VerifySignature Failed : RSADeformatter.VerifySignature Exception : {0} .", e));
            }

            return bVerifyResult;
        }

        public static byte[] ByteMD5Encrypt32(string password)
        {
            string cl = password;
            using MD5 md5 = MD5.Create();
            byte[] s = md5.ComputeHash(Encoding.UTF8.GetBytes(cl));
            return s;
        }

        public static string RSAPublicKeyString2XML(string publicKey)
        {
            RsaKeyParameters publicKeyParam = (RsaKeyParameters)PublicKeyFactory.CreateKey(Convert.FromBase64String(publicKey));
            return string.Format("<RSAKeyValue><Modulus>{0}</Modulus><Exponent>{1}</Exponent></RSAKeyValue>",
                Convert.ToBase64String(publicKeyParam.Modulus.ToByteArrayUnsigned()),
                Convert.ToBase64String(publicKeyParam.Exponent.ToByteArrayUnsigned()));
        }

        public static bool ValidateServerCertificate(object sender, X509Certificate certificate, X509Chain chain, SslPolicyErrors sslPolicyErrors)
        {
            return true;
        }
```

##### StringExtension.cs

```c#

        public static string UrlEncode(this string content, bool needUpper = false)
        {
            if (string.IsNullOrEmpty(content))
            {
                return string.Empty;
            }

            if (!needUpper)
            {
                return HttpUtility.UrlEncode(content);
            }

            var result = new StringBuilder();

            foreach (var per in content)
            {
                var temp = HttpUtility.UrlEncode(per.ToString());
                if (temp.Length > 1)
                {
                    result.Append(temp.ToUpper());
                    continue;
                }

                result.Append(per);
            }

            return result.ToString();
        }
```



链接：[阿里云 OSS Web 直传/回调/回调签名验证（.NET/C#/Layui） - 简书 (jianshu.com)](https://www.jianshu.com/p/33575a588acf)
