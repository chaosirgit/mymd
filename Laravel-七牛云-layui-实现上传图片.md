---
title: Laravel 七牛云 layui 实现上传图片
tags:
  - Laravel
  - PHP
  - 七牛云
  - layui
  - 上传
  - 图片
categories:
  - Laravel
abbrlink: 780b6301
date: 2018-06-08 12:47:33
---
### 安装
```php
composer require qiniu/php-sdk
```

### 使用

#### PHP 返回上传 token  
```php
<?php

namespace App\Http\Controllers\Admin;

use App\Setting;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use Qiniu\Auth;

class DefaultController extends Controller
{
    public function upload(){
        $accessKey = Setting::getValueByKey('qn_accessKey');
        $secretKey = Setting::getValueByKey('qn_secretKey');
        $bucket = Setting::getValueByKey('qn_bucket_static');
        $baseUrl = Setting::getValueByKey('qn_static_url');
        
        //构建鉴权对象
        $auth = new Auth($accessKey,$secretKey);
        
        //生成上传 token
        $token = $auth->uploadToken($bucket);

        return $token;
    }
}
```

####  layui 携带 token 请求七牛云上传接口

##### layui HTML 代码
```html
<div class="layui-form-item">
            <label class="layui-form-label" for="fileInput">缩略图</label>
            <div class="layui-input-block">
                <button type="button" class="layui-btn" id="thum">上传图片</button>
                <div class="layui-upload-list">
                    <div  id="thum_img"></div>
                </div>
            </div>
        </div>


        <div class="layui-form-item">
            <label class="layui-form-label" for="fileInput">详细图</label>
            <div class="layui-input-block">
                <button type="button" class="layui-btn" id="imgs">
                    上传
                </button>
                <div class="layui-upload-list">
                    <div id="imgs_list"></div>
                </div>
            </div>
        </div>
```

##### layui JS 代码
```javascript
     function uploadInsts(elem,multiple,number,imgDiv){
            if(multiple == null){
                multiple = false;
            }
            if(number == null){
                number = 0;
            }
            layui.use(['upload'],function () {
                var $ = layui.jquery
                    ,upload = layui.upload;
                upload.render({

                    elem: elem                      //上传按钮ID
                    ,url: 'https://up.qbox.me/'     //七牛云上传接口
                    ,data:{'token':'{{$token ?? null}}'}    //携带 token
                    ,multiple:multiple              //是否多个图片
                    ,number:number                  //最多上传图片数量
                    // ,before: function(obj){
                    //     //预读本地文件示例，不支持ie8
                    //     obj.preview(function(index, file, result){
                    //         $('#demo1').attr('src', result); //图片链接（base64）
                    //     });
                    // }
                    ,done: function(res){
                        //res 上传成功后七牛云返回数据 
                        $.ajax({
                            url:"{{url('admin/upload_save')}}"      //请求 php 接口
                            ,data:{'key':res.key,'multiple':multiple,'num':number} //res.key图片名
                            ,type:'post'
                            ,success:function(result){
                                if(result.error > 0){           //失败
                                    layer.msg(result.msg);
                                }else{                          //成功 拿到拼接后的图片url展示
                                    $(imgDiv).append("<img class='layui-upload-img' style='width:150px;height:150px;margin-right:3px;' src='"+result.msg+"'>");
                                }
                            }
                        });
                    }
                    ,error: function(){     //上传失败
                        layer.msg('请刷新重试');
                        return false;
                    }
                });
            });
    }
    
                uploadInsts('#thum',false,0,'#thum_img');//单图片
                uploadInsts('#imgs',true,5,'#imgs_list');//多图片
```

##### PHP 接收七牛云 key 拼接图片地址
```php
<?php

public function uploadSave(Request $request){
        $key = $request->get('key',null);
        $multiple = $request->get('multiple',false);
        $num = $request->get('number',1);
        if(empty($key)){
            return $this->error('上传图片错误请重试');
        }
        $type = 'image';
        $user_id = User::getAdminId();
        $uploads = new Uploads();
        $uploads->type = $type;
        $uploads->user_id = $user_id;
        $uploads->key = $key;
        $uploads->time = time();
        try{
            $uploads->save();
            $result = Setting::getValueByKey('qn_static_url').'/'.$key;
            return $this->success($result);
        }catch (\Exception $exception){
            return $this->error($exception->getMessage());
        }
    }
    
?>
```

##### layui + PHP + 七牛云 实现单个多个图片上传单选多选图片设计
1. 点击上传图片按钮公共弹出层函数
```javascript
function layer_show(title,url,w,h) {
            var width = w || null;
            var height = h || null;
            var areaValue;
            if (width != null) {
                areaValue = width + 'px';
                if (height != null) {
                    areaValue = [width + 'px', height + 'px'];
                }
            }else{
                areaValue = ['100%','100%'];
            }
                layui.use('layer', function () { //独立版的layer无需执行这一句
                    var $ = layui.jquery, layer = layui.layer; //独立版的layer无需执行这一句
                    layer.open({
                        type: 2 //此处以iframe举例
                        , title: title
                        , area: areaValue
                        , shade: 0
                        , maxmin: true
                        , content: url
                        , offset: '10px'

                    });
                });
        }
```

