# JWT Header Parameter Injection

The following ones are of particular interest to attackers.

* `jwk` (JSON Web Key) - Provides an embedded JSON object representing the key.
* `jku` (JSON Web Key Set URL) - Provides a URL from which servers can fetch a set of keys containing the correct key.
* `kid` (Key ID) - Provides an ID that servers can use to identify the correct key in cases where there are multiple keys to choose from. Depending on the format of the key, this may have a matching `kid` parameter.
