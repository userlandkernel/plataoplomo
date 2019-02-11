# plata o plomo
Plata O Plomo (Spanish: Silver or Lead) is a term used in Latin America for when someone is forced to accept a bribe.

He or she can either accept the bribe or get a lead bullet in the head.

This repository contains minor bugs and vulnerabilities that I found in iOS userland.

## iBooks Type Confusion [(Writeup)](writeups/ibooks.md) [(Poc)](exploits/ibooks_09_2018.sh)
A type confusion vulnerability in iBooks may lead to memory corruption.

An attacker with access to a paired device over usb can exploit this vulnerability to cause a persistant denial of service in the iBooks application.

An attacker may be able to corrupt memory in the iBooks application.

The issue can be fixed by improved serialization and sandboxing.

## MobileSlideShow Type Confusion [(Poc)](exploits/mobileslideshow_09_2018.sh)
A type confusion vulnerability in MobileSlideShow may lead to memory corruption.

## Assetsd Type Confusion [(Writeup)](writeups/assetsd.md)[(Poc)](exploits/assetsd_09_2018.sh)
A type confusion vulnerability in assetsd may lead to memory corruption.

The issue can be fixed by improved serialization.

## Launchd Type Confusion 
A type confusion vulnerability exists in the initialization of launchdaemons.

A local attacker may be able to execute arbitrary code with system privileges and persistancy.

The issue can be fixed by improved serialization.

## libsqlite3 Infoleak
A vulnerability exists in libsqlite3.

This is a known vulnerability but Apple has not patched it yet.

An attacker may be able to leak information about the address space of applications using the sqlite3 library.

## libsqlite3 Memory Corruption
A vulnerability exists in libsqlite3.

This is a known vulnerability but Apple has not patched it yet.

An attacker may be able to execute arbitrary code with system privileges in applications using the sqlite3 library.


## Webkit Denial of Service [(Poc)](exploits/webkit_09_2018.sh)
A remote attacker may be able to cause a denial of service in MobileSafari through maliciously crafted webcontent resulting in an application crash.

This vulnerability can be patched by improved memory management.

This issue seems to be patched in iOS 12.

_Quick Poc:_
```js
setInterval(function(){window.location.href='accounts://'+'A'.repeat(10000);},0.1);
```

# Kernel Denial of Service [(Writeup)](writeups/kernel-procalloc.md)[(Poc)](exploits/kernel-procalloc.c)
A local attacker may be able to cause a denial of service through a maliciously crafted application.

A logic flaw exists in process allocation.

This issue can be resolved through improved logic and improved sandboxing.

_Quick Poc:_
```Objc
for(int i = 0; i < 10000; i++) {
    dispatch_async(dispatch_get_main_queue(), ^(void){
        [NSThread detachNewThreadWithBlock:^(void){
              for(int j = 0; j < 10000; j++){
                execve("/SMOKE/A/LOT/OF/POT", NULL, NULL);
              }
        }];
    });
 ```

## Apple File Conduit Infoleak [Writeup](writeups/afc.md)
A local attacker may be able to list directories outside the sandbox.

A path traversal vulnerability exists in the afc service.

This issue can be resolved by improved path handling.

_Quick Poc:_
```sh
//afcclient can be found base64 encodedin data directory
afcclient ls ../../../../../../usr/libexec/ 
```

## Kernel Denial of Service [(Writeup)](writeups/kernel-zoneleak.md)[(Poc)](exploits/kernel-zoneleak.c)
A local attacker may be able to cause a denial of service through a maliciously crafted application.

This issue can be resolved by denying sandboxed processes to send large mach messages to a port and by denying the growth of the message queue on a machport.


## CVE-2019-7289 Shortcuts Path Traversal [(Writeup)](writeups/shortcuts-traversal.md)[(Poc)](exploits/shortcuts-traversal.dat)
Available for: Shortcuts 2.1.2 for iOS

A local user may be able to view senstive user information.

A parsing issue in the handling of directory paths was addressed with improved path validation.


## Shortcuts Arbitrary App launch [(Writeup)](writeups/shortcuts-lsapp.md)[(Poc)](exploits/shortcuts-lsapp.dat)
Available for: Shortcuts 2.1.2 for iOS

A local attacker may be able to launch hidden system applications.

A logical issue exists in the Apple Pay module allowing a bundle ID to be specified instead of defaulting to Apple pay.

## Shortcuts Type Confusion [(Writeup)](writeups/shortcuts-typeconfusion.md)[(Poc)](exploits/shortcuts-typeconfusion.dat)
Available for: Shortcuts 2.1.2 for iOS

A local attacker may be able to possibly execute arbitrary code with system privileges.

A logical issue in the passing of shortcut actions of a different type can lead to type confusion.

An attacker can use the Object Tree to leak an address therefore this issue may be exploitable.

