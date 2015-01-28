This is a [FreeRADIUS](http://freeradius.org/) [OAuth2 (OpenID Connect)](http://en.wikipedia.org/wiki/OpenID_Connect) perl module to handle authentication.  It was created to allow the users of a wireless 802.1X (WPA Enterprise) network to connect.

**N.B.** this module relies on [`grant_type=password`](https://tools.ietf.org/html/rfc6749#section-4.3) being supported by your OAuth2 provider

# Preflight

## Workstation

You will need to [have git installed on your workstation](http://git-scm.com/book/en/Getting-Started-Installing-Git), [cURL](http://curl.haxx.se/) and python.

**N.B.** Debian/Redhat users should be able to just type `sudo {apt-get,yum} install git curl python` whilst Mac OS X users should already have these tools present.

So we start off by fetching a copy of the project:

    git clone https://github.com/jimdigriz/freeradius-oauth2-perl.git
    cd freeradius-oauth2-perl

## Target RADIUS Server

Preferably running Debian `wheezy` 7.x, you should set up a working *default* installation of FreeRADIUS 2.2.x.  This can be done with:

    sudo apt-get install -yy --no-install-recommends freeradius freeradius-utils libwww-perl libconfig-ini-perl

**N.B.** if someone wants to step forward to help get this working on another UNIX system (*BSD, another Linux, Mac OS X, etc) and/or a later version of FreeRADIUS, then do get in touch

On the server, run:

    mkdir /opt/freeradius-perl-oauth2

From the project directory on your workstation, copy `main.pl` and `module` to `/opt/freeradius-perl-oauth2`:

# Configuration

## OAuth2 Discovery

If you run a *secure* HTTPS website at `https://example.com` then you can make use of the auto-discovery mechanism, by making:

    https://example.com/.well-known/openid-configuration

Generate an HTTP redirect depending on your authentication provider to:

 * **Microsoft Azure AD (Office 365):** `https://login.windows.net/example.com/.well-known/openid-configuration`
 * **Google Apps [not supported]:** `https://accounts.google.com/.well-known/openid-configuration`

If you do not have a secure main website, then you will need to inspect your authentication provider URLs manually, extracting the `authorization_endpoint` and `token_endpoint` entries, and add them to the configuration.

FIXME adding manual endpoints

## Cloud

### Microsoft Azure AD (Office 365)

1. go to https://portal.office.com and log in as an administrator
1. click on 'Admin' in the top right and select 'Azure AD' from the drop down menu
1. once the Azure page loads, click on 'Active Directory' from the left hand panel
1. highlight your main directory, and click on the arrow pointing right located to the right of its name
1. click on 'Applications' at the top
1. click on 'Add' located at the centre of the bottom of the page
1. a window will open asking 'What do you want to do?', click on 'Add an application my organization is developing'
1. enter in the application name `freeradius-oauth2-perl`, click on 'Native client application', and click on the next arrow
1. as a redirect URI, enter in `http://localhost:8000/code.html`, then click on the complete arrow
1. the application will now be added and you will be shown a preview page
1. before we leave, you will need the 'Client ID' (a long hex GUID), which you can find located under both 'Configure' (at the top of the page) or 'Update your Code' (under the 'Getting Started' title)

Using this Client ID, we now need to create an authorisation code, to do this, run a webserver from a terminal, inside the project with:

    python -m SimpleHTTPServer

Now in your web browser go to (replacing `CLIENTID` with your client ID):

    https://login.windows.net/common/oauth2/authorize?response_type=code&prompt=admin_consent&client_id=CLIENTID

You will be taken to a page asking you to permit freeradius-oauth2-perl access to enable sign-on and read users' profiles.  When you click on 'Accept' you will be redirected to a page that provides you with the authorisation code.  Take a copy of this, either via cut'n'paste or using the 'Export to File' link on that page.

Now `Ctrl-C` the python webserver as we have finished with it.

FIXME what to do with this file

### Google Apps

**N.B.** works in progress

## FreeRADIUS

On the server run as root:

    ln -T -f -s /opt/freeradius-perl-oauth2/module /etc/freeradius/modules/freeradius-perl-oauth2

Amend `/etc/freeradius/sites-available/default` to add `freeradius-perl-oauth2` at the right sections:

    authorize {
      ...
    
      files
    
      # after 'files'
      freeradius-perl-oauth2
    
      expiration
    
      ...
    }
    
    authenticate {
      ...
    
      eap
      
      # after 'eap'
      Auth-Type freeradius-perl-oauth2 {
        freeradius-perl-oauth2
      }
    }

# Testing

    radtest moo cow localhost 0 testing123 IGNORED 127.0.0.1

# Related Links

 * [Microsoft Azure REST API + OAuth 2.0](https://ahmetalpbalkan.com/blog/azure-rest-api-with-oauth2/)
 * [Authorization Code Grant Flow](https://msdn.microsoft.com/en-us/library/azure/dn645542.aspx)
 * [Configuring a Web Page Redirect](http://docs.aws.amazon.com/AmazonS3/latest/dev/how-to-page-redirect.html) - S3->Cloudfront redirects
