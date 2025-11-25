# Install certifi

If you’re encountering, SSLCertVerificationError: \[SSL: CERTIFICATE\_VERIFY\_FAILED] certificate verify failed: unable to get local issuer certificate, is common on macOS when Python is unable to verify SSL certificates due to a missing or inaccessible certificate store. This usually affects the ability to fetch data over HTTPS, such as when attempting to download or use resources like ngrok.

\
To resolve this issue on macOS, you need to install the certificates for Python. Follow these steps:

In this example I used python3.11 as the version replace it with python3.xx version in the below commands

&#x20;**1. Locate the Correct Path of Python Installation:**

To find out where Python 3.11 is installed on your system, use the which command. Open your Terminal and type:

```
which python3.11
```

Locate and Run the Install Certificates Script:

<figure><img src="../../.gitbook/assets/Install Python SSL Certifications.png" alt=""><figcaption></figcaption></figure>

Once you have the correct path to Python 3.11, you can attempt to locate the Install Certificates.command script. Typically, if you installed Python via Homebrew, the command might not directly exist, but Python installations downloaded from the official Python website typically include it.

Assuming you installed Python using Homebrew, you might not find an Install Certificates.command. Instead, you can ensure the certificates are installed by manually installing the certifi package and configuring your environment to use it. Here’s how you can do that:

2. **Install certifi:**

```
python3.11 -m pip install certifi
```

3. **Set the SSL\_CERT\_FILE Environment Variable:**

To manually specify the certificate file for Python to use, you can set the SSL\_CERT\_FILE environment variable to point to the certifi certificate file:

```
export SSL_CERT_FILE=$(python3.11 -m certifi)
```

4. **Retry Running Your Python Code:**

After setting up the certificates through certifi, try running your Python code again. This should resolve any SSL certificate issues.<br>

If you installed Python using a method other than Homebrew and the Python installation includes the Install Certificates.command, you would typically find it in the Python application directory within /Applications/Python 3.11 or similar. If this is the case, navigate to this directory in Finder, and you should be able to run the Install Certificates.command by double-clicking on it.

Ensure that you are using the correct paths and commands based on how and where Python is installed on your system.
