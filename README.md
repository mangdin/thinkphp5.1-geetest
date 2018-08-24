# thinkphp5.1-geetest
thinkphp5.1 geetest验证码

服务端验证代码
<pre>
            $config = Config::get('geetest.');
            $GtSdk = new \mangdin\geetest\GeetestLib($config['geetest_id'], $config['geetest_key']);
            $data = array(
                "client_type" => "web", #web:电脑上的浏览器；h5:手机上的浏览器，包括移动应用内完全内置的web_view；native：通过原生SDK植入APP应用的方式
                "ip_address" => \think\facade\Request::host(), # 请在此处传输用户请求验证时所携带的IP
            );
            if (Session::get('gtserver') == 1) {   //服务器正常
                $code = $GtSdk->success_validate($_POST['geetest_challenge'], $_POST['geetest_validate'], $_POST['geetest_seccode'],$data);
            }else{  //服务器宕机,走failback模式
                $code = $GtSdk->success_validate($_POST['geetest_challenge'], $_POST['geetest_validate'], $_POST['geetest_seccode']);
            }
            if (!$code){
                $this->error('验证码错误');
            }
            unset($data);
            unset($code);
            unset($config);
</pre>

服务端验证码生成代码
<pre>
public function geetest(){
        $config = Config::get('geetest.');
        $GtSdk = new \mangdin\geetest\GeetestLib($config['geetest_id'], $config['geetest_key']);
        $data = array(
            "client_type" => "web", #web:电脑上的浏览器；h5:手机上的浏览器，包括移动应用内完全内置的web_view；native：通过原生SDK植入APP应用的方式
            "ip_address" => \think\facade\Request::host(), # 请在此处传输用户请求验证时所携带的IP
        );
        $status = $GtSdk->pre_process($data, 1);
        \think\facade\Session::set('gtserver',$status);
        echo $GtSdk->get_response_str();
    }
</pre>


前台页面代码
```html
<div id="embed-captcha"></div>
<p id="wait" class="show">正在加载验证码......</p>
<p id="notice" class="hide">请先完成验证</p>
```
<pre>
<script src="http://apps.bdimg.com/libs/jquery/1.9.1/jquery.js"></script>
<script src="https://static.geetest.com/static/tools/gt.js"></script>
<script>
    var handlerEmbed = function (captchaObj) {
        $("#embed-submit").click(function (e) {
            var validate = captchaObj.getValidate();
            if (!validate) {
                $("#notice")[0].className = "show";
                setTimeout(function () {
                    $("#notice")[0].className = "hide";
                }, 2000);
                e.preventDefault();
            }
        });
        // 将验证码加到id为captcha的元素里，同时会有三个input的值：geetest_challenge, geetest_validate, geetest_seccode
        captchaObj.appendTo("#embed-captcha");
        captchaObj.onReady(function () {
            $("#wait")[0].className = "hide";
        });
        captchaObj.onSuccess(function () {
            //验证码成功之后执行的操作
        });
    };
    $.ajax({
        url: "{:url('index/login/geetest')}?t=" + (new Date()).getTime(), // 验证码地址加随机数防止缓存
        type: "get",
        dataType: "json",
        success: function (data) {
            // 使用initGeetest接口
            // 参数1：配置参数
            // 参数2：回调，回调的第一个参数验证码对象，之后可以使用它做appendTo之类的事件
            initGeetest({
                gt: data.gt,
                challenge: data.challenge,
                new_captcha: data.new_captcha,
                product: "embed", // 产品形式，包括：float，embed，popup。注意只对PC版验证码有效
                offline: !data.success // 表示用户后台检测极验服务器是否宕机，一般不需要关注
                // 更多配置参数请参见：http://www.geetest.com/install/sections/idx-client-sdk.html#config
            }, handlerEmbed);
        }
    });
</script>
</pre>
