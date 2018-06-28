MVC 应用（MVC Applications）
============================

在Phalcon，策划MVC操作背后的全部困难工作通常都可以通过 :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` 做到。这个组件封装了全部后端所需要的复杂操作，实例化每一个需要用到的组件并与项目整合在一起，从而使得MVC模式可以如期地运行。

单模块或多模块应用（Single or Multi Module Applications）
---------------------------------------------------------
通过这个组件，你可以运行各式各样的MVC结构，多模块跟单模块的区别就是，多模块会在路由返回的模块名称，之后去设置自动加载器、视图、调度器等各类服务和组件。

单模块（Single Module）
^^^^^^^^^^^^^^^^^^^^^^^
单一的MVC应用仅仅包含了一个模块。可以使用命名空间，但不是必需的。
这样类型的应用可能会有以下文件目录结构：

.. code-block:: php

    single/
        app/
            controllers/
            models/
            views/
        public/
            css/
            img/
            js/

如果未使用命名空间，以下的启动文件可用于编排MVC工作流：

.. code-block:: php

    <?php

    use Phalcon\Loader;
    use Phalcon\Mvc\View;
    use Phalcon\Mvc\Application;
    use Phalcon\Di\FactoryDefault;

    $loader = new Loader();

    $loader->registerDirs(
        array(
            '../apps/controllers/',
            '../apps/models/'
        )
    )->register();

    $di = new FactoryDefault();

    // 注册视图组件
    $di->set('view', function () {
        $view = new View();
        $view->setViewsDir('../apps/views/');
        return $view;
    });

    try {

        $application = new Application($di);

        echo $application->handle()->getContent();

    } catch (\Exception $e) {
        echo $e->getMessage();
    }

如果使用了命名空间，则可以使用以下启动文件（译者注：主要区别在于使用$loader的方式）：

.. code-block:: php

    <?php

    use Phalcon\Loader;
    use Phalcon\Mvc\View;
    use Phalcon\Mvc\Dispatcher;
    use Phalcon\Mvc\Application;
    use Phalcon\Di\FactoryDefault;

    $loader = new Loader();

    // 根据命名空间前缀加载
    $loader->registerNamespaces(
        array(
            'Single\Controllers' => '../apps/controllers/',
            'Single\Models'      => '../apps/models/',
        )
    )->register();

    $di = new FactoryDefault();

    // 注册调度器，并设置控制器的默认命名空间
    $di->set('dispatcher', function () {
        $dispatcher = new Dispatcher();
        $dispatcher->setDefaultNamespace('Single\Controllers');
        return $dispatcher;
    });

    // 注册视图组件
    $di->set('view', function () {
        $view = new View();
        $view->setViewsDir('../apps/views/');
        return $view;
    });

    try {

        $application = new Application($di);

        echo $application->handle()->getContent();

    } catch (\Exception $e) {
        echo $e->getMessage();
    }

多模块（Multi Module）
^^^^^^^^^^^^^^^^^^^^^^
多模块的应用使用了相同的文档根目录但拥有多个模块。在这种情况下，可以使用以下的文件目录结构：

.. code-block:: php

    multiple/
      apps/
        frontend/
           controllers/
           models/
           views/
           Module.php
        backend/
           controllers/
           models/
           views/
           Module.php
      public/
        css/
        img/
        js/

模块定义类（Module Define Class）
"""""""""""""""""""""""""""""""""
在`apps/`下的每一个目录都有自己的 MVC 结构。`Module.php` 文件定义了各个模块不同的配置，如自动加载器、视图和自定义服务：

.. code-block:: php

    <?php

    namespace Multiple\Frontend;

    use Phalcon\Loader;
    use Phalcon\Mvc\View;
    use Phalcon\DiInterface;
    use Phalcon\Mvc\Dispatcher;
    use Phalcon\Mvc\ModuleDefinitionInterface;

    class Module implements ModuleDefinitionInterface
    {
        /**
         * 注册自定义加载器
         */
        public function registerAutoloaders(DiInterface $di)
        {
            $loader = new Loader();

            $loader->registerNamespaces(
                array(
                    'Multiple\Frontend' => array(
                        '../apps/frontend/crontollers',
                        '../apps/frontend/models'
                    ),
                )
            );

            $loader->register();
        }

        /**
         * 注册自定义服务
         */
        public function registerServices(DiInterface $di)
        {
            // Registering the view component
            $di->set('view', function () {
                $view = new View();
                $view->setViewsDir('../apps/frontend/views/');
                return $view;
            });
        }
    }

.. code-block:: php

    <?php

    namespace Multiple\Backend;

    use Phalcon\Loader;
    use Phalcon\Mvc\View;
    use Phalcon\DiInterface;
    use Phalcon\Mvc\Dispatcher;
    use Phalcon\Mvc\ModuleDefinitionInterface;

    class BackendModule implements ModuleDefinitionInterface
    {
        /**
         * 注册自定义加载器
         */
        public function registerAutoloaders(DiInterface $di)
        {
            $loader = new Loader();

            $loader->registerNamespaces(
                array(
                    'Multiple\Backend\Controllers' => '../apps/backend/controllers/',
                    'Multiple\Backend\Models'      => '../apps/backend/models/',
                )
            );

            $loader->register();
        }

        /**
         * 注册自定义服务
         */
        public function registerServices(DiInterface $di)
        {
            // Registering a dispatcher
            $di->set('dispatcher', function () {
                $dispatcher = new Dispatcher();
                $dispatcher->setDefaultNamespace("Multiple\Backend\Controllers");
                return $dispatcher;
            });

            // Registering the view component
            $di->set('view', function () {
                $view = new View();
                $view->setViewsDir('../apps/backend/views/');
                return $view;
            });
        }
    }

还需要一个指定的启动文件来加载多模块的MVC架构：

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;
    use Phalcon\Mvc\Application;
    use Phalcon\Di\FactoryDefault;

    $di = new FactoryDefault();

    // 自定义路由
    // More information how to set the router up https://docs.phalconphp.com/zh/latest/reference/routing.html
    $di->set('router', function () {

        $router = new Router();

        $router->setDefaultModule("frontend");

        $router->add(
            "/login",
            array(
                'module'     => 'backend',
                'controller' => 'login',
                'action'     => 'index'
            )
        );

        $router->add(
            "/admin/products/:action",
            array(
                'module'     => 'backend',
                'controller' => 'products',
                'action'     => 1
            )
        );

        $router->add(
            "/products/:action",
            array(
                'controller' => 'products',
                'action'     => 1
            )
        );

        return $router;
    });

    try {

        // 创建应用
        $application = new Application($di);

        // 注册模块，包含设置模块定义类加载位置
        $application->registerModules(
            array(
                'frontend' => array(
                    'namespaceName' => 'Multiple\Frontend',
                    'className'     => 'Module',
                    'path'          => '../apps/frontend/Module.php',
                ),
                'backend'  => array(
                    'className' => 'Multiple\Frontend\BackendModule',
                    'path'      => '../apps/backend/Module.php',
                )
            )
        );

        // 处理请求
        echo $application->handle()->getContent();

    } catch (\Exception $e) {
        echo $e->getMessage();
    }

你也可以直接实例化模块定义类，类进行注册：

.. code-block:: php

    <?php

        require('../apps/backend/Module.php');
        require('../apps/frontend/Module.php');

        // 注册模块
        $application->registerModules(
            array(
                'frontend' => new FrontendModule,
                'backend'  => new BackendModule
            )
        );

如果你想在启动文件进行相关组件配置，你可以使用匿名函数来注册对应的模块：

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;

    // 创建视图组件
    $view = new View();

    // 设置视图组件相关选项
    // ...

    // Register the installed modules
    $application->registerModules(
        array(
            'frontend' => function ($di) use ($view) {
                $di->setShared('view', function () use ($view) {
                    $view->setViewsDir('../apps/frontend/views/');
                    return $view;
                });
            },
            'backend' => function ($di) use ($view) {
                $di->setShared('view', function () use ($view) {
                    $view->setViewsDir('../apps/backend/views/');
                    return $view;
                });
            }
        )
    );

当 :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` 有多个模块注册时，通常
每个都是需要的，以便每一个被匹配到的路由都能返回一个有效的模块。每个已经注册的模块都有一个相关的类来提供建立和启动自身的函数。
而每个模块定义的类都必须实现 registerAutoloaders() 和 registerServices() 这两个方法，这两个函数会在模块即被执行时被
:doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` 调用。

应用事件（Application Events）
------------------------------
:doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>` 可以把事件发送到 :doc:`EventsManager <events>` （如果它激活的话）。
事件将被当作"application"类型被消费掉。目前已支持的事件如下：

