# Owncloud

Docker compose configuration files for Owncloud with OpenID Connect AuthN/AuthZ and Ceph RADOS Gateway backend.
You need to set a bunch of environment variables in order to properly deploy the docker compose:

``` bash
ADMIN_PASSWORD=<admin_password>
ADMIN_USERNAME=<admin_username>
CONTACT_EMAIL=<your_email>
DB_NAME=<db_name>
DB_PASSWORD=<db_password>
DB_USERNAME=<db_username>
IAM_HOST=<IAM_hostname>
IAM_IP=<IAM_address>
MYSQL_ROOT_PASSWORD=<mysql_root_password>
OIDC_CLIENT_ID=<client_id>
OIDC_CLIENT_SECRET=<client_secret>
OIDC_PROVIDER=<IAM_URL>
OIDC_AUTHORIZATION_ENDPOINT=<IAM_Auth_endpoint>
OIDC_TOKEN_ENDPOINT=<IAM_token_endpoint>
OWNCLOUD_VERSION=<owncloud_version>
S3_ACCESS_KEY=<access_key>
S3_SECRET_KEY=<secret_key>
S3_ENDPOINT=<S3_endpoint>
S3_HOST=<S3_hostname>
S3_IP=<S3_address>
S3_REGION=<S3_region>
S3_CLASS=OCA\\Files_Primary_S3\\S3Storage
S3_ENABLED=True
S3_BUCKET=<S3_bucket>
S3_VERSION=2006-03-01
S3_PATHSTYLE=True
TRAEFIK_VERSION=<traefik_version>
HTTP_COOKIE_SAMESITE=None
```
 

## WebDAV with OpenID Connect

Rclone is able to use OpenID Connect for authentication by leveraging a so called OIDC-agent.

### Setting up the OIDC-agent

You need to install the [OIDC-agent](https://github.com/indigo-dc/oidc-agent) from your OS package repository (e.g. [Debian](https://github.com/indigo-dc/oidc-agent#debian-packages) or [MacOS](https://github.com/indigo-dc/oidc-agent#debian-packages)).


### Configuring the OIDC-agent

Run the following command to add a OpenID Connect profile to your OIDC-agent (see the script [create-oidc-client.sh](https://baltig.infn.it/fornari/owncloud/-/blob/main/scripts/create-oidc-client.sh)). It will open the login page of OpenID Connect identity provider where you need to log in if you don't have an active session.

``` bash
oidc-gen \
 --issuer https://${IAM_HOST}/ \
 --flow device \
 --scope max \
 oidc-owncloud
```

After a successful login or an already existing session you will be redirected to a page where you have to insert the code provided by oidc-gen. After inserting the code, you will be redirected to a success page of the OIDC-agent.
You will now be asked for a password for your account configuration, so that your OIDC session is secured and cannot be used by other people with access to your computer.


## Configure the WebDAV remote

First of all you need to set up your credentials and the WebDAV remote for Rclone. In this example we do this by setting environment variables. You might also set up a named remote or use command line options to achieve the same.

``` bash
export RCLONE_WEBDAV_VENDOR=owncloud
export RCLONE_WEBDAV_URL=https://${OWNCLOUD_HOST}/remote.php/webdav/
export RCLONE_WEBDAV_BEARER_TOKEN_COMMAND="oidc-token oidc-owncloud"
```


### Sync to the WebDAV remote

Now you can use Rclone to sync the local folder `/tmp/test` to `/test` in your Owncloud home folder.

``` bash
rclone sync :local:/tmp :webdav:/test
```

If your Owncloud doesn't use valid SSL certificates, you may need to use `rclone --no-check-certificate sync ...`.


### Mount WebDAV remote folder on local directory

You can also use Rclone to mount your remote Owncloud home folder on a local directory.

``` bash
mkdir owncloud-home
rclone mount :webdav:/ ./owncloud-home
```
 
If your Owncloud doesn't use valid SSL certificates, you may need to use `rclone --no-check-certificate mount ...`.
