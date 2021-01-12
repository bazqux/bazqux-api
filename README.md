BazQux Reader API
==========

It's a copy of Google Reader API.

The only thing you need to support BazQux Reader is to change endpoint URLs from
`google.com` to `bazqux.com` (or `www.google.com` to `www.bazqux.com`) in your code.

You need to have active account (either free trial or paid one) in [BazQux Reader](https://bazqux.com).
If you're client app developer you could mail you [account id](https://bazqux.com/whoami) to support@bazqux.com
and I will mark your account as forever free.

If you haven't registered via email & password (logged in via Google/Facebook/Twitter/OpenID)
you need to set username & password in settings (top-right corner) => Account.

API implementation is tested and works with [Reeder](https://reederapp.com/),
[Feeddler](https://itunes.apple.com/app/feeddler-rss-reader-pro-2/id919056339), [Web Subscriber](https://zoziapps.ch/web-subscriber/) and [Vienna RSS](http://www.vienna-rss.com). So there is a high probability it will work with your App without any hassle.

BazQux Reader also implements [Fever API](http://feedafever.com/api) ([local copy](FeverAPI.md)), which works in [Unread](https://www.goldenhillsoftware.com/unread/) and [ReadKit](https://readkitapp.com).

### Warning!

BazQux Reader does not automatically mark items as read
after 30 days. So please don't add `ot=CurrentTime-30days`
when you get unread items
(`s=user/-/state/com.google/reading-list&xt=user/-/state/com.google/read`).
You may miss some unread items this way.

In general it's not right approach to use `ot`. Right way is to get [ids](#item-ids) of all, unread and starred items limited by `n=<number_of_items>` then remove items that are no longer exists, fetch new items and sync unread/starred state. Read more in the next section.

### The Right Way to Sync

I'm tired of seeing new apps that try to download `reading-list` stream, passing `ot` (or, even worse, download each feed with `ot` flag — do you know that user can have 3000 feeds?) and then wondering why it doesn't work.

Few reasons why using `ot` is a **bad idea**:

* User subscribed to a new feed and it has posts before `ot` in `reading-list` that you won't sync (please, do not sync 3000 feeds or sync this new feed separately to solve this).

* User marked some 5 years old item (or even yesterday one) as unread. Oh, it's not in the `reading-list` or your feed filtered by `ot` again.

* BazQux Reader does not automatically mark items as read after 30 days. If you add `ot=CurrentTime-30days` you will miss some items.

* BazQux Reader removes old items from feeds to keep 500 last items per feed (to not blow up the database). So if you sync only new items you may get into situation when you will have items (in high volume feeds) in your app that are no longer in BazQux Reader.

* And BazQux Reader has filters. Real filters that really hide items (from both user and apps) instead of marking them as read. And user could change filters at any time. So any item from the past could be removed or added to the `reading-list` at any time and your app will miss them. And you (and me) will get a support request.

How to do it right? Use [item IDs](#item-ids), Luke!

General syncing process is:
- Fetch [list of feeds and folders](#subscriptions-list).
- Fetch [list of tags](#tag-list) (it contains folders too, so you need to remove folders found in previous call to get tags).
- Fetch [ids](#item-ids) of unread items

  https://bazqux.com/reader/api/0/stream/items/ids?output=json&s=user/-/state/com.google/reading-list&xt=user/-/state/com.google/read&n=1000
- Fetch ids of starred items

  https://bazqux.com/reader/api/0/stream/items/ids?output=json&s=user/-/state/com.google/starred&n=1000
- Fetch tagged item ids by passing `s=user/-/label/TagName` parameter.
- Remove items that are no longer in unread/starred/tagged ids lists from your local database.
- Fetch [contents of items](#fetching-individual-items) missing in database.
- Mark/unmark items read/starred/tagged in you app comparing local state and ids you've got from the BazQux.

Use [edit-tag](#tagging-items) to sync read/starred/tagged status from your app to BazQux.

### Main API endpoints

* https://bazqux.com/accounts/ClientLogin
* https://bazqux.com/reader/api/0
* https://bazqux.com/reader/atom

Getting lists of all/unread items/ids in json/atom formats,
marking items read/unread, starring, tagging,
adding/removing/renaming of subscriptions,
folders and tags (smart streams works like tags from API), custom subscriptions ordering -- everything is supported.

### Login

```
> curl https://bazqux.com/accounts/ClientLogin -d Email=foo -d Passwd=bar
Error=BadAuthentication

> curl https://bazqux.com/accounts/ClientLogin -d Email=realuser -d Passwd=***
SID=unused
LSID=unused
Auth=cltoken
```

Where `cltoken` is a client login token that you must pass to all other API calls in
`Authorization` header in form `GoogleLogin auth=cltoken`.

`cltoken` currently expires in 14 days. If you get `401 Unauthorized` reply from any other request earlier, you need to relogin with existing credentials. Show user the login dialog with ability to enter new credentials only if relogin has failed.

You can test most API calls right in your browser when you signed in BazQux Reader.

### Login error details

This is an extension to Google Reader API. Since BazQux Reader is a commercial app login can fail even with correct login & password if free trial or subscription has expired. This information is passed in `X-BQ-LoginErrorReason` HTTP response header. There are two possible values `YearSubscriptionExpired` and `FreeTrialExpired`. And there are two test accounts for it:

```
curl -v https://bazqux.com/accounts/ClientLogin -d Email=se -d Passwd=1
...
< X-BQ-LoginErrorReason: YearSubscriptionExpired
...

curl -v https://bazqux.com/accounts/ClientLogin -d Email=fte -d Passwd=1
...
< X-BQ-LoginErrorReason: FreeTrialExpired
...
```

I strongly suggest you to check this header and display corresponding error messages "Login failed: Your subscription has expired" or "Login failed: your free trial has expired". This can greatly reduce a number of "why I can't login today?" support requests.

### Ping

https://bazqux.com/reader/ping

```
> curl https://bazqux.com/reader/ping
Unauthorized

> curl https://bazqux.com/reader/ping -H "Authorization: GoogleLogin auth=cltoken"
OK
```

### Token

https://bazqux.com/reader/api/0/token

It's not used currently but may be used later the same way Google Reader uses it
(expires in 30 minutes, with "x-reader-google-bad-token: true" header set).

```
> curl https://bazqux.com/reader/api/0/token -H "Authorization: GoogleLogin auth=cltoken"
Token123
```

### Directory search

https://bazqux.com/reader/directory/search

```
> curl https://bazqux.com/reader/directory/search?q=foo -H "Authorization: GoogleLogin auth=cltoken"
Search is not yet supported
```

### User info

https://bazqux.com/reader/api/0/user-info

```
> curl https://bazqux.com/reader/api/0/user-info -H "Authorization: GoogleLogin auth=cltoken"
{"userId":"01234567890123456789","userName":"realuser","userProfileId":"01234567890123456789","userEmail":"realuser",
"isBloggerUser":true,"signupTimeSec":1234567890,"isMultiLoginEnabled":true}
```

BazQux Reader uses dummy `01234567890123456789` user id for all users and accept any user in Google Reader labels
or states `user/Abracadabra/label/MyFolder` = `user/01234567890123456789/label/MyFolder` = `user/-/label/MyFolder`.

### Preferences list

https://bazqux.com/reader/api/0/preference/list ([?output=json](https://bazqux.com/reader/api/0/preference/list?output=json))

The only preference is alphabetical sorting of subscriptions
(custom ordering is not yet available on website).

### Friend list

https://bazqux.com/reader/api/0/friend/list ([?output=json](https://bazqux.com/reader/api/0/friend/list?output=json))

Empty list for compatibility.

### Stream preferences list

https://bazqux.com/reader/api/0/preference/stream/list ([?output=json](https://bazqux.com/reader/api/0/preference/stream/list?output=json))

Contains information about sorting (alphabetical) and expanded/collapsed state of folders.
`sortId` is just a number of a feed or folder for current user. The same number is also first half of
[ItemId](#about-item-ids) so beware that ItemIds are not unique between users
while unique for single user.

### Set stream preferences

https://bazqux.com/reader/api/0/preference/stream/set

You may only set `k=subscription-ordering&s=...&v=...`. Other parameters are ignored.

### Tag list

https://bazqux.com/reader/api/0/tag/list ([?output=json](https://bazqux.com/reader/api/0/tag/list?output=json))

Only contains list of folders, tags, smart streams and some google specific feeds.

### Subscriptions list

https://bazqux.com/reader/api/0/subscription/list ([?output=json](https://bazqux.com/reader/api/0/subscription/list?output=json))

Always contain `htmlUrl` not depending on favicons setting. `firstitemmsec` is always dummy `1234567890000` (it seems
that no one use it).

### Subscriptions OPML

https://bazqux.com/reader/subscriptions/export

### Adding subscription

https://bazqux.com/reader/api/0/subscription/quickadd?quickadd=xkcd.com
([?output=xml](https://bazqux.com/reader/api/0/subscription/quickadd?quickadd=xkcd.com&output=xml))

BazQux Reader currently doesn't differ GET/POST and query string or form parameters. But you must use POST and append T=token
to be future proof (and to be compatible with Google Reader API).

### Editing subscription

https://bazqux.com/reader/api/0/subscription/edit

[...?ac=subscribe&s=feed/xkcd.com&t=XKCD](https://bazqux.com/reader/api/0/subscription/edit?ac=subscribe&s=feed/xkcd.com&t=XKCD)

[...?ac=edit&s=feed/http://xkcd.com/atom.xml&t=NewTitle&a=user/-/label/Comics&r=user/01234567890123456789/label/Foo](https://bazqux.com/reader/api/0/subscription/edit?ac=edit&s=feed/http://xkcd.com/atom.xml&t=NewTitle&a=user/-/label/Comics&r=user/01234567890123456789/label/Foo)

[...?ac=unsubscribe&s=feed/http://xkcd.com/atom.xml](https://bazqux.com/reader/api/0/subscription/edit?ac=unsubscribe&s=feed/http://xkcd.com/atom.xml)

You can put one subscription in many folders.

### Unread count

https://bazqux.com/reader/api/0/unread-count ([?output=json](https://bazqux.com/reader/api/0/unread-count?output=json))

### Item ids

https://bazqux.com/reader/api/0/stream/items/ids ([?output=json](https://bazqux.com/reader/api/0/stream/items/ids?output=json))

`s=user/-/state/com.google/reading-list`

`s=user/-/state/com.google/starred`

`s=user/-/state/com.google/read`

`s=user/-/state/com.google/broadcast` - empty results

`s=user/-/state/com.google/created` - empty results

`s=user/-/label/...` - folder, tag or smart stream

`s=feed/...`

`r=o` - oldest first ranking.

`xt=...` - everything possible in `s=`.

`it=...` - only messages with specific tags (starred items in feed). NB: It's an extension to GR API.

`ck=...` - ignored.

`ot=...` - minimum download time in seconds. It's time after which item appeared in reader, not the published time. Please don't add it when you get unread items list (there are no 30 days limit on a number of unread items). Internally 180 seconds are subtracted from `ot` to take caching into account (since previous call may not return last few items due to caching). So you can get few items before `ot`. Please, [do not use `ot`](#warning). 

`nt=...` - maximum download time in seconds.

`ts=...` - maximum published time in μs (1e-6 seconds). This could be used in [mark-all-as-read](#marking-all-as-read) call to implement "Mark older than N days" feature.

`c=...` - continuation from previous request (it's just an item id and hence never expire).

`n=...` - number of items. Default is 20, maximum is 50000.

`includeAllDirectStreamIds=true` - add source feed, tags and smart streams for each item.

Note that item ids are unique for one user but can overlap between users. Please use separate database for each account.

### Fetching individual items

https://bazqux.com/reader/api/0/stream/items/contents ([?output=atom](https://bazqux.com/reader/api/0/stream/items/contents?output=atom))

Pass all your [item ids](#about-item-ids) in `i=` parameters (like `i=item_id2&i=item_id_2&...&i=item_id_N`) in HTTP POST request. Although it's possible to add them to URL I recommend POST to not bump into URL length limit.

No more than 1000 items at once.

I suggest to fetch 50-100 items per call on mobile and 100-250 items per call on desktop. Fetching 1000 items at once may be a bit faster but requires more memory on mobile and syncing progress is less visible.

### About item ids

Item ids are 64 bit signed integers. To be compatible with Google Reader they are sometimes represented in long form:

`tag:google.com,2005:reader/item/<unsigned zero-padded 64 bit hexadecimal number>`

For example

`tag:google.com,2005:reader/item/000088960000047a` = `150177826473082`

`tag:google.com,2005:reader/item/80484b00000e8003` = `-9203023375158575101`

All API calls accept both item ids formats. Note that `/stream/items/ids` return ids in short form but `/stream/items/contents` in long form. You can safely convert them to 64 bit signed integer.

More here https://raw.githubusercontent.com/mihaip/google-reader-api/master/wiki/ItemId.wiki

### Fetching streams

https://bazqux.com/reader/api/0/stream/contents ([?output=atom](https://bazqux.com/reader/api/0/stream/contents?output=atom))

https://bazqux.com/reader/atom (= /stream/contents?output=atom)

The same options as in `stream/items/ids` are supported. When no subscription given defaults to reading-list.

`n=1000` maximum.

### Tagging items

https://bazqux.com/reader/api/0/edit-tag

`i=` - list of [item ids](#about-item-ids) (as in [/stream/items/contents](#fetching-individual-items)).

`a=` - add tag.

`r=` - remove tag.

Tags are: `user/-/state/com.google/read` for marking read/unread, `user/-/state/com.google/starred` for starring
and `user/-/label/...` for tagging.

No more than 10000 items to tag at once.

### Marking all as read

https://bazqux.com/reader/api/0/mark-all-as-read

`s=` - feed you want to mark as read (you could use all parameters that [/stream/items/ids](#item-ids) accepts).

No more than 50000 items are marked at once.

### Folder/tag/smart stream renaming

https://bazqux.com/reader/api/0/rename-tag?s=user/-/label/Comics&dest=user/-/label/NiceComics

### Folder/tag/smart stream removing

https://bazqux.com/reader/api/0/disable-tag?s=user/-/label/NiceComics

Feeds are not removed, only folder tag is (like in Google Reader).

### Subscriptions import

NB: This method is an extension to GR API.

https://bazqux.com/reader/api/0/import/opml

Post OPML data in `opml` parameter (in usual `application/x-www-form-urlencoded` form). It will return percent of feeds currently processed (integer number in plain text).

You can then poll (each 3 seconds for example)

https://bazqux.com/reader/api/0/import/percent-complete

till it return "100".

### Links

Description of API from Mihai Parparita (Google Reader project manager), I suggest you to look here first when you need details about API calls:

https://code.google.com/p/google-reader-api/w/list

Docs from Nick Bradbury (NetNewsWire developer):

http://inessential.com/2013/03/14/google_reader_api_documentation

One more descrption:

https://code.google.com/p/pyrfeed/wiki/GoogleReaderAPI
