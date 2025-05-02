# Lateral Movement and Pivoting

Group of techniques used by attackers to move around a network. It is part of a cycle where attackers access new machines and elevate privileges, and extract credentials if possible.

<figure><img src="../../../.gitbook/assets/image (651).png" alt=""><figcaption></figcaption></figure>

Before going further we need to understand the definition of User Access Control (UAC). UAC is simply a security feature that helps prevent unauthorized changes to the computer. It does this by asking for permission before allowing any certain actions or changes that require administrator rights.

This table below is an example usage of UAC:

<table><thead><tr><th align="center">Local Administrator</th><th align="center"></th><th align="center">Domain Administrator</th><th data-hidden></th></tr></thead><tbody><tr><td align="center">Local Administrator Group</td><td align="center"><mark style="color:red;"><strong>Group</strong></mark></td><td align="center">Domain Administrator Group</td><td></td></tr><tr><td align="center">Cannot execute remote admin tasks (RPC, SMB, WinRM). Can only RDP<br><br>*Note: built-in <strong>Administrator</strong> account is <strong>not affected</strong> by this</td><td align="center"><mark style="color:red;"><strong>Privileges</strong></mark></td><td align="center">Full admin rights and remotely like using RPC, SMB, WinRM, PsExec</td><td></td></tr><tr><td align="center">Limited unless UAC is disabled or bypassed</td><td align="center"><mark style="color:red;"><strong>Lateral Movement Impact</strong></mark></td><td align="center">More dangerous xD</td><td></td></tr></tbody></table>
