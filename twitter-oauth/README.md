Twitter OAuth with Revel 
==================================

The `twitter-oauth` sample application uses 
the [`mrjones/oauth`](https://github.com/mrjones/oauth) library to demonstrate:

* How to do the [`oauth`](http://oauth.net/) dance to authenticate an app to a `Twitter` account.
* Fetching mentions for that Twitter account
* Tweeting on behalf of that Twitter account

The core contents of the app are in the files
* `twitter-oauth/app/`
		* `models/`
			* [`user.go`](/twitter-oauth/app/models/user.go)   - User struct and in-memory data store
		* `controllers/`
			* [`app.go`](/twitter-oauth/app/models/app.go)    - The logic


OAuth Overview
===================================


An overview of the process:

1. The app generates a `request token` and sends the user to Twitter.
2. The `user` authorizes the app ie login with twitter
3. Twitter redirects the user to the provided `redirect url`, including a
   `verifier` in the parameters.
4. The app constructs a request to Twitter using the `request token` and
   the `verifier`, to which Twitter returns the `access token`.
5. The app henceforth uses the access token to operate Twitter on the user's behalf.



The `OAuth` process is governed by this configuration, which are twitter Api Keys..:

```go
var TWITTER = oauth.NewConsumer(
	"VgRrevelRunFooBarTruingindreamzw",
	"l8lOLyIF3peCFEvrEoTc8h4oFwieAFgPM6eegibberish",
	oauth.ServiceProvider{
		AuthorizeTokenUrl: "https://api.twitter.com/oauth/authorize",
		RequestTokenUrl:   "https://api.twitter.com/oauth/request_token",
		AccessTokenUrl:    "https://api.twitter.com/oauth/access_token",
	},
)
```


