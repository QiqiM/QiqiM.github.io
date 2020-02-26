---
title: php array
date: 2018-12-15 17:32:01
tags: [php, array]
---

### 1.php中如何循环二维数组
在php中，采用foreach循环来对二维索引数组进行遍历,下面的例子演示如何将三个关联数组，通过公有的key,在例子中是gid;组合成一个关联数组。
<!--more-->
#### a.初始化数据
```php
$paylist = array(
    0=>array(
        'gid'=> 1,
        'money'=> '100'
    ),  
    1=>array(
        'gid'=> 1,
        'money'=> '200'
    ),  
    2=>array(
        'gid'=> 4,
        'money'=> '300'
    ),  
    3=>array(
        'gid'=> 6,
        'money'=> '400'
    ),  
);

$rolelist = array(
    0=>array(
        'gid'=> 1,
        'name'=> 'xx1'
    ),  
    1=>array(
        'gid'=> 2,
        'name'=> 'xx2'
    ),  
    2=>array(
        'gid'=> 4,
        'name'=> 'xx4'
    ),  
    3=>array(
        'gid'=> 6,
        'name'=> 'xx6'
    ),  
);

$serverlist = array(
    0=>array(
        'gid'=> 1,
        'servername'=> 's1'
    ),  
    1=>array(
        'gid'=> 2,
        'servername'=> 's2'
    ),  
    2=>array(
        'gid'=> 4,
        'servername'=> 's4'
    ),  
    3=>array(
        'gid'=> 6,
        'servername'=> 's4'
    ),  
);

$gsidarr = array(1,1,4,6);
//去重排序
$gsidarr = array_unique($gsidarr);
$gsidarr = array_values($gsidarr);


```
#### b.构造数据(将公有key提取出来作为key值索引)
```php
$roleresult = array();
foreach($rolelist as $key => $value){
    $roleresult[$value['gid']] = $value;
}
echo '</br>----'.json_encode($roleresult).'</br>';
foreach($roleresult as $k=>$v){
    echo $v['name'].'==>'.$v['gid'].'--</br>';  //打印一下构造之后的数据
}

// 构造之后的数据roleresult
//{"1":{"gid":1,"name":"xx1"},"2":{"gid":2,"name":"xx2"},"4":{"gid":4,"name":"xx4"},"6":{"gid":6,"name":"xx6"}}

$serresult = array();
foreach($serverlist as $key => $value){
    $serresult[$value['gid']] = $value;
}
echo '</br>----'.json_encode($serresult).'</br>';
foreach($serresult as $k=>$v){
        echo $v['servername'].'==>'.$v['gid'].'--</br>';
}
// 构造之后的数据serverresult
// {"1":{"gid":1,"servername":"s1"},"2":{"gid":2,"servername":"s2"},"4":{"gid":4,"servername":"s4"},"6":{"gid":6,"servername":"s4"}}

```

#### c.将构造好的数据组合起来(通过公有key来取roleresult和serverresult的value)
```php
$ret = array();
foreach($paylist as $v){
    $data = array(
            'money'=>$v['money'],
            'gid'=>$v['gid'],
            'name'=>$roleresult[$v['gid']]['name'],
            'servername'=>$serresult[$v['gid']]['servername']
        );
        array_push($ret,$data);
}

//组装好之后的数据
/*
    [{"money":"100","gid":1,"name":"xx1","servername":"s1"},{"money":"200", 
    "gid":1,"name":"xx1","servername":"s1"},{"money":"300","gid":4,
    "name":"xx4","servername":"s4"},{"money":"400","gid":6,"name":"xx6",
    "servername":"s4"}]
*/

echo '</br>----'.json_encode($ret).'</br>';

```

### 2.最后附一个在线测试代码的网站

[在线测试网站](https://www.dooccn.com/php/)