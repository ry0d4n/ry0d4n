# Unpacking REvil Ransomware sodinokibi.exe
here i'll exaplain how to unpacak the famous <span style='color:red'>REvil</span> Ransomware.<br>
to download the original sample <a href='sample'>click here</a>.<br>
<hr>
## identifying packing <br>
we start first by loading the sample in <a href='http://www.exeinfo.byethost18.com/'>exeinfo</a> and it is showing that it is packed with an unknown packer.

![sodinokibi1](sodinokibi1.png)<br>

we have multiple choices to manually unpack the sample but i'll be choosing <a href='https://x64dbg.com/'>x32dbg</a> to unpack it, so fire up x32dbg and load the sample.

![sodinokibi2](sodinokibi2.png)<br>

at this point we dont know anything about the unpacking process or how it is implemented, one choice to use is to look for possible memory allocations and follow the addresses, you might get the unpacked version gets extracted inside the memory region<br>
to do this we have to look for <span style='color:red'>VirtualAlloc</span> API and follow the addresses returned in eax.

## dumping executable
press Ctrl + G and type VirtualAlloc and follow the jumps until you reach a call to the API, once you find it put a breakpoint and run until the breakpoint is hit.
  
now you have to make a single step forward and follow the value of eax in dump
  
![sodinokibi3](sodinokibi3.jpg)
  
as we can see its an empty space of memory but it is likely to be filled later with some content, usually malware authors tend to use <span style='color:red'>VirtualProtect</span> alot to change the protections on a piece of data in the virtual memory, it is an important API to put a breakpoint on too

put a breakpoint with bp VirtualProtect and run the executable, once you hit the breakpoint look to the empty region that was allocated before, you should see a PE file extracted in that region and thats the unpacked executable.

![sodinokibi4](sodinokibi4.png)

to dump the executable right click on the address in dump and select follow in memory map then right click on section and select dump memory to file.

until now we have an unpacked version of the malware but still we have no imports resolved.

## resolving IAT
in this malware specifically the imports are dynamically resolved at run time, it is calling a function at the beginning of execution and when analyzing that functions you'll notice that it is looping through hardcoded values which likely are the imports addresses

![sodinokibi5](sodinokibi5.png)

![sodinokibi6](sodinokibi6.png)
