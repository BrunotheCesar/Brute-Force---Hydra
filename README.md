# Brute-Force - Hydra


## *We're going to be looking for users who use weak or common passwords and attempt to guess their passwords through brute force using Hydra*

Upon accessing our vulnerable target, we are faced with a sign-in pop up. As we do not have any information about the host (besides knowing this is a brute force prone target), we will start with our brute force attempt using Hydra.

![1](https://github.com/LaraBruno/Brute-Force---Hydra/assets/37584600/420825d9-9191-4bbe-b923-3aabca5d25ca)


We want to generate the least amount of noise as possible, therefore, we are starting the process with a default password list, in hopes that our target did not change the server default credentials when setting it up.

In Kali, locating default password lists is effortless as they come loaded with the OS.

![2](https://github.com/LaraBruno/Brute-Force---Hydra/assets/37584600/3751b871-f698-4476-b732-f5bae474c434)


Success!

After signing in, we find an admin panel sign-in page. The password that we used previously didn't allow us in, so let's proceed with our task.

![3](https://github.com/LaraBruno/Brute-Force---Hydra/assets/37584600/aa07bd98-e483-4936-a064-d0fb9a678cb4)


When we try logging in with the admin credentials that we already have, the input is not pasted in the URL, indicating that the web application is using a POST form.
We can check what kind of services are supported by hydra and see if anything fits our criteria:

> hydra -h | grep services

Luckly, Hydra has a http post form module that will be useful here.

Hydra's man page explains how to use the tool in these scenarios:

> Syntax:
> 
> First is the page on the server to GET or POST to (URL).
> 
> Second is the POST/GET variables (taken from either the browser, proxy, etc.
> 
> with url-encoded (resp. base64-encoded) usernames and passwords being replaced in the
> "^USER^" (resp. "^USER64^") and "^PASS^" (resp. "^PASS64^") placeholders (FORM PARAMETERS)
> 
> Third is the string that it checks for an *invalid* login (by default)
> Invalid condition login check can be preceded by "F=", successful condition
> login check must be preceded by "S=".
> This is where most people get it wrong. You have to check the webapp what a
> failed string looks like and put it in this parameter! Add the -d switch to see
> the sent/received data!

It looks a bit intimidating, so let's break it down into sections.

We can start Hydra using a username/password list, or we can set it to a static, already known username/password list. 

Considering we are searching for a credential to get to the admin panel, and that "admin" account was already seen in this environment, let's set "admin" as the username and use a password list for the password.

> hydra -l admin -P /usr/share/wordlists/rockyou.txt -f 178.128.45.143 -s 30841

Great, now let's work on the http-post-form variables.

The URL for the login page is:
http://178.128.45.143:30841/login.php

Therefore, let's parse that and proceed with the other variables:

> http-post-form /login.php

We intercepted the log in attempt with Burp Suite, and we can see what are the variables being used for the username and password, let's add them into it.

![4](https://github.com/LaraBruno/Brute-Force---Hydra/assets/37584600/205457d3-a2eb-4a6a-9f07-822edd260c31)

> http-post-form /login.php:username=^USER^&password=^PASS^

The last variable we need, is a FAIL or SUCCESS string so Hydra knows that a password attempted was either correct or not.
The login button will likely not be present after logging in, so we can set a FAIL string to the log in button variable, meaning that if that button is still there, Hydra's password attempt failed.

Checking the page source, we can see the following piece of code for the log in button:

> form name='login' autocomplete='off' class='form' action='' method='post'

That not only gives us a string required to proceed, but it also confirms it uses the post method. We're on the right track!

> http-post-form /login.php:username=^USER^&password=^PASS^:F="<form name='login'"

If Hydra is still seeing that piece of code, it should proceed to the next password combination, until it finally finds a correct password.

> hydra -l admin -P /usr/share/wordlists/rockyou.txt -f 178.128.45.143 -s 30841 http-post-form "/login.php:username=^USER^&password=^PASS^:F=<form name='login'"

![5](https://github.com/LaraBruno/Brute-Force---Hydra/assets/37584600/1142abcc-e2d1-4db9-bb83-c659048368ca)

Success!

Hydra found the administrator credentials and we successfully logged into the admin panel! :-)
