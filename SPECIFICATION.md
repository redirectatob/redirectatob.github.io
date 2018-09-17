# Specification
## 1. Introduction
Sometimes, a URL may need to be obfuscated or encoded to bypass automated filters. Previously, two main solutions have been attempted: using a URL-shortener, and paying for hosting with server-side script capabilities. The former prevented automated generation of obfuscation, for CAPTCHAs were likely in place to prevent clogging the databases used. This tool, **redirectatob**, is therefore documented: it requires only static hosting to work, for all scripting is performed client-side.
### 1.1. Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).
## 2. General Format
A 'redirectatob' is a URI (as specified in [RFC3986](https://tools.ietf.org/html/rfc3986) with a fragment component (the segment after the number sign `#`). This fragment component encodes a URI. Hosts SHOULD redirect to this URI after decoding.
## 3. Encoding
The steps to generate the fragment component when attempting to encode some valid target URI string *target* are below
*NOTE*: In JavaScript, `location.hash` includes the preceding number sign `#`, and then the fragment component. This section does not describe this number sign.

1. Set the variable *encoded* to: the value of *target*
2. If *encoded* contains a codepoint outside the range U+0000 to U+00FF, then:
  1. Set *encoded* to: *encoded* but prefixed with an exclamation mark `!` (U+0021)
  2. Set *encoded* to: the result of encodeURIComponent when *uriComponent* is set to *encoded*, as defined in [18.2.6.5 of ECMAScript 2015](https://www.ecma-international.org/ecma-262/6.0/#sec-encodeuricomponent-uricomponent)
3. Set *encoded* to: the result of btoa when performed with an argument of *encoded*, as defined in the [HTML5 Specification for btoa](https://www.w3.org/TR/html50/webappapis.html#dom-windowbase64-btoa)
4. The fragment component is then the value of *encoded*

If encoding the fragment component for the official redirectatob host (https://redirectatob.github.io), then there exists no representation for *target*, unless *target* begins with any of:
- http://
- https://
- data:

Decoders MUST be able to decode those above three schemes, and SHOULD redirect to all of them. Decoders MAY decode other schemes, but should consider the security impact of doing so.

## 4. Decoding
The steps to decode a fragment component *fragment*, which was generated through the above encoding steps, into some target URI, are listed below:
*NOTE*: In JavaScript, `location.hash` includes the preceding number sign `#`, and then the fragment component. This section does not describe this number sign.

1. Set the variable *decoded* to be the value of *fragment*
2. Set *decoded* to the result of atob with an argument of *decoded*, as defined in the [HTML5 specification for atob](https://www.w3.org/TR/html50/webappapis.html#dom-windowbase64-atob)
3. If the first character of *decoded* is an exclamation mark `!` (U+0021), then:
  1. Set *decoded* to: *decoded*, but with its first character removed
  2. Set *decoded* to: the result of decodeURIComponent when *encodedURIComponent* is set to *decoded*, as defined in [18.2.6.3 of ECMAScript 2015](https://www.ecma-international.org/ecma-262/6.0/#sec-decodeuricomponent-encodeduricomponent)
4. The value of *decoded* is the URL that was encoded.

The decoder SHOULD then redirect to the URI.

## 5. Extending redirectatob, error-handling

An error has occured if one of the following has occured:
- A function that is required for encoding or decoding has thrown an error when performing its function
- A scheme that the host has disallowed is being encoded or decoded
- The URI is not well-formed

A redirectatob encoder or decoder SHOULD show relevant feedback to the user regarding the error.

A redirectatob encoder or decoder MAY build upon the format, hence allowing something which would be an error message to instead be converted into functionality.

## 6. Security Considerations

Redirectatob's software is designed for redirection performed by ECMAScript within a host environment of a browser. Before allowing schemes like javascript:, careful consideration should be taken to ensure that such a script could not manipulate the displayed URL (for example by removing the fragment component) with `location.pushState`; and then change the contents of the website to attempt to defame the person who owns that site.
