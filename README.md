BazQux Reader API
==========

BazQux Reader clones Google Reader API. 

The only thing you need to support BazQux Reader is to change
google.com to bazqux.com in your code.

To set login/password you need to sign into [BazQux Reader](https://bazqux.com)
and go to Options (top-right corner) => Mobile login.

Main API endpoints:
* https://www.bazqux.com/accounts/ClientLogin
* https://www.bazqux.com/reader/api/0
* https://www.bazqux.com/reader/atom

Most of the API is supported. The only two things missing are
starring and tagging (API calls just returns OK but no starred items
or tags are created).

Getting lists of all/unread items/ids in json/atom formats,
marking items read/unread, adding/removing/renaming of
subscriptions and folders -- everything is supported.

### Login

```
> curl https://www.bazqux.com/accounts/ClientLogin -d Email=foo -d Passwd=bar
Error=BadAuthentication

> curl https://www.bazqux.com/accounts/ClientLogin -d Email=realuser -d Passwd=***
SID=unused
LSID=unused
Auth=cltoken
```

Where `cltoken` is a random session token that you must pass to all other API calls in
`Authorization` header in form `GoogleLogin auth=cltoken`.

You can also test most of API calls right in your browser when you signed in BazQux Reader.

### Ping

https://www.bazqux.com/reader/ping

```
> curl https://www.bazqux.com/reader/ping
Unauthorized

> curl https://www.bazqux.com/reader/ping -H "Authorization: GoogleLogin auth=cltoken"
OK
```

### Token

https://www.bazqux.com/reader/api/0/token

It's not used currently but maybe used later the same way Google Reader uses it 
(expire in 30 minutes, with "x-reader-google-bad-token: true" header set).

```
> curl https://www.bazqux.com/reader/api/0/token -H "Authorization: GoogleLogin auth=cltoken"
Token123
```

### Directory search

https://www.bazqux.com/reader/directory/search

```
> curl https://www.bazqux.com/reader/directory/search?q=foo -H "Authorization: GoogleLogin auth=cltoken"
Search is not yet supported
```

### User info

https://www.bazqux.com/reader/api/0/user-info

```
> curl https://www.bazqux.com/reader/api/0/user-info -H "Authorization: GoogleLogin auth=cltoken"
{"userId":"01234567890123456789","userName":"realuser","userProfileId":"01234567890123456789","userEmail":"realuser",
"isBloggerUser":true,"signupTimeSec":1234567890,"isMultiLoginEnabled":true}
```

BazQux Reader uses dummy `01234567890123456789` user id for all users.

### Preferences list

https://www.bazqux.com/reader/api/0/preference/list ([?output=json](https://www.bazqux.com/reader/api/0/preference/list?output=json))

The only preference is alphabetical sorting of subscriptions.

### Friend list

https://www.bazqux.com/reader/api/0/friend/list ([?output=json](https://www.bazqux.com/reader/api/0/friend/list?output=json))

Always empty.

### Stream preferences list

https://www.bazqux.com/reader/api/0/preference/stream/list ([?output=json](https://www.bazqux.com/reader/api/0/preference/stream/list?output=json))

Contains information about sorting (alphabetical) and expanded/collapsed state of folders. 
`sortId` is just a number of a feed or folder for current user. The same number is also first half of
[ItemId](https://code.google.com/p/google-reader-api/wiki/ItemId) so beware that ItemIds are not unique between users
while unique for single user.

### Tag list

https://www.bazqux.com/reader/api/0/tag/list ([?output=json](https://www.bazqux.com/reader/api/0/tag/list?output=json))

Only contains list of folders plus some google specific feeds. Not item tags present since tagging is not yet supported
in BazQux Reader.

### Subscriptions list

https://www.bazqux.com/reader/api/0/subscription/list ([?output=json](https://www.bazqux.com/reader/api/0/subscription/list?output=json))

Always contain `htmlUrl` not depending on favicons setting. `firstitemmsec` is always dummy `1234567890000` (it seems
that no one use it).

### Adding subscription

https://www.bazqux.com/reader/api/0/subscription/quickadd?quickadd=xkcd.com
([?output=xml](https://www.bazqux.com/reader/api/0/subscription/quickadd?quickadd=xkcd.com&output=xml))

BazQux Reader currently doesn't differ GET/POST and query string or form parameters. But you must use POST and append T=token 
to be future proof.

### Editing subscription

https://www.bazqux.com/reader/api/0/subscription/edit

(?ac=subscribe&s=feed/xkcd.com&t=XKCD)[https://www.bazqux.com/reader/api/0/subscription/edit?ac=subscribe&s=feed/xkcd.com&t=XKCD]
(?ac=unsubscribe&s=feed/xkcd.com&t=XKCD)[https://www.bazqux.com/reader/api/0/subscription/edit?ac=subscribe&s=feed/xkcd.com&t=XKCD]
