# PHP Web 框架架构思考

> PHP 为 Web 应用而生，几乎所有有名的 PHP 项目都是 Web 应用，任何一个 Web 应用或多或少依赖一个 Web 框架，无论是项目本身造的轮子还是项目基于其它框架。

一个 PHP Web 框架的必要组成部分无非分为一下三点。

- 解析请求，获得输入。
- 定位执行者，将输入给指定执行者。
- 输出执行后的相应，获得输出。

上述三点其实就是 MVC 的 Controller 相关的点。对 Web 框架来说，MVC 中 除了 Controller  的Model、View，都不必要是框架自带的，可以用其它独立的开源组件替代。

这里标注三个有各自鲜明特性的框架分别是

- Yii2
- Symfony
- Laravel

## Yii2 

Yii2 的优点在与易于理解，由于 Symfony、Laravel 等框架使用了非常多的中间件、事件驱动的概念，你会发现从入口文件 index.php 进去很难知道最终如何调用到了某个 Controller 的 Action ，因为找到 Action 并调用是 Web 框架的核心功能，这个流程很难看懂的话，就很真正掌握 Web 框架本身，而 Yii2 的优点便体现于此。

### Yii2 整体架构

Yii2 的整体流程非常简单易懂，从 index.php 进入后，首先会读取配置文件，通过配置文件对 Application 进行初始化并调用其 run 方法。在 run 方法中，可以清晰的看到 Yii2 如何对 Request 进行解析，如何读取请求中 URL 信息并 router 到对应的 Controller 和 Action 中。

Yii2 的另一个特点是大量使用了类继承。首先其定义了 \yii\base\Object 类，定义构造器和魔术方法等。基于 \yii\base\Object 扩展成 \yii\base\Component 类，加入了事件和用来扩展类能力的 Behavior 特性。基于 \yii\base\Component 扩展成 \yii\base\Model 类，加入了属性验证功能，用来接收和验证外部输入。基于 \yii\base\Model 扩展成 \yii\db\ActiveRecord，用来加入数据持久化和关联数据获取的功能。

这样的层层递进的基于集成的扩展，可以把每一层的功能定义清楚，方便使用者理解，让用户有选择地扩展自己的类，有点类似典型的 java 应用。但是，这样做也有明显的缺点。因为基于 Yii2 的开发或多或少需要继承 \yii\base\Object 类，所以这些开发的产出，包括扩展、模块很难兼容其它框架。因为这个原因，导致 Yii2 社区中创造的东西的影响，无法传播到其它框架的使用者，导致其流行度没有 Symfony，以及基于 Symfony 的 Laravel 高。如果大家感兴趣，我后面可以介绍如果借用 Symfony 和 Laravel 在 Yii2 中写出更好的代码。

Yii2 被称为 A web framework out of the box，一站式的 Web 框架，也是有其道理的。一方面其框架本身包含了构建网站所需的各种功能，不需要到处选合适的组件来创造自己的应用。另一方面，一旦选用了这个框架，就依赖了这个整个框架，比较难只用框架中的一小部分，或将已有工作其迁移到其它框架。

## Symfony

// TODO

## Laravel

// TODO