2. PHP 接收 url 中的参数，并显示弹出层视图，要有回调函数
```php
<?php
public function uploadShow(Request $request){
        $limit = $request->get('limit',9);
        $type = $request->get('type','radio');          //图片选择是单选还是多选
        $callback = $request->get('callback',null);     //回调函数名

        $user_id = User::getAdminId();
        $results = Uploads::where('user_id',$user_id)->orderBy('id','desc')->paginate($limit);
        $token = Uploads::getQiniuUpToken();            //七牛云上传 token
        return view('admin.upload.show')->with(['results'=>$results->items(),'type'=>$type,'callback'=>$callback,'token'=>$token]);
    }
    ?>
```

3. 上传图片弹出层视图
```blade
<form class="layui-form" action="">
        <div class="layui-form-item">
            <div class="layui-input-block">
        <div class="layui-upload">
            <button type="button" class="layui-btn" id="upload_img"><i class="layui-icon"></i>上传图片</button>
        </div>
            </div>
        </div>

        <div class="layui-form-item" pane="">
            <label class="layui-form-label">选择图片</label>
            <div class="layui-input-block">
            <!--type 为单选框还是多选框-->
                @foreach($results as $result)
                <input class="layui-inline" type="{{$type}}" name="like[]" lay-skin="primary" title="<img style='width:150px;height:150px;' src='{{$result->img_url}}' class='imgs'>" value="{{$result->img_url}}">
                @endforeach
            </div>
        </div>

        <div class="layui-form-item">
            <div class="layui-input-block">
                <button class="layui-btn" lay-submit="" lay-filter="demo1">确定</button>
                <button type="reset" class="layui-btn layui-btn-primary">重置</button>
            </div>
        </div>
    </form>
```

4. 弹出层-上传图片按钮
```javascript
//elem          jQuery ID 选择器  
//multiple      单图 or 多图上传 
//number        多图上传最大可选图片数量
 function uploadInsts(elem,multiple,number){
            if(multiple == null){
                multiple = false;
            }
            if(number == null){
                number = 0;
            }
            layui.use(['upload'],function () {
                var $ = layui.jquery
                    ,upload = layui.upload;
                upload.render({

                    elem: elem
                    ,url: 'https://up.qbox.me/'         //七牛云上传接口
                    ,data:{'token':'{{$token ?? null}}'}    //携带上传 token
                    ,multiple:multiple
                    ,number:number
                    // ,before: function(obj){
                    //     //预读本地文件示例，不支持ie8
                    //     obj.preview(function(index, file, result){
                    //         $('#demo1').attr('src', result); //图片链接（base64）
                    //     });
                    // }
                    ,done: function(res){
                        //上传成功后七牛云返回 key 提交到后端接口保存图片地址及其他信息
                        $.ajax({
                            url:"{{url('admin/upload_save')}}"      
                            ,data:{'key':res.key,'multiple':multiple,'num':number}
                            ,type:'post'
                            ,success:function(result){
                                if(result.error > 0){
                                    layer.msg(result.msg);
                                }else{                      //后端保存成功后重载此页面
                                    location.reload();      
                                }
                            }
                        });
                        //上传成功
                    }
                    ,error: function(){
                        layer.msg('请刷新重试');
                        return false;
                    }
                });
            });
        }
```

5. 弹出层-确定按钮
```javascript
//点击确定后给执行父级页面传入的回调函数名,并关闭此页面
        function backParent() {
            layui.use('form', function () {
                var $ = layui.jquery
                    , form = layui.form;

                //注意：parent 是 JS 自带的全局对象，可用于操作父页面
                var index = parent.layer.getFrameIndex(window.name); //获取窗口索引

                //监听提交
                form.on('submit(demo1)', function (data) {
                    //给父页面传值
                    var s = 0;
                    var res = new Array();
                    for (var i in data.field) {
                        res.push(data.field['like[' + s + ']']);
                        s++;
                    }
                    //把构建好的数据传入回调函数
                    parent.{{$callback}}(res);      //调用父级页面传入的回调函数名，这里的回调函数名是从 php 返回的.
                    parent.layer.close(index);      //关闭此页面

                    return false;
                });
            });
        }
```

6. 父级页面-定义回调函数
```javascript
        function imgsList(res){
            layui.use('element',function(){
                var $ = layui.jquery;
                for(var i=0;i<res.length;i++){
                    if(res[i] != undefined){
                        var html = '';
                        html += '<img src="'+res[i]+'" style="width:150px;height:150px;">';
                        html += '<button class="layui-btn img_delete" type="button" >删除</button>';

                        $('#imgs_list').append(html);
                    }

                }
            });
        }
```

7. 事件实例
```blade
<button type="button" class="layui-btn" onclick="layer_show('上传图片','{{url('admin/upload_select')}}?type=checkbox&callback=imgsList',800,600)" >
```


