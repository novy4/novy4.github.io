+++ 
draft = false
date = 2018-11-22T11:07:35+02:00
title = "Telegram bot on python"
slug = "python-telegram-bot" 
tags = [
    "python",
    "development",
    "telegram"
]
categories = [
    "development"
]
+++

### Introduction

Once upon a time I had a need to for a telegram bot that will reposts news from external channels to my own channel to stay up-to-day with news from different sources in one place. Doing some short research, I didn’t found suitable open-source script for my needs and I decided to write my own script on python. It is not ideal, but it works :-)

There is a library called <a href="https://telethon.readthedocs.io/en/stable/" rel="Telethon"> Telethon </a> that makes easy to write python applications that can interact with Telegram. Think of it as a wrapper that has already done the heavy job for us, so we can focus on developing an application.

### Library Installation
Installation can be done via simple command
```
pip3 install telethon
```

### Coding

Starting our script with importing library components and creating a client
```
from telethon import TelegramClient
from telethon import utils
import time
import os, sys

```

Let's assign the api id, hash and my phone number

```
api_id = XXXXXX
api_hash ='xxxxxxxxxxxxxxxxxxxxxxx'
phone_number = '+XXXXXXXXXXX'
```

To start our client, we have to initialize it
```
client = TelegramClient('me', api_id, api_hash)
client.start()
```

And now we have to create a logic that will take the last message from the history and to store it value in a variable. This way we setting variable for a very first time when we are starting the script
```
for message in client.get_message_history('icodrops', limit=1):
    last = message.id
    print(message.id)

``` 

To run our script in a loop, we are adding while operator. Then have another loop that assigning message body to the variable $cryptomessage  
```
while True:

    for message in client.get_message_history('telegram_channel_name', min_id=last): 
        cryptomessage = message.message
```

With the operator if we are checking if current message id is greater than the first one that was assigned before then we rewriting original variable $last with a current message id.
```
if message.id > last:
            last = message.id
```

Next if operator is verifying that message is not empty, as a result we are changing the encoding of the message and sending the message to our telegram channel. Else we are sending a capture of an image that sometimes appears instead of text
```
if cryptomessage != '':
            mesg1 = cryptomessage.encode('utf-8') 
            client.send_message('cryptoanalizatorfeed', mesg1)
            time.sleep(1)
        else:
            client.send_message('cryptoanalizatorfeed', message.media.caption)
```
Waiting for the 30 seconds and checking the whole logic again
```
time.sleep(30)
```

So, lets see the whole script

```
#!/usr/bin/python3

from telethon import TelegramClient
from telethon import utils
import time
import os, sys

#My API values
api_id = XXXXXX
api_hash ='xxxxxxxxxxxxxxxxxxxxxxx'
phone_number = '+XXXXXXXXX'

client = TelegramClient('me', api_id, api_hash)
client.start()

for message in client.get_message_history('icodrops', limit=1):
    last = message.id
    print(message.id)

while True:

    for message in client.get_message_history('icodrops', min_id=last): 
        cryptomessage = message.message
        if message.id > last:
            last = message.id
        if cryptomessage != '':
            mesg1 = cryptomessage.encode('utf-8')
            client.send_message('cryptoanalizatorfeed', mesg1)
            time.sleep(1)
        else:
            client.send_message('cryptoanalizatorfeed', message.media.caption)
    time.sleep(30)
```

This script can be wrapped into a docker image in combination with environment variables for $api_id, $api_hash and $phone_number. In the next article I will show how to wrap such a script into a docker image. 
