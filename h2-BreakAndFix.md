# Task X) Summary
> Read/watch/listen and summarize     
> <https://terokarvinen.com/application-hacking/>     

## X.1) Broken access control
> OWASP Top 10: <https://owasp.org/Top10/2021/A01_2021-Broken_Access_Control/index.html> (link added 21.1.2026)     
> Note: I have abbreviated _Access Control_ to _AC_ in some instances!     

**A01:2021 - Broken Access Control**     
- Common AC vulnerabilites
  - Principle of least privilege violation.
  - Bypassing AC checks by tampering with the URL, internal app state, the HTML page or API requests
  - Insicure direct object references, meaning someone is able to view or edit another account by providing it's unique identifier.
  - API access with missing AC for POST
  - Privilege elevation
  - Metadata manipulation
  - CORS misconfiguration

**How to prevent?**
- Deny by default / principle of least privilege
- Implement AC mechanisms once, and re-use through application
- Disable web server directory listing and make sure file metadata and backup files are not present within web roots
- Log AC failures and implement alerts
- API rate limiting     



## X.2) Fuzzing
> Find Hidden Web Directories - Fuzz URLs with ffuf <https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/> (link added 21.1.2026)

**Background:**
- Web servers often have non-linked secret directories
- You could try finding them manually by typing all the possibilities, but `fuff` can do it automatically!
- Fuff is a web fuzzer, created by [joohoi](<https://github.com/joohoi>)

**Get fuzzing**
- Easy to install: `$ sudo apt-get install ffuf`
- The tool is installed, still need a wordlist/dictionary. For example: <https://github.com/danielmiessler/SecLists>
- Then specify wordlist and target and get going:
  - `$ ./ffuf -w common.txt -u http://<target>/FUZZ`
  - `-w` flag for **wordlist**
  - `-u` flag for **target url**
- Helpful:
  - `./fuff ` gives you params and filters to use with the tool     


## X.3) AC vulns and privilege escalation
> <https://portswigger.net/web-security/access-control> (link added: 21.1.2026)

**What is Access Control?**
- Put simply, it answers two questions, **who?** && **what?**
  - "_What actions_ is an _entity X_ allowed to take?"d

**Vertical AC**
- Restrict access to sensitive _functionality_ to specific type of users

**Horizontal AC**
- Restricts access to _resources_ to specific users

**Context-dependent AC**
- Restrict access to functionality and resources _based upon the state of the application, or the user's interaction with it_

**Note:**
- The rest of the article gives examples on common vulnerabilities and broken AC
- Check the task `X.1` above for that!    


## X.4) Writing reports
> A guide for writing technical reports     
> <https://terokarvinen.com/2006/raportin-kirjoittaminen-4/> (link added: 21.1.2026)     

**The meat and bone:**
- **Repeatability & Clarity**
  - Write everything down as clearly as you can, so that somebody following along (doing everything you did), can arrive at the same exact outcome
  - This may include, but not limited to:
    - Computer specs
    - Environment
    - Timestamps
    - Internet connection
    - etc..
  - Also remember to report on any failures and unexpected stuff!
  - Remember to include troubleshooting methods/steps
- **Easy to read**
  - Use clear language and check spelling
  - Use headings
  - Write just enough, don't be tedious!
- **Always reference any material you use!**


**The no-no zone:**
- Straight up lying
- Plagiarisation     

****

# My facilities
**HOST a.k.a the provider**
- Lenovo ThinkPad L490
  - CPU: Intel i3-8145U (4) @ 3.900GHz 
  - GPU: Intel WhiskeyLake-U GT2 UHD Graphics 620
  - Total Memory: 8GB
  - Total Disk size: 256GB
  - OS: Ubuntu 24.04.3 LTS x86_64
  - Kernel: 6.8.0-90-generic
- Running the VM: KVM/QEMU Standard PC (Q35 + ICH9, 2009) (pc-q35-8.2)
- Managed by: Virtual Machine Manager v.4.1.0, powered by libvirt.

**VM a.k.a the attacker**
- OS: Kali GNU/Linux Rolling x86_64
- Kernel: Linux 6.18.5+kali-amd64
- Total Memory: 4GB
- Total Disk size: 28GB     


  
# A) Break-in and enter
> Objective:     
> Break into 010-staff-only     
> Reveal admin password. (It contains the string "SUPERADMIN")        
> <https://terokarvinen.com/hack-n-fix/> (link added: 21.1.2026)      

## Setting up
It all began with the command:
```bash
$ mkdir terosChallenge && cd terosChallenge && wget https://terokarvinen.com/hack-n-fix/teros-challenges.zip
```
This creates a directory, moves into it and downloads the necessities.     
We can then decompress the contents:
```bash
$ unzip teros-challenges.zip
```    
We get a new directory containing the challenge:
- <img width="303" height="170" alt="Screenshot from 2026-01-22 18-03-00" src="https://github.com/user-attachments/assets/73b62eb8-1848-40c1-af31-983217299204" />     

We then start the challenge by running:
```bash
$ ./staff-only.py
```
And paste the address we're given into a browser
- <img width="578" height="378" alt="Screenshot from 2026-01-22 18-09-25" src="https://github.com/user-attachments/assets/c1f3a805-7e78-46ba-8571-274889e92e96" />     

## Walkthrough
By typing in the pin code found at the bottom of the page (123), we're able to recover a password
- <img width="992" height="326" alt="Screenshot from 2026-01-22 18-50-40" src="https://github.com/user-attachments/assets/3fb716d3-3b98-48d3-a6ed-b08697fc937c" />     

The website is kind enough to let us know the SQL query used to recover it: 
```sql
SELECT password FROM pins WHERE pin='<input>';
```
So I tried to inject it by typing the following attach into the input field:
```sql
' OR 1=1--
``` 
- <img width="485" height="130" alt="Screenshot from 2026-01-22 18-32-36" src="https://github.com/user-attachments/assets/e9ed4b6d-96cf-4b0d-ad70-9771be2c7034" />     
But it throws an error "Please enter a number.", so we have to try something else..     

----
## Try #2:
I inspected the html page `ctrl+shift+i`, and found a form field with `input type="number"`, so I changed it to `type="string"` and ran the same injection again
- <img width="483" height="312" alt="Screenshot from 2026-01-22 18-47-36" src="https://github.com/user-attachments/assets/26f39074-6a72-48ea-b0d5-5bbb7babd497" />    
This time around, we got a different answer:
- <img width="620" height="169" alt="Screenshot from 2026-01-22 18-47-52" src="https://github.com/user-attachments/assets/0b212682-df7a-428c-9372-8de2c4f97429" />    
Only problem is, `foo` is not the one we're looking for...     

----

## Try #3:
My next attempt was trying to crack it from the command line, and I found a super useful website explaining how to use `curl` for a POST request.     
> Link: <https://reqbin.com/req/c-g5d14cew/curl-post-example>     

The command I used:
```bash
$ curl -X POST http://127.0.0.1:5000 -H "Content-Type: application/x-www-form-urlencoded" -d "pin=' OR 1=1--"
```
But we're still only getting a `foo` response from the server...
- <img width="604" height="151" alt="Screenshot from 2026-01-23 15-22-58" src="https://github.com/user-attachments/assets/771836c2-7892-4d13-8653-4624103c7e12" />          

----

## Try #4:
I was still destined to perform the injection attack, so I kept trying to change the input field type. (Which I later found out doesn't matter as long as you're able to input a string..)     

I went back to the web page, opened up the developer tool and changed the `type` to `= nvarchar` (this is the part where a simple `=string` is enough). I then typed in the injection:
```sql
' OR 1=1--
```
<img width="703" height="558" alt="Screenshot from 2026-01-23 16-11-45" src="https://github.com/user-attachments/assets/938a4ad5-2811-4201-8a95-f2b150388aa2" />     

Still only giving us `foo`...     

I also tried another variation of the attack like so:
```sql
' OR '1=1
```
Because we know the query, we can simply add an apostrophe before the 1, and leave the comment character out, as the app code will close the quote for us.     
But still nothin....      
- <img width="703" height="558" alt="Screenshot from 2026-01-23 16-16-20" src="https://github.com/user-attachments/assets/730a4ce5-7cd2-4258-b3bd-e76b6cfc829b" />     
Before accepting defeat, I skimmed through what the tip section is saying (linked in the assigment):     
- <img width="797" height="940" alt="Screenshot from 2026-01-23 16-23-57" src="https://github.com/user-attachments/assets/7d64f5f3-fab2-481d-86cf-97c35359e593" />     
Apparently i'm very close, but something is not clicking for me right now, so i'll come back to this!     

----

# (Took a 40min lunch break....)
After a well deserved break I came back to the computer feeling fresh, and _finally internalised_ the fact that the injected query is always going to be true, so it's just delivering us the _first match from the table_. I needed a way to filter it somehow. The tip section pushed me in the right direction, but only when I took the break was I able to internalise this. (Reminder: take breaks! 😃)      
As we also learned from reading the tips, you could use the `LIMIT` operator, but that would require some manual labour, and I want to get straight to the sauce, so here we go:     

## Try #5
I searched online on how to filter the query response, and was going through some SQL syntax. I tried a few things, and finally found something that worked:
```sql
' UNION SELECT pin || '=' || password FROM pins WHERE password LIKE "%ADMIN%"--
```
<img width="1113" height="554" alt="Screenshot from 2026-01-23 23-09-19" src="https://github.com/user-attachments/assets/a61fd145-c96d-4af5-952d-5ce94d149a7f" />     

## Explanation
The `UNION` operator allows us to create a new query, where we then join together the pin and password by using -> `|| '=' ||`.     
We then **filter** the result using a simple regex with the `LIKE` operator.     
This results in us being the new owners of the `pin` and corresponding `password` for the `admin` user.   
(Note: The regex only works because we know the admin password contains the word ADMIN)




# B) Fixing from source
> Now that we know how to break it, let's fix it!     

My ==gameplan== here is to remove the hint from `index.html` page, and then check the source code on how we can enforce the input type for the query.

## staff-only.py
```python
#!/usr/bin/python3
# Copyright 2018-2024 Tero Karvinen http://TeroKarvinen.com
#########################################
# WARNING: Purposefully VULNERABLE APP! #
#########################################

from flask import Flask, render_template, request # sudo apt-get install python3-flask
from flask_sqlalchemy import SQLAlchemy # sudo apt-get install python3-flask-sqlalchemy
from sqlalchemy import text

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
db = SQLAlchemy(app)

@app.route("/", methods=['POST', 'GET'])
def hello():
        if "pin" in request.form:
                pin = str(request.form['pin'])
        else:
                pin = "0"

        sql = "SELECT password FROM pins WHERE pin='"+pin+"';"
        row = ""
        with app.app_context():
                res=db.session.execute(text(sql))
                db.session.commit()
                row = res.fetchone()

        if row is None:
                password="(not found)"
        else:
                password=row[0]
        return render_template('index.html', password=password, pin=pin, sql=sql)

def runSql(sql):
        with app.app_context():
                res=db.session.execute(text(sql))
                db.session.commit()
                return res

def initDb():
        # For simplifying the demo, passwords are also incorrectly stored as plain text. 
        # However, that's not the only thing that's wrong.
        runSql("CREATE TABLE pins (id SERIAL PRIMARY KEY, pin VARCHAR(17), password VARCHAR(20));")
        runSql("INSERT INTO pins(pin, password) VALUES ('321', 'foo');")
        runSql("INSERT INTO pins(pin, password) VALUES ('123', 'Somedude');")
        runSql("INSERT INTO pins(pin, password) VALUES ('11112222333', 'SUPERADMIN%%rootALL-FLAG{Tero-e45f8764675e4463db969473b6d0fcdd}');")
        runSql("INSERT INTO pins(pin, password) VALUES ('321', 'loremipsum');")

if __name__ == "__main__":
        print("WARNING: Purposefully VULNERABLE APP!")
        initDb()
        app.run() # host="0.0.0.0" to serve non-localhost, e.g. from vagrant; debug=True for debug

```

## Fixed version
> For brevity, i'm only going to present the parts that were changed     


### The hello() function
```python
@app.route("/", methods=['POST', 'GET'])
def hello():
        pin = 0
        password = "not here"
        error = None

        if request.method ==  'POST': 
                try:
                        pin = int(request.form['pin'])
                except (KeyError, ValueError):
                        error = "You think you slick huh? Only numbers allowed here!"
                        return render_template('index.html', password=password, pin=pin, error=error)

        if "pin" in request.form:
                pin = str(request.form['pin'])
        else:
                pin = "0"

        sql = "SELECT password FROM pins WHERE pin='"+pin+"';"
        row = ""
        with app.app_context():
                res=db.session.execute(text(sql))
                db.session.commit()
                row = res.fetchone()
        if row is None:
                password="(not found)"
        else:
                password=row[0]
        return render_template('index.html', password=password, pin=pin)
```

### The initDB() function
```python
def initDb():

        runSql("CREATE TABLE pins (id SERIAL PRIMARY KEY, pin INT(17), password VARCHAR(20));")
```


## Execution
Let's try to put our pin code into the field:     
<img width="1090" height="374" alt="Screenshot from 2026-01-24 15-40-07" src="https://github.com/user-attachments/assets/cc538589-ec32-4946-959a-a5b35e0529e4" />     
But as soon as we type something else:       
<img width="1090" height="374" alt="Screenshot from 2026-01-24 15-38-16" src="https://github.com/user-attachments/assets/e283891d-51ff-404b-9768-48e701a22a71" />      

## Explanation





