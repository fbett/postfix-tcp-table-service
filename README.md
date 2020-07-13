# Postfix: TCP Table Service

As i was not able to find a detailed description of the expected results, here are the results of my research:

Manual:
http://www.postfix.org/tcp_table.5.html

## General Requirements:
- Request format: "get [subject]"
- Each request ends with a new line character "\n"
- Each reply must end with a new line character "\n"
- Subject: All non-ascii characters including spaces are encoded (in PHP use rawurldecode to decode)
- Postfix will may reuse an existing client connection
- Postfix will may open multiple client connections
- If there are multiple scopes, postfix will request each of them until there is a match.
- If there are multiple scopes, postfix will request sometimes only the exact address without requesting the other scopes.

##  How to Test using "postmap"
postmap -q "subject" tcp:[IP]:[Port]]

## Reply Format
Result           | Reply
---------------- | ---------------
Not found/Reject | 500 SPACE text NEWLINE
Error            | 400 SPACE text NEWLINE
Success          | 200 SPACE text NEWLINE

This format is mandatory. An empty text is not accepted.

Text should be encoded like the subject (in PHP use rawurlencode to encode)

If no result is found, the result should be:
```
500 NO%20RESULT\n
```

## Example: Address Lookup Table (f.e. sender_bcc_maps)
Scope: Domain or exact email address (including extension)

Probably unexpected: This lookup is performed for outgoing and incoming messages.

Result          | Reply
--------------- | ---------------
Match not found | 500 NO%20RESULT\n
Match found     | 200 [encoded whitespace seperated list of bcc recipients]\n

## Example: transport_maps
Scope: General (request subject: *) or exact email address (including extension)

Result          | Reply
--------------- | ---------------
Match not found | 500 NO%20RESULT\n
Match found     | 200 [encoded transport map]\n

For local delivery (subject is an account and there is no transport map set)
```
200 [encoded ":"]\n
```

If subject is not an account and there is no transport map set, but should be redirected:
```
500 NO%20RESULT\n
```

## Example: Access Lookup Table (f.e. check_recipient_access)
Scope: Exact email address (including extension)

Result   | Reply
-------- | ---------------
Continue | 200 DUNNO\n
OK       | 200 OK\n

If reply is "500 NO%20RESULT\n", message will be rejected (recipient not found)

Please see access manual for other actions:
http://www.postfix.org/access.5.html

## Example: virtual_mailbox_maps
Scope: Domain or exact email address (including extension)

Result          | Reply
--------------- | ---------------
Match not found | 500 NO%20RESULT\n
Match found     | 200 [encoded whitespace seperated list of email addresses]

In case of a "catch all" match, return:
```
200 [encoded "@domain.tld"]\n
```

Return a "match not found", if the message should be local delivered (result of transport_map is ':')

## Example: virtual_mailbox_domains
Scope: Domain

Result          | Reply
--------------- | ---------------
Match not found | 500 NO%20RESULT\n
Match found     | 200 [encoded domain name, f.e. "domain.tld"]\n

## Example: virtual_alias_maps
Scope: Domain or exact email address (including extension)

Result          | Reply
--------------- | ---------------
Match not found | 500 NO%20RESULT\n
Match found     | 200 [encoded mailbox map, f.e. path information ]\n

Return a "match not found", if the message should not be local delivered (result of transport_map is not ':')
