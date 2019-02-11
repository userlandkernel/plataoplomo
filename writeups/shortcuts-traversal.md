# Shortcuts Path Traversal

It is not uncommon for new features to be vulnerable at release because many times programmers were rushed to work towards a deadline.  
Programmers take security serious but usually apply security only in the last stage of the development and therefore such easy flaws as path traversal can exist.  
However, with Apple Inc. you may have expected differently as Apple takes security and privacy very seriously in their mobile Operating System which is one of the main reasons why people choose Apple.  
As soon as I discovered the new Shortcuts Application I took a look at all of its features and found it a very useful app.  
Due to the many actions and features and the app being new I also expected it to contain security issues.  
I expected two issues to be in the application that I made a simple testcase for:

- Can a user abuse the file actions (CRUD) to write or read files outside the container, by traversing a implicit specified path.
- Can a user cause type confusion by an object of specific type be passed to an action that requires another type.

My first assumption turned out to be correct immediately, firstly I simply used the file:// URI, but luckily this did not give me access to directories outside the app sandbox.  
Secondly I tried creating a file in the upper directory with succession proving my assumptions.  
I then tried to read outside of the sandbox but this luckily did not work.  
Then based on earlier findings I knew that directories can be treated as files and the sandbox does not explicitly protect directories.  
I found a way to confuse two actions to see a folder as a file and sent it through imessage so that it can be uploaded to anywhere, this because it seems the normal save file action was checking the origin of the folder.  
So what happened underneath?

- A WFFile object is used and its path is set to a folder
- The object is passed to the next action that will read out the WFFile without checking whether it is a folder thus returning the raw representation of the folder.  
- The next action is invoked and uses the raw folder structure instead of a file, as again object type was not checked here.  

With the traversal I was able to copy the Shortcuts application and take an early look. 
Appstore binaries are always encrypted and information about the encryption can be found in a specific mach-o segment.  
However, this encryption is known to be very weak and not straightforward: It does not encrypt all data.  
Therefore I was able to derrive classnames from the Frameworks and see the entitlements used by the app per framework.  
I also found a python debugging script that the developers forgot to remove in their production framework.  
Knowing the entitlements and classes I now knew the app was written mainly in Objective-C and what the permissions of the app were.  

Because the shortcuts app runs as mobile the path traversal can be used to access any directory owned by mobile and copy it out.  
This gives access to the cache of keystrokes, sms database and even whatsapp messages!  

In my other writeup you will be able to read my hunt for typeconfusion vulnerabilities. 
Below you can find the shortcut that I used:

https://www.icloud.com/shortcuts/c3a65b80294f4802b052081dac13851a
