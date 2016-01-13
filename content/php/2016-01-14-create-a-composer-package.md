+++
title = "创建一个PSR-4规范的composer包"
author = "frostwong"
type = "post"
date = "2016-01-09T21:11:33+08:00"
description = "创建一个PSR-4规范的composer包"
draft = false
tags = ["Composer", "PSR-4"]
+++

### 下载安装Composer
到[composer官网](https://getcomposer.org/download/)按照指示下载。官方建议的是用`curl -sS https://getcomposer.org/installer | php`这条命令。可能有些同学会迷惑这是在干什么，这其实是为了保证你下载到的总是最新的composer。分析一下，首先看到了管道，把前面curl获取的结果交给php执行，执行的结果就是下载了一个最新的`composer.phar`到当前目录下。这时虽然也可以说已经能用了，但通常我还会把它链接到`/usr/local/bin`目录下，方便使用。
`sudo ln -s /home/frost/packages/composer.phar /usr/local/bin/composer`
这时在终端输入`composer`就应该能看到它的帮助信息了。

### 配置Packagist
原本这不是必需的，但由于众所周知的原因，我还是建议配置一下。具体见[这里](http://pkg.phpcomposer.com/)。
`composer config -g repositories.packagist composer http://packagist.phpcomposer.com`就可以了。

## composer.json
简单起见，这里使用monolog/monolog为例。
要使用composer，首先需要一个`composer.json`文件，它描述了项目依赖关系及其他一些元信息。
### `require`关键字
`require`可能是你第一个也是唯一需要制定的东西。它用来告诉Composer你的项目需要依赖哪些包。

```json
{
	"require": {
		"monolog/monolog": "1.0.*"
	}
}
```

如你所见，`require`接受一个包含包名(monolog/monolog)和版本限制(1.0.*)的映射的对象。
### 包名
包名由vendor名和项目名组成。两者经常会是相同的——vendor名的存在只是为了避免名称冲突。它可以允许两个人创建同一个名称的库`json`，这样两个包名就可能是`igorw/json`和`seldaek/json`了。
这里我们“需要”`monolog/monolog`，所以vendor名和项目名一样。建议对项目使用唯一的名字。稍后还允许同一个命名空间下的多个项目。如果你在维护一个库，这可以让你很容易的把它分成多个解耦的小部分。
### 包版本
上面的例子中我们“需要”版本`1.0.*`，这表示`1.0`版本的所有分支。
版本限制可以用多种不同的方式，[这里](https://getcomposer.org/doc/articles/versions.md)给出了详细的解释。
### 稳定性
默认情况下只考虑稳定版本。如果你也希望用RC, beta, alpha或者开发版，可以用[稳定性标识](https://getcomposer.org/doc/04-schema.md#package-links)。为了不用为每个包单独设置稳定性标识，还可以用[最小稳定性](https://getcomposer.org/doc/04-schema.md#minimum-stability)
### 安装依赖
要安装依赖，只需要执行`composer install`即可。
这会找到符合版本限制的最新版本的`monolog/monolog`，并且把它下载到`vendor`目录。把第三方的代码放在一个名为`vendor`的目录是一个规范。在本例中，会把它放在`vendor/monolog/monolog`。
> 如果你使用git，通常会把`vendor`目录放在`.gitignore`中。

你会注意到`composer install`命令还会在当前目录下生成一个`composer.lock`文件。
### `composer.lock`锁文件
安装完依赖后，Composer会把它安装的精确版本号写入`composer.lock`文件中。它把项目锁定在某个特定的版本号。
把`composer.lock`文件和`composer.json`一起提交到版本控制中。这很重要，因为`install`命令会检查锁文件是否存在，如果存在，它就会下载指定的精确版本，否则就会按照`composer.json`的描述下载符合约束的最新版本。
这意味着每个下载了你的项目的人看到的都是**完全**相同的代码，不会因为依赖更新了就自动更新依赖。`update`命令会将依赖更新到符合要求的最新版本，然后更新锁文件。
`update`命令也可以指定单独的包名来更新指定的包。
### Packagist
Packagist是Composer的信息库。一个Composer信息库本质上是一个包的源——从这里你可以获取包。Packagist希望成为每个人用的中心信息库。这意味着你可以自动`require`任何这里存在的包。
### 自动加载
对于指定了自动加载信息的库，Composer会生成一个`vendor/autoload.php`文件。你可以放心的`include`这个文件而Composer会完成剩下的工作。
```php
require __DIR__ . '/vendor/autoload.php
```
这让使用第三方代码很方便。比如，如果你的项目依赖Monolog，你可以马上开始使用它的类，它会被自动加载。

```php
$log = new Monolog\Logger('name');
$log->pushHandler(new Monolog\Handler\StreamHandler('app.log', Monolog\Logger::WARNING));
$log->addWarning('FOO');
```

你还可以把自己的代码添加到`composer.json`的`autoload`字段。

```json
{
	"autoload": {
		"psr-4": {"Acme\\": "src/"}
	}
}
```

Composer就会为Acme命名空间注册一个PSR-4的自动加载器。
这里你定义の一个命名空间到目录的映射。`src`目录会是你的项目的根目录，和`vendor`在同级目录。比如文件`src/Foo.php`包含`Acme\Foo`类。
在添加了`autoload`字段之后，需要重新运行`dump-autoload`来重新生成`vendor/autoload.php`文件。包含这个文件还会返回一个自动加载器的实例，你可以把返回值存储下来并且添加更多的命名空间。这在测试时会很有用。比如

```php
$loader = require __DIR__ . '/vender/autoload.php';
$loader->add('Acme\\Test\\', __DIR__);
```

