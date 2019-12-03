# Not-So-PrivateKeys

People are often the biggest risk in InfoSec.  
Like when they leave factory RSA private keys for firmware personalization (IMG4 ticket and baseband) in production code.  
That's exactly what Apple did in one of their macOS private frameworks.

The MobileDevice framework contained RSA private keys in the strings section of the binary.  
They seem to be used for development devices internally used by Apple.
Its up to anyone to take this further and test whether they can be used for personalizing firmware on production devices too.

To encourage other reverse engineers to do what is my passion, to document the unknown, I publish this document and the private keys as a manifesto and motivation for fellow reverse engineers.  
Because of that, chances are big that this repository will be taken down under the enforcement of the DMCA.  
Personally I think that would be unlawful due to the constraints of the act, but neither do I have the legal means to make that claim when it would happen.  
Taking down this repository for DMCA violation is an act of weakness and disrespectful to the owl of Athena.
If that will happen, I will find my freedom elsewhere and perhaps leave GitHub forgood.
Just remember that a meal never tastes great if you don't add onions.  
Those with culinairy principes will know what I mean by that.  

## Fantastic keys and where to find them

Path: /System/Library/PrivateFrameworks/MobileDevice.framework/MobileDevice  
How?  

1. Run the scripts (dump_keys.py, dump_certs.py, dump_pubkeys.py) in the [Exploits](https://github.com/userlandkernel/plataoplomo/tree/master/exploits) folder of this repository on a mac.  
2. You now will have all the private keys and certificates in your current directory.  
3. (Optional) If you're familiar with r2 and know how binaries work thus where to look for strings, then good news: The private keys have a symbol that exactly explains what they are used for.  
4. Profit!

## Future research
- Am not attempting any personalization with this keys, got other more important projects that need work (sorry).
- Be smart, try, perhaps curse a million times, succeed or find other cool insecurities :)
- Use img4tool by Tihmstar if that helps with the personalization process, well if you get it to compile xD
