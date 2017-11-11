.. _refClientCredentialsQuickstart:
使用 Client Credentials 保护 API 
==========================================

本文 quickstart 介绍了使用 IdentityServer保护API最基本的场景。

在这种情况下，我们将定义一个API和一个想要访问它的Client。
Client将在IdentityServer上请求Access token，并使用它来访问API。

定义API
^^^^^^^^^^^^^^^^
Scopes 定义您要保护系统中的资源， 例如 APIs。

由于我们正在使用内存配置进行此演练 - 所有您需要做的事情
添加一个API, 是创建一个``piResource``类型的对象并设置适当的属性。

添加一个文件 (例如 ``Config.cs``) 到你的项目，和以下代码::

    public static IEnumerable<ApiResource> GetApiResources()
    {
        return new List<ApiResource>
        {
            new ApiResource("api1", "My API")
        };
    }

定义 client
^^^^^^^^^^^^^^^^^^^
下一步是定义一个可以访问这个API的客户端。

对于这种情况，现在client和IdentityServer还没有用户和进行身份验证
添加下面的代码到你的配置::

    public static IEnumerable<Client> GetClients()
    {
        return new List<Client>
        {
            new Client
            {
                ClientId = "client",

                // 没有交互式用户, 使用clientid / secret进行身份验证。
                AllowedGrantTypes = GrantTypes.ClientCredentials,

                // secret 认证
                ClientSecrets =
                {
                    new Secret("secret".Sha256())
                },

                // 客户端访问的scopes
                AllowedScopes = { "api1" }
            }
        };
    }

配置 IdentityServer
^^^^^^^^^^^^^^^^^^^^^^^^
使用定义的scopes和client配置 IdentityServer, 你需要添加代码到 ``ConfigureServices`` 方法. 
你可以使用扩展方法
将这些相关的stores和data加到DI系统中::

    public void ConfigureServices(IServiceCollection services)
    {
        // 配置 identity server 与 in-memory stores, keys, clients and resources
        services.AddIdentityServer()
            .AddDeveloperSigningCredential()
            .AddInMemoryApiResources(Config.GetApiResources())
            .AddInMemoryClients(Config.GetClients());
    }

这样 - 如果你运行服务器并浏览浏览器
``http://localhost:5000/.well-known/openid-configuration``, 你应该看到所谓的发现文件。
这将被您的客户端和API用于下载必要的配置数据。

.. image:: images/1_discovery.png

添加 API
^^^^^^^^^^^^^
接下来, 添加 API 到你的解决方案。 

您可以使用ASP.NET Core Web API模板。
重复, 我们建议您控制端口并使用与您所用的相同的技术像以前一样配置Kestrel和启动配置文件。

**controller**

添加一个新的controller到您的API项目中::

    [Route("identity")]
    [Authorize]
    public class IdentityController : ControllerBase
    {
        [HttpGet]
        public IActionResult Get()
        {
            return new JsonResult(from c in User.Claims select new { c.Type, c.Value });
        }
    }
这个controller将稍后将用于测试授权要求，以及通过API查看claims identity。

**Configuration**

最后一步是将authentication services添加到DI和authentication middleware到管道中。
这些会:

* 验证传入的token以确保它来自受信任的发行者
* 验证令牌是有效的用于这个api（aka scope）

添加 `IdentityServer4.AccessTokenValidation` NuGet package 到你项目。

.. image:: images/1_nuget_accesstokenvalidation.png

像这样更新``Startup``::

    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvcCore()
                .AddAuthorization()
                .AddJsonFormatters();

            services.AddAuthentication("Bearer")
                .AddIdentityServerAuthentication(options =>
                {
                    options.Authority = "http://localhost:5000";
                    options.RequireHttpsMetadata = false;

                    options.ApiName = "api1";
                });
        }

        public void Configure(IApplicationBuilder app)
        {
            app.UseAuthentication();

            app.UseMvc();
        }
    }


``AddAuthentication`` 添加 authentication services 到 DI 和配置 ``"Bearer"`` 为默认scheme。
``AddIdentityServerAuthentication`` 添加 IdentityServer access token 验证处理程序到 DI ，提供给authentication services使用。
``UseAuthentication`` 添加authentication 中间件到管道中，以便每次调用都会自动执行authentication认证。

如果您使用浏览器导航到控制器 (``http://localhost:5001/identity``), 
你应该会返回一个401状态码。 这意味着您的API需要一个凭证(credential)。

就是这样，API现在由IdentityServer保护。

创建client
^^^^^^^^^^^^^^^^^^^
最后一步是编写一个获得access token的客户端, 然后使用这个
access token访问API。为此, 添加一个控制台程序到你的解决方案 (see full code `here <https://github.com/IdentityServer/IdentityServer4.Samples/blob/release/Quickstarts/1_ClientCredentials/src/Client/Program.cs>`_).

这个 token endpoint 在 IdentityServer 实现 OAuth 2.0 协议, 你可以使用
 HTTP 去访问他。 然而, 我们需要一个名为 IdentityModel的库, 那个
将协议交互封装在易于使用的API中。

添加 `IdentityModel` NuGet package 到你的程序.

.. image:: images/1_nuget_identitymodel.png

IdentityModel 包含一个发现endpoint的client库。
这样你只需要知道IdentityServer的基地址， 实际的
端点地址可以从元数据中读取::

    // 从元数据发现 endpoints
    var disco = await DiscoveryClient.GetAsync("http://localhost:5000");
    if (disco.IsError)
    {
        Console.WriteLine(disco.Error);
        return;
    }

接下来，您可以使用``TokenClient``类来请求token。
你需要用token endpoint,client id 和 secret 去创建实例, 

接下来你可以使用 ``RequestClientCredentialsAsync`` 方法为API请求一个token::

    // request token
    var tokenClient = new TokenClient(disco.TokenEndpoint, "client", "secret");
    var tokenResponse = await tokenClient.RequestClientCredentialsAsync("api1");

    if (tokenResponse.IsError)
    {
        Console.WriteLine(tokenResponse.Error);
        return;
    }

    Console.WriteLine(tokenResponse.Json);


.. note:: 将access token从控制台复制并粘贴到 `jwt.io <https://jwt.io>`_ 去解析原始token.

最后一步是调用API。

要将access token发送到API，通常使用HTTP  Authorization header。
这是使用 ``SetBearerToken`` 扩展方法完成::

    // call api
    var client = new HttpClient();
    client.SetBearerToken(tokenResponse.AccessToken);

    var response = await client.GetAsync("http://localhost:5001/identity");
    if (!response.IsSuccessStatusCode)
    {
        Console.WriteLine(response.StatusCode);
    }
    else
    {
        var content = await response.Content.ReadAsStringAsync();
        Console.WriteLine(JArray.Parse(content));
    }

输出应该是这样的:

.. image:: images/1_client_screenshot.png

.. note:: 默认情况下， access token 将包含 claims 相关的 scope, lifetime (nbf and exp),  client ID (client_id) and the issuer name (iss).

进一步实验
^^^^^^^^^^^^^^^^^^^

本演练集中于迄今为止的成功之路

* 客户端能够请求令牌
* 客户端可以使用令牌来访问API

您现在可以尝试引发错误，以了解系统的行为， 例如

* 尝试连接到IdentityServer ，当它不运行（不可用）。
* 尝试使用无效的client id或secret来请求令牌。
* 尝试在令牌请求期间询问无效范围。
* 尝试在不运行时调用API（不可用）。
* 不要将令牌发送到API。
* 将API配置为需要与令牌中的scope不同的scope。
