---
layout: post
tags: CTF, ctf, Codegate
title: 'Codegate 2014 CTF, web "120" write-up'
---

### Task
You are given a URI (all happens in the `http://58.229.183.24/5a520b6b783866fd93f9dcdaf753af08/` route) that leads to `index.php`, the same but ends with `index.phps` and is an alleged source code of the former, finally, `index.php` contains a link to `auth.php`

`index.phps` listing below

~~~php
    <?php
    session_start();

    $link = @mysql_connect('localhost', '', '');
    @mysql_select_db('', $link);

    function RandomString()
    {
      $filename = "smash.txt";
      $f = fopen($filename, "r");
      $len = filesize($filename);
      $contents = fread($f, $len);
      $randstring = '';
      while( strlen($randstring)<30 ){
        $t = $contents[rand(0, $len-1)];
        if(ctype_lower($t)){
        $randstring .= $t;
        }
      }
      return $randstring;
    }

    $max_times = 120;

    if ($_SESSION['cnt'] > $max_times){
      unset($_SESSION['cnt']);
    }

    if ( !isset($_SESSION['cnt'])){
      $_SESSION['cnt']=0;
      $_SESSION['password']=RandomString();

      $query = "delete from rms_120_pw where ip='$_SERVER[REMOTE_ADDR]'";
      @mysql_query($query);

      $query = "insert into rms_120_pw values('$_SERVER[REMOTE_ADDR]', '$_SESSION[password]')";
      @mysql_query($query);
    }
    $left_count = $max_times-$_SESSION['cnt'];
    $_SESSION['cnt']++;

    if ( $_POST['password'] ){

      if (eregi("replace|load|information|union|select|from|where|limit|offset|order|by|ip|\.|#|-|/|\*",$_POST['password'])){
        @mysql_close($link);
        exit("Wrong access");
      }

      $query = "select * from rms_120_pw where (ip='$_SERVER[REMOTE_ADDR]') and (password='$_POST[password]')";
      $q = @mysql_query($query);
      $res = @mysql_fetch_array($q);
      if($res['ip']==$_SERVER['REMOTE_ADDR']){
        @mysql_close($link);
        exit("True");
      }
      else{
        @mysql_close($link);
        exit("False");
      }
    }

    @mysql_close($link);
    ?>

    <head>
    <link rel="stylesheet" type="text/css" href="black.css">
    </head>

    <form method=post action=index.php>
      <h1> <?= $left_count ?> times left </h1>
      <div class="inset">
      <p>
        <label for="password">PASSWORD</label>
        <input type="password" name="password" id="password" >
      </p>
      </div>
      <p class="p-container">
        <span onclick=location.href="auth.php"> Auth </span>
        <input type="submit" value="Check">
      </p>
    </form>
~~~

That's all that's given. Going thru the `index.phps`:

+ If you just hit the page, the script will create a PHP session with a counter `$_SESSION['cnt']`. Every time you hit the page using *the same session* (meaning the same PHPSESSID - that's how PHP "knows" your client), you get the counter incremented by one till it goes up to `120`.

+ On the first visit to the page, your client IP is written to the DB via `$_SERVER[REMOTE_ADDR]` while any other record with the same `ip` database field gets removed (by `delete from rms_120_pw where ip='$_SERVER[REMOTE_ADDR]'`). Along with the IP, a new "random" password is written to the same table.

+ The password is generated by `RandomString()`, which reads publicly accessible `smash.txt` (article of the decade!) and fetches lowercase ASCII string 30 characters in length. That's the password that gets written to the DB.

+ If counter is overflown (120 requests), the session gets renewed: a new password is being generated and written to the DB together with the *same* client IP.

+ If there's a `POST` request to the script with a `password` form parameter, then the page runs `eregi()` against `$_REQUEST['password']`, with an attempt to prevent SQL-injection, however does it poorly as you will see in the *Solution* section.
<br><br>

### Solution
+ If you do a `POST` request to `index.php` with the following payload:

~~~
password='+or+password+like+'%
~~~

then the resulting SQL query is `"select * from rms_120_pw where (ip='$_SERVER[REMOTE_ADDR]') and (password='' or password like '%')"` and will bring "True" as the result of the script's processing. This bypasses `eregi()` but leaves you with the question "What's next?".

+ Recall, there's another script `auth.php` which doesn't seem to be prone to the same SQL attacks. Chances are, you're supposed to retrieve the password from the `rms_120_pw` table and submit it to `auth.php` for it will query the same table.

+ Claim: we can bruteforce the password, based upon responses from `index.php` "True" or "False" when doing SQL-injection. The payload below will tell if the first characted of the password string is "a":

~~~
password='+or+password+like+'a%
~~~


+ If the script returns "True" then the guess is correct and you can proceed to the next character (and so up to 30, which is the password length), otherwise "False" which means you have to try the next character in this position.

+ Recall, each request eats up one attempt. If we are allowed a handful of 120 attempts, the alphabet is lowercase ASCII (according to `RandomString()`) then there `120/26 = 4` characters you can find in the worst case, having consumed all the attempts. This is a no-go.


There are various search algorithms that differentiate in their running time. However, this is not really needed here. Although, the number of attempts is limited to `120`, you have to look at what is limiting it: `$_SESSION['cnt']`. If you carefully look at the source, you might see that what's written to the database is the remote IP, not the PHP session id, per se. That is, every time you request a page without a `PHPSESSID` cookie being sent to the server, but via the same IP, that IP still goes to the DB. In other words, you may request `index.php` even a thousand times (in theory), every time without a cookie, but storing the `PHPSESSID` that the server gave you. While doing this, the secret password is being regenerated every single time, until the very last request. With the last request, you stop raping the server, store all the cookies you fetched (esp. the last one) and of course, the same `$_SERVER['REMOTE_ADDR']` is stored in the DB.

What this gives us, is the ability to consume attempts while bruteforcing the password, without being worried of the password getting overwritten. **As long as you don't deplete 120 attempts, the password remains the same stored in the table**. With that in mind the algorithm becomes simple:

+ Procure cookies (for the winter) enough to make `26*30` requests at worst. Thanks to `LIKE` statement it's not `26^30`. That's just `7` cookies. Again, there are ways to minimize the effort to many less attempts, but I want to consider the worst case to know the margin.

+ Start bruteforcing `index.php` using payloads like `password='+or+password+like+'abcdefg%`

+ If the number of attempts is close to `120`, supply another cookie and go from there.

That simple.


### POC
[https://git.io/fAdpn](https://git.io/fAdpn)

### Notes
*   If you are to use Python with `python-requests` module, then be informed that [python-requests](https://github.com/kennethreitz/requests) *always* URL-encodes the payload (`data` parameter) which bit me again. If you know how to "officially" tell it not to - let us know in the comments.

*   Usual one: I noticed the CTF server to go up in ping time by 2 seconds (!) after this challenge opened

*   Please share your password search algorithm if any


> Read the manual if unsure, post comment(s) if unclear.