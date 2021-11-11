# 博客系统开发笔记

仅记录开发笔记和部分对于概念的理解，不系统记录概念的具体信息。

[TOC]

## 测试驱动开发 TDD

这里不展开写，后续会在 Architect 分类里写一篇关于 TDD 的笔记。// TODO

具体就是 

先写测试，跑测试，出报错，根据报错去写程序，再测试直到成功，成功后重构代码。

> 参考资料: [TDD 開發五步驟，帶你實戰 Test-Driven Development 範例](https://tw.alphacamp.co/blog/tdd-test-driven-development-example)

## 单元测试

在 Symfony 项目中使用单元测试 PHPUnit 包可以使用 symfony/flex 维护的官方测试包 `test`

使用命令安装

```powershell
# 将 test 安装到 --dev 环境下
composer req test --dev 
```

symfony/flex 会为我们创建 

- `/tests` 目录
- `bin/phpunit` 工具运行测试
- `.env.test` 编写测试环境
- `phpunit.xml.dist` 编写单元测试框架配置

```powershell
# 使用 make 组件自动生成测试类
php bin/console make:test
```

> 参考文档: [PHPUnit 手册](https://phpunit.readthedocs.io/zh_CN/latest/)

## ORM 组件

Symfony/flex 提供的组件包 symfony/orm 中集成的 orm 组件是 `doctrine`，这里简单写下部署过程，具体内容查看官方文档学习即可。

关于 ORM 的概念参考 Architect 分类中的 ORM 文章 // TODO

```powershell
# 安装 orm 组件
composer req orm 
```

> 参考文档： [Doctrine 文档](https://www.doctrine-project.org/projects/doctrine-orm/en/2.10/index.html#welcome-to-doctrine-2-orm-s-documentation)

## 功能开发

### 创建文章和评论

#### Entity

Entity 的概念也会写在 Architect 中 // TODO

Entity 就是一个具有 setter/getter 方法简单对象，一般在 ORM 中用来代表单一个数据表。

这里先创建一个文章类和评论类来结构化储存博客系统的主要信息

在安装了 ORM 后，会在 src 目录下自动生成 Entity 目录用来存储简单对象

```powershell
# 使用 make 创建符合 psr 规范的简单 PHP 对象
# 根据提示可以填写字段信息
php bin/console make:entity Post
php bin/console make:entity Comment
```

另外，可以为 Entity 添加 Relation 字段来表示外键信息。

例如 Post 与 Comment 就是 OneToMany 的关系于是可以使用命令：

```powershell
php bin/console make:entity Post
comments
OneToMany
```

来设置 Post 和 Comment 的对应关系。

#### Entity 写入数据库

在 ORM 系统中，把数据从内存（new Entity() 对象）存入到数据库或其他系统中的方式称为 `persist` ( 持久化 )。

在安装了 `orm` 到项目中后，可以使用由 `Doctrine\ORM` 提供的 `EntityManager` 来管理 Entity 们，包括对他们进行持久化的操作。

那么如何获得 `EntityManager` 对象呢？

这里首先要引入集成测试的概念，在之前的测试方法中，我们一直使用的 `TestCase` 进行单元测试（UnitTest），单元测试的原则就是，仅对单一功能进行测试，比如某一 Entity 对象、某一工具类等，但是对于复杂的功能，**使得单一的对象需要其他依赖来完成整体测试时就需要使用集成测试来完成了**。





### 文章工厂类