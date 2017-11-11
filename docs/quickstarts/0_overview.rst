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

``AddIdentityServer`` registers the IdentityServer services in DI. It also registers an in-memory store for runtime state.
This is useful for development scenarios. For production scenarios you need a persistent or shared store like a database or cache for that.
See the :ref:`EntityFramework <refEntityFrameworkQuickstart>` quickstart for more information.

The ``AddDeveloperSigningCredential`` extension creates temporary key material for signing tokens.
Again this might be useful to get started, but needs to be replaced by some persistent key material for production scenarios.
See the :ref:`cryptography docs <refCrypto>` for more information.

.. Note:: IdentityServer is not yet ready to be launched. We will add the required services in the following quickstarts.

Modify hosting
^^^^^^^^^^^^^^^

By default Visual Studio uses IIS Express to host your web project. This is totally fine,
except that you won't be able to see the real time log output to the console.

IdentityServer makes extensive use of logging whereas the "visible" error message in the UI
or returned to clients are deliberately vague.

We recommend to run IdentityServer in the console host. 
You can do this by switching the launch profile in Visual Studio.
You also don't need to launch a browser every time you start IdentityServer - you can turn that off as well:

.. image:: images/0_launch_profile.png

In addition, it will be helpful to run IdentityServer on a consistent URL for these quickstarts.
You should also configure this URL in the launch profile dialog above, and use ``http://localhost:5000/``.
In the above screenshot  you can see this URL has been configured.

.. Note:: We recommend to configure the same port for IIS Express and self-hosting. This way you can switch between the two without having to modify any configuration in your clients.

To then choose the console host when you launch, you must select it in the launch menu from Visual Studio:

.. image:: images/0_choose_launch.png

How to run the quickstart
^^^^^^^^^^^^^^^^^^^^^^^^^
As mentioned above every quickstart has a reference solution - you can find the code in the 
`IdentityServer4.Samples <https://github.com/IdentityServer/IdentityServer4.Samples>`_
repo in the quickstarts folder.

The easiest way to run the individual parts of a quickstart solution is to set the startup mode to "current selection".
Right click the solution and select "Set Startup Projects":

.. image:: images/0_startup_mode.png

Typically you start IdentityServer first, then the API, and then the client. Only run in the debugger if you actually want to debug.
Otherwise Ctrl+F5 is the best way to run the projects.
