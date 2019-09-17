---
title: laravel解析Excel文件
date: 2019-08-04 10:31:36
categories: laravel
---

## 安装

```bash
#该插件已经更新到了3.1版本，目前我项目中使用的是2.1，这篇文章也是针对2.1版本的
composer require maatwebsite/excel=2.1.*
```

## 配置

在laravel项目的`config/app.php`文件中，找到`providers`和`aliases`数组，分别在两个数组中添加下面的内容。

```php
'providers'=>[
    ...

    Maatwebsite\Excel\ExcelServiceProvider::class,
],

'aliases'=>[
    ...

    'Excel' => Maatwebsite\Excel\Facades\Excel::class,
]
```

配置好后，在项目的命令行中执行下列命令，可以在项目中生成`config/excel.php`文件，可以通过该文件对该插件进行配置。

```bash
php artisan vendor:publish --provider="Maatwebsite\Excel\ExcelServiceProvider"
```

## 解析Excel文件

直接调用`load`方法，第一个参数是导入的文件名或文件地址。解析过后可以通过`toArray()`方法将解析后的结果转换为数组。

```php
$arr = [];
\Excel::load('企业导入模板.xlsx', function($reader) {
            $arr = $reader->get()->toArray();
        });
```

## 注意
如果出现因为中文表头导致解析数据不完整的时候，请在`config/excel.php`文件中对`to_ascii`进行修改。

```php
'to_ascii'=>false,
```

如需其他配置问题，请参考[官方文档](https://laravel-excel.com/ "官方文档")