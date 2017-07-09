---
layout: post
title:  phpredis扩展调用 redis的scan方法报错
date:   2017-06-29 21:30:00 +0800
categories: redis
tag: 随笔
---

* content 
{:toc}



官方文档的例子如下：


{% highlight php %}
<?php 
/* Without enabling Redis::SCAN_RETRY (default condition) */
$it = NULL;
do {
    // Scan for some keys
    $arr_keys = $redis->scan($it);

    // Redis may return empty results, so protect against that
    if ($arr_keys !== FALSE) {
        foreach($arr_keys as $str_key) {
            echo "Here is a key: $str_key\n";
        }
    }
} while ($it > 0);
echo "No more keys to scan!\n";

/* With Redis::SCAN_RETRY enabled */
$redis->setOption(Redis::OPT_SCAN, Redis::SCAN_RETRY);
$it = NULL;

/* phpredis will retry the SCAN command if empty results are returned from the
   server, so no empty results check is required. */
while ($arr_keys = $redis->scan($it)) {
    foreach ($arr_keys as $str_key) {
        echo "Here is a key: $str_key\n";
    }
}
echo "No more keys to scan!\n";
?>

{% endhighlight php %}

运行后会发现报了下面的错误：

{% highlight php %}
Warning: Parameter 1 to Redis::scan() expected to be a reference, value given
{% endhighlight php %}

scan的第一个参数必须是个引用传值

解决方案如下（代码只列出了主要部分，不是完整代码）：

{% highlight php %}
<?php

        $iterator = null;
        $arguments = array(&$iterator,'match_str*',100);
        $redis->setOption(Redis::OPT_SCAN, Redis::SCAN_RETRY);

        do{
            #bug 问题
            #$res = $redis->scan($iterator,'match_str*',100);
            $res = call_user_func_array(array($redis, 'scan'), $arguments);

            foreach($res as $value) {
                $this->del($value);
                echo $value." 已删除\n";
            }
        } while($iterator >0);
?>

{% endhighlight php %}