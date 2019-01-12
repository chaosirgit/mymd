---
title: 扫描 PHP 危险文件脚本
tags:
  - Laravel
  - 脚本
  - PHP
categories:
  - 代码
keywords:
  - Laravel
  - 脚本
  - PHP
abbrlink: 8b68010d
date: 2019-01-12 13:30:20
---
## 上码
Laravel `php artisan make:command ScanHorse`:

```php
<?php

namespace App\Console\Commands;

define('MYFULLPATH', str_replace('\\', '/', (__FILE__)));

use Carbon\Carbon;
use Illuminate\Console\Command;

class ScanHorse extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'scan_horse';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = '扫描 PHP 木马';
    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    public $php_code = array();

    public $count = 0;
    public $scanned = 0;

    public $ignore = array('.', '..','Console' ); //忽略目录

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $setting = array();
        if ($this->confirm('是否扫描所有文件?')){
            $setting['is_all'] = true;
        }else{
            $setting['is_all'] = false;
            $setting['ext'] = $this->ask('请输入要追加的文件后缀,默认为','php | php? | phtml | shtml');
        }
        $dir = $this->ask('请输入扫描路径:',base_path());
        $dir = substr($dir,-1)!="/"?$dir."/":$dir;
        if ($this->confirm('是否开始扫描?')){
            $start_time = Carbon::now();
            $this->info('扫描开始--'.$start_time->toDateTimeString());
            ini_set('memory_limit','2048M');
            if($setting['is_all'] == true)
            {
                $is_ext="(.+)";
            }else{
                if(!empty(trim($setting['ext'])))
                {
                    $is_user = explode("|",$setting['ext']);
                    if(count($is_user)>0)
                    {
                        foreach($is_user as $key=>$value)
                            $is_user[$key]=trim(str_replace("?","(.)",$value));
                        $is_ext = "(\.".implode("($|\.))|(\.",$is_user)."($|\.))";
                    }
                }
            }
            $this->php_code = $this->getCodeArray();

            if(!is_readable($dir)){
                $dir = base_path();
            }

            $this->doScan($dir,$is_ext);
            $end_time = Carbon::now();
            $this->info('扫描结束--'.$end_time->toDateTimeString());
            $spent = $end_time->diffInSeconds($start_time);
            $this->info('耗时 '.$spent.' 秒,共 '.$this->scanned.' 个文件,其中 '.$this->count.' 个可疑文件');


        }



    }

    public function doScan($path,$is_ext){

        $replace=array(" ","\n","\r","\t");
        $dh = @opendir( $path );
        while(false!==($file=readdir($dh))){
            if( !in_array( $file, $this->ignore ) ){
                if( is_dir( "$path$file" ) ){
                    $this->doScan("$path$file/",$is_ext);
                } else {
                    $current = $path.$file;
                    if(MYFULLPATH==$current) continue;
                    if(!preg_match("/$is_ext/i",$file)) continue;
                    if(is_readable($current))
                    {
                        $this->scanned++;
                        $content=file_get_contents($current);
                        $content= str_replace($replace,"",$content);
                        foreach($this->php_code as $key => $value)
                        {
                            if(preg_match("/$value/i",$content))
                            {
                                $this->count++;
                                $filetime = date('Y-m-d H:i:s',filemtime($current));
                                $reason = explode("->",$key);
//                                $url = str_replace(REALPATH,HOST,$current);
                                preg_match("/$value/i",$content,$arr);
                                $this->error('-------------------------------------');
                                $this->error('NO.'.$this->count);
                                $this->error('文件: '.$current);
//                                $this->error('url: '.$url);
                                $this->error('更新时间: '.$filetime);
                                $this->error('原因: '.$reason[0]);
                                $this->error('特征: '.$reason[1]);
                                error_log('文件: '.$current.PHP_EOL.'更新时间: '.$filetime.PHP_EOL.'原因: '.$reason[0].PHP_EOL.'特征: '.$reason[1].PHP_EOL.'-------------------------------------'.PHP_EOL,3,'./scan.log');
                                break;
                            }else{

                            }
                        }
                    }
                }
            }
        }
        closedir( $dh );
    }


    public function getCodeArray(){
            return array(
                '后门特征->cha88.cn'=>'cha88\.cn',
                '后门特征->c99shell'=>'c99shell',
                '后门特征->phpspy'=>'phpspy',
                '后门特征->Scanners'=>'Scanners',
                '后门特征->cmd.php'=>'cmd\.php',
                '后门特征->str_rot13'=>'str_rot13',
                '后门特征->webshell'=>'webshell',
                '后门特征->EgY_SpIdEr'=>'EgY_SpIdEr',
                '后门特征->tools88.com'=>'tools88\.com',
                '后门特征->SECFORCE'=>'SECFORCE',
                '后门特征->eval("?>'=>'eval\((\'|")\?>',
                '可疑代码特征->system('=>'system\(',
                '可疑代码特征->passthru('=>'passthru\(',
                '可疑代码特征->shell_exec('=>'shell_exec\(',
                '可疑代码特征->exec('=>'exec\(',
                '可疑代码特征->popen('=>'popen\(',
                '可疑代码特征->proc_open'=>'proc_open',
                '可疑代码特征->eval($'=>'eval\((\'|"|\s*)\\$',
                '可疑代码特征->assert($'=>'assert\((\'|"|\s*)\\$',
                '危险MYSQL代码->returns string soname'=>'returnsstringsoname',
                '危险MYSQL代码->into outfile'=>'intooutfile',
                '危险MYSQL代码->load_file'=>'select(\s+)(.*)load_file',
                '加密后门特征->eval(gzinflate('=>'eval\(gzinflate\(',
                '加密后门特征->eval(base64_decode('=>'eval\(base64_decode\(',
                '加密后门特征->eval(gzuncompress('=>'eval\(gzuncompress\(',
                '加密后门特征->eval(gzdecode('=>'eval\(gzdecode\(',
                '加密后门特征->eval(str_rot13('=>'eval\(str_rot13\(',
                '加密后门特征->gzuncompress(base64_decode('=>'gzuncompress\(base64_decode\(',
                '加密后门特征->base64_decode(gzuncompress('=>'base64_decode\(gzuncompress\(',
                '一句话后门特征->eval($_'=>'eval\((\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)',
                '一句话后门特征->assert($_'=>'assert\((\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)',
                '一句话后门特征->require($_'=>'require\((\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)',
                '一句话后门特征->require_once($_'=>'require_once\((\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)',
                '一句话后门特征->include($_'=>'include\((\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)',
                '一句话后门特征->include_once($_'=>'include_once\((\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)',
                '一句话后门特征->call_user_func("assert"'=>'call_user_func\(("|\')assert("|\')',
                '一句话后门特征->call_user_func($_'=>'call_user_func\((\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)',
                '一句话后门特征->$_POST/GET/REQUEST/COOKIE[?]($_POST/GET/REQUEST/COOKIE[?]'=>'\$_(POST|GET|REQUEST|COOKIE)\[([^\]]+)\]\((\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)\[',
                '一句话后门特征->echo(file_get_contents($_POST/GET/REQUEST/COOKIE'=>'echo\(file_get_contents\((\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)',
                '上传后门特征->file_put_contents($_POST/GET/REQUEST/COOKIE,$_POST/GET/REQUEST/COOKIE'=>'file_put_contents\((\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)\[([^\]]+)\],(\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)',
                '上传后门特征->fputs(fopen("?","w"),$_POST/GET/REQUEST/COOKIE['=>'fputs\(fopen\((.+),(\'|")w(\'|")\),(\'|"|\s*)\\$_(POST|GET|REQUEST|COOKIE)\[',
                '.htaccess插马特征->SetHandler application/x-httpd-php'=>'SetHandlerapplication\/x-httpd-php',
                '.htaccess插马特征->php_value auto_prepend_file'=>'php_valueauto_prepend_file',
                '.htaccess插马特征->php_value auto_append_file'=>'php_valueauto_append_file'
            );
    }
}


```
