PHP practice
--------------

# 安全实践
1. 输入值转义addslashes & stripslashes
	htmlspecialchars()
2. [apache bench](http://www.oschina.net/question/12_5473?fromerr=gk6QJISZ)
3. [php header详解](http://blog.csdn.net/rainysia/article/details/8131174)
4. **过滤数据:filter_input, filter_var**
5. at定时任务 & crontab例行任务
6. SplFileInfo 文件信息类
7. 迭代器
[迭代器介绍](http://www.cnblogs.com/ellisonDon/archive/2013/02/26/2932978.html)
｀｀｀
ArrayIterator,LimitIterator，RecursiveArrayIterator //数组迭代器,
$arr = range(1,10);
$arriter = new ArrayIterator($arr);
$iter = new LimitIterator($arriter,3,5);//迭代器，偏移量，限制数量
foreach($iter as $ar){
 echo $ar."\n";
}

RecursiveIteratorIterator，RecursiveTreeIterator //将迭代结构变成一维结构
ParentIterator//过滤迭代器的非父元素，只找出拥有子元素的键值


AppendIterator//合并两个数组迭代
//$iter1,$iter2为arrayiterator
$iter = new AppendIterator();
$iter ->append($iter1);
$iter ->append($iter2);//对iter的操作等同于对arrayiterator

FilterIterator，RegexIterator//过滤迭代器,匹配迭代器

DirectoryIterator,RecursiveDirectoryIterator//目录迭代访问
$path="/wamp/";//访问目录
$Directoryiter = new DirectoryIterator($path);//当前目录迭代
$RecursiveDirectoryiter = new RecursiveDirectoryIterator($path);//递归迭代
$treeiter = new RecursiveTreeIterator($RecursiveDirectoryiter);//可视化递归迭代
foreach( $Directoryiter as $fileinfo){//列出当前目录
 echo $fileinfo."\n";
}
foreach( $treeiter as $fileinfo){//递归列出当前目录下所有文件
 echo $fileinfo."\n";
}
｀｀｀	
	
8. Xss
```
<?php
function remove_xss($val) {
   // remove all non-printable characters. CR(0a) and LF(0b) and TAB(9) are allowed
   // this prevents some character re-spacing such as <java\0script>
   // note that you have to handle splits with \n, \r, and \t later since they *are* allowed in some inputs
   $val = preg_replace('/([\x00-\x08,\x0b-\x0c,\x0e-\x19])/', '', $val);
   // straight replacements, the user should never need these since they're normal characters
   // this prevents like <IMG SRC=@avascript:alert('XSS')>
   $search = 'abcdefghijklmnopqrstuvwxyz';
   $search .= 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
   $search .= '1234567890!@#$%^&*()';
   $search .= '~`";:?+/={}[]-_|\'\\';
   for ($i = 0; $i < strlen($search); $i++) {
      // ;? matches the ;, which is optional
      // 0{0,7} matches any padded zeros, which are optional and go up to 8 chars
      // @ @ search for the hex values
      $val = preg_replace('/(&#[xX]0{0,8}'.dechex(ord($search[$i])).';?)/i', $search[$i], $val); // with a ;
      // @ @ 0{0,7} matches '0' zero to seven times
      $val = preg_replace('/(�{0,8}'.ord($search[$i]).';?)/', $search[$i], $val); // with a ;
   }
   // now the only remaining whitespace attacks are \t, \n, and \r
   $ra1 = array('javascript', 'vbscript', 'expression', 'applet', 'meta', 'xml', 'blink', 'link', 'style', 'script', 'embed', 'object', 'iframe', 'frame', 'frameset', 'ilayer', 'layer', 'bgsound', 'title', 'base');
   $ra2 = array('onabort', 'onactivate', 'onafterprint', 'onafterupdate', 'onbeforeactivate', 'onbeforecopy', 'onbeforecut', 'onbeforedeactivate', 'onbeforeeditfocus', 'onbeforepaste', 'onbeforeprint', 'onbeforeunload', 'onbeforeupdate', 'onblur', 'onbounce', 'oncellchange', 'onchange', 'onclick', 'oncontextmenu', 'oncontrolselect', 'oncopy', 'oncut', 'ondataavailable', 'ondatasetchanged', 'ondatasetcomplete', 'ondblclick', 'ondeactivate', 'ondrag', 'ondragend', 'ondragenter', 'ondragleave', 'ondragover', 'ondragstart', 'ondrop', 'onerror', 'onerrorupdate', 'onfilterchange', 'onfinish', 'onfocus', 'onfocusin', 'onfocusout', 'onhelp', 'onkeydown', 'onkeypress', 'onkeyup', 'onlayoutcomplete', 'onload', 'onlosecapture', 'onmousedown', 'onmouseenter', 'onmouseleave', 'onmousemove', 'onmouseout', 'onmouseover', 'onmouseup', 'onmousewheel', 'onmove', 'onmoveend', 'onmovestart', 'onpaste', 'onpropertychange', 'onreadystatechange', 'onreset', 'onresize', 'onresizeend', 'onresizestart', 'onrowenter', 'onrowexit', 'onrowsdelete', 'onrowsinserted', 'onscroll', 'onselect', 'onselectionchange', 'onselectstart', 'onstart', 'onstop', 'onsubmit', 'onunload');
   $ra = array_merge($ra1, $ra2);
   $found = true; // keep replacing as long as the previous round replaced something
   while ($found == true) {
      $val_before = $val;
      for ($i = 0; $i < sizeof($ra); $i++) {
         $pattern = '/';
         for ($j = 0; $j < strlen($ra[$i]); $j++) {
            if ($j > 0) {
               $pattern .= '(';
               $pattern .= '(&#[xX]0{0,8}([9ab]);)';
               $pattern .= '|';
               $pattern .= '|(�{0,8}([9|10|13]);)';
               $pattern .= ')*';
            }
            $pattern .= $ra[$i][$j];
         }
         $pattern .= '/i';
         $replacement = substr($ra[$i], 0, 2).'<x>'.substr($ra[$i], 2); // add in <> to nerf the tag
         $val = preg_replace($pattern, $replacement, $val); // filter out the hex tags
         if ($val_before == $val) {
            // no replacements were made, so exit the loop
            $found = false;
         }
      }
   }
   return $val;
}
?>
```

9. 数据验证正则
```
$validate = array(
            'require'   =>  '/\S+/',
            'email'     =>  '/^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/',
            'url'       =>  '/^http(s?):\/\/(?:[A-za-z0-9-]+\.)+[A-za-z]{2,4}(?:[\/\?#][\/=\?%\-&~`@[\]\':+!\.#\w]*)?$/',
            'currency'  =>  '/^\d+(\.\d+)?$/',
            'number'    =>  '/^\d+$/',
            'zip'       =>  '/^\d{6}$/',
            'integer'   =>  '/^[-\+]?\d+$/',
            'double'    =>  '/^[-\+]?\d+(\.\d+)?$/',
            'english'   =>  '/^[A-Za-z]+$/',
        );
```

10. php开启内置服务器 `php -S localhost:4000 -c /usr/php.ini`
11. **上传文件类型判断使用finfo读到类型**
12. 遍历文件夹
```
function ls($dirname)
{
 $dir=opendir($dirname);
 //list file first
 while(false !==($file=readdir($dir)))
 {
  if($file !='.' && $file !='..' && !is_dir($dirname.'/'.$file))
  {
   echo $dirname.": ".$file. " - file</br>";
  }
 }
 closedir($dir);
 $dir=opendir($dirname);
 //list document first
 while(false !==($file=readdir($dir)))
 {
  
  if($file !='.' && $file !='..' && is_dir($dirname.'/'.$file))
  {
   echo $dirname.": ".$file. " - document</br>";
   $subname=$dirname.'/'.$file;
   ls($subname);
  }
 }
 closedir($dir);
}
```

13. 网页缓存设置
｀｀｀
<?php 

header('Expires: Mon, 26 Jul 1997 05:00:00 GMT'); 

header('Cache-Control: no-store, no-cache, must-revalidate'); 

header('Cache-Control: post-check=0, pre-check=0', FALSE); 

header('Pragma: no-cache'); 

?>
｀｀｀

14. mysqli 使用prepare绑定数据插入
｀｀｀
------------------------------------prepare  插入时使用bind_param---------------------
$query="INSERT INTO table values (?,?,?,?)";
$stmt =$db->prepare($query);
$stmt =$stmt->bind_param('sssd',$v1,$v2,$v3,$v4);//第一个参数为后面几个参数值的数据类型
$stmt->execute();
echo $stmt->affected_rows;
$stmt->close();
｀｀｀



