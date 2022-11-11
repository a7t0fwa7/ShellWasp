# ShellWasp
ShellWasp is a new tool to faciliate creating shellcode utilizing syscalls, released at DEF CON 30 on August 12, 2022. It will also soon be a briefing at Black Hat Middle East and Africa this November, 2022! It is likely some more sample syscall shellcodes will be made released then or shortly thereafter.

ShellWasp automates building templates of syscall shellcode. The template is intended to be just that - a template. The user still must determine what parameter values to use and how to generate them. The intent is to simplify the process for those desiging to create syscall shellcode by hand. Nearly all user-mode syscalls supported, including all the ones I could find function prototypes for. ShellWasp also solves the syscall portability problem for syscalls. It uses the 32-bit PEB to identify OS build. ShellWasp creates a syscall arrray, allowing the current syscall values (SSNs) to be found at runtime, rather than having to be hardcoded, which can limit how you can use them across OS builds. ShellWasp takes care of managing the syscall array, so if a syscall is used multiple times, there will only be one entry in the syscall array. Thus, ShellWasp will allows syscall values (SSNs) to be obtained dynamically.

ShellWasp supports Windows 7/10/11. It uses both existing syscall tables, as well as newly created syscall tables for newer versions of Windows 10 & 11. These are created through an original process, parsing WinDbg results. It is important to note that my research has indicated that in newer OS builds, some syscalls no longer increment by 1, so ShellWasp utilizes precomputed syscall tables in JSON format.

The shellcode size created by ShellWasp is relatively small in size. Users can select the OS builds to support, and it is recommend to use perhaps just some of the most recent ones from Windows 7/10/11, rather than every possible one. This can help keep size more manageable. Additionall, the way in which syscalls are called differs from Windows 7 and Windows 10/11. ShellWasp will automatically take care of that based on the selections the user makes. We have created syscall shellcode that works across all three OS, using our technique.

## Using ShellWasp
The Assembly generated by ShellWasp is compact in size. Additionally, as many people have automatic Windows update, it may be desirable to select only more recent OS builds, rather than every possible one, and this helps reduce size as well. Users can easily and quickly rearrange syscalls in shellcode. 

ShellWasp only supports Windows 7/10/11 at the moment, as a desing choice. It is easy to select desired Windows releases via config file or UI. Changes can also be saved to the to config.


![image](https://github.com/Bw3ll/ShellWasp/blob/main/images/shellwasp1.png?raw=true)

![image](https://github.com/Bw3ll/ShellWasp/blob/main/images/shellwasp3.png?raw=true)

![image](https://user-images.githubusercontent.com/49998815/201258739-bc8e4f11-d737-4a1f-a8e5-7f827f701717.png)
Note: You select the OS builds to target--it is not necessary to target every single build--and you select the syscalls to use. The above is just a random illustration. ShellWasp takes care of a lot of the details, but you still need to build out the parameters and required structures.

## Usage
Download via GitHub and run it on the command line, e.g. `py shellWasp.py`

Desired settings for selected OS builds/releases as well as Windows syscalls can be added to the config file. They also may be added or changed in the UI. 

## Updates
On Nov. 1, 2022, support was added for Windows 10 22H2 and Windows 11 22H2. These are the newest Windows releases. Note: we do not support Insider preview builds nor Server. 

On Nov. 1, 2022, the mechanism by which the pointer to the syscall array is preserved has been changed. In testing shellcode with chains of several Windows syscalls, some stability issues were noted with values on the stack. In order to avoid those issues, it was decided to change the stack cleanup (`add esp, 0xXX`) and `pop edi`,  to `mov edi, [esp+0xYY]` - YY being the number of bytes that would have been "cleaned" from the stack. The `push edi` that follows is retained. ShellWasp maintains a pointer to the syscall array at edi, and since the actual syscall itself destroys the value contained in edi, there needs to be a way to restore it, after the return from the far jump to kernel-mode. It was felt this new Assembly would be a more stable way to accomplish this. Of course, another option could be to have a pointer to the syscall array stored at some location on ebp or other memory, and then that could be used to restore EDI. That would in some ways be simpler, as it would be possible to avoiding needing to count the number of bytes to go back. However, it was felt that `mov edi, [esp+0xYY]` would be safer for novices. If it was stored elsewhere in memory at a fixed location, such as the stack, it could be possible to accidentally overwrite it. Both approaches take minimal time and effort. 

## Installation
A setup file is provided to help ensure the correct libraries are installed. 
To set up ShellWasp, simply use the following command: `py setup.py install`

You may substitute *py* with *python* as need be. This will make sure that Colorama and keystone-engine are installed. Keystone is used to compile the Assembly, which ensures it is valid Assembly. However, the generated bytes are not currently displayed, as ShellWasp is intended to produce a template, whose parameters need to be customized. This may be enabled as an optional setting in the future.

If you do not wish to use the setup.py file, you can also install these manually. `pip install keystone-engine` and `pip install colorama` may be used for that purpose.

## Coming Soon
I will release a script that can be used in WinDbg to determine the SSNs (syscall values) for a given release. A supporting script can be used to convert it to a Python dictionary/json. In all, it takes a few minutes of effort, though you must have WinDbg on the OS you are trying to extract values from, and it should not be affected by EDR. The scripts will need to be cleaned up before they can be released. Note that I discovered that syscall values (SSNs) no longer are predictable once sorted, whichmakes some of the underlying concepts behind some leading syscall techniques not always reliable, as explained at DEFCON. This is a relatively new development, that I had not seen any discussion on. Many SSNs do remain predictable, but there are many that are decidedly NOT, and some in a rather deceptive fashion (it may look predictable if truncated!). Again, this will cause some popular techniques to not always work as intended. In my opinion, this seems to be a Microsoft mitigation effort. Thus, using pre-computed values seems most reliable. When looking at some tools to parse NTDLL, I noticed inaccurate results being produced. I determined the cause was that it was truncating the SSN. It would truncate it from a DWORD to a WORD, cutting off part of the value. That also seems to be a likely mitigation effort. Yes, some SSNs are now several digits long. The solution is relatively simple - use a debugger, like WinDbg, and script the process, and you avoid errors. I did not make an exhaustive study of these types of syscall tools that just parse NTDLL; I just noticed simply that one I was using was producing inaccurate results, and I knew I could write something very quickly, that would be guaranteed to be error-free. Ultimatley, what I created was just a quick script to help obtain accurate SSNs for new releases for ShellWasp. Others are welcome to use it themselves if they wish to use it for their own syscall tools or syscall-related purposes.

## License
This project is released under the terms of the MIT license.

## Note
Please note that this GitHub is currently incomplete, and it will be updated with more information at a later date, including install instructions.

Some UI design aesthetics inspired by Tarek Abdelmotaleb.