+---------------------------------+-------------------------------------+--------------+--------------+
| 事件名                          | 触发条件                            | 能否中止操作 | 能否返回值   |
+=================================+=====================================+==============+==============+
| boot                            | 当应用处理它首个请求时被执行        | Yes          | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| beforeHandleRouter              | 当应用处理它首个请求时被执行        | Yes          | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| afterHandleRouter               | 当应用处理它首个请求时被执行        | Yes          | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| beforeStartModule               | 在初始化模块之前，仅当模块被注册时  | Yes          | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| afterStartModule                | 在初始化模块之后，仅当模块被注册时  | Yes          | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| beforeCheckUseImplicitView      | 在检查是否启用视图之前              | No           | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| afterCheckUseImplicitView       | 在检查是否启用视图之后              | No           | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| beforeHandleRequest             | 在执行分发环前                      | Yes          | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| afterHandleRequest              | 在执行分发环后                      | Yes          | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| beforeRenderView                | 在执行视图渲染之前                  | No           | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| afterRenderView                 | 在执行视图渲染之后                  | No           | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| beforeSendResponse              | 在发送响应之前                      | No           | No           |
+---------------------------------+-------------------------------------+--------------+--------------+
| afterSendResponse               | 在执行视图渲染之后                  | No           | No           |
+---------------------------------+-------------------------------------+--------------+--------------+

