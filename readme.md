# Yaxy

[Описание на русском языке](readme_ru.md)

Yaxy is a proxy-server for a web developer, it substitutes the required resources with simple rules.

## Installation

If you haven't installed NodeJS already, [then do it] (http://nodejs.org/), then

     npm install-g yaxy

## Running

     yaxy - config my-yaxy-config.txt - port 9999

If you do not specify `--config`, Yaxy will look for yaxy-config.txt file in the current directory. The server will listen on port `8558`, if no `--port` specified.

## Configuration File Format

The configuration file is read line by line. Blank lines, lines beginning with `#` and unknown strings are ignored.

Rules can be combined in sections. Start of the section is a line enclosed in square brackets, the contents of a line can be any.

    # Rules out of sections

    [Section 1]
    # Rules for section #1

    [Section 2]
    # Rules for section #2

If a section name starts with `#`, then all rules of this section will be ignored.

    # Rules out of sections

    [#Section 1]
    # Section #1 rules
    # will be ignored

    [Section 2]
    # Rules for section #2

## Rules

Rules are written like:

    url => replacement

`url` -- domain address string for rule to work. `http://` at the beginning of line is not obligaroty. E.g., rule `www.yandex.ru => ...` will work for all resources at www.yandex.ru domain, while `yandex.ru/yandsearch => ...` -- only for Yandex search.

Sometimes exact address match is required, i.e. you need to modify only site's main page, leaving the rest of it intact. For this specify `!` at the beginning of url, `http://` is still optional, but trailing slash(es) are a must for exact matching. E.g., replaceing Yandex main page:  `!www.yandex.ru/ => ...`.


If first two cases are not enough, you may use regexp for left side, enclosed with `/`. So, if you'll need to modify all requests to .ru domain, use:  `/^http://[^/]+\.ru// => ...`. Please note, regexp uses whole url, including `http://`.

### Simple url substitute

The right side of a rule specifies the replacement for captured match of a rule's left side. E.g., rule `google.ru => yandex.ru` will substitute all request from `google.ru` to `yandex.ru`, so request `http://google.ru/` goes to `http://yandex.ru/`, and request to `http://google.ru/foo/bar` goes to `http://yandex.ru/foo/bar`.
More specific rule `google.ru/foo => yandex.ru/foo` will substitute request to `http://google.ru/foo/bar`, while leaving intact `http://google.ru/baz`. Please note, if you need to keep path in a domain, you must specify it on both parts of a rule, because `google.ru/foo => yandex.ru` will replace request `http://google.ru/foo/bar` to `http://yandex.ru/bar`.

When you have a regexp in a left side of a rule, right side is treated as a template for captured groups. E.g., we replace all *.ru domain requests to *.com: `/^(http://[^/]+\.)ru(/.*)$/ => $1com$2`.

### Displaying local files

You can have your little static-server, using `file://` protocol in the right part of a rule.

    # windows
    host.com/some/path => file://c:/www/host.com
    # linux
    host.com/some/path => file:///home/www/host.com

Directory index file -- index.html. For above example, url `http://host.com/some/path` will return file `/home/www/host.com/index.html`, and url `http://host.com/some/path/empty.gif` returns `/home/www/host.com/empty.gif`.

### Using data:uri

If your substitute is simple and you don't want to create a file, you may use data:uri.

    host.com/some/path => data:text/html;<script type="text/javascript">alert('Hello!');</script>

Then all urls, starting with `host.com/some/path` will return response with `<script type="text/javascript">alert('Hello!');</script>`.

    host.com/some/path/png => data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQAQMAAAAlPW0iAAAABlBMVEUAAAD///+l2Z/dAAAAM0lEQVR4nGP4/5/h/1+G/58ZDrAz3D/McH8yw83NDDeNGe4Ug9C9zwz3gVLMDA/A6P9/AFGGFyjOXZtQAAAAAElFTkSuQmCC

Get an image for all urls, starting with `host.com/some/path/png`

If you have a regexp in rule's left side, you may use it's captured groups in the right side.

    /^http://test.my/\?name=(.*)/ => data:text/html;<script>alert('Hello, $1!');</script>

### Leaving url intact

If url must be left intact, use `$` in rule's right side.

    host.com/some/path => $

Sometimes you don't need to modify a request, but only use some modifiers for it (see below).

### Cancelling a request

If right side of rule is left empty, then request will be cancelled (even no http headers will be returned).

## Modifiers

Modifiers are strings, starting with `$` (leading spaces don't matter). First goes modifier name, after space -- arguments. Modifier corresponds to the rule it is written after. If modifier is written at the file's beginning (before any rule), it applies to all rules. If modifier is written at the section's beginning, then it applies to all section rules.

Please note, global modifiers applies to *rules* (i.e. requests, matching these rules), and not all requests through proxy server.

### Modifying GET-parameters

    # Add or modify parameter
    $SetQueryParam from=yaxy

    # Removing parameter
    $RemoveQueryParam from

### Cookie manipulations

Cookies are modified for server, so while browser sends an original cookie, server receives the modified one.

    # Adding/modifying specific cookie
    $SetCookie user=YaxyUser

    # Removing a cookie
    $RemoveCookie ssid

### Modifying HTTP headers

    # Setting a request header
    $SetRequestHeader X-Requested-With: Yaxy

    # Setting a response header
    $SetResponseHeader X-Proxy: Yaxy

    # Removing request header
    $RemoveRequestHeader Referer

    # Removing response header
    $RemoveResponseHeader Content-Type

### Modifying response status

    # Always HTTP 200 for response (even when it's not)
    $StatusCode 200

### Response delay

    #Respond at least 5 seconds later
    $Delay 5
