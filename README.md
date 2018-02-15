<img align="left" src="logo.png" width="240" />

##### [Redirections encoded with `btoa`](http://redirectatob.github.io)

Maybe it'll help if you think of them instead of redirect`atob`s. Basically redirectatob is a webpage that you pass a URL to in the location hash (i.e. domain/page.html#hash, but encoded with btoa. The webpage will convert the encoded hash back into a URL using atob through the client's JavaScript. I made it so that I could give this URL - http://www.woodus.com/den/games/dq4ds/mugshots.php - to a friend of mine. The site's blacklist was preventing me from posting the link, and, as it had happened to me multiple times before, I decided I needed to make a redirect service.

I don't own a server, though, so just using the client's `btoa`/`atob` worked.

Well, that's mostly the story... but for any developers who wish to generate these redirectatobs automatically, I encourage you to read on.

# Specification

A redirectabob is a URL which is under the `redirectatob.github.io` host, referencing the `index.html` file in the root directory; and, including a valid hash string which encodes the data for where the redirectatob should send the user when they load the page. It is highly recommended that the redirectatob will contain a forward slash after the host part, as ommiting it may cause an additional redirect to occur before the user is sent to their final destination.

## Encoding destination URLs into hashes

Usually, the hash is simply the `btoa()` JavaScript function run on the destination URL. Sometimes, however, this is not possible.

### Valid schemas

Any destination URL must specify a schema, and that schema must be one of

- http
- https
- data

You should not attempt to encode any other schema, as that will make the redirectatob invalid.

### Encoding a URL

#### In the Latin1 range

If a destination URL contains NO characters OUTSIDE the Latin1 character range, then it should be encoded through the following steps

1) Base64 encode the URL, in a way that replicates JavaScript's `btoa()` function.

#### Outside the Latin1 range

If a destination URL contains ONE OR MORE characters OUTSIDE the Latin1 range, then it should be encoded through the following steps...

1) Place a `!` character BEFORE the URL (to the left of the schema part).
2) Percent encode all characters in the URL, AS IF IT IS PART OF A QUERYSTRING (i.e. encode the host part too, if applicable). The percent encoding should be done in a way that replicates JavaScript's `encodeURIComponent()` function.
3) Base64 encode the resulting string, in a way that replicates JavaScript's `btoa()` function.

## Decoding hashes into destination URLs

An encoded hash can be converted into a destination URL. Redirectatob will do this, and then redirect a user to their final destination if a valid redirectatob was provided.

1) Base-64 decode the hash (without the `#`), in a way that replicates JavaScript's `atob()` function.
2) If the decoded value begins with an `!` character, that character should be stripped and the rest of the string should be percent decoded, AS IF IT WERE PART OF A QUERYSTRING (i.e. decode the host part too, if applicable). The percent decoding should be done in a way that replicates JavaScript's `decodeURIComponent()` function.
3) Check if the schema of the URL is one of
   - http
   - https
   - data
   And if it isn't, then these steps MUST be stopped and the URL MUST be rejected as invalid. This is important from a security standpoint.
4) Redirect the user to the URL of the resulting string. It is not necessery to ensure the URL leads to a host which resolves; nor that the destination page responds with a status code in the range 200-299; however, you MAY check that the URL is well-formed.
