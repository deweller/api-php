# elvanto API PHP Library

This library is all set to go with version 1 of the <a href="http://elvanto.com/api/" target="_blank">elvanto API</a>.

## Authenticating

The elvanto API supports authentication using either <a href="http://elvanto.com/api/getting-started/#oauth" target="_blank">OAuth 2</a> or an <a href="http://elvanto.com/api/getting-started/#api_key" target="_blank">API key</a>.

### Using OAuth 2

This library provides functionality to help you obtain an Access Token and Refresh token. The first thing your application should do is redirect your user to the elvanto authorization URL where they will have the opportunity to approve your application to access their elvanto account. You can get this authorization URL by using the `authorize_url()` method, like so:

```php
require_once('elvanto_API.php');

$elvanto = new elvanto_API();

$authorize_url = $elvanto->authorize_url(
	'Client ID for your application',
    'Redirect URI for your application',
    'The permission level your application requires',
    'Optional state data to be included'
);
// Redirect your users to $authorize_url.
```

If your user approves your application, they will then be redirected to the `redirect_uri` you specified, which will include a `code` parameter, and optionally a `state` parameter in the query string. Your application should implement a handler which can exchange the code passed to it for an access token, using `exchange_token()` like so:

```php
require_once('elvanto_API.php');

$elvanto = new elvanto_API();

$result = $elvanto->exchange_token(
    'Client ID for your application',
    'Client Secret for your application',
    'Redirect URI for your application',
    'A unique code for your user' // Get the code parameter from the query string.
);

$access_token = $result->access_token;
$expires_in = $result->expires_in;
$refresh_token = $result->refresh_token;
// Save $access_token, $expires_in and $refresh_token.
```

At this point you have an access token and refresh token for your user which you should store somewhere convenient so that your application can look up these values when your user wants to make future elvanto API calls.

Once you have an access token and refresh token for your user, you can authenticate and make further API calls like so:

```php
require_once('elvanto_API.php');

$auth_details = array(
    'access_token' => 'your access token',
    'refresh_token' => 'your refresh token'
);
$elvanto = new elvanto_API($auth_details);

$results = $elvanto->call('people/getAll');
var_dump($results);
```

All OAuth tokens have an expiry time, and can be renewed with a corresponding refresh token. If your access token expires when attempting to make an API call, you will receive an error response, so your code should handle this. Here's an example of how you could do this:

```php
require_once('elvanto_API.php');

$auth_details = array(
    'access_token' => 'your access token',
    'refresh_token' => 'your refresh token'
);
$elvanto = new elvanto_API($auth_details);

$results = $elvanto->call('people/getAll');
if (isset($results->error)) {
    // If you receive '121: Expired OAuth Token', refresh the access token.
    if ($results->error->code == 121) {
        list($new_access_token, $new_expires_in, $new_refresh_token) =
            $elvanto->refresh_token();
        // Save $new_access_token, $new_expires_in, and $new_refresh_token.
    }
    // Make the call again.
    $results = $elvanto->call('people/getAll');
}
var_dump($results);
```

### Using an API key

```php
require_once('elvanto_API.php');

$auth_details = array('api_key' => 'your API Key');
$elvanto = new elvanto_API($auth_details);

$results = $elvanto->call('people/getAll');
var_dump($results);
```

## Documentation

Documentation can be found on the <a href="http://elvanto.com/api/" target="_blank">elvanto API website</a>.

## Updates

Follow our <a href="http://twitter.com/elvantoAPI" target="_blank">Twitter</a> to keep up-to-date with changes in the API.

## Support

If you come across any bugs, or have any suggestions, please <a href="https://elvanto.com/forums/forum/3/api-developers/" target="_blank">post in the forum</a> or contact us <a href="http://elvanto.com/contact/" target="_blank">via our website</a>.
