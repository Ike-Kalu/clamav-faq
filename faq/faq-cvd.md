# ClamAV Virus Database FAQ

## How do I keep my virus database up to date?

ClamAV comes with _FreshClam_, a tool which periodically checks for new database releases and keeps your database up to date. It is encouraged that you update to at least version 0.103.2, which respects our bandwidth limitations.

---

## How often is the virus database updated?

The virus database is usually updated once or twice per day. Sign up for our [VirusDB mailing list](https://lists.clamav.net/mailman/listinfo/clamav-virusdb) to see our response times to new threats. The virusdb team tries to keep up with the latest threats in the wild.  You can contribute to make the virusdb updating process  more efficient by submitting samples of viruses via our "[Contact](https://www.clamav.net/contact)" page on ClamAV.net.

---

## The last CVD update crashed my ClamAV installation. Why?

Before publishing a CVD update, we verify that it can be correctly loaded by the last two stable release series of ClamAV. Please stay tuned to our [EOL policy](https://www.clamav.net/documents/end-of-life-policy-eol) for what versions are actively supported.

---

## The last CVD update detects a lot of false positives on my system. Why?

Before publishing a CVD update, we test it for false positives using the latest stable release of ClamAV. If you want to avoid problems with false positives, you must run the latest stable version of ClamAV.

---

## I tried to submit a sample through the web interface, but it said the sample is already recognized by ClamAV. My ClamScan tells me it's not. I have already updated my database and ClamAV engine, what's wrong with my setup?

Please run ClamScan with the `--alert-broken` option. Also check that FreshClam and ClamScan are using the same path for storing/reading the database.

---

## I found an infected file in my HD/USB/mailbox, but ClamAV doesn't recognize it yet. Can you help me?

Our virus database is kept up to date with the help of the community. Whenever you find a new virus which is not detected by ClamAV you should [complete this form](https://www.clamav.net/reports/malware). The virusdb team will review your submission and update the database if necessary. Before submitting a new sample: - check that the value of `DatabaseDirectory`, in both `clamd.conf` and `freshclam.conf`, is the same - and update your database by running FreshClam to ensure you've scanned it with the latest virus database.

---

## I'm running ClamAV on a lot of clients on my local network. Can I serve the cvd files from a local server so that each client doesn't have to download them from your servers?

Sure, you can find more details on our [Private Local Mirror page](https://www.clamav.net/documents/private-local-mirrors).

1. If you want to take advantage of incremental updates, install a proxy server and then configure your FreshClam clients to use it (watch for the HTTPProxyServer parameter in `freshclam.conf`).

2. The second possible solution is to:

  * Configure a local webserver on one of your machines (say `machine1.mylan`)

  * Let FreshClam download the `*.cvd` files from [http://database.clamav.net](http://database.clamav.net) to the webserver's *DocumentRoot*.

  * Finally, change `freshclam.conf` on your clients so that it includes:


    `DatabaseMirror machine1.mylan`

    `ScriptedUpdates off`

    First the database will be downloaded to the local webserver and then the other clients on the network will update their copy of the database from it.

    _Important_:  For this to work, you have to add `ScriptedUpdates off` on all of your machines!

---

## I can't wait for you to update the database! I need to use the new signature NOW!

No problem, save your own signatures in a text file with the appropriate extension (see [our signature writing documentation](https://www.clamav.net/documents/creating-signatures-for-clamav) for more information). Put the signature file in the same directory where the `.cvd` files are located. ClamAV will load it after the official `.cvd` files. You do not need to sign the `.db` file.

---

## Can I download the virusdb manually?

This practice is discouraged, please use either FreshClam or CvdUpdate to update your definitions.  Please check out our [FreshClam FAQ](https://www.clamav.net/documents/freshclam-faq) and our [Private Mirror Documentation](https://www.clamav.net/documents/private-local-mirrors) for further information and links to CvdUpdate.

---

## I am getting error codes such as 403, 429, etc when FreshClam (or other update system) attempts to download updates

Are you attempting to download safebrowsing.cvd and getting a 403?  If so, [take a look at this blog post](https://blog.clamav.net/2021/04/are-you-still-attempting-to-download.html), otherwise check out our [Freshclam FAQ](https://www.clamav.net/documents/freshclam-faq) under the section on "Error Codes"

---

## I can't resolve `current.cvd.clamav.net`! Is there a problem with your/my DNS servers?

`current.cvd.clamav.net` has got only a TXT record, not a type A record! Try this command:

`$ host -t txt current.cvd.clamav.net`

Please note that some not RFC compliant DNS servers (namely the one shipped with the *Alcatel* (now *Thomson*) **SpeedTouch 510 modem**) can't resolve `TXT` record. If that's the case, please recompile ClamAV with the flag `--enable-dns-fix` if using `./configure` or `-D ENABLE_FRESHCLAM_DNS_FIX=ON` if using CMake.

---

For other questions regarding issues with Freshclam, see our [Freshclam Troubleshooting FAQ](https://www.clamav.net/documents/troubleshooting-faq).
