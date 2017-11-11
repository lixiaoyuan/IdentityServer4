.. _refResosurceOwnerQuickstart:
使用密码保护API
=================================

OAuth 2.0 password 授权(grant)模式允许资源所有者 使用一个client发送username和password
到token service 获取代表该用户的access token。

password 授权(grant)模式下建议仅用于可信的应用程序。
一般来说，当你想验证用户和访问令牌(access token)时，使用interactive
OpenID Connect会好得多。

虽然, 这种授权模式使我们能够向用户接受概念，快速启动IdentityServer。这就是为什么我们展示它。

添加 users
^^^^^^^^^^^^
就像资源(aka scope)和客户端的内存存储一样,也有一个用户.

.. note:: 查看基于ASP.NET Identity 快速入门，了解有关如何正确存储和管理用户账户。

 ``TestUser`` 类 代表测试用户及用户Claims. 我们将用过下面代码添加到配置类中创建几个用户。:

首先将下面的 using 语句添加到config.cs 文件中::

    using IdentityServer4.Test;

    public static List<TestUser> GetUsers()
    {
        return new List<TestUser>
        {
            new TestUser
            {
                SubjectId = "1",
                Username = "alice",
                Password = "password"
            },
            new TestUser
            {
                SubjectId = "2",
                Username = "bob",
                Password = "password"
            }
        };
    }

然后向IdentityServer注册用户::

    public void ConfigureServices(IServiceCollection services)
    {
        // configure identity server with in-memory stores, keys, clients and scopes
        services.AddIdentityServer()
            .AddDeveloperSigningCredential()
            .AddInMemoryApiResources(Config.GetApiResources())
            .AddInMemoryClients(Config.GetClients())
            .AddTestUsers(Config.GetUsers());
    }

这个 ``AddTestUsers`` 扩展方法在底层做了以下事情

* 添加了对资源所有者密码授权的支持。
* 增加了对用户相关服务的支持，通常由登录用户界面使用（我们将在下一个quickstart中使用）。
* 添加对基于用户的配置文件服务的支持（您将在下一个quickstart中了解更多信息）

为资源所有者密码授权添加一个Client
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
你可以简单地通过改变添加对我们现有Client的授权类型的
``AllowedGrantTypes`` 属性。如果你需要Client能够使用这两种授权类型也是支持的。

通常，你希望为资源所有者用例创建单独的客户端
将以下内容添加到您的Client配置::

    public static IEnumerable<Client> GetClients()
    {
        return new List<Client>
        {
            // other clients omitted...

            // resource owner password grant client
            new Client
            {
                ClientId = "ro.client",
                AllowedGrantTypes = GrantTypes.ResourceOwnerPassword,

                ClientSecrets = 
                {
                    new Secret("secret".Sha256())
                },
                AllowedScopes = { "api1" }
            }
        };
    }

使用密码授权请求token
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
现在client看起来非常类型于我们做的credentials授权类型。
现在主要的区别就是Client会以某种方式手机用户的密码, 并在token请求期间将其发送给token服务。

 在这里IdentityModel's 再次帮助 ``TokenClient`` ::

    // request token
    var tokenClient = new TokenClient(disco.TokenEndpoint, "ro.client", "secret");
    var tokenResponse = await tokenClient.RequestResourceOwnerPasswordAsync("alice", "password", "api1");

    if (tokenResponse.IsError)
    {
        Console.WriteLine(tokenResponse.Error);
        return;
    }

    Console.WriteLine(tokenResponse.Json);
    Console.WriteLine("\n\n");

当您将token发送到 identity API端点时，你会注意到一个小的
但与client credentials授权相比有着重要的不同。这个access token 将包含
唯一标识用户的 ``sub`` 声明(claim) 。 这个 "sub" 声明(claim) 可以用过调用API的内容来查看，并且也可以在控制台应用程序上查看。
