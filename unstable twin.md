

### Unstable machine from tryhackme and it's a great and undertaking room. so let's begin.....


# Namp Scan.
![Nmap Result](https://user-images.githubusercontent.com/45504631/132653954-a34c70da-2d42-4fa4-a51d-da0b3ab2a40f.png)


We can see there is nothing much more to explore and see what going on. there is a simple web page running on port 80. And ssh on port 22 which one is by default.

### In Enumeration there is one key to look into web directory so let's start..
![Directory Scan.png](https://user-images.githubusercontent.com/45504631/132654224-a0d33b26-71de-42ac-98eb-8da07a7105ed.png)
 web can see there is just one directory in /info but we go to that we can see there is just one message.


 


It means there is a game play-off APIs we need to justify this request into burp so we can get a piece of more information about what's going on request.

### Interpect request throw burp..

![image](https://user-images.githubusercontent.com/45504631/132654944-579ecbdf-eec8-46f6-9bbe-c53bc8bccbe8.png)
It's look normal but when can see It's here build number too. but when I enter on THM It's say wrong flag so I'm try another way but I interpect request again It's changed. I mean build number change by send request again and agian. 
![image](https://user-images.githubusercontent.com/45504631/132655182-a6deddee-bb0d-46cb-a2ed-eb9f814e6b57.png)

When executed multiple times, this request returns different build numbers and server names. It seems like two servers run on the host. You can observe this by observing the HTTP header for both scans.

```
curl -X POST http://10.10.35.201/api
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>405 Method Not Allowed</title>
<h1>Method Not Allowed</h1>
<p>The method is not allowed for the requested URL.</p>
```

  When I curl /API directory It's saying there is one more directory named API I guess by searching about "How API work". so I'm curious to see why the API directory did not appear on scans. Then I try multiple things by changing wordlist but It's not working. after so much hard work I find this 
![image](https://user-images.githubusercontent.com/45504631/132656629-490d5e08-94ae-46e0-9d18-525e5851597c.png)     Defualt configuration of dirseach 
     
We can see by Default dirsearch support some response code and also when I take a close look at the curl request It says ``405 Method Not Allowed``. It means that the API page response code is 405 and we can try ffuf because It's fast and simple...

![image](https://user-images.githubusercontent.com/45504631/132656838-6d0fc891-1218-4ae6-83ae-81352807a299.png)
Found this /api directory and now let's move to Fuzz another directory in /api address...

![image](https://user-images.githubusercontent.com/45504631/132656867-cea01318-0cc3-4b89-8576-2f89c6f38025.png)

A login page I found..

```
curl -X POST http://10.10.35.201/api/login
"The username or password passed are not correct."
```
## SQL Injection Using Curl

There is a hard part in this room so started to try another SQL payloads for discover tables. And the end found pdf on exploit-DB 

> https://www.exploit-db.com/docs/english/41397-injecting-sqlite-database-based-applications.pdf 

Lot's of hard requests and using too much payloads I get payloads to discover version 

> P : UNION SELECT 1,sqlite_version()--

![image](https://user-images.githubusercontent.com/45504631/132657188-455a0110-f975-452a-be38-cb44e331ce58.png)
Now expose the tables..

> P : union SELECT tbl_name FROM sqlite_master  WHERE type='table' and tbl_name NOT like 'sqlite_%

![image](https://user-images.githubusercontent.com/45504631/132656977-e99de36f-b086-48a6-8ff1-b5ca28a3db13.png)

Let's started to look into users first..

> P : UNION SELECT username, password FROM users--

![image](https://user-images.githubusercontent.com/45504631/132657269-297ba99b-0677-461f-9e78-60cb8f313b9d.png)

Now take a look in notes to find ssh cridential.

> P : Union ALL SELECT 1,notes FROM notes---


![image](https://user-images.githubusercontent.com/45504631/132657301-4346add4-2acd-4f71-8bff-6f8adb446fb1.png)
So yeah It's ssh cred and is in hash form. we first to crack down on this..

## HASH CRACKING 

using some identifire we can see this is a SHA-512 hash.
We can crack this using ``John`` and wordlist ``rockyou.txt``

![image](https://user-images.githubusercontent.com/45504631/132663687-2738c890-a91a-4045-9e66-dea942d51dbd.png)

we get a ssh password of user marry_ann. and let's jump into ssh.

![image](https://user-images.githubusercontent.com/45504631/132663772-d2096775-0faa-4a5b-9e0e-7ee0656faf5a.png)

Get a ssh welcome and user flag too.

Now time to final flag and nor there is no flag for root..
 
Hint file in home directory server_notes.txt 

![image](https://user-images.githubusercontent.com/45504631/132663820-b8b30931-379c-4eb6-86c6-9b29d238739d.png)
it says about one directory which contains all family pictures so let's find the directory and found one /opt.

![image](https://user-images.githubusercontent.com/45504631/132663888-a8c3039c-ec21-4e97-b007-44b364ea3d13.png)

yeah after looking we can think this is steganography. but the biggest prob is to share those files in the machine for further process.

And I tried different things like `python, Apache` and many other methods but failed! 

Moving to look what inside main_5000.py we found this to take all images to my machine.

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

And yeah we can see there is /get_image?name= perimeter to get those images according to this code. after using some google research I get how to curl these images in my machine.

![image](https://user-images.githubusercontent.com/45504631/132664206-fefee14b-5f25-4e4f-aa0f-c514d81c45d9.png)

Using these to get all images.



![image](https://user-images.githubusercontent.com/45504631/132664326-629afbe1-d4a7-4216-9eb1-afdddb9a71f9.png)
## steganography

Time to get a final flag.

try steghide to get a what hidden message inside in images.

![image](https://user-images.githubusercontent.com/45504631/132664337-66bbb66f-6fec-425f-85cf-ebf748004a54.png)
At last extracting, I found notes.txt file on every image and there is an all notes file with some hash and line denoted by colors like this.

```
Red - 1DVsdb2uEE0k5HK4GAIZ

```

One notes file like-named notes.txt seems there is some hint to crack those hash.

> Hint : You need to find all my children and arrange in a rainbow!

It means arrang all images rainbow colour like this.
![image](https://user-images.githubusercontent.com/45504631/132664761-554cacd6-0f74-4cbf-a695-97b0e0250d8f.png)
After containing all images in rainbow color sequence.

>hash : 1DVsdb2uEE0k5HK4GAIZPS0Mby2jomUKLjvQ4OSwjKLNAAeCdl2J8BCRuXVXeVYvs6J6HKpZWPG8pfeHoNG1

It's base62 hash decode this using CyberChief and get a final.

![image](https://user-images.githubusercontent.com/45504631/132664838-fd579a4f-903a-4ffc-96c8-9822167ac3ca.png)
