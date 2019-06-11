# Fix maps in Picasa

Close Picasa and and perform following steps:

* Open Registry Editor - `regedit.exe`.
* Navigate to: `HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_BROWSER_EMULATION`.
* Create a new DWORD entry with name: `picasa3.exe` and Hexadecimal value: `2af8` (Decimal: 11000).

Open Picasa and embedded maps module should now work.
