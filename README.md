BazQux Reader API
==========

BazQux Reader clones Google Reader API. 

The only thing you need to support BazQux Reader is to change
google.com to bazqux.com in your code.

To set login/password you need to sign into (BazQux Reader)[https://bazqux.com]
and go to Options (top-right corner) => Mobile login.

Main API endpoints:
    https://www.bazqux.com/accounts/ClientLogin
    https://www.bazqux.com/reader/api/0
    https://www.bazqux.com/reader/atom

Most of the API is supported. The only two things missing are
starring and tagging (API calls just returns OK but no starred items
or tags are created).

Getting lists of all/unread items/ids in json/atom formats,
marking items read/unread, adding/removing/renaming of
subscriptions and folders -- everything is supported.
