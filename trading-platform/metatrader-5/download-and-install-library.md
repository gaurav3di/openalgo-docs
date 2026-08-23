# Download & Install Library

**Download the MQL5 Libray and Unzip it**

{% file src="../../.gitbook/assets/OpenAlgo.zip" %}

**Open MQL5 -> File -> Open DataFolder as shown below**

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

**Navigate to \MQL5\Include folder and paste the Unzipped OpenAlgo Folder here as shown below**

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Now the Installation Process is completed if you open MetaEditor you should see the Folders with Library Files as shown below

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

The `OpenAlgo` folder contains four files:

* `OpenAlgoApi.mqh` - the six order functions. This is the only file you include.
* `CommonDefs.mqh` - the `Exchanges`, `ProductTypes` and `PriceTypes` enums.
* `UrlParser.mqh` - splits your OpenAlgo URL into host, path and port.
* `WinINet.mqh` - the HTTP transport, built on `wininet.dll`.

Put the folder at `MQL5\Include\OpenAlgo`, so the full path to the main file is `MQL5\Include\OpenAlgo\OpenAlgoApi.mqh`. That matches the `#include <OpenAlgo/OpenAlgoApi.mqh>` line used on the next page. If MetaEditor was already open, close and reopen it so it picks up the new include folder.

Because `WinINet.mqh` imports `wininet.dll` and `kernel32.dll`, every EA built on this library needs **Allow DLL imports** enabled. Set it per EA in the Common tab of the EA properties dialog, or globally under Tools, Options, Expert Advisors.

