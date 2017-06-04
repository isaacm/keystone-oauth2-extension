# keystone-oauth2-extension
OpenStack Keystone extension to enable OAuth 2.0.

## How to Install
To install this extension in Keystone, you have to do the following:

1. Place the `oauth2` folder inside the `keystone/contrib` folder in your Keystone project.

2. Place the files in `tests/` inside the `keystone/tests` folder in your Keystone project.

3. This extension implements an auth plugin. You need to add the `plugins/oauth2.py` module to the `keystone/auth/plugins` folder in your Keystone project.

   > The files inside the `config` folder contain everything you need to **add** to your Keystone settings files (`etc/keystone.conf` and `etc/keystone-paste.ini`). If you are an experienced user, you can check those files and **skip steps 4-6**. Should you prefer to set up everything step by step, please read on.

4. Since this extension is augmenting a pipeline (see [Keystone docs](http://docs.openstack.org/developer/keystone/extension_development.html#modifying-the-keystone-paste-ini-file) for more info), a corresponding `filter:` section is necessary to be introduced in your `etc/keystone-paste.ini` file. Just place the following:
   ```
   [filter:oauth2_extension]
   paste.filter_factory = keystone.contrib.oauth2.routers:OAuth2Extension.factory
   ``` 
5. In order for the extension to work, it must be placed in the `pipeline`.

6. Edit the `[auth]` section in your `keystone.conf` file (the one placed in the `etc` folder in your Keystone project), to include OAuth 2.0 auth method, just like this:
   <pre>
   # Default auth methods. (list value)
   methods=external,password,token,<b>oauth2</b>
   </pre>

7. Define new policies in your `policy.json` file (the one placed in the `etc` folder in your Keystone project) for the following targets: 
   ```
   identity:list_authorization_codes
   identity:revoke_access_token
   identity:request_authorization_code
   ```
The file `config/policy.json` contains default values you can use, as well as other required policies which Keystone should include by default.

8. Create a config registration file `keystone/conf/oauth2.py`, by copying `keystone/conf/oauth1.py` and change the string `oauth1` to `oauth2` in the content.
Add oauth2 import and module to conf_modules in `keystone/conf/__init__.py`. Then add this:
	```
	keystone.auth.oauth2 =
	default = keystone.auth.plugins.oauth2:OAuth2
	keystone.oauth2 =
    sql = keystone.contrib.oauth2.backends.sql:OAuth2
	```
To the `keystone/setup.cfg` file 

9. Check Python dependencies. This extension uses [OAuthLib](https://oauthlib.readthedocs.org/en/latest/), tested to work with versions >=0.7.2, <=1.0.3. This is already a dependency in Keystone and you should not need to install it again, but if you are not using the standard Keystone installation, make sure to add it.

10. Create database tables. Copy `migrate_repo/latest_add_oauth2_tables.py` to `keystone/common/sql/migrate_repo/versions/`, and replace the leading `latest` to the latest_version+1, where latest_version is the keystone repo's latest version numbe, then execute:
`tools/with_venv.sh keystone-manage db_sync` to create oauth2 related tables


