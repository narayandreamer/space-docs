---
title: "Encrypting"
description: "Encrypting"
date: 2020-06-18T12:08:12+05:30
draft: false
weight: 1
---

Often our app has to store certain private information of our users. Such private data needs to be encrypted before storing for compliances. Space Cloud allows you to encrypt such fields in your request/response easily with the `encrypt` rule.

## How it works

The syntax for `encrypt` rule is:

{{< highlight javascript >}}
{
  "rule": "encrypt",
  "fields": "<array-of-fields>",
}
{{< /highlight >}}

> **The encrypt rule will always get resolved unless there's some problem with the configured AES key.**

The `encrypt` rule replaces the `fields` specified in the rule with their encrypted value. These fields can be present either in the request or response.

The values of the encrypted fields can be retrieved by using the [decrypt]() rule.

> **Performing an encrypt operation generates a different output each time even for the same input value. If you want to generate same output each time for a value, you should use the `hash` rule. However, hashed fields cannot be decrypted back.** 

### Encryption algorithm

Space Cloud uses AES encryption (CFB mode) to encrypt/decrypt fields.


The AES key used for encryption is configurable. Whenever you create a project through the Mission Control, it configures a random AES key for that project. This AES key can be changed later from the project settings in Mission Control. The AES key used for encryption should be a 32 byte string that is base64 encoded.


### Example

Let's say we want to encrypt the `email` and `name` fields of user before inserting it into users table. This is how we can use the `encrypt` rule to do that:
  
{{< highlight javascript >}}
{
  "rule": "encrypt",
  "fields": ["args.doc.email", "args.doc.name"]
}
{{< /highlight >}}

`args.doc` is nothing but a variable containing the document/record that the user is trying to insert. 

You can even encrypt the fields sent back to user in response by using the `args.res` variable. You can check out the [list of available variables]() in security rules for each operation.

Let's say the document to be inserted (`args.doc`) was:
{{< highlight javascript >}}
{
  "id": "1",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "dob": "26-04-1997",
  "role": "user"
}
{{< /highlight >}}

After passing through the `encrypt` rule, the `args.doc` would look like this:
{{< highlight javascript >}}
{
  "id": "1",
  "name": "oJsFxb2wVCA=",
  "email": "gJsFxbOQVCC2cOWoMQZFAMd7AWM=",
  "dob": "26-04-1997",
  "role": "user"
}
{{< /highlight >}}

This encrypted data will then be inserted by the crud module of Space Cloud.

## Combining encrypt with other rules

Encrypt rule can be easily combined with any other data masking operations or authorization logic by using the `and` rule. Check out the [documentation of and rule]().

**Example:** Allow a record to be inserted in users table only if the length of username is greater than 10. The email field in the record should be encrypted, while the password field should be hashed. ([hash rule]()). Here's how you can write this access control logic using `and` rule:

{{< highlight javascript >}}
{
  "rule": "and",
  "clauses": [
    {
    "rule": "match",
    "eval": ">",
    "type": "number",
    "f1": "length(args.doc.username)",
    "f2": 10 
    },
    {
      "rule": "encrypt",
      "fields": ["args.doc.email"]
    },
    {
      "rule": "hash",
      "fields": ["args.doc.password"]
    }    
  ]
}
{{< /highlight >}}

With the above security rule, a record will only get inserted whenever the `match` clause gets resolved, since the `encrypt` and `hash` rules always gets resolved. However, due to the nature of `and` rule, the `encrypt` and `hash` rules will only get processed when the `match` rule passes since they are after the `match` rule.