# Drupal 8

[TOC]

## Configuración de Proxy para salida a internet

```bash
/**
 * External access proxy settings:
 *
 * If your site must access the Internet via a web proxy then you can enter the
 * proxy settings here. Set the full URL of the proxy, including the port, in
 * variables:
 * - $settings['http_client_config']['proxy']['http']: The proxy URL for HTTP
 *   requests.
 * - $settings['http_client_config']['proxy']['https']: The proxy URL for HTTPS
 *   requests.
 * You can pass in the user name and password for basic authentication in the
 * URLs in these settings.
 *
 * You can also define an array of host names that can be accessed directly,
 * bypassing the proxy, in $settings['http_client_config']['proxy']['no'].
 */
$settings['http_client_config']['proxy']['http'] = '172.26.67.48:3128';
$settings['http_client_config']['proxy']['https'] = '172.26.67.48:3128';
$settings['http_client_config']['proxy']['no'] = ['127.0.0.1', 'localhost','drupal8dev'];

```

https://www.drupal.org/project/varnish 

## One time link para cambiar la contraseña

```
[drupadmin@lvwdrup8devapp drup8agesic]$ drush uli -l https://drup8agesic.corp.ute.com.uy
https://drup8agesic.corp.ute.com.uy/user/reset/1/1519217557/E1XwkM1LXCsW5aSyT3E_W-GeboeQ4UwXUVNlKtyqFe4/login

[drupadmin@lvwdrup8devapp drup8agesic]$ drush help uli
Display a one time login link for user ID 1, or another user.

Examples:
drush user:login                            Open default web browser and browse to homepage, logged in as uid=1.
drush user:login --name=ryan node/add/blog  Open default web browser (if configured or detected) for a one-time 
                                            login link for username ryan that redirects to node/add/blog.
drush user:login --browser=firefox --mail=drush@example.org  Open firefox web browser, and login as the user 
                                                             with the e-mail address drush@example.org.
Arguments:
  [path] Optional path to redirect to after logging in.

Options:
  --name[=NAME]                 A user name to log in as. If not provided, defaults to uid=1. [default: "1"]
  --browser[=BROWSER]           Optional value denotes which browser to use (defaults to operating system 
                                default). Use --no-browser to suppress opening a browser. [default:"true"]
  --redirect-port=REDIRECT-PORT A custom port for redirecting to (e.g., when running within a Vagrant 
                                environment)
  --no-browser                  Negate --browser option.
Aliases: uli, user-login
```

https://devcentral.f5.com/questions/load-balance-drupal-site 

