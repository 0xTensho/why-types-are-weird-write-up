Opening the instance of the challenge we have a login page, trying sql injections and default login don't work. However when viewing the code ( by hitting CTRL+U ), there is a comment that jumps out :
![[Pasted image 20230807005501.png]]
It does tell us that there is some source.php file. Accessing it returns what looks like some code handling the login :
![[Pasted image 20230807005603.png]]
We see that to be able to login as admin we need to provide admin as the username and a password which hash will equal zero. However since we are in PHP and since the code is not using the strict PHP equal ( `===` ), this is vulnerable to PHP type juggling. More information can be found [here](https://www.youtube.com/watch?v=KOy6QFKZFGQ), but basically in php, every string starting with 0e and followed by numbers will be treated as `0*10^numbers`. For instance, `'2e3' == 2000` is true in php. So any string starting with 0e will be equal to zero. Thanks to [this](https://github.com/spaze/hashes), we already have some magic words which hashes start with 0e ! I'll select the first one from the sha1 list : `aaroZmOk`. When entering this as a password, it works, and we have a form that let us enter a number and see a record in the database ( I didn't have time to screenshot this, sorry ).

When we supply input like `12,'")()` or something like that, it errors out. So it might be vulnerable to an SQL injection. And indeed, having this as a payload works : `0 UNION SELECT 1,2,3--`, and the three fields returned are what we expected, 1 2 and 3.

Then we can A) Enumerate manually (which I did and encourage you doing) using [this](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md) or B) use a tool like sqlmap, which works just fine here.

When we enumerate we see a table called power_users, and dumping all data from this table gives this :

( payload : `0 UNION SELECT username, password,3 FROM power_users -- aaaa` )

![[Pasted image 20230807011451.png]]

I don't remember which one of the two is the real flag but it doesn't matter, it works.