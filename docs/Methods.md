# Methods

These are all methods available in the vue app via `$auth`.

* All `success` and `error` functions will receive proper context from currently called component.

### ready

* Fires once on the initial app load to pre-load users (if set).

~~~
<div v-if="$auth.ready()">
    <vue-router></vue-router>
</div>
<div v-if="!$auth.ready()">
    Site loading...
</div>
~~~

### redirect

* Returns either an object if a redirect occurred or `null`.
* The object is in the form `{type: <string>, from: <object>, to: <object>}` Where `type` is one of `401`, `403`, `404` and the `from` and `to` objects are just copies of the route transitions.

~~~
var redirect = this.$auth.redirect();

this.$auth.login({
    redirect: {name: redirect ? redirect.from.name : 'account'},
});
~~~

### user

* Returns the currently stored users data.
* Update the current user by passing in an object.

~~~
<div>
    {{ $auth.user().email }}
</div>
~~~

~~~
this.$auth.user(userObject);
~~~


### check

* Check to see if the user is logged in.
* It also accepts arguments to check for a specific role or set of roles.

~~~
<a v-if="!$auth.check()" v-link="'/login'">login</a>
<a v-if="$auth.check('admin')">admin</a>
<a v-if="$auth.check(['admin', 'manager')]">manage</a>
<a v-if="$auth.check()" v-on:click="$auth.logout()">logout</a>
~~~

### other

* Checks if secondary "other" user is logged in.

~~~
<a v-if="$auth.other()" v-on:click="logoutOther()">logout other</a>
~~~

### disableOther

* Disables other using the default token until it is re-enabled (or logged out).
* This allows you to login is as "another" user but still perform requests as an admin.

~~~
this.$auth.disableOther();

this.$http.get('users'); // Will run as admin.

this.$auth.enableOther();
~~~

### enableOther

* See disableOther.

### token

* Returns the currently activated token if none are specified.
* Can also specify a specific token, but only `other` and `default` will actually return anything unless setting your own token.
* Can set a token with optional second argument, set null to use default naming conventions.

~~~
var token = this.$auth.token();
var token = this.$auth.token('other');
var token = this.$auth.token('default');

this.$auth.token(null, token);
this.$auth.token('test', token);

var token = this.$auth.token('test');
~~~


### fetch

* Fetch the user (again) allowing the users data to be reset (from the api).
* Data object is passed directly to http method.

~~~
this.$auth.fetch({
    params: {},
    success: function () {},
    error: function () {},
    // etc...
});
~~~

### refresh

* Manually refresh the token.
* The refresh will always fire on boot, to disable this override the `expiredToken` option method.
* Can be used in conjunction with `expiredToken` and `token` to write custom refreshes.
* If any custom expiration custom logic needs to be done (for instance decoding and checking expiration date in jwt token) override the `expiredToken` method and return `boolean`.
* Data object is passed directly to http method.

~~~
this.$auth.refresh({
    params: {}, // data: {} in axios
    success: function () {},
    error: function () {},
    // etc...
});
~~~

### register

* Convenience method for registration.
* Data object is passed directly to http method.
* Accepts `autoLogin` parameter to attempt login directly after register.
* Accepts `rememberMe` parameter when used in conjunction with `autoLogin` equal to `true`.
* Accepts `redirect` parameter which is passed directly to router.

~~~
this.$auth.register({
    params: {}, // data: {} in axios
    success: function () {},
    error: function () {},
    autoLogin: true,
    rememberMe: true,
    redirect: {name: 'account'},
    // etc...
});
~~~

### login

* Data object is passed directly to http method.
* Accepts `rememberMe` parameter.
* Accepts `redirect` parameter which is passed directly to router.
* Accepts `fetchUser` param which allows disabling fetching user after login.

~~~
this.$auth.login({
    params: {}, // data: {} in axios
    success: function () {},
    error: function () {},
    rememberMe: true,
    redirect: '/account',
    fetchUser: true
    // etc...
});
~~~

### logout

* Data object is passed directly to http method.
* Accepts `redirect` parameter which is passed directly to router.
* Accepts `makeRequest` parameter which must be set to `true` to send request to api. Otherwise the logout just happens locally by deleting tokens.

~~~
this.$auth.logout({
    makeRequest: true,
    params: {}, // data: {} in axios
    success: function () {},
    error: function () {},
    redirect: '/login',
    // etc...
});
~~~

### loginOther

* Data object is passed directly to http method.
* Accepts `redirect` parameter which is passed directly to router.

~~~
this.$auth.loginOther({
    params: {}, // data: {} in axios
    success: function () {},
    error: function () {},
    redirect: {name: 'account'},
    // etc...
});
~~~

### logoutOther

* Data object is passed directly to http method.
* Accepts `redirect` parameter which is passed directly to router.
* Also accepts `makeRequest` parameter same as `logout` method.

~~~
this.$auth.logoutOther({
    makeRequest: true,
    params: {}, // data: {} in axios
    success: function () {},
    error: function () {},
    redirect: {path: '/admin'},
    // etc...
});
~~~

### oauth2

* Convenience method for OAuth2.
* Initial request is to third party.
* Second call is to api server.
* Accepts `code` parameter which should be set to `true` when the code is set.
* Accepts `provider` parameter which hooks into data for third party.
* Third party data should follow format such as `facebookData`, `facebookOath2Data`. Check options section for more info.
* Accepts `redirect` parameter which is passed directly to router.

~~~
if (this.$route.query.code) {
    this.$auth.oauth2({
        code: true,
        provider: 'facebook',
        params: { // data: {} in axios
            code: this.code
        },
        success: function(res) {},
        error: function (res) {}
        redirect: {path: '/account'},
        // etc
    });
}
else {
    this.$auth.oauth2({
        provider: 'facebook'
    });
}
~~~