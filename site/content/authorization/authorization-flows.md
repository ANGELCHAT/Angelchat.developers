---
weight: 20
---

# App authorization flows
All apps integrated with chat.io must be first created in [chat.io Developers Console](https://console.chat.io).

**Note**: If you want to use [Customer API](../customer-api) you will need an additional authorization flow. Check the [Customer Authorization](#customer-authorization) section.

When a user starts using your app, they will see what parts of his account your app will have access to. When they grant the access, your app will receive an `access_token` that will let you authorize [Agent API](../agent-api) or [Configuration API](../configuration-api) calls.


There are a few scenarios in which you can acquire an `access_token`:

* [Sign in with chat.io](#sign-in-with-chat-io) – the easiest way to get access to any chat.io account.
* [Public web apps (coming soon)](#public-web-apps) – JavaScript apps available to all chat.io customers.
* [Private web apps](#private-web-apps) – JavaScript apps available to agents from single chat.io account.
* [Private server-side apps](#server-side-apps) – backend apps installed on a single chat.io account.
* Public server-side apps (coming soon) – backend apps available to all chat.io customers.

## Sign in with chat.io
"Sign in with chat.io" button is the easiest way to collect `access_token` from a chat.io user.

<!-- screenshot -->
In this scenario, chat.io user enters your website with a "Sign in with chat.io" button installed. After clicking the button, they enter chat.io login and password in a pop-up window and grant access to some parts of their account.

In return, you acquire an `access_token` which can be used to call [Agent API](/customer-api) methods.

Read more how to implement this flow in a dedicated ["Sign in with chat.io"](../sign-in-with-chatio) article.

## Public web apps

**Coming soon**

Public web apps are JavaScript applications that can access any chat.io customer account.

To set up your own public app, you must define the URL of the app and the list of scopes – parts of chat.io account your app will have access to. chat.io customer who enters your app URL is be asked to enter their login and password and grant access for your app.

Then, the user is redirected to your app with `access_token` included in the URL.

<!-- screenshot -->
### 1. Create the app
Go to [Developers Console](https://console.chat.io) and create a new "chat.io OAuth 2.0 Client" app. **Redirect URI** is the address of your app that will receive `access_token` in a URL. **Scopes** is a list of permissions your app will get.

### 2. Redirect to chat.io OAuth Server

> Example redirection to chat.io OAuth Server:

```
https://accounts.chat.io/
  ?response_type=token
  &client_id=2261a58dfe1420acc0dc1bd77158f7ac
  &redirect_uri=https%3A%2F%2Fmy-application.com
  &state=i8XNjC4b8KVok4uw5RftR38Wgp2BFwql
```

When a user runs your app, you should redirect their browser to the following URL:

`https://accounts.chat.io/`

with the following URL params:

* **response_type=token**
* **client_id** – you received it when you created the app in [Developers Console](https://console.chat.io).
* **redirect_uri** – URL of your web application that chat.io OAuth Server will redirect the user back after successful authorization. It must be one of the URLs that you entered when creating the app in the previous step.
* **state** – you can provide here any value that might be useful to your application. It is strongly recommended to include an anti-forgery token here mitigate the [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery).

### 3. Acquire the access token

> Example redirection back to your app:

```
https://my-application.com/
  #access_token=1/fFBGRNJru1FQd44AzqT3Zg
  &token_type=Bearer
  &expires_in=1209600
  &state=i8XNjC4b8KVok4uw5RftR38Wgp2BFwql
```

After successful authorization, the user is redirected back to your app. The URL will include a number of params in the hash section of the URL:

* **access_token** – token you can use to call [REST API](/rest-api) methods on behalf of the user.
* **expires_in** – number of seconds the `access_token` will be valid. When it expires, you will need to repeat the authorization process to get the new `access_token`.
* **state** – value of the `state` param that you passed in redirection to chat.io OAuth Server.
* **token_type=Bearer**

Your application should remember `access_token` in localStorage or a cookie until it expires. Caching the token prevents you from redirecting the user to chat.io OAuth Server every time he visits your app.

<!--### Example app

> This example private web application fetches chat.io agents list and logs it in the console.

> `index.html` file:

```html
<!DOCTYPE html>
<html>
<body>
<script src="//ajax.googleapis.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>

<script>
function getHashParam(key) {
  var params = location.hash.substring(1).split('&');
  var value = params.find(function(item) {
		return item.split('=')[0] === key;
	});
  return value ? value.split('=')[1] : '';
}

var clientId = '<client_id>';
var redirectUri = 'http://localhost/test.html';
var accessToken = getHashParam('access_token');

if (accessToken) {
  $.ajax({
    url: './get_agents.php',
    dataType: 'json',
    data: {
      access_token: accessToken
    },
    success: (agents) => {
      console.log(agents);
    }
  });
} else {
	location.href = 'https://accounts.chat.io/' +
      '?response_type=token' +
      '&client_id=' + clientId +
      '&redirect_uri=' + redirectUri;
}
</script>
</body>
</html>
```

> `get_agents.php` file:

```php
<?php
$accessToken = $_GET['access_token'];

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://api.livechatinc.com/agents');
curl_setopt($ch, CURLOPT_HTTPHEADER, array(
	'Content-Type: application/json',
	'X-API-Version: 2',
	'Authorization: Bearer ' . $accessToken
));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);

$result = curl_exec($ch);
curl_close($ch);

echo $result;
```


This example app fetches [Agents list](/rest-api#list-all-agents) and returns it in the browser's console.

You should update `clientId` and `redirectUri` params from the example to match actual values. You can set up these params when creating the app in [chat.io Developers Console](https://console.chat.io). Your app must also have access to **read agents list** scopes. Scopes are also configured in the Developers Console.

Please note that the actual REST API request is performed by the backend script (in this case, PHP) because chat.io REST API does not allow cross-domain requests.

-->

## Private web apps

Private web apps are JavaScript applications that are available only to agents from a single chat.io account. If you want to build an internal app for your chat agents only, this is a good way to go.

Private web apps work the very same way like [public web apps](#public-web-apps). Please refer to that documentation to understand how it works.

To start building a private web app, set it up [chat.io Developers Console](https://console.chat.io).

## Server-side apps

Server-side apps are applications that have access to user's data for unlimited time. Private and public server-side apps have the same authorization flows. Public server-side apps are not available yet.

When your application wants to acquire the `access_token`, you must redirect the user to chat.io OAuth Server only once. After successful authorization, the user is redirected back to your app along with a single-use authorization code.

Your application exchanges the authorization code for an `access_token` and `refresh_token`. From now now, you can generate new `access_tokens` indefinitely without any action required from the user.

<!-- screenshot -->
### 1. Create the app

Go to [Developers Console](https://console.chat.io) to create a new server-side app. **Redirect URI** is the address of your app that will receive authorization `code` in a URL. **Scopes** is a list of permissions your app will get.

### 2. Redirect o chat.io OAuth Server

> Example redirection to chat.io OAuth Server:

```
https://accounts.chat.io/
  ?response_type=code
  &client_id=86pp8cqeg2ac5fimbs8gibluu16ugyvs
  &redirect_uri=https%3A%2F%2Fmy-application.com
  &state=f3NtEuZ5AuxsmnVAzcyLGm17aAaltJTv
```

Start with redirecting your user to the following address: `https://accounts.chat.io/`

Required URL parameters:

* **response_type=code**
* **client_id** – you received it when you created the app in [Developers Console](https://console.chat.io).
* **redirect_uri** – URL of your web application that chat.io OAuth Server will redirect the user back after successful authorization. It must be one of the URLs that you entered when creating the app in the previous step.

Optional URL parameters:

* **state** – provide any state that might be useful to your application. It will be included in the redirection to `redirect_uri` endpoint.<br>It is strongly recommended to include an anti-forgery token in the state to mitigate the [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery).

### 3. Acquire the code

> Example redirection back to your app:

```
https://my-application.com/
  ?code=4/P7q7W91a-oMsCeLvIaQm6bTrgtp7
  &state=f3NtEuZ5AuxsmnVAzcyLGm17aAaltJTv
```

When the user approves the access request, they are redirected back to your app. The URL will include `code` param which should be used to acquire `access_token` and `refresh_token`.

If the user does not approve the request, chat.io OAuth Server will not redirect the user to your application.

### 4. Exchange code for access token and refresh token

> Example request:

```shell
curl "https://accounts.chat.io/token" \
  -X POST \
  -d "grant_type=authorization_code&\
  code=4/P7q7W91a-oMsCeLvIaQm6bTrgtp7&\
  client_id=86pp8cqeg2ac5fimbs8gibluu16ugyvs&\
  client_secret=nBdMN8d7MEp1YYo3&\
  redirect_uri=https://my-application.com"
```

To exchange the `code` for an `access_token` and `refresh_token`, you should perform an HTTP POST request to the following URL:

`https://accounts.chat.io/token`

Required parameters:

* **grant_type=authorization_code**
* **code** – the authorization code returned from the initial request.
* **client_id** – you received it when you created the app in [Developers Console](https://console.chat.io).
* **client_secret** - you received it when you created the app in [Developers Console](https://console.chat.io).
* **redirect_uri** – URL of your web application that chat.io OAuth Server will redirect the user back after successful authorization. It must be one of the URLs that you entered when creating the app in the previous step.

> Example response:

```json
{
  "access_token": "1/fFAGRNJru1FTz70BzhT3Zg",
  "refresh_token": "-/khgiugfalskdbcakg2347o8326",
  "expires_in": 3920,
  "token_type": "Bearer"
}
```

<br>
The response will include the following params in JSON format:

* **access_token** – token you can use to call [REST API](/rest-api) methods on behalf of the user.
* **expires_in** – number of seconds the `access_token` will be valid. When it expires, you will need to generate new `access_token` using `refresh_token` (read [Using the refresh token](#using-the-refresh-token) for more details).
* **refresh_token** – token that can be used to generate new access tokens.
* **token_type=Bearer**

### Using the refresh token

> Example request:

```shell
curl "https://accounts.chat.io/token" \
  -X POST \
  -d "grant_type=refresh_token&\
  refresh_token=-/khgiugfalskdbcakg2347o8326&\
  client_id=86pp8cqeg2ac5fimbs8gibluu16ugyvs&\
  client_secret=nBdMN8d7MEp1YYo3"
```

To obtain a new `access_token`, your application must send an HTTP POST request to the following endpoint:

`https://accounts.chat.io/token`

Required parameters:

* **grant_type=refresh_token**
* **refresh_token** – value of refresh token you received in the previous step.
* **client_id** – you received it when you created the app in [Developers Console](https://console.chat.io).
* **client_secret** - you received it when you created the app in [Developers Console](https://console.chat.io).

> Example response:

```json
{
  "access_token": "1/fFAGRNJru1FTz70BzhT3Zg",
  "refresh_token": "-/khgiugfalskdbcakg2347o8326",
  "expires_in": 3920,
  "token_type": "Bearer"
}
```

<br>
The response will include the following params in JSON format:

* **access_token** – token you can use to call [REST API](/rest-api) methods on behalf of the user.
* **expires_in** – number of seconds the `access_token` will be valid. When it expires, you will need to generate new `access_token` using `refresh_token` (read [Using the refresh token](#using-the-refresh-token) for more details).
* **refresh_token** – token that can be used to generate new access tokens.
* **token_type=Bearer**

### Revoking tokens

> Revoking a token:

```shell
curl "https://accounts.chat.io/token\
  ?token=<access_token|refresh_token>"
  -X DELETE
```

In some cases a user may wish to revoke access given to an application. The token can be an access token or a refresh token. If the token is an access token and it has a corresponding refresh token, the refresh token will also be revoked.

### Validating the access token

> Validating the `access_token`:

```shell
curl "https://accounts.chat.io/info"
  -H "Authorization: Bearer <access_token>"
```

You can validate an `access_token` by making an HTTP request to the following endpoint:

`https://accounts.chat.io/info`

Please note that refresh tokens are not supported for direct validation. If `access_token` was obtained with with the use of a `refresh_token`, response will return both tokens.

> Example response:

```json
{
  "access_token": "hgDBq88bSw-UHBXnm1Q_Bw",
  "expires_in": 3564,
  "refresh_token": "wB_Q1mnXBHU-wSb88qBDgh", // optional
  "client_id": "14c26ac43f2123dcf8a76ff58b319930",
  "scope": "email,manage_accounts",
  "token_type": "Bearer",
  "entity_id": "name@domain.com",
  "license_id": 1520
}
```

<br>
The response will include the following params in JSON format:

* **access_token** – token you can use to call [REST API](/rest-api) methods on behalf of the user.
* **expires_in** – number of seconds the `access_token` will be valid. When it expires, you will need to generate new `access_token` using `refresh_token` (read [Using the refresh token](#using-the-refresh-token) for more details).
* **refresh_token** – token that can be used to generate new access tokens.
* **client_id** – you received it when you created the app in [Developers Console](https://console.chat.io).
* **scope** – comma-separated list of permissions that `access_token` has access to.
* **token_type=Bearer**
* **entity_id** – chat.io's user login.
* **license_id** – chat.io's user account number.

## Notes

### Limitations

There is currently a limit of 25 refresh tokens per client per user. When limit is reached, the oldest token is automaticaly revoked (with rabbitmq publishing). 

Another limitation is 3 redirects in 30 seconds to chat.io OAuth 2.0 server per client per user. When limit is reached, server redirects to error page with `too_many_redirects` error detail.

### Redirect URI considerations

Client configuration allows adding many redirect URIs. The redirect URIs are separated by comma. Authorization request redirect URI is valid when matches one of the client's configuration URIs.

URI is composed of several parts:

* scheme (`http://`, `https://`) - is required and must match exactly,
* host (`google.pl`, `localhost:3000`, ...) - hostname or ip and optional port, is required and must match exactly,
* path (`/info`, `/test`, ...) - is optional, client redirect URI path must be a substring of authorization request redirect path, path traversals are forbidden,
* query (`?size=20`, ...) - is forbidden,
* fragment (`#paragraph`) - is forbidden.
