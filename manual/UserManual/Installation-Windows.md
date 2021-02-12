# Installing ClamAV on Windows

If you wish to build ClamAV from source using Visual Studio 2015, please head over to the [Win32 ClamAV Build Instructions](https://github.com/Cisco-Talos/clamav-devel/blob/dev/0.101/win32/README.md) located in our source release materials on [ClamAV.net](https://www.clamav.net/downloads) and on [GitHub](https://github.com/Cisco-Talos/clamav-devel).

---

## Install using the ClamAV Windows Installer

Important: Installing ClamAV using the Installer will require Administrator privileges.

1. Download: <http://www.clamav.net/downloads/production/ClamAV-0.103.1.exe>
2. Locate the file in your Downloads directory.
3. Right-click on `ClamAV-0.103.1.exe` and select `Run as administrator`. You may receive a warning message along the lines of "Windows protected your PC".  Select `More info` and then select `Run anyway`.
4. Select `I accept the agreement` and click `Next`.
5. Click `Next` again. If you've removed a previous installation of ClamAV, you may receive the prompt "The folder ... already exists...". If you do, select `Yes`.
6. Click `Install`.
7. Click `Finish`.
8. Press the Windows-key and type `powershell` but _DO NOT_ press `Enter`. Right-click on `Windows PowerShell` at the top of the menu and select `Run as administrator`. Your computer may warn you `Do you want to allow this app to make changes to your device?`  Click `Yes`.
9. Verify that the prompt in the PowerShell window looks like this:
  <pre>
      PS C:\WINDOWS\system32>
  </pre>

10. In the Adminstrator PowerShell window, enter the following to navigate to the ClamAV install directory:
  <pre>
      cd "c:\program files\clamav"
  </pre>

Continue on to "First Time Set-Up" below...

---

## Install using the ClamAV Portable Install Package

1. Download: <https://www.clamav.net/downloads/production/clamav-0.103.1-win-x64-portable.zip>
2. Unzip it.
3. Open the `clamav-0.103.1-win-x64-portable` directory.
4. Hold down Shift and then right-click on the background in the current directory (but not on one of the files). Select `"Open PowerShell window here"`. If that option doesn't appear, try again.

Continue on to "First Time Set-Up"...

---

## First Time Set-Up

In the PowerShell window, perform the following tasks:

* Run:
  <pre>
      copy .\conf_examples\freshclam.conf.sample .\freshclam.conf
      copy .\conf_examples\clamd.conf.sample .\clamd.conf
  </pre>
* Run:
  <pre>
      write.exe .\freshclam.conf
  </pre>

  WordPad will pop up. Delete the line that says "Example". You may also wish to set additional options to enable features or alter default behavior, such as the receive-timeout. Save the file and close WordPad.
* Run:
  <pre>
      write.exe .\clamd.conf
  </pre>

  WordPad will pop up. Delete the line that says "Example". You may also wish to set additional options to enable features or alter default behavior, such as enabling logging. Save the file and close WordPad.

---

## Additional notes about the config files and database directories

The install directory is but one of a few locations ClamAV may search for configs and for signature databases.

### Config files path search order

1. The content of the registry key:
   "HKEY_LOCAL_MACHINE/Software/ClamAV/ConfDir"
2. The directory where libclamav.dll is located:
   "C:\Program Files\ClamAV"
3. "C:\ClamAV"

### Database files path search order

1. The content of the registry key:
  "HKEY_LOCAL_MACHINE/Software/ClamAV/DataDir"
2. The directory "database" inside the directory where libclamav.dll is located:
  "C:\Program Files\ClamAV\database"
3. "C:\ClamAV\db"

---

## Next Steps

---

### Download the Signature Databases

Before you can start the ClamAV scanning engine (using either `clamd` or `clamscan`), you must _first_ have ClamAV Virus Database (.cvd) file(s) installed in the appropriate location on your system. The default location for these database files is C:\Program Files\ClamAV\database, the database directory of your ` (in Windows).

Continuing in the PowerShell window:

1. Run:
  <pre>
      .\freshclam.exe
  </pre>
2. freshclam will download some files and drop them in the database directory. This can take a minute or two depending on how fast your internet connection is. The files are a pretty large.
3. You are now ready to perform scans with ClamAV. If you using the portable install package, you may now copy the entire `clamav-0.100.1-win-x64-portable` directory to the computer(s) you wish to scan.

---

### Steps to Perform Basic Scanning

* Run this to scan the files in the current directory:
  <pre>
      .\clamscan.exe .
  </pre>

  This will scan the current directory. At the end of the scan, it will display a summary. If you notice in the clamscan output, it only scanned something like 60 files, even though there are more files in subdirectories. By default, clamscan will only scan files in the current directory.

* Run this to scan all the files in the current directory:
  <pre>
      .\clamscan.exe --recursive .
  </pre>

* Run this to scan ALL the files on your C: drive, it will take **quite** a while. Keep in mind that you can cancel it at any time by pressing `Ctrl-C`:
  <pre>
      .\clamscan --recursive C:\
  </pre>

* For more information on ways you can use clamscan, run:
  <pre>
      .\clamscan.exe --help
  </pre>

---

### Faster a-la-carte Scanning with ClamD

You may have noticed that `clamscan.exe` takes a while to get started. This is because it loads the signature database each time you start a scan. If you require faster scanning of individual files, you will want to use `clamd.exe` with `clamdscan.exe` instead. Please note that many of the command line options for clamscan should instead be configured in `clamd.conf` when using `clamd.exe`.

Continuing in the PowerShell window:

1. Run:
  <pre>
      .\clamd.exe
  </pre>

  The application will take a moment to load and then appear to hang, but it is in fact waiting for scanning commands from `clamdscan.exe`.

2. Open a second PowerShell window as you did above, in the same directory.

3. In the second PowerShell window, you can now run `clamdscan.exe` much the same way you did with `clamscan.exe` above.
  <pre>
      .\clamdscan.exe .
  </pre>

---

## Additional Notes about Windows-specific Issues

### Globbing

Since the Windows command prompt doesn't take care of wildcard expansion, minimal emulation of unix glob() is performed internally. It supports `*` and `?` only.

### File paths

Please always use the backslash as the path separator. SMB Network shares and UNC paths are supported.

#### Socket and libclamav API Input

The Windows version of ClamAV requires all the input to be UTF-8 encoded.

This affects:

- The API, notably the cl_scanfile() function
- ClamD socket input, e.g. the commands SCAN, CONTSCAN, MUTLISCAN, etc.
- ClamD socket output, i.e replies to the above queries

For legacy reasons ANSI (i.e. CP_ACP) input will still be accepted and processed as before, but with two important remarks:

1. Socket replies to ANSI queries will still be UTF-8 encoded.
2. ANSI sequences which are also valid UTF-8 sequences will be handled as UTF-8.

As a side note, console output (stdin and stderr) will always be OEM encoded, even when redirected to a file.