以下示例演示了如何将侦听器绑定到组件：

.. code-block:: php

    <?php

    use Phalcon\Events\Manager as EventsManager;

    $eventsManager = new EventsManager();

    $application->setEventsManager($eventsManager);

    $eventsManager->attach(
        "application",
        function ($event, $application) {
            // ...
        }
    );

禁用视图组件（Disable View Component）
--------------------------------------
MVC 应用默认开启视图组件，以下示例演示了如何禁用视图组件：

.. code-block:: php

    <?php

    $application->useImplicitView(false);

JSON 输出（JSON Response）
^^^^^^^^^^^^^^^^^^^^^^^^^^
MVC 应用默认开启视图组件，以下示例演示了如何禁用视图组件：

.. code-block:: php

    class IndexController extends Phalcon\Mvc\Controller {
        public function indexAction() {
            return ['name' => 'Phalcon7'];
        }
    }

    $di = new Phalcon\Di\FactoryDefault();
    $di->response->setContentType('application/json');
    $application = new Phalcon\Mvc\Application();
    $application->attachEvent('beforeSendResponse', function($event, $res){
        // Change data
        $data = $this->dispatcher->getReturnedValue();
        $res->setJsonContent(['data' => $data]);
    });
    $application->useImplicitView(false);

    echo $application->handle('/index/index')->getContent();

HMVC 请求（HMVC request system）
--------------------------------
在 HMVC 的父子 MVC 之间，调度控制器、路由以及视图组件是分离的，以下示例演示了如何完成 HMVC 请求：

.. code-block:: php

    <?php

    class HmvcController extends Phalcon\Mvc\Controller
    {

        public function oneAction()
        {
            echo $this->app->request('/hmvc/two');
        }

        public function twoAction()
        {
            echo $this->dispatcher->getActionName();
        }
    }

.. highlights::

    慎用 `exit` 可以使用抛出异常 :doc:`Phalcon\\ContinueException <../api/Phalcon_ContinueException>` 退出 action，注入调度控制器、路由以及视图组件时不要使用 `require_once` 返回对象。
