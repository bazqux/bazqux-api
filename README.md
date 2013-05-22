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

https://www.bazqux.com/reader/api/0/preference/list
https://www.bazqux.com/reader/api/0/preference/list?output=json

The only preference is alphabetical sorting of subscriptions.
