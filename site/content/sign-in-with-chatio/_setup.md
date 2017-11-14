# Setup

## 1. Create new app
Create new "LiveChat OAuth 2.0 Client" app in <a href="https://developers.livechatinc.com/console/">Developers Console</a>. You will receive a new `client_id` that we will use in the next steps.

Please note that **Redirect URI** field must match the URL of the website with "Sign in with LiveChat" button installed. The button will not work on any other URL addresses.


## 2. Include SDK library
```html
<script src="//cdn.livechatinc.com/accounts/accounts-sdk.min.js"></script>
```
Create a simple HTML page and include the following JavaScript library.

## 3. Prepare button container

> Native "Sign in with LiveChat" button

```html
<div class="livechat-login-button"></div>
```

> Custom "Sign in with LiveChat" button

```js
// javascript
var instance = AccountsSDK.init({ ... });
```
```html
<!-- html -->
<a href="" onclick="instance.openPopup()">Sign in with LiveChat</a>
```

Insert HTML code to prepare the container for "Sign in with LiveChat" button. Its `class` attribute must have value of `livechat-login-button`. The button will be automatically inserted into the container.

If you prefer to design your own button, you can do that, too. Just bind the [`openPopup()`](#instance-openpopup) method of `AccountsSDK` instance to the `onclick` attribute for your link or button.


## 4. Initialize the SDK

```html
<script>
var instance = AccountsSDK.init({
  client_id: '<your_client_id>',
  onIdentityFetched: (error, data) => {
    if (data) {
      console.log('User authorized!');
      console.log('License number: ' + data.license);
    }
  }
});
</script>
```

Insert the following JavaScript code before the closing `</body>` tag.

That's it!

Your users will see "Sign in with LiveChat" button if they are not yet logged in or they didn't grant access for your app.

If they are already logged in, you will immediately receive valuable information about your user, such as their `access_token` or `license` number.
