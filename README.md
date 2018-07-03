# Simple OAuth2 StackExchange

This library is a wrapper around [Simple OAuth2 Library](https://github.com/lelylan/simple-oauth2)

Specially made for [Authorization Code Flow](https://tools.ietf.org/html/draft-ietf-oauth-v2-31#section-4.1) with StackExchange.

## Requirements

Latest Node 8 LTS or newer versions.

## Getting started

```
npm install --save simple-oauth2 simple-oauth2-stack-exchange
```

or 

```
yarn add simple-oauth2 simple-oauth2-stack-exchange
```

### Usage

```js
const simpleOAuth2StackExchange = require('simple-oauth2-stack-exchange');
const stackExchange = simpleOAuth2StackExchange.create(options);
```

`stackExchange` object exposes 3 keys:
* authorize: Middleware to request user's authorization.
* getToken: Middleware for callback processing and exchange the authorization token for an `access_token`
* oauth2: The underlying [simple-oauth2](https://github.com/lelylan/simple-oauth2) instance.

### Options

**Required options**

| Option       | Description                                                                                |
|--------------|--------------------------------------------------------------------------------------------|
| clientId     | Your App Id.                                                                               |
| clientSecret | Your App Secret Id.                                                                        |
| callbackURL  | Callback configured when you created the app.                                              |


**Other options**

| Option           | Default                      | Description                                                                                                                                                                               |
|------------------|------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| scope            | []                           | https://api.stackexchange.com/docs/authentication                                                                                                                                         |
| state            | ''                           | Your CSRF anti-forgery token. More at: https://auth0.com/docs/protocols/oauth2/oauth-state                                                                                                |
| returnError      | false                        | When is false (default), will call the next middleware with the error object. When is true, will set req.tokenError to the error, and call the next middleware as if there were no error. |
| authorizeHost    | 'https://stackoverflow.com'  |                                                                                                                                                                                           |
| authorizePath    | '/oauth'                     |                                                                                                                                                                                           |
| tokenHost        | 'https://stackoverflow.com'  |                                                                                                                                                                                           |
| tokenPath        | '/oauth/access_token'        |                                                                                                                                                                                           |
| authorizeOptions | {}                           | Pass extra parameters when requesting authorization.                                                                                                                                      |
| tokenOptions     | {}                           | Pass extra parameters when requesting access_token.                                                                                                                                       |


## Example

### Original boilerplate

```js
const oauth2 = require('simple-oauth2').create({
  client: {
    id: process.env.STACK_EXCHANGE_APP_ID,
    secret: process.env.STACK_EXCHANGE_APP_SECRET
  },
  auth: {
    authorizeHost: 'https://stackoverflow.com'
    authorizePath: '/oauth',

    tokenHost: 'https://stackoverflow.com',
    tokenPath: '/oauth/access_token'
  }
});

router.get('/auth/stack-exchange', (req, res) => {
  const authorizationUri = oauth2.authorizationCode.authorizeURL({
    redirect_uri: 'http://localhost:3000/auth/stack-exchange/callback',
  });

  res.redirect(authorizationUri);
});

router.get('/auth/stack-exchange/callback', async(req, res) => {
  const code = req.query.code;
  const options = {
    code,
    redirect_uri: 'http://localhost:3000/auth/stack-exchange/callback'
  };

  try {
    // The resulting token.
    const result = await oauth2.authorizationCode.getToken(options);

    // Exchange for the access token.
    const token = oauth2.accessToken.create(result);

    return res.status(200).json(token);
  } catch (error) {
    console.error('Access Token Error', error.message);
    return res.status(500).json('Authentication failed');
  }
});
```

### With SimpleOAuth2StackExchange

```js
const simpleOAuth2StackExchange = require('simple-oauth2-stack-exchange');

const stackExchange = simpleOAuth2StackExchange.create({
  clientId: process.env.STACK_EXCHANGE_APP_ID,
  clientSecret: process.env.STACK_EXCHANGE_APP_SECRET,
  callbackURL: 'http://localhost:3000/auth/stack-exchange/callback'
});

// Ask the user to authorize.
router.get('/auth/stack-exchange', stackExchange.authorize);

// Exchange the token for the access token.
router.get('/auth/stack-exchange/callback', stackExchange.accessToken, (req, res) => {
  return res.status(200).json(req.token);
});
```
