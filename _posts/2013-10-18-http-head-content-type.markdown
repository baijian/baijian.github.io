---
layout: post
title: "http-head-content-type"
date: 2013-10-18 22:49
comments: true
categories: 
---

`Content-Type` is one component of `HTTP header fields`, both `request header` and `response header`.
In `HTTP`(Hypertext Transfer Protocol), severial header fields like `Content-Type` define the
operating parameters of an HTTP transaction.

The header fields are tarnsmitted after the request or response line, which is the first line of a
message. The end of the headers is indicated by an empty field. A core set of fields is standardlized
by the `IETF` in `RFC` and must be implemented by all HTTP-compliant protocol implementations.
Non-standard header fields were conventionally marked by prefixing the field name with `X-`.
There are no limits to the size of each header field name or value or to the number of headers in
standard itself. However, most servers, clients and proxy software impose some limits for practical
and security reasons.`Apache 2.3` by default limits each header size to 8190 bytes and there can by at
most 100 headers in a single request.


`Content-Type` is the `MIME type` of the body of the request which is used with `POST` and `PUT` requests.
Let's see some examples:

* Request header

    Content-Type: application/x-www-form-urlencoded

    Content-Type: multipart/form-data; boundary=AaB03x

Also in request response header, `Content-Type` indicate the MIME type of response content.

* Response header

    Content-Type: text/html; charset=utf-8

An `Internet media type` is a two-part identifier for file formats on the Internet.
The identifiers were originally defined in `RFC 2046` for use in email send through
`SMTP`, but their use expanded to other protocols such as HTTP. These types are called
`MIME type` and are sometimes referred to as `Content-types`.The original name `MIME type`
referred to usage to identify non-ASCII parts of email messages composed using the
`MIME`(Multipurpose Internet Mail Extensions) specification.

A media type is composed of two or more parts, a type, a subtype and zero or more optional parameters.
For example, subtypes of `text` have an optional `charset` parameter that can be included to indicate
the character encoding, and subtypes of multipart type often define a boundary between parts.

Web browsers also support various media types. This enables the browser to display or output files
that are not in HTML format. Media type specification is also an important information source
for search engines for the classification of data files on the web.

Prior to RFC 6648, experimental or no-standard media types were prefixed with `x-`.Subtypes that
begin with vnd. are vendor-specific, subtypes that begin with prs. are int the personal or vanity tree.
New media types can be created with procedures outlined in RFC 4288.

There are a list of [common media types](http://en.wikipedia.org/wiki/Internet_media_type#List_of_common_media_types) 
and list of 
[common media subtype prefixes](http://en.wikipedia.org/wiki/Internet_media_type#List_of_common_media_subtype_prefixes)

### application/x-www-form-urlencoded

This content type is the default content type of `POST` method.
Forms submitted with this content type must be encoded as follows:

1. Control names and values are escaped. Space characters are replaced by `+`, and then reserved characters
are escaped as described in RFC1738. Non-alphanumeric characters are replaced by `%HH`, a percent sign and
two hexadecimal digits representing the ASCII code of the character. Line breaks are represented as `CR LF`
pairs(ie., `%0D%0A`).

2. The control names/values are listed in the order they appear in the document. The name is separated from
the value by `=` and name/value pairs are separated from each other by `&`.

### multipart/form-data

This type defined in RFC 2388 for additional information about file uploads.
The content type `application/x-www-form-urlencoded` is inefficient for sending large quantities of binary
data or text containing non-ASCII characters. The content type `multipart/form-data` should be used for
submitting forms that contain files, non-ASCII data, and binary data.

The content `multipart/form-data` follows the rules of all multipart MIME data streams as outlined in
RFC2045. The definition of `multipart/form-data` is avaiable at the IANA registry.

A `multipart/form-data` message contains a series of parts, each representing a successfull control.
As with all multipart MIME types, each part has an optional `Content-Type` header that defaults to
`text/plain`. User agents should supply the `Content-Type` header, accompanied by a `charset` parameter.

Each part is expected to contain:

1. a `Content-Disposition` header whose value is form-data.

2. a name attribute specifying the control name of the corresponding control.Control names original
encoded in non-ASCII character sets may be encoded using the method outlined in RFC2045.

### Examples

* upload a file request header example

```
    Content-Type: multipart/form-data; boundary=AaB03x

    --AaB03x
    Content-Disposition: form-data; name="inputname1"

    InputText
    --AaB03x
    Content-Disposition: form-data; name="fileinputname"; filename="file.txt"
    Content-Type: text/plain

    ... contents of file1.txt ...
    --AaB03x--
```

* upload a file and a image example

```
    Content-Type: multipart/form-data; boundary=AaB03x

    --AaB03x
    Content-Disposition: form-data; name="inputname1"

    InputText
    --AaB03x
    Content-Disposition: form-data; name="files"
    Content-Type: multipart/mixed; boundary=BbC04y

    --BbC04y
    Content-Disposition: file; filename="file1.txt"
    Content-Type: text/plain

    ... contents of file1.txt ...
    --BbC04y
    Content-Disposition: file; filename="file2.gif"
    Content-Type: image/gif
    Content-Transfer-Encoding: binary

    ...contents of file2.gif...
    --BbC04y--
    --AaB03x--
```

For detail you can see from [W3C](http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4.1).

At last I want to tell you a situation I have ever meet with which is the purpose to write this article.
I write a API for a APP developer
to use, I use `php` to develop my api, and get post parameters with `$_POST` as we phper knows.
But what's strange, I can not get his post parameters within `$_POST` array. When I use `curl -d`,
It's ok. So I think of `Content-Type`, I want to compare different content type with `curl -H` command.
When I use `multipart/form-data` header, ofcourse `API` can not get the post parameters. So I judge
it must be the APP developer request my api with error `Content-Type` header, I tell he to add a header
`Content-Type: application/x-www-form-urlencoded` before post my api, then it is ok. It tell us, do not
use any http libs which you search from internet without rewrite it. It may be some bugs in it, especially
searching with `Baidu` and chinese languages.

