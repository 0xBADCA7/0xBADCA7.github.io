---
layout: post
title: Dynamic form parameters against SQLMap kiddies et al
---

### Intro

During one of my pentesting nights I have come across a funky server that was generating login forms in a weird way:

    <form name="LoginFormdXa5Bap003G38Fk7qFc84G_" method="POST">
    	<input type="text" name="login8dGs84Hlks8dfHs8dsdlF" value="">
    	<input type="password" name="passG9dJ348ddHlspaZgfda2" value="">
    	<input type="submit" name="submit" value="Login">
    </form>

> "Hmm.. interesting" - I thought to myself

I refreshed the page and witnessed the strings after `LoginForm`, `login` and `pass` change.

> "Pretty good!" - again thought I to my humble self and submitted a forged form with random values to the server

The server spitted out a nice descriptive error and I decided that it's time to crack open `SQLMap` and see if there's anything interesting there and if the field names are really _that_ dynamic. It turned out that `sqlmap` politely fetched every new response from the server with the "correct" field names and found an injection somewhere in the mid of its run.

> "What a lame thing!" - I told myself finally.

Apparently, there was some sort of "protection" from tampering with the login form data but it was completely useless against those who fire up every single scanner/tool on Earth against your application and copy-paste their SQLi-way out to your database(s).

### Proposal & examples

It was clear to me should there be a client-side feature mangling form field names, SQLMap won't pick it up and proceed with what really came back from the server with the initial request.
I coded up a quick example of such HTML page with a JavaScript piece that appends left and right halves of SHA1 hash to username and password fields respectively. SHA1 hash is based on the current UNIX timestamp. What's the deal with SHA1? See the PHP script after this one.

	<doctype html>
	<head>
		<title>SQLMap dynamic parameters</title>
	</head>
	<body>
		<form id="sqliForm" method="POST">
			<input id="username" type="text" name="username">
			<input id="password" type="password" name="password">

			<input type="submit" value="Submit">
		</form>
		<script src="http://crypto-js.googlecode.com/svn/tags/3.1.2/build/rollups/sha1.js"></script>
		<script>
			document.addEventListener
			?
			document.addEventListener('DOMContentLoaded', init, false )
			:
			document.attachEvent('onreadystatechange', init)

			function init()
			{
				var form = document.getElementById('sqliForm');

				form.addEventListener
				?
				form.addEventListener('submit', submitForm, false)
				:
				form.attachEvent('submit', submitForm);

			}

			function submitForm()
			{
				var u = document.getElementById('username');
				var p = document.getElementById('password');
				var ts = (new Date()).getTime().toString().substr(0, 10);
				var hash = CryptoJS.SHA1(ts).toString();

				u.name = 'username' + hash.substr(0, 20);
				p.name = 'password' + hash.substr(20);
			}
		</script>
	</body>
	</html>

