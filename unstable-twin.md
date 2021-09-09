---
title: "unstable twin"
tags: ""
---

### Unstable machine from tryhackme and its a great and undertaking room. so let's begain.....


# Namp Scan.
![Nmap Result.png](https://boostnote.io/api/teams/jShH61kcp/files/0b3edb4309cefd3ca626614bc9cdb6ec7b79c25b786ea71817dac1e34bcfd5e4-Nmap%20Result.png)

We can clearly see there is nothing much more for explore and see what going on. there is simple web page runnging on port 80. And ssh on port 22 which one is by defualt.

### In Enumeration there is one key to look into web directory so let's start..
![Directory Scan.png](https://boostnote.io/api/teams/jShH61kcp/files/a416ca6c2e788534b99250b5b4d3ab2523fa0463bccb6f5a174dd0f323807e5f-Directory%20Scan.png)
 web can clearly see there is just one directory in /info but we goes to that we can see there is just one message.


 
![burp11.png](https://boostnote.io/api/teams/jShH61kcp/files/c5b5fae0821ea4a66b0bd586e7c755890b55dfba61f40d68d4569e110e29f1f7-burp11.png)

It means there is game play off APIs we need to just this request into burp so we can get a more information about what's actually going on request.

### Interpect request throw burp..

![burp1.png](https://boostnote.io/api/teams/jShH61kcp/files/09c0d7b4d9bc989a13146473c11899afbd33f81ac07ab4e4f7379ae744f6db6d-burp1.png)

It's look normal but when can see It's here build number too. but when I enter on THM It's say wrong flag so I'm try another way but I interpect request again It's changed. I mean build number change by send request again and agian. 
![burp2.png](https://boostnote.io/api/teams/jShH61kcp/files/d7abc2ebf46c27ac9bcc3e0f5d9fc6e1e005bdaf837f8404839d5697c9ab4765-burp2.png)

When executed multiple times, this request returns different build numbers and server names. It seems like two servers run on the host. You can observe this by observing the HTTP header for both scans.

```
curl -X POST http://10.10.35.201/api
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>405 Method Not Allowed</title>
<h1>Method Not Allowed</h1>
<p>The method is not allowed for the requested URL.</p>
```

  When I curl /api directory It's say there is one more directory named api i guess by searching about "How API work". so I'm curious to see why api directory not appear on scans. Then I try multiple thing by changing wordlist but It's not working. after so much hard work I find this 
![Untitled.png](https://boostnote.io/api/teams/jShH61kcp/files/ae6ca12a950cb414684b946733c2c3e6701bee014ef4f5d9ec16606a34e7ef62-Untitled.png)
     Defualt configuration of dirseach 
     
We can clearly see by Defualt dirsearch support some respons code and also when I take a close look on curl request It's say ``405 Method Not Allowed``. It mean that api page respons code is 405 and we can try ffuf becasue It's fast and simple..

![ffuf scan 2.png](https://boostnote.io/api/teams/jShH61kcp/files/10de420ef256a24baefaeb4eec985ae1e3d6d5040b834eb1902ca18e4a6d1d79-ffuf%20scan%202.png)

Found this /api directory and now let's move to Fuzz another directory in /api address..

![ffuf Scan.png](https://boostnote.io/api/teams/jShH61kcp/files/9297d9d15aeab1b43417980765f30159cd81855e5c183de936508689a6469a50-ffuf%20Scan.png)

A login page I found..

```
curl -X POST http://10.10.35.201/api/login
"The username or password passed are not correct."
```
## SQL Injection Using Curl

There is a hard part in this room so started try another another sql payloads for discover tables. And the end found pdf on exploit-db 

> https://www.exploit-db.com/docs/english/41397-injecting-sqlite-database-based-applications.pdf 

Lot's of hard request and using to much payloads I get payloads to discover verion 

> P : UNION SELECT 1,sqlite_version()--

![curl for Version.png](https://boostnote.io/api/teams/jShH61kcp/files/94de3b5cb39f8f27a5456e9edf3bbc9f035cbb3dd3c28e4ac09866934526ddfc-curl%20for%20Version.png)

Now expose the tables..

> P : union SELECT tbl_name FROM sqlite_master  WHERE type='table' and tbl_name NOT like 'sqlite_%

![curl for checking table_names.png](https://boostnote.io/api/teams/jShH61kcp/files/89701ea4f119947079d3406a1cda172b98419e511becc541f205ab422fd01f03-curl%20for%20checking%20table_names.png)

Let's started to look into users first..

> P : UNION SELECT username, password FROM users--

![curl for user tabe.png](https://boostnote.io/api/teams/jShH61kcp/files/286bf918ac53c919f4b515a8b8a7801153787db63c8184e45d5cfdf207f6fe5b-curl%20for%20user%20tabe.png)



Now take a look in notes to find ssh cridential.

> P : Union ALL SELECT 1,notes FROM notes---


![curl for notes table.png](https://boostnote.io/api/teams/jShH61kcp/files/844b03fe32b7e7199a8d9765635facdbcba454a01e1a9181138139b637894795-curl%20for%20notes%20table.png)

So yeah It's ssh cred and is in hash form. we first to crack down this..

## HASH CRACKING 

using some identifire we can see this is a SHA-512 hash.
We can crack this using ``John`` and wordlist ``rockyou.txt``

![hash cracking.png](https://boostnote.io/api/teams/jShH61kcp/files/1222d2d7ec8b01edaa09e9e8437023b8c486dafad852c66ce7cf1c46290a280c-hash%20cracking.png)


we get a ssh password of user marry_ann. and let's jump into ssh.

![ssh welcome.png](https://boostnote.io/api/teams/jShH61kcp/files/ae46423533cc272afce3fd8a146b3635a2fb56521142f4b8ea4c00d917952ae5-ssh%20welcome.png)


Get a ssh welcome and user flag too.

Now time to final flag and nor there is no flag for root..
 
Hint file in home directory server_notes.txt 

![find_in_note_ssh.png](https://boostnote.io/api/teams/jShH61kcp/files/9e5d1e4d78a91464b17894ec3285e4f25f68f54acbb341ac38011c88713cd926-find_in_note_ssh.png)

it's says about one directory which contains all family pictures so let's find directory and found one /opt.

![photo album.png](https://boostnote.io/api/teams/jShH61kcp/files/f12f4031cd8d400161c434afc39b97919860ae66970d6456ee98762ac1cea2c5-photo%20album.png)


yeah after looking we can clearly think this is a steganography. but the biggest prob is share those files in machine for futher process.

And I tried different things like `python,Apache` and many other methods to but failed! 

Moving to looking what indside main_5000.py we found this to take all images to my machine.

```
@app.route('/get_image')
def get_image():
    if request.args.get('name').lower() == 'vincent':
        filename = 'Twins-Danny-DeVito.jpg'
        return send_file(filename, mimetype='image/gif')
    elif request.args.get('name').lower() == 'julias':
        filename = 'Twins-Arnold-Schwarzenegger.jpg'
        return send_file(filename, mimetype='image/gif')
    elif request.args.get('name').lower() == 'mary_ann':
        filename = 'Twins-Bonnie-Bartlett.jpg'
        return send_file(filename, mimetype='image/gif')
    return '', 404

```

And yeah we can see there is /get_image?name= peremeter to get those images according to these code. after using some google research I get how to curl these images in my machine.

![geting those images for steghide.png](https://boostnote.io/api/teams/jShH61kcp/files/5043009c0e3b54c7598d6805be84779d5b0aaffda5373f06437b2245427b71bb-geting%20those%20images%20for%20steghide.png)

Using these I can get all images.



![all image in my machine.png](https://boostnote.io/api/teams/jShH61kcp/files/3aa37468142520da1326aa63e2c8a794380c07f3c5d51eba0726a9d9c14189c5-all%20image%20in%20my%20machine.png)

## steganography

Time to get a final flag.

try steghide to get a what hidden message inside in a images.

![steghide all.png](https://boostnote.io/api/teams/jShH61kcp/files/b000330bba861fc3ca66bdca7ba73c93c2518636741c53b5c2990bc0e1137f69-steghide%20all.png)

At last extracting I found notes.txt file on every images and there is a all notes file with some hash and line denoted by colors like this.

```
Red - 1DVsdb2uEE0k5HK4GAIZ

```

One notes file like named notes.txt seems there is some hint to crack those hash.

> Hint : You need to find all my children and arrange in a rainbow!

It means arrang all images rainbow colour like this.
![Untitled.png](https://boostnote.io/api/teams/jShH61kcp/files/045bfce6b248f7162d164100daf9a31e097604ddf1e95deb416d80a096a4e380-Untitled.png)

After containing all images in rainbow color sequence.

>hash : 1DVsdb2uEE0k5HK4GAIZPS0Mby2jomUKLjvQ4OSwjKLNAAeCdl2J8BCRuXVXeVYvs6J6HKpZWPG8pfeHoNG1

It's base62 hash decode this using CyberChief and get a final.

![Untitled.png](https://boostnote.io/api/teams/jShH61kcp/files/cc2ea403b1bc5fde8cfef6d436d419517854b296ba3f02d018bc1c52192646a0-Untitled.png)
