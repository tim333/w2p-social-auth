# w2p-social-auth
Automatically exported from code.google.com/p/w2p-social-auth


web2py plugin for python-social-auth

http://drive.google.com/uc?export=view&id=0B2UQ0R99VPSuUU1pVFFSLUI0a0E&nonsense=something_that_ends_with.png

Description

This is a web2py plugin to integrate python-social-auth.

This plugin is a wrapper around the python-social-auth (PSA) library to make it easier to use it in web2py.

This plugin integrates the PSA flow in web2py and provides some basic forms as a starting point.

This is still beta-quality code and actively under development.

Do not rely on it for production code just yet.

Please check regularly for updates.

Requirements

A couple of things are assumed:

You know what a web2py plugin is.
You are familiar with python-social-auth. and it's features and capabilities.
You have read the PSA documentation
the dependencies are the same as for python-social-auth:

python-oauth2
six
requests
requests-oauthlib
python-openid
You need to install those in your app "modules" folder.

Installation

The plugin is wrapped in a simple example web2py app. You can create a new app using the admin interface and then copy over the files in this repo. If you want to include it in a different app you can use the admin interface to pack the plugin and install it in the other app. Or you could copy over the plugin files manually.

The requirements as described above are not included in the app so those still need to be placed in the /modules folder.

You need to edit db.py to adjust it to your situation. (API keys, db connection, other settings)

Configuration

To hook it up to web2py Auth you can use the plugin_social_auth.utils.SocialAuth class or extend from it. This will provide a very basic page with a buttons (or dropdown) for all configured backends, a field for manual OpenID input and a simple interface to manage all associated accounts.

If you need something more fancy you could always extend utils.SocialAuth and override the relevant methods to return custom forms.

from plugin_social_auth.utils import SocialAuth auth = SocialAuth(db)

You need at least these settings for auth:

```

Enable the "username" field

auth.define_tables(username=True)

Disable certain auth actions unless you're also using web2py account registration

auth.settings.actions_disabled = ['register', 'change_password', 'request_reset_password'] ```

Configure the plugin in one of your models. All PSA settings can be configured like this. But at least you need to configure your API keys and which backends to use.

``` plugins = PluginManager()

Configure your API keys

plugins.social_auth.SOCIAL_AUTH_TWITTER_KEY = my_twitter_consumer_key plugins.social_auth.SOCIAL_AUTH_TWITTER_SECRET = my_twitter_consumer_secret plugins.social_auth.SOCIAL_AUTH_FACEBOOK_KEY = my_facebook_app_id plugins.social_auth.SOCIAL_AUTH_FACEBOOK_SECRET = my_facebook_app_secret plugins.social_auth.SOCIAL_AUTH_LIVE_KEY = my_live_client_id plugins.social_auth.SOCIAL_AUTH_LIVE_SECRET = my_live_client_secret

Configure PSA with all required backends

plugins.social_auth.SOCIAL_AUTH_AUTHENTICATION_BACKENDS = ( # You need this one to enable manual input for openid. # It must not be configured in SOCIAL_AUTH_PROVIDERS (below) 'social.backends.open_id.OpenIdAuth',

'social.backends.persona.PersonaAuth',
'social.backends.live.LiveOAuth2',
'social.backends.twitter.TwitterOAuth',
'social.backends.facebook.FacebookOAuth2')
Configure the providers that you want to show in the login form.

:

The backend name is the "name" property in the relevant backend class as configured in SOCIAL_AUTH_AUTHENTICATION_BACKENDS.

The display name is how you want the backend to be represented in the ui (Buttons, dropdown and messages)

plugins.social_auth.SOCIAL_AUTH_PROVIDERS = { 'live': 'Live', 'twitter': 'Twitter', 'facebook': 'Facebook', 'persona': 'Mozilla Persona'}

Configure app index URL. This is where you are redirected after logon when

auth.settings.logout_next is not configured.

If both are not configured there may be no redirection after logout! (returns 'None')

plugins.social_auth.SOCIAL_AUTH_APP_INDEX_URL = URL('init', 'default', 'index')

```

User Interface

w2p-social-auth can be configured to show a dropdown or buttons. 'dropdown' does not require javascript (except for Persona backend) and 'buttons' requires js and jquery to be loaded. Configure like below if you want dropdown. plugins.social_auth.SOCIAL_AUTH_UI_STYLE = 'dropdown'

Configure like below to disable the bootstrap styles for the buttons: plugins.social_auth.SOCIAL_AUTH_UI_BOOTSTRAP = False

