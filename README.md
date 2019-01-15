TLSharp
-------------------------------

_Unofficial_ Telegram (http://telegram.org) client library implemented in C#. Please refer to (https://github.com/sochix/TLSharp) for the original version and further documentation.
[![Build status](https://ci.appveyor.com/api/projects/status/95rl618ch5c4h2fa?svg=true)](https://ci.appveyor.com/project/sochix/tlsharp)
[![NuGet version](https://badge.fury.io/nu/TLSharp.svg)](https://badge.fury.io/nu/TLSharp)

# Sample code
```csharp
using System;
using System.Threading.Tasks;
using TeleSharp.TL;
using TLSharp.Core;
using System.Linq;
using TeleSharp.TL.Messages;
using System.Collections.Generic;

🚩 Check out [TeleJS](https://github.com/RD17/TeleJS) - a pure JavaScript implementation of Telegram MTP protocol

It's a perfect fit for any developer who would like to send data directly to Telegram users or write own custom Telegram client.

:star2: If you :heart: library, please star it! :star2:

# Table of contents

- [How do I add this to my project?](#how-do-i-add-this-to-my-project)
- [Dependencies](#dependencies)
- [Starter Guide](#starter-guide)
  - [Quick configuration](#quick-configuration)
  - [First requests](#first-requests)
  - [Working with files](#working-with-files)
- [Available Methods](#available-methods)
- [Contributors](#contributors)
- [FAQ](#faq)
- [Donations](#donations)
- [Support](#support)
- [License](#license)

# How do I add this to my project?

Install via NuGet

```
	> Install-Package TLSharp
```

or build from source

1. Clone TLSharp from GitHub
1. Compile source with VS2015 or MonoDevelop
1. Add reference to ```TLSharp.Core.dll``` to your awesome project.

# Dependencies

TLSharp has a few dependenices, most of functionality implemented from scratch.
All dependencies listed in [package.conf file](https://github.com/sochix/TLSharp/blob/master/TLSharp.Core/packages.config).

# Starter Guide

## Quick Configuration
Telegram API isn't that easy to start. You need to do some configuration first.

1. Create a [developer account](https://my.telegram.org/) in Telegram. 
1. Goto [API development tools](https://my.telegram.org/apps) and copy **API_ID** and **API_HASH** from your account. You'll need it later.

## First requests
To start work, create an instance of TelegramClient and establish connection

```csharp 
   var client = new TelegramClient(apiId, apiHash);
   await client.ConnectAsync();
```
Now you can work with Telegram API, but ->
> Only a small portion of the API methods are available to unauthorized users. ([full description](https://core.telegram.org/api/auth)) 

                var contacts = await client.GetContactsAsync();
                Console.WriteLine("Contacts:");
                foreach (var contact in contacts.Users.OfType<TLUser>())
                {
                    var contactUser = contact as TLUser;
                    Console.WriteLine($"\t{contact.Id} {contact.Phone} {contact.FirstName} {contact.LastName}");
                }


                var dialogs = (TLDialogs) await client.GetUserDialogsAsync();
                Console.WriteLine("Channels: ");
                foreach (var channelObj in dialogs.Chats.OfType<TLChannel>())
                {
                    var channel = channelObj as TLChannel;
                    Console.WriteLine($"\tChat: {channel.Title}");
                }

                Console.WriteLine("Groups:");
                TLChat chat = null;
                foreach (var chatObj in dialogs.Chats.OfType<TLChat>())
                {
                    chat = chatObj as TLChat;
                    Console.WriteLine($"Chat name: {chat.Title}");
                    var request = new TLRequestGetFullChat() { ChatId = chat.Id };
                    var fullChat = await client.SendRequestAsync<TeleSharp.TL.Messages.TLChatFull>(request);

                    var participants = (fullChat.FullChat as TeleSharp.TL.TLChatFull).Participants as TLChatParticipants;
                    foreach (var p in participants.Participants)
                    {
                        if (p is TLChatParticipant)
                        {
                            var participant = p as TLChatParticipant;
                            Console.WriteLine($"\t{participant.UserId}");
                        }
                        else if (p is TLChatParticipantAdmin)
                        {
                            var participant = p as TLChatParticipantAdmin;
                            Console.WriteLine($"\t{participant.UserId}**");
                        }
                        else if (p is TLChatParticipantCreator)
                        {
                            var participant = p as TLChatParticipantCreator;
                            Console.WriteLine($"\t{participant.UserId}**");
                        }
                    }

                    var peer = new TLInputPeerChat() { ChatId = chat.Id };
                    var m = await client.GetHistoryAsync(peer, 0, 0, 0);
                    Console.WriteLine(m);
                    if (m is TLMessages)
                    {
                        var messages = m as TLMessages;

  //find recipient in contacts
  var user = result.Users
	  .Where(x => x.GetType() == typeof (TLUser))
	  .Cast<TLUser>()
	  .FirstOrDefault(x => x.Phone == "<recipient_phone>");
	
  //send message
  await client.SendMessageAsync(new TLInputPeerUser() {UserId = user.Id}, "OUR_MESSAGE");
```

Full code you can see at [SendMessage test](https://github.com/sochix/TLSharp/blob/master/TLSharp.Tests/TLSharpTests.cs#L87)

To send message to channel you could use the following code:
```csharp
  //get user dialogs
  var dialogs = (TLDialogsSlice) await client.GetUserDialogsAsync();

  //find channel by title
  var chat = dialogs.Chats
    .Where(c => c.GetType() == typeof(TLChannel))
    .Cast<TLChannel>()
    .FirstOrDefault(c => c.Title == "<channel_title>");

  //send message
  await client.SendMessageAsync(new TLInputPeerChannel() { ChannelId = chat.Id, AccessHash = chat.AccessHash.Value }, "OUR_MESSAGE");
```
Full code you can see at [SendMessageToChannel test](https://github.com/sochix/TLSharp/blob/master/TLSharp.Tests/TLSharpTests.cs#L107)
## Working with files
Telegram separate files to two categories -> big file and small file. File is Big if its size more than 10 Mb. TLSharp tries to hide this complexity from you, thats why we provide one method to upload files **UploadFile**.

```csharp
	var fileResult = await client.UploadFile("cat.jpg", new StreamReader("data/cat.jpg"));
```

TLSharp provides two wrappers for sending photo and document

```csharp
	await client.SendUploadedPhoto(new TLInputPeerUser() { UserId = user.Id }, fileResult, "kitty");
	await client.SendUploadedDocument(
                new TLInputPeerUser() { UserId = user.Id },
                fileResult,
                "some zips", //caption
                "application/zip", //mime-type
                new TLVector<TLAbsDocumentAttribute>()); //document attributes, such as file name
```
Full code you can see at [SendPhotoToContactTest](https://github.com/sochix/TLSharp/blob/master/TLSharp.Tests/TLSharpTests.cs#L125) and [SendBigFileToContactTest](https://github.com/sochix/TLSharp/blob/master/TLSharp.Tests/TLSharpTests.cs#L143)

To download file you should call **GetFile** method
```csharp
	await client.GetFile(
                new TLInputDocumentFileLocation()
                {
                    AccessHash = document.AccessHash,
                    Id = document.Id,
                    Version = document.Version
                },
                document.Size); //size of fileChunk you want to retrieve
```

Full code you can see at [DownloadFileFromContactTest](https://github.com/sochix/TLSharp/blob/master/TLSharp.Tests/TLSharpTests.cs#L167)

# Available Methods

For your convenience TLSharp have wrappers for several Telegram API methods. You could add your own, see details below.

1. IsPhoneRegisteredAsync
1. SendCodeRequestAsync
1. MakeAuthAsync
1. SignUpAsync
1. GetContactsAsync
1. SendMessageAsync
1. SendTypingAsync
1. GetUserDialogsAsync
1. SendUploadedPhoto
1. SendUploadedDocument
1. GetFile
1. UploadFile
1. SendPingAsync
1. GetHistoryAsync

**What if you can't find needed method at the list?**

Don't panic. You can call any method with help of `SendRequestAsync` function. For example, send user typing method: 

```csharp

  //Create request 
  var req = new TLRequestSetTyping()
  {
    Action = new TLSendMessageTypingAction(),
    Peer = new TLInputPeerUser() { UserId = user.Id }
  };

  //run request, and deserialize response to Boolean
  return await client.SendRequestAsync<Boolean>(req);
``` 

**Where you can find a list of requests and its params?**

The only way is [Telegram API docs](https://core.telegram.org/methods). Yes, it's outdated. But there is no other source.
Latest scheme in JSON format you can find [here](https://gist.github.com/aarani/b22b7cda024973dff68e1672794b0298)

## What things can I Implement (Project Roadmap)?

### Release 1.0.0

* [DONE] Add PHONE_MIGRATE handling
* [DONE] Add FILE_MIGRATE handling
* Add Updates handling
* [DONE] Add NuGet package
* [DONE] Add wrappers for media uploading and downloading
* Store user session as JSON

# FAQ

#### What API layer is supported?
The latest one - 66. Thanks to Afshin Arani for his TLGenerator

#### I get a xxxMigrationException or a MIGRATE_X error!

TLSharp library should automatically handle these errors. If you see such errors, please open a new Github issue with the details (include a stacktrace, etc.).

#### I get an exception: System.IO.EndOfStreamException: Unable to read beyond the end of the stream. All test methos except that AuthenticationWorks and TestConnection return same error. I did every thing including setting api id and hash, and setting server address.-

You should create a Telegram session. See [configuration guide](#sending-messages-set-up)

#### Why do I get a FloodException/FLOOD_WAIT error?
It's likely [Telegram restrictions](https://core.telegram.org/api/errors#420-flood), or a bug in TLSharp (if you feel it's the latter, please open a Github issue). You can know the time to wait by accessing the FloodException::TimeToWait property.

#### Why does TLSharp lacks feature XXXX?

TLSharp only covers basic functionality of the Telegram protocol, you can be a contributor or a sponsor to speed-up developemnt of any more new features.

#### Where else to ask for help?
If you think you have found a bug in TLSharp, create a github issue. But if you just have questions about how to use TLSharp, use our gitter channel (https://gitter.im/TLSharp/Lobby) or our Telegram channel (https://t.me/joinchat/AgtDiBEqG1i-qPqttNFLbA).

                foreach(var user in allUpdates.Users)
                {
                    Console.WriteLine($"{user}");
                }

                foreach (var chat in allUpdates.Chats)
                {
                    Console.WriteLine($"{chat}");
                }
            }
        }

        private void MarkMessageRead(TelegramClient client, TLAbsInputPeer peer, int id)
        {
	    // An exception happens here but it's not fatal.
            try
            {
                var request = new TLRequestReadHistory();
                request.MaxId = id;
                request.Peer = peer;
                client.SendRequestAsync<bool>(request).Wait();
            }
            catch {}

        }
    }
}

```

# Support
If you have troubles while using TLSharp, I can help you for an additional fee. 

My pricing is **219$/hour**. I accept PayPal. To request a paid support write me at Telegram @sochix, start your message with phrase [PAID SUPPORT].

# Contributors
* [Afshin Arani](http://aarani.ir) - TLGenerator, and a lot of other usefull things
* [Knocte](https://github.com/knocte)

# License

**Please, provide link to an author when you using library**

The MIT License

Copyright (c) 2015 Ilya Pirozhenko http://www.sochix.ru/

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
