设置和概述
==================

有两个基本的方法来启动一个新的IdentityServer项目:

* 从头开始
* 从Visual Studio中的ASP.NET Identity模板开始

如果你从头开始, 我们提供了几个帮手 in-memory stores, 所以你从一开始就不必担心持久性。

如果您从ASP.NETIdentity 开始, 我们提供了一个简单的方法来整合。

快速入门提供了各种常见IdentityServer场景的分步说明。
他们从基础开始，变得更复杂，建议你按顺序做。

每个快速入门都有一个参考解决方案 -你可以在 
`IdentityServer4.Samples <https://github.com/IdentityServer/IdentityServer4.Samples>`_
文件夹中找到quickstarts 中的代码。

基本设置
^^^^^^^^^^^
屏幕截图显示Visual Studio - 但这不是要求.

**创建 quickstart IdentityServer**

首先创建一个新的ASP.NET Core项目。

.. image:: images/0_new_web_project.png

然后选择“空白”项目。

.. image:: images/0_empty_web.png

接下来, 添加 `IdentityServer4` nuget package:

.. image:: images/0_nuget.png
    
或者，您可以使用软件包管理器控制台通过运行以下命令来添加依赖项:

    "Install-Package IdentityServer4"

.. note:: IdentityServer内部版本号1.x目标ASP.NET Core 1.1, IdentityServer内部版本号2.x以ASP.NET Core 2.0为目标。

IdentityServer使用通常的模式来配置和向ASP.NET Core主机添加服务。
在``ConfigureServices``中配置所需的服务并添加到DI系统中。 
在``Configure``中间件被添加到HTTP管道。

修改你的``Startup.cs``文件看起来像这样::

    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddIdentityServer()
                .AddDeveloperSigningCredential();
        }

        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseIdentityServer();
        }
    }

``AddIdentityServer`` 在DI中注册IdentityServer服务。它还为运行时状态注册了一个内存中的存储。
这对于开发场景很有用。 对于生产场景，您需要持久存储或共享存储，如数据库或缓存。
有关更多信息，请参阅:`EntityFramework <refEntityFrameworkQuickstart>` 快速入门。

``AddDeveloperSigningCredential`` 扩展为签名令牌创建临时密钥。
一开始这可能是有用的，但是在生产场景需要用一些持续的持久性密钥来替代 。
有关更多信息，请参阅:`cryptography docs <refCrypto>` 。

.. Note:: IdentityServer 尚未准备好启动。 我们将在以下快速入门中添加所需的服务。.

修改项目 hosting
^^^^^^^^^^^^^^^

默认情况下，Visual Studio使用IIS Express来托管您的Web项目。 这很好，
但是您将无法看到实时日志输出到控制台。

IdentityServer广泛使用日志记录，而UI中的“visible”错误消息或者返回给客户故意含糊不清。

我们建议在控制台上运行 IdentityServer 。 
您可以通过在Visual Studio中切换启动配置文件来完成此操作。
每次启动IdentityServer时，也不需要启动浏览器，也可以关闭此功能：

.. image:: images/0_launch_profile.png

另外，在这些一致UR的quickstarts上运行IdentityServer会很有帮助。
Y您还应在上面的启动配置文件对话框中配置此URL，使用 ``http://localhost:5000/``.
在上面的截图中你可以看到这个URL已经被配置。

.. Note:: 我们建议为IIS Express和自托管配置相同的端口。这样，您可以在两者之间切换，而无需修改客户端中的任何配置。

要在启动时选择控制台主机，必须在Visual Studio的启动菜单中选择它：

.. image:: images/0_choose_launch.png

如何运行quickstarts
^^^^^^^^^^^^^^^^^^^^^^^^^
如上所述，每个快速入门都有一个参考解决方案 - 您可以在
`IdentityServer4.Samples <https://github.com/IdentityServer/IdentityServer4.Samples>`中找到代码


运行快速启动解决方案的各个部分的最简单方法是将启动模式设置为“当前选择”。
右键单击解决方案并选择“设置启动项目”：

.. image:: images/0_startup_mode.png

通常，首先启动IdentityServer，然后启动API，然后启动客户端。 如果你真的想调试，只能在调试器中运行。
否则，Ctrl + F5是运行项目的最佳方法。
