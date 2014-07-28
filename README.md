[![Build Status](https://travis-ci.org/nicklockwood/RequestUtils.svg)](https://travis-ci.org/nicklockwood/RequestUtils)


Purpose
--------------

RequestUtils is a collection of category methods designed to simplify the process of HTTP request construction and manipulation in Cocoa. It extends NSString, NSURL and NSURLRequest with some additional utility methods that were left out of the standard API.


Supported OS & SDK Versions
-----------------------------

* Supported build target - iOS 8 / Mac OS 10.9 (Xcode 6.0, Apple LLVM compiler 6.0)
* Earliest supported deployment target - iOS 5.1 / Mac OS 10.7
* Earliest compatible deployment target - iOS 4.3 / Mac OS 10.6.8

NOTE: 'Supported' means that the library has been tested with this version. 'Compatible' means that the library should work on this iOS version (i.e. it doesn't rely on any unavailable SDK features) but is no longer being tested for compatibility and may require tweaking or bug fixes to run correctly.


ARC Compatibility
------------------

RequestUtils makes use of conditional compilation to automatically work with both ARC and non-ARC projects. There is no need to exclude RequestUtils files from the ARC validation process, or to convert RequestUtils using the ARC conversion tool.


Thread Safety
--------------

All the RequestUtils methods should be safe to use concurrently on multiple threads.


Installation
--------------

To use the RequestUtils categories in an app, just drag the RequestUtils.h and .m files (demo files and assets are not needed) into your project and import the header file into any class where you wish to make use of the RequestUtils functionality, or include it in your prefix.pch file to make it available globally within your project.


NSString Extensions
----------------------

RequestUtils extends NSString with the following methods:

	- (NSString *)URLEncodedString;
	
This is a method to URL-encode a string so that it may be safely used within a URL path or query parameter. This method is different from the standard `stringByAddingPercentEscapesUsingEncoding:` method because that only ensures that the string does not contain characters that are invalid for use in a string, but it doesn't encode characters that are valid to use in a url (e.g. "?", "&") but which cannot be used inside a query string parameter or path component without affecting the URL structure.

	- (NSString *)URLDecodedString:(BOOL)decodePlusAsSpace;
	
This reverses the effects of `URLEncodedString` by replacing all percent escape sequences with their original characters. Internally this method uses to `stringByReplacingPercentEscapesUsingEncoding:` with NSUTF8StringEncoding. The parameter `decodePlusAsSpace` will cause "+" characters to be converted to spaces, which is useful if the parameters were generated by a web form.

    - (NSString *)stringByAppendingURLPathExtension:(NSString *)extension;
    
NSString has a useful method `stringByAppendingPathExtension:` which can be used to add a path extension to the end of the string, correctly separated by a dot. Unfortunately the `stringByAppendingPathExtension:` method is not URL-aware, and incorrectly strips the double `//` from URLs when a path is appended. `stringByAppendingURLPathExtension:` is much smarter, and not only avoids stripping the `//` from the URL schema, but will also correctly insert the path extension *before* the URL query or fragment string.
    
    - (NSString *)stringByDeletingURLPathExtension;
    
This is a URL-aware version of the `stringByDeletingURLPathExtension` method. It will remove the last component of the path without disrupting the query or fragment strings, or mangling `//` (if present).
    
    - (NSString *)URLPathExtension;
	
This is a URL-aware version of the `pathExtension` method. It will return the path file extension without including the query or fragment strings (if present).
	
    - (NSString *)stringByAppendingURLPathComponent:(NSString *)str;
    
NSString has a useful method `stringByAppendingPathComponent:` which can be used to add path components to the end of the string whilst correctly adding a dividing `/` and avoiding double `/` if one or both path components already include it. Unfortunately the `stringByAppendingPathComponent:` method is not URL-aware, and incorrectly strips the double `//` from URLs when a path is appended. `stringByAppendingURLPathComponent:` is much smarter, and not only avoids stripping the `//` from the URL schema, but will also correctly insert the path component *before* the URL query or fragment string.
    
    - (NSString *)stringByDeletingLastURLPathComponent;
    
This is a URL-aware version of the `stringByDeletingLastPathComponent` method. It will remove the last component of the path without disrupting the query or fragment strings, or mangling `//` (if present).
    
    - (NSString *)lastURLPathComponent;
    
This is a URL-aware version of the `lastPathComponent` method. It will return the last component of the path without including the query or fragment strings (if present).
	
	+ (NSString *)URLQueryWithParameters:(NSDictionary *)parameters;
	
This method generates a URL query string from a dictionary of keys and values. The values in the dictionary can either be individual values such as strings or numbers, or arrays of values, in which case the last value in the array will be used by default. All key and values in the dictionary will be automatically URL-encoded. See below for more options around how parameter arrays are handled.
	
	+ (NSString *)URLQueryWithParameters:(NSDictionary *)parameters options:(URLQueryOptions)options;
	
This method generates a URL query string from a dictionary of keys and values, but provides an additional `options` argument to control how parameter arrays are handled. All keys and values in the dictionary will be automatically URL encoded. See `URLQueryOptions` below for discussion of the options.
	
	- (NSString *)URLQuery;
	
This method attempts to find and return a URL query string within the string. It is similar to the `query` method of NSURL, but works better with partial URLs/URIs or URLs with non-standard structures, e.g. bespoke schemas.
	
	- (NSString *)stringByDeletingURLQuery;
	
This method finds and removes the URL query from the string. It is URL fragment/hash aware and will leave the URL fragment untouched if present.
	
    - (NSString *)stringByReplacingURLQueryWithQuery:(NSString *)query

Replaces the current query string (if present) with the specified query string. Passing nil or an empty string as the query parameter is equivalent to calling `stringByDeletingURLQuery`;
	
	- (NSString *)stringByAppendingURLQuery:(NSString *)query;
	
This method will append a URL query to the string. It is URL fragment/hash aware and will insert the query string before the # if present. It will also correctly join the string to an existing query string if present. If there is already a query string present, the two strings will be concatenated with no regard for duplicate parameters, however a joining `&` will be inserted if needed. For more options around merging query strings, see below:
	
	- (NSString *)stringByMergingURLQuery:(NSString *)query;
	
This method will append a URL query to the string, merging any duplicate values. Duplicate query parameter keys will be replaced by the last value encountered for that key. For more merging options, see below:
	
	- (NSString *)stringByMergingURLQuery:(NSString *)query options:(URLQueryOptions)options;
	
This method will append a URL query to the string, merging any duplicate values. By default, duplicate query parameters will be replaced by the last value encountered for that key, but you can override that behaviour using the `options` argument. See `URLQueryOptions` below for discussion of the options.
	
	- (NSDictionary *)URLQueryParameters;
	
This method attempts to locate a query string within the string and then splits it into a dictionary of key/value pairs. Each key and value will be URL decoded. If duplicate parameter keys are encountered, the last value encountered for that key is used. To override this behaviour, use `URLQueryParametersWithOptions:` instead.

	- (NSDictionary *)URLQueryParametersWithOptions:(URLQueryOptions)options;
	
This method attempts to locate a query string within the string and then splits it into a dictionary of key/value pairs. Each key and value will be URL decoded. The `options` argument controls how duplicate parameters should be handled. See `URLQueryOptions` below for discussion of the options.
		
	- (NSString *)URLFragment;
	
This method returns the URL fragment identifier or hash part of the string. It is similar to the `fragment` method of NSURL, but works better with partial URLs/URIs or URLs with non-standard structures, e.g. bespoke schemas.
	
	- (NSString *)stringByDeletingURLFragment;
	
This method finds and removes the URL fragment identifier from the string (if present).
	
	- (NSString *)stringByAppendingURLFragment:(NSString *)fragment;
		
This method will append a URL fragment identifier to the string. It will also correctly concatenate the string to an existing fragment identifier if present. No attempt is made to analyse the structure of the fragment, or handle path concatenation correctly within the fragment string. If you need to do that, generate the fragment separately and then set it in one go.

	- (NSURL *)URLValue;
	
This method converts the string to a URL. It is similar to using `[NSURL URLWithString:]`, but with a couple of benefits: 1) It doesn't throw an exception if the string is nil, and 2) It automatically detects if the string is a file path and uses `[NSURL fileURLWithString:]` instead of `[NSURL URLWithString:]`, thereby eliminating a common source of bugs.
	
	- (NSURL *)URLValueRelativeToURL:(NSURL *)baseURL;
	
This method converts the string to a URL. It is similar to using `[NSURL URLWithString:relativeToURL]`, but won't throw an exception if the string is nil.
    
    - (NSString *)base64EncodedString;
    
Encodes the string as UTF8 data and then encodes that as a base-64-encoded string without any wrapping (line breaks). For more advanced base 64 encoding options, check out my Base64 library (https://github.com/nicklockwood/Base64).
    
    - (NSString *)base64DecodedString;
    
Treats the string as a base-64-encoded string and returns an autoreleased NSString object containing the decoded data, interpreted using UTF8 encoding. Any non-base-64 characters in the string are ignored, so it is safe to use a string containing line breaks or other delimiters. For more advanced base 64 decoding options, check out my Base64 library (https://github.com/nicklockwood/Base64).


NSURL Extensions
----------------------

RequestUtils extends NSURL with the following methods:

	+ (NSURL *)URLWithComponents:(NSDictionary *)components;
	
This builds a URL from the supplied dictionary of components. The dictionary may contain any or all of the following: scheme, host, port, user, password, path, parameterString, query, fragment (there are a set of constants defined for all of these values). All values are optional and the method will attempt to build as much of the URL as possible given the values supplied. All values supplied should be NSStrings, with the exception of the URLQueryComponent, which can be either an NSString or an NSDictionary of query parameters.
	
	- (NSDictionary *)components;
	
Returns a dictionary of URL components containing any or all of the following: scheme, host, port, user, password, path, parameterString, query, fragment.
	
	- (NSURL *)URLWithScheme:(NSString *)scheme;
	
Sets or replaces the current URL scheme with the supplied value.
	
	- (NSURL *)URLWithHost:(NSString *)host;
	
Sets or replaces the current URL host with the supplied value.
	
	- (NSURL *)URLWithPort:(NSString *)port;
	
Sets or replaces the current URL port with the supplied value.
	
	- (NSURL *)URLWithUser:(NSString *)user;
	
Sets or replaces the current URL user with the supplied value.
	
	- (NSURL *)URLWithPassword:(NSString *)password;
	
Sets or replaces the current URL password with the supplied value.
	
	- (NSURL *)URLWithPath:(NSString *)path;
	
Sets or replaces the current URL path with the supplied value.
	
	- (NSURL *)URLWithParameterString:(NSString *)parameterString;
	
Sets or replaces the current URL parameterString with the supplied value.
	
	- (NSURL *)URLWithQuery:(NSString *)query;
	
Sets or replaces the current URL query with the supplied value.
	
	- (NSURL *)URLWithFragment:(NSString *)fragment;
	
Sets or replaces the current URL fragment with the supplied value.


NSURLRequest Extensions
----------------------

RequestUtils extends NSURLRequest with the following methods:

    + (id)HTTPRequestWithURL:(NSURL *)URL method:(NSString *)method parameters:(NSDictionary *)parameters;
    
Creates a new, autoreleased NSURLRequest using the specified URL, HTTP method and parameters. For GET request, the parameters are encoded in the query string. For all other methods the parameters are encoded in the POST body.
    
    + (id)GETRequestWithURL:(NSURL *)URL parameters:(NSDictionary *)parameters;
    
Creates a new, autoreleased NSURLRequest using the GET method and the specified URL and parameters. The Parameters are URL encoded and set as the query string of the request URL. If the URL already includes a query string, the passed parameters will be appended to the existing query according to the rules used for the NSString `stringByMergingURLQuery:` extension method.
    
    + (id)POSTRequestWithURL:(NSURL *)URL parameters:(NSDictionary *)parameters;
    
Creates a new, autoreleased NSURLRequest using the POST method and the specified URL and parameters. The parameters are URL encoded and set as the body of the request.
    
    - (NSDictionary *)GETParameters;
    
Returns the GET (query string) parameters of the request as a dictionary.
    
    - (NSDictionary *)POSTParameters;
    
Returns the POST (request body) parameters of the request as a dictionary.

    - (NSString *)HTTPBasicAuthUser;
    
Returns the HTTP basic auth user. If the request has an `Authorization` header then that will be used to get the user, otherwise if there is a user specified in the URL itself, that will be used instead.
    
    - (NSString *)HTTPBasicAuthPassword;
    
Returns the HTTP basic auth password. If the request has an `Authorization` header then that will be used to get the password, otherwise if there is a password specified in the URL itself, that will be used instead.
    
    
NSMutableURLRequest Extensions
----------------------

RequestUtils extends NSMutableURLRequest with the following methods:

    - (void)setGETParameters:(NSDictionary *)parameters;
    - (void)setGETParameters:(NSDictionary *)parameters options:(URLQueryOptions)options
    
This method sets the GET parameters of the request using a dictionary. The Parameters are URL encoded and set as the query string of the request URL. Any existing query string parameters will be replaced by the new values. The optional options argument allows you to control how the query parameters are serialised (see the URLQueryOptions section below for details).
    
    - (void)addGETParameters:(NSDictionary *)parameters options:(URLQueryOptions)options

This method works like the `setGETParameters:` method except that the parameters are merged with the existing request parameters instead of replacing them. The rules by which the merging occurs are controlled by the options argument (see the URLQueryOptions section below for details).
    
    - (void)setPOSTParameters:(NSDictionary *)parameters;
    - (void)setPOSTParameters:(NSDictionary *)parameters options:(URLQueryOptions)options
    
This method sets the POST parameters of the request using a dictionary. The parameters are URL encoded and set as the body of the request. Any existing request body contents will be replaced. The optional options argument allows you to control how the POST parameters are serialised (see the URLQueryOptions section below for details).

    - (void)addPOSTParameters:(NSDictionary *)parameters options:(URLQueryOptions)options

This method works like the `setPOSTParameters:` method except that the parameters are merged with the existing POST parameters instead of replacing them. The rules by which the merging occurs are controlled by the options argument (see the URLQueryOptions section below for details).

    - (void)setHTTPBasicAuthUser:(NSString *)user password:(NSString *)password;

This method sets the HTTP basic auth username and password for the request. The username and password are set using the `Authorization` header of the request.


URLQueryOptions
---------------------

The query string manipulation methods offer some options for how query strings should be generated or interpreted. The options can be combined using the | (pipe) or + (plus) operators. Some of these options are mutually exclusive however, and should not be combined.

	URLQueryOptionDefault
	
The default option if no alternative is specified. When generating query strings from dictionaries using `URLQueryWithParameters:`, this maps to `URLQueryOptionUseArrays` meaning that any arrays of values will be expanded into multiple parameters.

When parsing query strings into parameter dictionaries using `URLQueryParameters` this maps to `URLQueryOptionKeepLastValue` meaning that if duplicate parameter keys are encountered, the value of the last duplicate will be used.
	
	URLQueryOptionKeepLastValue
	
This is the default option when parsing query strings into parameter dictionaries using `URLQueryParameters`. When this option is used and duplicate parameter keys are encountered, the value of the last duplicate will be used.

If this option is used when generating a query string from a dictionary using `URLQueryWithParameters:options:` and an array of values is provided for a given parameter, only the last value will be used to construct the query string.
	
	URLQueryOptionKeepFirstValue
	
If this option is used when parsing query strings into parameter dictionaries using `URLQueryParametersWithOptions:` and duplicate parameter keys are encountered, the first encountered value with a given key will be used.

If this option is used when generating a query string from a dictionary using `URLQueryWithParameters:options:` and an array of values is provided for a given parameter, only the first value will be used to construct the query string.
	
	URLQueryOptionUseArrays
	
If this option is used when parsing query strings into parameter dictionaries using `URLQueryParametersWithOptions:` and duplicate parameter keys are encountered, or the key has a "[]" array suffix, the values will be gathered into an array. The resultant parameter array will therefore contain a mix of array and string values, and you will need to add code to check the type of each dictionary entry when processing.

If this option is used when generating a query string from a dictionary using `URLQueryWithParameters:options:` and an array of values is provided for a given parameter, duplicate parameters will be added to the query string with that key.
	
	URLQueryOptionAlwaysUseArrays
	
If this option is used when parsing query strings into parameter dictionaries using `URLQueryParametersWithOptions:`, each dictionary entry will be created as an array, even if there is only one parameter with a given key (in which case the array will contain one item). The advantage of this is that you can guarantee that every dictionary entry will be an array, avoiding the need to add code to check the type of each dictionary entry when processing.

When generating a query string from a dictionary using `URLQueryWithParameters:options:`, this option is equivalent to `URLQueryOptionUseArrays` unless `URLQueryOptionUseArraySyntax` is also enabled. If `URLQueryOptionUseArraySyntax` is enabled, using `URLQueryOptionUseArrays` means that every key in the resultant query string will be suffixed with "[]", even if there is only one value associated with it. This may then impact how those parameters are interpreted later (i.e. as single-item arrays rather than strings). See `URLQueryOptionUseArraySyntax` for more discussion.
	
	URLQueryOptionUseArraySyntax

Unlike the other options, which are mutually exclusive, `URLQueryOptionUseArraySyntax` can be combined with `URLQueryOptionUseArrays` or `URLQueryOptionAlwaysUseArrays` to affect output. When generating a query string from a dictionary using `URLQueryWithParameters:options:`, setting this option means that any key with multiple value (or any key *period* if `URLQueryOptionAlwaysUseArrays` is used) will be suffixed with the "[]" syntax to indicate that it is part of a parameter array.

Use of the "[]" key suffix is not an official part of the RFC 1808 URL specification, but it is an ad-hoc standard used by a number of popular web server platforms including PHP, and so it can be useful to be able to interpret and/or generate query strings in this format. By default, this option is disabled.

`URLQueryOptionUseArraySyntax` has no effect when parsing query strings using the `URLQueryParameters:` method.


Release Notes
---------------

Version 1.0.4

- Fixed bug in -[NSURL URLWithPath:]

Version 1.0.3

- Fixed crash when request parameter dictionary keys or values are not strings
- Improved test coverage

Version 1.0.2

- Fixed bug in stringByAppendingURLQuery: method
- Updated to use native base64 support
- Now conforms to -Weverything warning level

Version 1.0.1

- Updated for Xcode 4.6
- URLEncoding now copes gracefully with  non-string values (e.g NSNumber)
- Fixed bug in stringByMergingURLQuery:options: method

Version 1.0

- Initial release.