For the completeness I have put this code into a [single PHP file][1] which acts as a serving script at the same time for convenience:

	<?php
	// Based on UNIX timestamp in seconds hash
	$timehash = sha1(time());

	/*
	@DEBUG
	print($timehash);
	print "<br>";
	print substr($timehash, 0, strlen($timehash) / 2);
	print "<br>";
	print substr($timehash, -strlen($timehash) / 2);
	@@
	*/

	// Generate one-time tokens
	$username_nonce = ''; // substr($timehash, 0, strlen($timehash) / 2);
	$password_nonce = ''; // substr($timehash, -strlen($timehash) / 2);

	$db = mysqli_connect('localhost', 'root', 'root', 'demo_db', '8889');

	if (!$db)
	{
		printf("Connect failed: %s\n", mysqli_connect_error());
		exit();
	}

	// Looking for anything that may look like
	// our username and password field names
	$keys = array_keys($_POST);
	$usernameMatch = preg_grep('/^username(.*)$/', $keys);
	$passwordMatch = preg_grep('/^password(.*)$/', $keys);

	if (sizeof($usernameMatch) === 1 && sizeof($passwordMatch) === 1)
	{
		$usernameFieldName = array_values($usernameMatch);
		$passwordFieldName = array_values($passwordMatch);

		$username = $_POST[$usernameFieldName[0]];
		$password = $_POST[$passwordFieldName[0]];

		// Get the both parts of the hash back
		$lHash = substr($usernameFieldName[0], strlen('username'));
		$rHash = substr($passwordFieldName[0], strlen('password'));

		//@DEBUG
		//printf("Time hash:<br>%s<br>Reconstructed hash from client:<br>%s:%s",
		//		$timehash, $lHash, $rHash);
		//@@

		$found = false;

		// If the request has been received within 3
		// seconds of token generation...
		for ($i=0;$i<=3;$i++)
		{
			if ( $lHash . $rHash === sha1(time()-$i) )
			{
				printf("<br>%s", "You nailed it!");

				auth($db, $username, $password);
				$found = true;
				break;
			}
		}

		// Too late submitting the form
		if (!$found)
			printf("<br>%s", "You are too slow...");
	}

	function auth($mysqli_link, $username, $password)
	{
		//@DEBUG
		// printf("Username: %s, password: %s", $username, $password);
		// printf("<br>%s", "SELECT * FROM settings WHERE user='${username}' AND password='${password}'");
		//@@

		if (isset($username) && isset($password))
		{
			// SQLi here ofc.
			$result = mysqli_query($mysqli_link, "SELECT id, user, password FROM settings WHERE user='${username}' AND password='${password}'")
				or die("<br/><br/>" ."Couldn't execute the query. " . mysqli_error($mysqli_link));

			$row = mysqli_fetch_assoc($result);

			if ($row)
			{
				// Dumps out on successful retrieval from DB
				print("<br>Result dump:<br>");
				var_dump($row);
			}
		}
	}
	?>
	<doctype html>
	<head>
		<title>SQLMap dynamic parameters</title>
	</head>
	<body>
		<form id="sqliForm" action="<?=$_SERVER['SCRIPT_NAME']?>" method="POST">
			<input id="username" type="text" name="username<?=$username_nonce?>">
			<input id="password" type="password" name="password<?=$password_nonce?>">

			<input type="submit" value="Submit">
		</form>
		<script src="http://crypto-js.googlecode.com/svn/tags/3.1.2/build/rollups/sha1.js"></script>
		<script>
			document.addEventListener
			?
			document.addEventListener('DOMContentLoaded', init, false )
			:
			document.attachEvent('onreadystatechange', init)

			function init()
			{
				var form = document.getElementById('sqliForm');

				form.addEventListener
				?
				form.addEventListener('submit', submitForm, false)
				:
				form.attachEvent('submit', submitForm);

			}

			function submitForm()
			{
				var u = document.getElementById('username');
				var p = document.getElementById('password');
				var ts = (new Date()).getTime().toString().substr(0, 10);
				var hash = CryptoJS.SHA1(ts).toString();

				u.name = 'username' + hash.substr(0, 20);
				p.name = 'password' + hash.substr(20);
			}
		</script>
	</body>
	</html>

As you can see now, the server side of the script looks for anything that resembles "username" and "password" in the `$_POST` array, builds up a SHA1 hash from the two halves and then checks if the hash is that of a UNIX timestamp produced withing the last 3 seconds.

One can expect `sqlmap` to pick up only "username" and "password" field names at best. And so it does, which brings in malformed hash (in fact, an empty string) to the server-side script and thus filters out script-kiddies (_sqlmap-kiddies_) out. Could it ever be simpler?

## As it came initially from the server:
![imgur](http://i.imgur.com/if7TAz3.png "As it came initially from the server")


## What was actually sent out with form:
![imgur](http://i.imgur.com/D6aOh5v.png "What was actually sent out")


## SQLMap doesn't deal with client-side code so it failed:
![imgur](http://i.imgur.com/KEGP6uu.png "SQLMap failed")



### Coda
We've looked at a very simple but effective way of deterring "not-so-determined" attackers from testing your web application against SQL-injection. _It doesn't fix the issues in the code_ that lead to SQL-injection in your app though. However, if there _is_ one it's probably better that a "certified" pentester discovers it rather than a script-kiddie (it's not a lot of fun seeing "pwn3d" in your admin console on a Monday morning). On top of that, it feels a little funny (\*snigger\*) when you see a bunch of failed requests to your application with wrong parameters passed.

So far - so good, and I'm on my way of preparing a new [post][2] on how to work around such "protection" using the same good 'ol `sqlmap`.

> Read the manual if unsure, post comment(s) if unclear.

[1]: https://git.io/fAdpl "gist @ GitHub"
[2]: https://breaking.into.systems/read/2013/beat-dynamic-parameters-with-sqlmap "Beat dynamic parameters with SQLMap"
