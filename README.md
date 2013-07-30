BazQux Reader API
==========

It's a copy of Google Reader API. 

The only thing you need to support BazQux Reader is to change endpoint URLs from
`google.com` to `bazqux.com` in your code.

To set login/password you need to sign into [BazQux Reader](https://bazqux.com)
and go to the Options (top-right corner) => Mobile login.

API implementation is tested and works with [Mr.Reader](http://mrreaderblog.curioustimes.de/post/52060909928/supported-google-reader-alternatives-part-two), 
[Feeddler](http://blog.chebinliu.com/2013/06/feeddler-status-update.html), [JustReader](http://justreader.net) and [Vienna RSS](http://www.vienna-rss.org). So there is a high probability it will work with your App without any hassle.

### Main API endpoints

* https://www.bazqux.com/accounts/ClientLogin
* https://www.bazqux.com/reader/api/0
* https://www.bazqux.com/reader/atom

Getting lists of all/unread items/ids in json/atom formats,
marking items read/unread, starring, tagging, 
adding/removing/renaming of subscriptions, 
folders and tags, custom subscriptions ordering -- everything is supported.

NB: Starred items, tagging and custom subscriptions ordering
are not yet available in BazQux Reader web interface
but already available through API.

### Warning!

BazQux Reader does not automatically mark items as read
after 30 days. So please don't add `ot=CurrentTime-30days`
when you get unread items 
(`s=user/-/state/com.google/reading-list&&xt=user/-/state/com.google/read`). 
You may miss some unread items this way.

### Login

```
> curl https://www.bazqux.com/accounts/ClientLogin -d Email=foo -d Passwd=bar
Error=BadAuthentication

> curl https://www.bazqux.com/accounts/ClientLogin -d Email=realuser -d Passwd=***
SID=unused
LSID=unused
Auth=cltoken
```

Where `cltoken` is a client login token that you must pass to all other API calls in
`Authorization` header in form `GoogleLogin auth=cltoken`.

You can test most API calls right in your browser when you signed in BazQux Reader.

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

It's not used currently but may be used later the same way Google Reader uses it 
(expires in 30 minutes, with "x-reader-google-bad-token: true" header set).

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

BazQux Reader uses dummy `01234567890123456789` user id for all users and accept any user in Google Reader labels
or states `user/Abracadabra/label/MyFolder` = `user/01234567890123456789/label/MyFolder` = `user/-/label/MyFolder`. 

### Preferences list

https://www.bazqux.com/reader/api/0/preference/list ([?output=json](https://www.bazqux.com/reader/api/0/preference/list?output=json))

The only preference is alphabetical sorting of subscriptions 
(custom ordering is not yet available on website).

### Friend list

https://www.bazqux.com/reader/api/0/friend/list ([?output=json](https://www.bazqux.com/reader/api/0/friend/list?output=json))

Empty list for compatibility.

### Stream preferences list

https://www.bazqux.com/reader/api/0/preference/stream/list ([?output=json](https://www.bazqux.com/reader/api/0/preference/stream/list?output=json))

Contains information about sorting (alphabetical) and expanded/collapsed state of folders. 
`sortId` is just a number of a feed or folder for current user. The same number is also first half of
[ItemId](https://code.google.com/p/google-reader-api/wiki/ItemId) so beware that ItemIds are not unique between users
while unique for single user.

### Set stream preferences

https://www.bazqux.com/reader/api/0/preference/stream/set

You may only set `k=subscription-ordering&s=...&v=...`. Other parameters are ignored.

### Tag list

https://www.bazqux.com/reader/api/0/tag/list ([?output=json](https://www.bazqux.com/reader/api/0/tag/list?output=json))

Only contains list of folders, tags and some google specific feeds.

### Subscriptions list

https://www.bazqux.com/reader/api/0/subscription/list ([?output=json](https://www.bazqux.com/reader/api/0/subscription/list?output=json))

Always contain `htmlUrl` not depending on favicons setting. `firstitemmsec` is always dummy `1234567890000` (it seems
that no one use it).

### Subscriptions OPML

https://www.bazqux.com/reader/subscriptions/export

### Adding subscription

https://www.bazqux.com/reader/api/0/subscription/quickadd?quickadd=xkcd.com
([?output=xml](https://www.bazqux.com/reader/api/0/subscription/quickadd?quickadd=xkcd.com&output=xml))

BazQux Reader currently doesn't differ GET/POST and query string or form parameters. But you must use POST and append T=token 
to be future proof (and to be compatible with Google Reader API).

### Editing subscription

https://www.bazqux.com/reader/api/0/subscription/edit

[...?ac=subscribe&s=feed/xkcd.com&t=XKCD](https://www.bazqux.com/reader/api/0/subscription/edit?ac=subscribe&s=feed/xkcd.com&t=XKCD)

[...?ac=edit&s=feed/http://xkcd.com/atom.xml&t=NewTitle&a=user/-/label/Comics&r=user/01234567890123456789/label/Foo](https://www.bazqux.com/reader/api/0/subscription/edit?ac=edit&s=feed/http://xkcd.com/atom.xml&t=NewTitle&a=user/-/label/Comics&r=user/01234567890123456789/label/Foo)

[...?ac=unsubscribe&s=feed/http://xkcd.com/atom.xml](https://www.bazqux.com/reader/api/0/subscription/edit?ac=unsubscribe&s=feed/http://xkcd.com/atom.xml)

You can put one subscription in many folders.

### Unread count

https://www.bazqux.com/reader/api/0/unread-count ([?output=json](https://www.bazqux.com/reader/api/0/unread-count?output=json))

### Item ids

https://www.bazqux.com/reader/api/0/stream/items/ids ([?output=json](https://www.bazqux.com/reader/api/0/stream/items/ids?output=json))

`s=user/-/state/com.google/reading-list`

`s=user/-/state/com.google/starred`

`s=user/-/state/com.google/read`

`s=user/-/state/com.google/broadcast` - empty results

`s=user/-/state/com.google/created` - empty results

`s=user/-/label/...` for folders or tags

`s=feed/...`

`r=o` for oldest first ranking.

`xt=...` - everything possible in `s=`.

`it=...` - only messages with specific tags (starred items in feed). NB: It's an extension to GR API

`ck=...` is ignored.

`ot=...` - please don't add it when you get unread items list.

`nt=...`

`c=...` continuation from previous request (it's just an ItemId and hence never expire).

`n=50000` maximum, 20 default.

Note that item ids are unique for one user but can overlap between users. Please use separate database for each account.

### Fetching individual items

https://www.bazqux.com/reader/api/0/stream/items/contents ([?output=atom](https://www.bazqux.com/reader/api/0/stream/items/contents?output=atom))

No more than 10000 items at once.

### Fetching streams

https://www.bazqux.com/reader/api/0/stream/contents ([?output=atom](https://www.bazqux.com/reader/api/0/stream/contents?output=atom))

https://www.bazqux.com/reader/atom (= /stream/contents?output=atom)

The same options as in `stream/items/ids` are supported. When no subscription given defaults to reading-list.

`n=10000` maximum.

### Tagging items

https://www.bazqux.com/reader/api/0/edit-tag

`user/-/state/com.google/read` for marking read/unread, `user/-/state/com.google/starred` for starring
and `s=user/-/label/...` for tagging.

No more than 10000 items to tag at once.

### Folder/tag renaming

https://www.bazqux.com/reader/api/0/rename-tag?s=user/-/label/Comics&dest=user/-/label/NiceComics

### Folder/tag removing

https://www.bazqux.com/reader/api/0/disable-tag?s=user/-/label/NiceComics

Feeds are not removed, only folder tag is (like in Google Reader).

### Subscriptions import

NB: This method is an extension to GR API. 

https://www.bazqux.com/reader/api/0/import/opml

Post OPML data here. It will return percent of feeds currently processed (integer number in plain text).

You can then poll (each 3 seconds for example)

https://www.bazqux.com/reader/api/0/import/percent-complete

till it return "100".
