---
description: >-
  Dalam zaman kuno Harimau Selatan, dikatakan bahawa, kadang-kadang kunci kepada
  masa depan tersembunyi di tempat yang jelas.
---

# Kapla Harimau Selatan

When redirected to the website, we will be greeted with this&#x20;

<figure><img src="../../../.gitbook/assets/image (542).png" alt=""><figcaption></figcaption></figure>

Luckily there is a path hinted at the source code

<figure><img src="../../../.gitbook/assets/image (543).png" alt=""><figcaption></figcaption></figure>

There is some PHP source code in the txt path

```php
<?php

# Allow the script to be accessed only from a specific origin (https://127.0.0.1)
header("Access-Control-Allow-Origin: https://127.0.0.1");

# Define the header name we want to check ('Origin')
$headerName = 'Origin';

# Define the expected value of the 'Origin' header
$headerValue = 'https://127.0.0.1';

# Define a secondary custom header name we want to check ('X-Custom-Header')
$secondaryHeaderName = 'X-Custom-Header';

# Define the expected value of the 'X-Custom-Header', which is base64 encoded
$secondaryHeaderValue = 'Sm9ob3IganVnYSBkaWtlbmFsaSBzZWJhZ2FpIEdfX19fX19fIG9sZWggb3JhbmcgU2lhbQ==';

# Convert the 'Origin' header name to a format suitable for the $_SERVER array
# (e.g., 'Origin' -> 'HTTP_ORIGIN')
$headerKey = 'HTTP_' . strtoupper(str_replace('-', '_', $headerName));

# Convert the 'X-Custom-Header' name to a format suitable for the $_SERVER array
# (e.g., 'X-Custom-Header' -> 'HTTP_X_CUSTOM_HEADER')
$secondaryHeaderKey = 'HTTP_' . strtoupper(str_replace('-', '_', $secondaryHeaderName));

# Check if both the expected headers exist in the request
if (isset($_SERVER[$headerKey]) && isset($_SERVER[$secondaryHeaderKey])) {

    # Retrieve the actual value of the 'Origin' header from the request
    $actualValue = $_SERVER[$headerKey];

    # Retrieve the actual value of the 'X-Custom-Header' from the request
    $actualSecondaryValue = $_SERVER[$secondaryHeaderKey];

    # Compare the actual values to the expected values
    if ($actualValue === $headerValue && $actualSecondaryValue === $secondaryHeaderValue) {
        # If both headers match the expected values, output the flag
        echo "The flag is 3108{this-is-fake-flag}";
    } else {
        # If the headers exist but don't match the expected values, output "Close enough"
        echo "Close enough";
    }
} else {
    # If one or both headers are missing, output "Denied!"
    echo "Denied!";
}

?>
```

After analyzing a bit there is some base64 at the custom header

<figure><img src="../../../.gitbook/assets/image (544).png" alt=""><figcaption></figcaption></figure>

Time to Google!

<figure><img src="../../../.gitbook/assets/image (545).png" alt=""><figcaption></figcaption></figure>

So the answer is "Gangganu". Now, based on this snippet of the code, if our values of the two headers match we should get the real flag

```php
# Compare the actual values to the expected values
    if ($actualValue === $headerValue && $actualSecondaryValue === $secondaryHeaderValue) {
        # If both headers match the expected values, output the flag
        echo "The flag is 3108{this-is-fake-flag}";
```

The two headers that we should add to fulfill the requirements are:

`Origin: htps://127.0.0.1`

`X-Custom-Header: Gangganu`

Pass it on Burp

<figure><img src="../../../.gitbook/assets/image (546).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (547).png" alt=""><figcaption></figcaption></figure>

Flag: <mark style="color:red;">`3108{d941697cea9e3f341864780b68416961}`</mark>
