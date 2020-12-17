To build scan_image project, 

you must put [libpng](https://github.com/ShadowsocksR-Live/libpng.git) project and it submodule (zlib) to 

root folder of zbar project.

and use VS 2010+ open zbar_qrcode.sln to build it.

like this:

```
zbar
| 
+--- ...
+---java
+---libiconv
+---libpng      <-- libpng files place this folder
+---perl
+--- ...
+---win32
|       readme.md    <-- this readme file
+---zbar
+--- ...
```
