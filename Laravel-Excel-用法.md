---
title: Laravel Excel 用法
tags:
  - Laravel
  - PHP
  - Excel
  - Composer
categories:
  - Laravel
abbrlink: 489422f2
date: 2018-06-08 12:44:33
---

## Laravel-Excel 插件用法
### 安装
#### `composer` 安装
```
composer require "maatwebsite/excel:~2.1.0"
```

#### 安装完成后，修改 `config/app.php` 在 `providers` 数组内追加如下内容
```php
Maatwebsite\Excel\ExcelServiceProvider::class,
```

#### 同时在 `aliases` 数组内追加如下内容:
```php
'Excel' => Maatwebsite\Excel\Facades\Excel::class,
```

#### 生成配置文件 `config/excel.php` :
```php
php artisan vendor:publish --provider="Maatwebsite\Excel\ExcelServiceProvider"
```

### 用法

#### 解析 Excel 文件

```php
ini_set ('memory_limit', '1024M');
$data = Excel::load('excel.xlsx',function ($reader){
        },'UTF-8')->toArray();
```

#### 将数据导成 Excel 文件
```php
// 导出 Excel 并能直接在浏览器下载
# $export_file_name = 要生成的文件名
Excel::create($export_file_name, function ($excel) {
    $excel->sheet('Sheetname', function ($sheet) {
        $sheet->appendRow(['name', 'age']);
        $sheet->appendRow(['LiLei', '22']);
        $sheet->appendRow(['HanMeimei', '22']);
    });
})->download('xls');

// 导出 Excel 并存储到指定目录
Excel::create($export_file_name, function ($excel) {
    $excel->sheet('Sheetname', function ($sheet) {
        $sheet->appendRow(['name', 'age']);
        $sheet->appendRow(['LiLei', '22']);
        $sheet->appendRow(['HanMeimei', '22']);
    });
})->store('xls', $path);
```
