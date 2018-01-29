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

