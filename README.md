# ank-mail
因为调用的时候有些参数是必填项,如果不填写就会报错错,所以下面把邮件类封闭成一个函数来调用
```
function send_mail($conf = array())
{
    //ini_set("memory_limit","100M");
    $host     = isset($conf['host']) ? $conf['host'] : '';
    $port     = isset($conf['port']) ? $conf['port'] : '';
    $username = isset($conf['username']) ? $conf['username'] : '';
    $password = isset($conf['password']) ? $conf['password'] : '';

    $toname  = isset($conf['toname']) ? $conf['toname'] : '';
    $toemail = isset($conf['toemail']) ? $conf['toemail'] : '';

    $fromname  = isset($conf['fromname']) ? $conf['fromname'] : ''; //来源名称
    $fromemail = isset($conf['fromemail']) ? $conf['fromemail'] : ''; //来源邮箱

    $subject    = isset($conf['subject']) ? $conf['subject'] : '';
    $body       = isset($conf['body']) ? $conf['body'] : '';
    $extra_msg  = isset($conf['extra_msg']) ? $conf['extra_msg'] : '';
    $attachment = isset($conf['attachment']) ? $conf['attachment'] : '';
    $port       = $port ?: 25;
    $fromname   = $fromname ?: $username;

    if (empty($host)) {
        return '[host]主机地址不能为空';
    }
    if (empty($username)) {
        return '[username]smtp服务器用户名不能为空';
    }
    if (empty($password)) {
        return '[password]smtp服务器密码不能为空';
    }
    if (empty($fromname)) {
        return '[fromname]寄件人名字不能为空';
    }
    if (empty($fromemail)) {
        return '[fromemail]寄件人邮箱不能为空';
    }

    if (empty($toname)) {
        return '[toname]收件人名字不能为空';
    }
    if (empty($toemail)) {
        return '[toemail]收件人邮箱不能为空';
    }
    if (empty($subject)) {
        return '[subject]邮件主题不能为空';
    }
    if (empty($body)) {
        return '[body]邮件内容不能为空';
    }

    // import('Ainiku.PHPMailer');
    $mail = new \ank\PHPMailer();

    $mail->SMTPDebug = 0; // 关闭SMTP调试功能
    $mail->IsSMTP(); // send via SMTP
    $mail->Host     = $host; // SMTP servers
    $mail->SMTPAuth = true; // turn on SMTP
    $mail->Username = $username; // SMTP username
    $mail->Password = $password; // SMTP password
    $mail->From     = $fromemail; // 发件人邮箱
    $mail->FromName = $fromname; // 发件人
    $mail->CharSet  = "utf-8"; // 这里指定字符集！
    $mail->Encoding = "base64";
    $mail->AddAddress($toemail, $toname); // 收件人邮箱和姓名
    $mail->IsHTML(true); // send as HTML
    //$mail->SMTPSecure = 'ssl';   // 使用安全协议用qq邮箱时要用
    $mail->Subject  = $subject; // 邮件主题
    $mail->Body     = $body;
    $mail->WordWrap = 50; // set word wrap 换行字数
    $mail->AltBody  = "text/html";
    $mail->AddReplyTo($fromemail, $fromname); //快捷回复
    if ($attachment) {
        foreach ($attachment as $key => $value) {
            if (!file_exists($value['path'])) {
                return '附件文件路径不存在';
            }
            if (filesize($value['path']) > 1024 * 1024 * 10) {
                return '附件不能大于10M';
            }
            $mail->AddAttachment($value['path'], $value['name']);
        }
    }
    //$mail->AddAttachment("sendmail.rar");     // attachment 附件
    //$mail->AddAttachment("psd.jpg", "psd.jpg");  //添加一个附件并且重命名

    if ($mail->Send()) {
        return true;
    } else {
        return config('app_debug') ? $mail->ErrorInfo : '邮件发送错误!';
    }

}
```
##调用方法
```
$result = send_mail([
    'host'       => 'smtp.163.com',
    'port'       => 25,
    'username'   => 'deariloveyoutoever',
    'password'   => '01227328',
    'fromemail'  => 'deariloveyoutoever@163.com',
    'fromname'   => '我是网易邮箱',

    'toemail'    => '735579768@qq.com',
    'toname'     => '你好克立兄',
    'subject'    => '内容的一个摘要',
    'body'       => '这个是邮件内容',
    //附件为一个数组格式如下
    'attachment' => [
        [
            'path' => 'e:/OpenHostsEdit.bat',
            'name' => 'host.txt',
        ],
        [
            'path' => 'E:/GitServer/curl-7.60.0.zip',
            'name' => 'curl.zip',
        ],
    ],
]);
if ($result !== true) {
    die($result);
}
die('send mail success');
```