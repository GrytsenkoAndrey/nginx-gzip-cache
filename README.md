# nginx-gzip-cache
How to enable gzip and Expires headers into the nginx

## Enable GZIP

### Test files

Create a file named test.html in the default Nginx directory using truncate. This extension denotes that it’s an HTML page:

```
sudo truncate -s 1k /var/www/html/test.html
```

Let’s create a few more test files in the same manner: one jpg image file, one css stylesheet, and one js JavaScript file:

```
sudo truncate -s 1k /var/www/html/test.jpg
sudo truncate -s 1k /var/www/html/test.css
sudo truncate -s 1k /var/www/html/test.js
```

The next step is to check how Nginx behaves with respect to compressing requested files on a fresh installation with the files we have just created.

### Checking the default Behavior

Let’s check if the HTML file named test.html is served with compression. The command requests a file from our Nginx server and specifies that it is fine to serve gzip compressed content by using an HTTP header (Accept-Encoding: gzip):

```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.html
```

In response, you should see several HTTP response headers:

```
Output
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 09 Feb 2021 19:04:25 GMT
Content-Type: text/html
Last-Modified: Tue, 09 Feb 2021 19:03:41 GMT
Connection: keep-alive
ETag: W/"6022dc8d-400"
Content-Encoding: gzip
```

In the last line, you can see the Content-Encoding: gzip header. This tells us that gzip compression was used to send this file. That’s because Nginx has gzip compression enabled automatically even on the fresh Ubuntu 20.04 installation.

However, by default, Nginx compresses only HTML files. Every other file will be served uncompressed, which is less than optimal. To verify that, you can request our test image named test.jpg in the same way:

```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.jpg
```

The result should be slightly different than before:

```
Output
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 09 Feb 2021 19:05:49 GMT
Content-Type: image/jpeg
Content-Length: 1024
Last-Modified: Tue, 09 Feb 2021 19:03:45 GMT
Connection: keep-alive
ETag: "6022dc91-400"
Accept-Ranges: bytes
```

### Configuring nginx's GZIP settings

To change the Nginx gzip configuration, open the main Nginx configuration file in nano or your favorite text editor:

```
sudo nano /etc/nginx/nginx.conf
```

Find the gzip settings section, which looks like this:

**/etc/nginx/nginx.conf**

```
. . .
##
# `gzip` Settings
#
#
gzip on;
gzip_disable "msie6";

# gzip_vary on;
# gzip_proxied any;
# gzip_comp_level 6;
# gzip_buffers 16 8k;
# gzip_http_version 1.1;
# gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
. . .

```

You can see that gzip compression is indeed enabled by the gzip on directive, but several additional settings are commented out with # sign and have no effect. We’ll make several changes to this section:

- Enable the additional settings by uncommenting all of the commented lines (i.e., by deleting the # at the beginning of the line)
- Add the gzip_min_length 256; directive, which tells Nginx not to compress files smaller than 256 bytes. Very small files barely benefit from compression.
- Append the gzip_types directive with additional file types denoting web fonts, icons, XML feeds, JSON structured data, and SVG images.

After these changes have been applied, the settings section should look like this:

```
. . .
##
# `gzip` Settings
#
#
gzip on;
gzip_disable "msie6";

gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_min_length 256;
gzip_types
  application/atom+xml
  application/geo+json
  application/javascript
  application/x-javascript
  application/json
  application/ld+json
  application/manifest+json
  application/rdf+xml
  application/rss+xml
  application/xhtml+xml
  application/xml
  font/eot
  font/otf
  font/ttf
  image/svg+xml
  text/css
  text/javascript
  text/plain
  text/xml;
. . .
```

Save and close the file to exit.

To enable the new configuration, restart Nginx:

```
sudo systemctl restart nginx
```

## Testing

```
curl -H "Accept-Encoding: gzip" -I http://localhost/test.css
```

Now gzip is compressing the file:

```
Output
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 09 Feb 2021 19:21:54 GMT
Content-Type: text/css
Last-Modified: Tue, 09 Feb 2021 19:03:45 GMT
Connection: keep-alive
Vary: Accept-Encoding
ETag: W/"6022dc91-400"
Content-Encoding: gzip
```