The plugin uses Font-Awesome to create the icons for the buttons. But obviously there are much more backends than there are icons in the font. So if you are using backends which do not have an icon available and you don't want to see empty spaces for icons, you can disable the icon in the buttons like this. (Or you could provide your own icons using CSS or something. that's up to you) plugins.social_auth.SOCIAL_AUTH_UI_ICONS = False

Exception handling

The default Exception handler will catch only python-social-auth exceptions, shows a flash message and logs the error. You can define your own handler by extending from plugin_social_auth.utils.W2pExceptionHandler or implement your own.

You can configure it like this:

import sys _path = plugins.social_auth.SOCIAL_AUTH_EXTRA_BACKEND_MODULE_PATH = 'mybackends' sys.modules[_path] = __import__(_path)

User Details

By default PSA updates user fields with new values when the user logs on with an new social account. This behavior can be overridden by configuring the SOCIAL_AUTH_PROTECTED_USER_FIELDS. When you leave this behaviour default it makes sense to make certain fields readonly because they could be overridden anyway when the user logs on with a new account. Fields that you want the user to be able to change, you should configure using SOCIAL_AUTH_PROTECTED_USER_FIELDS

```

Make user props readonly since these will automatically be updated

when the user logs on with a new social account anyway.

for prop in ['first_name', 'last_name', 'username', 'email']: auth.settings.table_user[prop].writable = False ```

Multiple account association

python-social-auth supports associating multiple social accounts with a single user. This works as follows:

When a user is logged-in and logs in (again) with a new social account, this social account is associated with the currently logged-in user. A user can not disconnect the last associated social account.

The plugin shows a basic form to manage account associations using the 'associations' action:

{application}/plugin_social_auth/user/associations

You could also use the functions defined in utils.SocialAuth to create your own forms and maybe include it in a profile page or somewhere else.

New account confirmation

Because of the way PSA handles multiple accounts association (see below), it's IMO very easy for users to unintended create a new account when they meant to associate an existing account with a new social account. If you enable this setting, after the user has logged on to the social account and when it's detected that new account is going to be created but before the new web2py account is actually created and associated, a dialog will be shown to ask the user for confirmation. When the user confirms, the pipeline will continue where it left off.

You can enable it using this setting (default is False):

SOCIAL_AUTH_CONFIRM_NEW_USER

You can customize the message like this:

auth.messages.confirm = <myconfirmmessage>

can be a string or a callable that returns a HTML helper.

Mozilla Persona

to use Mozilla Persona, use this backend:

'social.backends.persona.PersonaAuth'

If you want to use Persona, you need to configure the setting SOCIAL_AUTH_ENABLE_PERSONA (see table below)

Note:

Because persona requires JavaScript to work, when JavaScript is disabled in the browser, the persona item will automatically be hidden from the dropdown menu (in case your using the dropdown ui).

Extra settings

Extra settings that can be configured besides the PSA settings:

NameDefaultDescription SOCIAL_AUTH_EXTRA_BACKEND_MODULE_PATHIf you want to add or extend backends you can define them in your own module and configure the module path: SOCIAL_AUTH_EXCEPTION_HANDLERplugin_social_auth.utils.W2pExceptionHandlerPath to custom exception handler SOCIAL_AUTH_APP_INDEX_URLYour app index page. By default this is where web2py redirects to after logout SOCIAL_AUTH_ENABLE_PERSONAFalseWhen set to "True" the plugin will include the required javascript in the view SOCIAL_AUTH_CONFIRM_NEW_USERFalseShow a confirmation form when a new account will be created SOCIAL_AUTH_PROVIDERSThe providers that you want to show in the login form SOCIAL_AUTH_DISALLOWED_OPENID_HOSTS'www.google.com', 'google.com'Google OpenID should not be used with manual OpenID entry because Google may use different ID's for different domains so it's not guaranteed to always stay the same. If you want to use Google OpenId, use: social.backends.google.GoogleOpenId (will use email as ID)
http://blog.stackoverflow.com/2010/04/openid-one-year-later/
http://blog.stackoverflow.com/2009/04/googles-openids-are-unique-per-domain/
SOCIAL_AUTH_UI_BOOTSTRAPTrueUse Twitter Bootstrap styles for buttons SOCIAL_AUTH_UI_STYLEbuttons'buttons' or 'dropdown' as ui SOCIAL_AUTH_UI_ICONSTrueAdd icons to buttons
Project Information

The project was created on Feb 18, 2014.

License: New BSD License
4 stars
hg-based source control
Labels: 
web2py OAuth OpenID social plugin

