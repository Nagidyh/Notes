[TOC]

## OOP

通过 ”形“ 与 ”本“ 来理解类与对象的关系。

原则上来讲，大部分 OOP 类需要有如下理解

- 定义了属性以及方法，可以通过方法来对属性进行加工。

### Zend 原理

#### PHP 源码类型定义

首先从 `zend` 源码上来看 PHP 对变量的定义：

```c
#zend/zend_types.h
typedef union _zend_value {
    zend_long lval; 		/* long value */
    double dval; 			/* double value */
    zend_refcounted *counted;
    zend_string *str;
    zend_array *arr;
    zend_object *obj; 		// 用于对象
    zend_resource *res;
    zend_reference *ref;
    zend_ast_ref *ast;		// 用于常量表达式
	zval *zv;
    void *ptr;
    zend_class_entry *ce;
    zend_function *func;
    struct {
        uint32_t w1;
        uint32_t w2;
    } ww;
} zend_value;
```

其中，`zend_object_value` 就是变量中的一个结构。

接着我们来看 `zend_object_value` 结构的具体组成。

#### PHP Object 的底层存储结构

![image-20211118092259497](https://notes-image-host.oss-cn-beijing.aliyuncs.com/img/image-20211118092259497.png)

```c
#zend/zend_tpyes.h
struct _zend_object {
    zend_refcounted_h gc;
    uint32_t	handle;	// TOOD: may be remove ???
    zend_class_entry *ce;	// 入口
    const zend_object_handlers *handlers;
    HashTable *properties;
    zval properties_table[1];
}
```

在 PHP5 中，对象在底层的实现是采取的 ”属性数组 + 方法数组“ 来实现的。

在上面的源码中：

- `ce` 是存储该对象的类结构，在对象初始化时保存了类的入口，相当于类指针的作用。
- `properties` 是一个 HashTable，用来存放对象属性。guards 用来阻止递归调用。

类的标准方法在 `zend/zend_object_handlers.h` 文件中定义，具体实现规则在 `zend/zend-object_handlers.c` 文件中。

> // TODO 后续会仔细学习 PHP Zend 和拓展开发，届时再做详细的源码阅读。

#### 原理简单总结

- 类是定义一系列属性和操作的模板，而对象则把属性进行具体化，然后**交给类处理**。
- 对象就是数据，**对象本身不包含方法**，但是对象有一个 “指针” 指向一个类，这个类可以有方法。
- 方法描述不同属性所导致的不同表现。
- 类和对象是不可分割的，有对象就必定有类与其对应。
  - 此处也有一个特殊情况：由标量强制类型转换 `(object)` ，没有类与其对应。此时，PHP 中一个称为 “孤儿” 的 `stdClass` 类就会收留这个变量。

### 魔术方法

PHP 的魔术方法由双下划线定义 `__`，是具有特殊的作用一些方法。可以看做就是 PHP 的 “语法糖”。

语法糖往往指的是，**没有给语言本身添加新功能**，而是指提供了一些较为易读、易用的**写法**，在 PHP 中的引用、SPL 等都属于语法糖。

例如 `__construct` 与 `__destruct` 方法就是常用的魔术方法。

#### PHP 的重载

在 Java 中的重载指的是，在 Runtime 中，会根据签名的不同来调用不同的方法。比如 Java 就支持多个不同的构造函数，只要保证方法的签名不同即可。

但在 PHP 中，重载的意思是指的是**动态的 “创建” 类属性和方法**。

比如 `__set` 和 `__get` 方法，就被归类为**魔术重载**方法。

```php
public function __set($name, $value) {
    echo "Setting $name to $value \r\n";
    $this->$name = $value;
}
public function __get($name) {
	return $name ?? 'undefined';
}
```

`mixed __call(string $name, array $arguments)`: 当调用一个不可访问的方法（未定义，或不可见）时，`__call` 方法就会被调用，其中 `$name` 是要被调用的方法名，`$arguments` 是参数数组。

其实在很多的 MVC 框架中，`__call` 的使用是比较频繁的，例如一个控制器调用了一个不存在的方法，那么就可以在 `__call` 中进行一些友好的处理。

使用 `__callStatic` 实现一个简单的 ORM 模型。

```php
# ActiveRecord
abstract class ActiveRecord 
{
    protected static $table;
    protected $fieldvalues;
    public $select;
    
    static function findById($id) {
        $query = 'SELECT * FROM'
            .static::$table
            ."WHERE id = $id";
        return self::createDomain($query);
    }
    function __get($fieldname) {
        return $this->fieldvalues[$fieldname];
    }
    static function __callStaic($method, $args) {
        $field = preg_replace('/^findBy(\w*)$/', '${1}', $method);
        $query = 'SELECT * FROM'
            .static::$table
            ."WHERE $filed = '.$args[0].'";
        return self::createDomain($query);
    }
    private static function createDomain($query) {
        $klass = get_called_class();
        $domain = new $klass();
        $domain->fieldvalues = array();
        $domain->select = $query;
        foreach ($kalss::$fields as $field => $type) {
            $domain->fieldvalues[$field] = 'TODO: set from sql result';
        }
        return $domain;
    }
}

# Customer
class Customer extends ActiveRecord {
    protected static $table = 'custdb';
    protected static $fields = array(
    	'id' => 'int',
        'email' => 'varchar',
        'lastname' => 'varchar'
    );
}

# Sales
class Sales extends ActiveRecord {
    protected static $table = 'salesdb';
    protected static $fields = array(
    	'id' => 'int',
        'item' => 'varchar',
        'qty' => 'int'
    );
}

## Test
assert("SELECT * FROM custdb WHERE id = 123" == Customer::findById(123)->select);
assert("TODO: set from sql result" == Customer::findById(123)->email);
assert("SELECT * FROM salesdb WHERE id = 321" == Sales::findById(321)->select);
assert("SELECT * FROM custdb WHERE Lastname = 'Denoncourt'" == Customer::findByLastname('Denoncourt')->select);
```

`__call` 还有一个比较常用的场景，比如想用 PHP 实现类似 JS 的链式操作。

`strlen(trim($str));` 写成 `$str->trim()->strlen()` 。

可以实现一个 String 类，使用 `__call` 方法，在调用 trim 和 strlen 的时候执行 call_user_func 即可。

### 与 Java 的区别

PHP 由于有了 `__set` 和 `__get` 魔术方法，使得动态的增加对象属性字段变得很方便，而对于 Java 来说，要实现类似的效果，就必须借助反射 API 或直接修改编译后字节码来实现。

这就是动态语言的优势，简单、灵活。

### 命名空间与自动加载

#### 命名空间

广义上讲，命名空间是**一种封装事务的方法**。在很多地方都有这种抽象概念。例如在操作系统中，目录用来将文件进行分组，对于目录中的文件来说，目录就扮演了命名空间的角色。

先看一段代码：

```php
<?php 
namespace A\B\C;
class Exception extends \Exception {}
$a = new Exception('hi');
$b = new \Exception('h1');
$c = new ArrayObject;
```

- 第二行、创建了命名空间 `A\B\C`，这就表示整个代码下的文件，都属于这个命名空间。

- 第三行，新建了一个 `Exception` 类，它继承了 `\Exception` 类，首先，我们创建的这个 `Exception` 类是属于 `A\B\C` 这个命名空间的。它所继承的 `\Exception` 类前面有一个 `\`，这个 `\` 代表的是**根命名空间**，PHP 规定，**所有的 PHP 内置类，都默认属于 `\` 命名空间下**，有继承关系则继承上一级的命名空间。

- 第四行，在默认没有声明命名空间时创建的对象就默认使用当前命名空间。

- 第五行，使用了命名空间，所以声明的是根命名空间下 PHP 的内置 `Exception` 对象。

- 第六行，我们想创建一个 `ArrayObject`类，PHP 内置是有这个类的，但是由于我们没有加命名空间限定，所以默认使用了当前命名空间，显然当前命名空间并没有这个类，因此就会报错。

  如果想要正常的使用这个类，就需要和第五行一样，使用根命名空间。

  **PHP 不会自动退化到根命名空间。**

  但存在一个例外，对于函数和常量来说，如果当前命名空间不存在该函数或常量，PHP 会自动使用全局空间中的函数和常量。（这也是可以直接使用 PHP 内置函数和常量的原因）

#### 自动加载

在 PHP 中，为了能够使用某一个类，需要先加载这个类所在的文件。

在 PHP 5.1.2 之前，只能使用 `include/require` 来加载，从 PHP 5.1.2 起，PHP 提供了自动加载类的方法 —— `spl_autoload_register`。这个方法的原型是这样的：

`bool spl_autoload_register([callable $autoload_function [, bool $throw = true [, bool $prepend = false]]])`

- `autoload_function`：欲注册的自动装载函数。如没有提供参数，则自动注册 `autoload` 的默认实现函数 `spl_autoload()`。
- `throw`：该参数设置了 `autoload_function` 无法成功注册时，`spl_autoload_register` 是否抛出异常。
- `prepend`：如果是 true，`spl_autoload_register` 会添加函数到队列首，而不是队列尾。在一个项目存在多个自动加载器时，这个参数很有用。

有了这个方法，就不需要每个文件都去加载了。

```php
<?php 
class Loader 
{
    public static function loadClass()
    {
        
    }
    public static function loadLibClass($class)
    {
        include('Lib.driver.db.'.$class);
        include('Helper.'.$class);
    }
}
spl_autoload_register(array('Loader','LoadLibClass'));
```

这样，在创建一个对象时，就不需要显示的使用 `include` 和 `require` 来加载这个类了。但需要注意的一点是，如果使用了命名空间，那么由于在使用命名空间的时候，class 类名中带了命名空间限制符，在解析并加载时需要注意并做特殊处理。

正是因为有了命名空间与自动加载，才让 PHP 生态发展出了 `composer` 包机制，进而保证能任意引入第三方类库。

> 简单提一下，后续再做详细笔记 // TODO

### 继承与多态

OOP 的优势在于复用。继承与多态都是对类进行复用，继承是对类级别的复用，多态是方法级别的复用。

PHP 有多态吗？

#### 继承与组合

提到继承，就不得不提组合。组合其实就是在一个类中，创建另一个类的对象，并声明为本对象的属性。而继承，则是在类的层面进行实现，声明一个类为另一个类的子类，那么他就会继承来自父类的所有方法和属性，也有自己本身单独的方法与属性。

语法上，在继承中使用 `::`（范围解析操作符）来使用父类的方法和属性，PHP 通过 `parent` 与 `self` 指代自身与父类。此外，`::` 操作符还用于调用类常量和静态方法。

继承与组合的差异，更多是在概念层面，其中继承代表的关系更多的是 “**is、as**” 而组合代表的是 "**need**"，从方法复用的角度考虑，如果两个类具有很多相同的代码和方法，那么就可以抽像出一个公共父类提供这些公共方法，然后两个类作为子类，提供个性的方法，这种情况使用继承的语意更好。

但是在实际开发中，继承与组合的取舍往往不是很明了。所以需要遵循 “低耦合” 的标准。

解耦，是要解除模块之间的依赖性，使得模块之间都相对独立，模块之间的接口尽量相对少而简单。

与组合相比，继承的耦合度相当的高，但是也有其优点，继承是便于扩展的。

在设计继承关系时，应该遵循以下几点：

- 精心设计专门用于被继承的类，继承树的抽象层应该比较稳定，一般不能多于三层。
- 对于不是专门用于被继承的类，禁止其被继承，也就是使用 `final` 修饰，这样做的好处是既可以防止重要方法被非法复写，也便于让 IDE 寻找优化的机会。
- 优先考虑组合以提高代码重用性。
- 子类是一种特殊类型，而不是父类的一个角色。
- 子类应该是父类的基础上拓展，而不是覆盖或使其功能失效。
- 底层代码多用组合，顶层/业务层代码多用继承。底层使用组合可以提高效率，避免对象过于。顶层使用继承可以提高灵活性，让业务使用更方便。

#### 多继承与 Traits

在 C ++ 中，一个类可以继承多个父类，组合两个父类的功能，C ++ 就是以这种模型来增加继承的灵活性，但是这种模型往往会带来 “菱形问题”，为其使用带来了不少困难，模型变得十分复杂，因为大多数语言都放弃了这个模型。

如果多重继承过于复杂，那么其他语言是怎么增强继承的拓展性的呢？在 PHP 5.4 中，引入的语法结构 `Traits` 是一种很好的解决方案，他的来源是 C++ 、Ruby 和 Mixin 以及 Scale 里的 Traits。

Traits 一种除 extends、implements 外另一种拓展对象的方式， Traits 既可以使单继承语言获得多继承的灵活，又避免了多继承带来的种种问题。

> 后续添加详细的 Traits 用法和原理笔记

#### 多态

多态确切含义是指，同一类对象收到相同的消息，会得到不同的结果。而这个消息是不可预测的。

多态，顾名思义，就是多种状态，多种结果。

以 Java 为例子，Java 是一种强类型的语言，因此变量和函数的返回都是有状态的。比如实现一个 add 函数的功能，他的参数可能是 int 也可能是 float ，而返回值也可能是 int 和 float。在这种情况下，add 函数是有状态的，它有多种可能的运行结果。在实际使用时，编译器会自动匹配适合的那个函数。这属于函数重载的概念。需要说明，**重载并不是 OOP 的概念，和多态也不是一个概念，它属于多态的一种实现方式。**

多态性是一种通过多种状态或阶段描述相同对象的编程方式，它真正的意义在于：**实际开发中，只要关心一个接口或基类的编程，而不必去关心一个对象所属的具体类。**

##### PHP 的多态

很多人讲，PHP 是没有多态的，而 PHP 官方手册也找不到对多态的详细描述。

其实因为 PHP 作为一门脚本语言，其本身是不需要的多态这个概念的，因为其本身就是多态的。

举个例子

```php
<?php 
class Employee 
{
	protected function working()
    {
        echo 'need overload';
	}
}

class Teacher extends Employee
{
	public function working()
    {
		echo 'read after me ~ Abandon';
    }
}

class Coder extends Employee
{
    public function working()
    {
        echo 'writing bug ...'
    }
}

function doprint($obj)
{
	if ('Employee' == get_class($obj)) {
        echo 'Error';
    } else {
        $obj->working();
    }
}
doprint(new Teacher()); // read after me ~ Abandon
doprint(new Coder());	// writing bug ...
doprint(new Employee());// need overload
```

通过传入对象所属的类不同来调用其同名方法，得到的结果不同，这是多态吗？如果站在 C++ 的角度来说，这不是多态，这只是不同类对象的不同表现而已。C ++ 中的多态指的是在运行过程时对象的具体化，指同一类对象调用相同的方法返回的结果不同。

C ++ 的例子：

```C++
#include <cstdlib>
#include <iostream>
/**
虚函数实现多态
*/
using namespace std;
class father {
	public:
        father():age(30){cout<<"father constructor，age"<<age<<"\n";}
        ~father():{cout<<"father distruct"<<"\n";}
        void eat(){cout<<"father eating 3kg"<<"\n";}
        virtual void run(){cout<<"father run 10km"<<"\n";} // 虚函数
    protected:
    	int age;
};
class son:public father{
    public:
    	son(){cout<<"son construct"<<"\n";}
    	~son(){cout<<"son distruct"<<"\n";}
    	void eat(){cout<<"son eating 1kg"<<"\n";}
    	void run(){cout<<"son run 3km"<<"\n";}
    	void cry(){cout<<"crying"<<"\n"}
};

int main(int argc, char *argv[])
{
	father *pf = new son; 
    pf->eat();
    pf->run();
    delete pf;
    system("PAUSE");
    return EXIT_SUCCESS;
}

// result:
"father construct, age 30"
"son construct"
"father eating 3kg"
"son run 3km"
"father distruct"
```

上面的代码首先定义了一个父类 `father`，然后定义了一个子类 `son`，子类继承了父类的一些方法，并拥有自己的方法。

通过 `father *pf = new son;` 语句创建了一个派生类（子类），并把该派生类的对象赋值给了基类（父类）指针，然后用该指针访问父类中的 `eat` 和 `run` 方法。

根据运行结果来看，由于 `run` 方法在父类中加了 `virtual` 关键字，表示该函数有多种形态，可能被多个对象所拥有。也就是说，多个对象在调用同一个同名函数时，会产生不同的结果。

这个例子与 PHP 的有何不同呢？

C ++ 这个例子所创建的**对象是一个指向父类的子对象**，还可以创建更多的派生类对象，然后向上转型为父类对象。这些对象，都是父类的同一类对象，但是在**运行时，却能调用到其派生类的同名函数**。

而 PHP 例子中，无论是 Coder 还是 Teacher 他们指向的类，都是他们所对应的 Coder 和 Teacher ，在调用时候只是简单的不同类的对象调用而已，所以说他不是 C ++ 立场上的多态的。

这里体外话讲讲自己对于 Java 多态的理解，在 Java 中，编译器通过动态绑定来调用虚方法（我也不知道这个说法是否有误，待以后详细学习后可能会更正），与 C ++ 相似 Java 在声明对象时候也是 `Father pf = new Son();` Son 被向上转型成为了 Father 类的对象，所有标注了静态绑定的方法 `final`、`static`、`private` 将直接指向 Father 类的方法本身，而其他方法如果有子类的实现 `Override` 将会被编译为 JVM 指令 `invokevirtual` 可见，Java 本身也是通过查找子类的虚方法来实现多态的，但是 Java 将指针的概念淡化了。

而由于 PHP 自身本就是弱类型的语言，所以在声明 Son 时，自然而然就指向了 Son 类型的 run 方法，即在调用过程中，实际调用的就是 Son 类中的 run 方法。在源码层面也是通过 `ce` 访问到类的方法直接进行调用，所以 PHP 并不存在 Java 和 C++ 概念中的多态（指相同类型的对象调用同一个方法，结果不同），因为 PHP 自身就是多态的。

PHP 没有对象转型机制，所以不能像 C++ Java 一样实现讲派生类赋值给基类，再在调用函数时动态的改变其指向的操作。在 PHP 中对象都是确定的，是不同类型的对象，所以从这个角度讲，这并不是真正的多态。

那么 PHP 有没有办法实现动态的调用函数指向呢？其实是可以实现的，PHP 中的多态实现：

```php
<?php 
interface Workable 
{
	public function working();    
}

class Teacher implements Workable 
{
    public function working() 
    {
        echo 'read after me ~ keep on';
    }
}

class Coder implements Workable
{
	public function working()
    {
        echo 'writing feature ...';
    }
}

function doprint(Workable $w)
{
    $w->working();
}
$a = new Teacher();
$b = new Coder();
doprint($a);
doprint($b);
```

在这段代码中，函数 `dorpint(Workable $w)` 限定了参数类型，所以无论是 Teacher 还是 Coder 在调用函数执行时，都被看做了 Workable 的实现，因此**他们是同一类型，但是却调用了不同的方法**。这就是多态。

弱类型的语言的多态实现和概念与强类型语言相比，是有一定区别。

相比而言弱类型语言实现多态会更加的简单、灵活。

总结几点

- 多态指同一类对象在运行时的具体化。
- PHP 是弱类型语言，实现多态更简单、灵活。
- 类型转换不是多态。
- PHP 中的父类和子类看做 ”继父“ 和 ”继子“ 的关系，它们存在继承关系，但不存在血缘关系。因此子类无法向上转型为父类，从而失去多态最典型的特征。
- 多态的本质就是 if ... else ，只不过实现的层级不同。

### 面向接口编程

面向接口编程更形象的是说法是面向规范编程，这一点在 PHP 的中体现的非常明显。

比如在 Java 中，由 JDBC 提供了接口具体规范，而类似 Mysql 之类的数据库去实现相应的规范，开发者只用安装 Mysql 的 SDK 本身，并不需要关心具体实习只用面向接口便可以使用，这就是面向规范编程的典型。

通常在大型项目里，会把代码进行分层和分工。核心开发人员和技术经理编写核心流程和代码，往往是以接口的形式给出，而基础开发人员则针对这些接口，填充代码，如数据库操作等。这样，核心技术人员把更多的精力投入到了技术攻关和业务逻辑中。

前端针对接口编程，只管在 Action 层调用 Service，不需要关心具体实现。

后端负责 Dao 和 Service 的接口实现，就实现了代码分工合作。

#### PHP 中的接口

先来看一段 PHP 中的代码：

```php
interface Mobile
{
    public function run();
}

class Plain implements Mobile
{
    public function run()
    {
        echo 'I am plain'.;
    }
    public function fly()
    {
        echo 'go fly ~';
	}
}

class Car implements Mobile
{
    public function run()
    {
        echo 'I am Car';
    }
}

class Machine
{
	function demo(Mobile $a)
    {
        $a->fly();	// 该接口是没有这个方法的
    }
}

$obj = new Machine();
$obj->demo(new Plain());	// 运行成功
$obj->demo(new Car());		// 运行失败
```

这段代码实际上是错误的，不符合接口语义。但是在 PHP 中，对 plain 的示例进行检测时，是可以运行的。也就是说，PHP 只关心是否实现这个方法，而不关心接口语义是否正确。

按理说，接口应该是起一个强制规范和契约作用，但是这里接口的约束并没有奇效，也打破了契约。

在 Java 中，接口就是一种 **type** ， 如果你打破了我们之间的契约，你的行为变得无法控制，那就是非法的。是无法通过编译的，这符合正常逻辑，也起到了接口作为规范的作用。

接口不仅是规范的实现者，还是接口的执行者，不允许调用接口中不存在的方法。

但是从上面的例子可以看出，PHP 的接口作为规范和契约的作用打了折扣。

在 PHP 中接口的语义是有限的，在 PHP 中接口可以淡化为设计文档，为团队起到一个基本的契约作用。

由于 PHP 是弱类型语言，且强调灵活，所以并不推荐大规模的使用接口，而是仅在部分 “内核” 使用接口，因为 PHP 的接口已经失去了很多接口应该拥有的语义。

所以从语义上考虑，可以更多的使用抽象类而不是接口。

另外，PHP 5 对 OOP 特性也做了一些增强，其中就有一个 SPL （PHP 标准库）的尝试。SPL 中实现了一些接口，其中最主要的是 `Iterator` 迭代器接口，通过实现这个接口，就能使对象能够用于 foreach 结构，从而在形式上比较统一。

比如 SPL 中就有一个`DirectoryIterator`类，这个类在继承 `SplFileInfo` 类的同时，实现了 `Iterator`、`Traversable`、`SeekableIterator` 这三个接口，那么这个类的实例在获得了父类 `SplFileInfo` 的全部功能外，还能够实现 `Iterator` 接口所展示的操作。

```php
# Iterator 接口原型
current()
    # This method returns the current index's value. You are solely responsible for tracking what the current index is as the interface does not do this for you.
key()
    # This method return the value of the current index's key. For foreach loops this is extermely important so that the key value can be populated.
next()
    # This method moves the internal index forward one entry.
rewind()
    # This method should reset the internal index to the first element.
valid()
    # This method should return true or false if there is a current element. It is called after rewind() or next().
```

如果一个类实现了 `Iterator` 接口，就必须实现这五个方法，如果实现了这五个方法，那么就可以很容易对这个类的实例进行迭代。

这里的 `DirectoryIterator` 类之所以使用 foreach 进行迭代，是因为 SPL 已经为其实现了 `Iterator` 接口。

使用接口设计时应该注意以下几点：

- 接口作为一种规范和契约存在。作为规范，接口应该保证可用性；作为契约，接口应该保证可控性。
- 接口只是一个声明，一旦使用了 interface 关键字，就应该实现它。
- PHP 中接口有两个不足，一是没有契约限制（可以不实现定义的方法），二是缺少足够多的内部接口。

常见的很多设计模式中，大部分都是围绕接口展开的。

### 反射 API

这里不展开写了，PHP 的反射和 Java 的反射在使用上非常类似。

总而言之，反射是在 OOP 编程中由语言本身提供的一组可以对对象进行内省的 API，可以通过这一组 API 来实现根据对象找到他的类本身。

这里简单记录一下常用的几个方法

```php
# 反射 API
$reflect = new \ReflectionObject($object);
$props = $reflect->getProperties();
foreach ($props as $prop) {
    print $prop->getName().'\n';
}
$methods = $reflect->getMethods();
foreach ($methods as $method) {
    print $method->getName().'\n';
}

# get_class 方法获取类指针
// 返回对象关联数组
var_dump(get_object_vars($student));
// 类属性
var_dump(get_class_vars(get_class($student)));
// 返回由类的方法名组成的数组
var_dump(get_class_methods(get_class($student)));
// 获取所属类
get_class($student);
```

常用于通过动态代理实现 Hook ，来优化代码等场景。

不过也与 Java 类似，反射的内存开销非常大，也会破坏 OOP 本身的封装性。

### 异常和错误处理

首先是区分错误（Error）和异常（Exception）错误指的是在编译过程中，无法通过语义检查的、无法运行的。而异常指的是在运行过程中，不符合预期的情况，属于业务和逻辑的中断。

与 Java 作一个简单的比较：

```php
# 除 0 
<?php 
$a =  null;
try {
    $a = 5/0;
} catch(exception $e) {
    $e->getMessage();
    $a = -1;
}
echo $a;
# reslut 
Warning: Division by zero in ....
...
```

Java 中捕获异常

```java
public class ExceptionTry 
{
    public static void tp() throws ArithmeticException 
    {
        int a;
        a = 5/0;
        System.out.println('result':+a);
    }
    
    public static void main(String[] args)
    {
        int a;
        try {
            a = 5/0;
            System.out.println('result:'+a);
        } catch (ArithmeticException e) {
            e.printStackTrace();
        } finally {
            a = -1;
            System.out.println('reslut:'+a);
        }
        try {
            ExceptionTry.tp();
        } catch (Exception e) {
            System.out.println('catched Exception');
        }
    }
}

// result
java.lang.ArithmeticException: / by zero
    at ....
result: -1
catched Exception
```

从上面的结果其实不难看出，对于除零这种场景，PHP 认为这是一个错误，直接触发了编译器的 warning，而不会进入异常处理流程，故最终 $a 的值并不是 -1。也就是说，这个错误并没有正常的进入异常处理流程，只有当你主动 throw 后，PHP 才能够捕获异常（也有一些异常 PHP 能够自动捕获）。

而对于 Java，除零属于 ArithmeticExcepiton，会对其进行捕获，并进入异常处理流程。

也就是说，PHP 通常是无法自动捕获有意义的异常的，它把所有不正常的情况都视作了错误，你想要捕获这个异常，就得使用 if...else 结构，保证代码是正常的，然后判断被除数是否为零，则手动 throw 出异常，再捕获。PHP 能够自动捕获的异常主要有两大类，pdoexception 和 reflectionexception。	

PHP 应该在什么情况下使用异常：

#### 悲观预测

假设一个场景，程序员悲观的认为自己的这段代码在高并发条件下可能产生死锁，那么他就会悲观的抛出异常，在发生死锁时进行捕获，对异常进行细致的处理。

#### 程序的需要、对业务的关注

假设一个场景，有一个上传文件的业务需求，要把上传的文件保存在一个目录里，并在数据库中插入一个这个文件的记录，那么这两个步骤就是互相关联、密不可分的一个集成业务，缺一不可。文件保存失败，而插入成功会导致无法下载文件；而保存成功但没有记录会导致文件变成死文件，永远得不到下载。

那么假设文件保存没有提示，但是保存失败会抛出异常，访问数据库也一样，插入成没有提示，失败则自动抛出异常，就可以把这两个有可能抛出异常的代码段包在一个 try 语句里，然后用 catch 捕获错误，在 catch 代码段删除没有被记录到的文件或是没有文件的记录，以保证业务的数据一致性。

如果只是象征性的 try...catch，然后打印一个报错，最后 over。这样的异常不如不用。

异常的两种处理方式：

发生后立刻捕获：

```php
try {
	if ( exception happen ) thrwo new Excepiton('exception msg');
} catch (Exception $e) {
    execute Exception
}
```

如果业务很重要，那么异常越早处理越好，以保证程序在意外情况下能保持业务处理的一致性。比如一个操作有多个前提步骤，突然最后一个步骤异常了，那么其他前提操作都要消除掉才行，以保证数据的一致性。并且在这种核心业务下，有大量代码来做善后工作，进行数据补救，这是一种比较悲观的而又很重要的异常。我们应该把异常消灭在局部，避免异常的扩散。

分散拋出异常、集中捕获：

```php
function func {
	if ( exception happen ) thrwo new Excepiton('exception msg');
}
// 集中处理
try { 
	func()
} catch (Exception $e) {
    excute Exception
}
```

如果异常不那么重要，并且在单一入口、MVC风格的应用中，为了保持代码的流程和统一，则常常采用这种异常处理方式。这种方式更强调业务流程走向，对善后工作并不关心。这是一种次要异常，其将异常集中处理从而是流程更专一。

异常处理可以看做是一种事务，还可以看成是一种内建的恢复系统。如果程序某部分失败，异常将恢复到某个已知的稳定点，而这个点就是程序的上下文环境，而 try 块里面的代码就保存 catch 所要知道的程序上下文信息。

#### 语言级别的健壮性要求

在语言健壮性这一点，PHP 是不足的。以 Java 为例，Java 是一种面向企业级开发的语言，强调健壮性。Java 支持多线程，Java 认为，多线程被中断这种情况是彻彻底底的无法预料和避免的。所以 Java 规定，凡是用了多线程，就必须正视这种情况。要么抛出，要么处理。总之，必须面对 `InterruptedException`，不准回避。当然你可以什么都不做，但你必须意识到。



异常，就是无法控制的运行时错误，会导致出错时中断正常的逻辑运行，该异常代码后面的逻辑都不能继续运行。那么 try...catch 处理的好处就是：**可以把异常造成的逻辑中断破坏降到最小范围内，并且经过补救处理后不影响业务逻辑的完整性；乱拋异常和只拋不捕获，或捕获而不补救，会导致数据混乱。**这就是异常处理的重要作用，通过精确控制运行时的流程，在程序中断时，有预见的进行补救，是逻辑流程能够回归正轨。

#### PHP 的异常处理



#### PHP 7 的异常处理

