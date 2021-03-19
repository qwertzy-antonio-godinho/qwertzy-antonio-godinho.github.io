---
layout: post
title: Abusing Internet Archive
subtitle: Case study - Abusing Internet Archive to deliver Malware
tags: [malware, analysis]
---

# Twitter post

Having finished setup my lab environment, the time had come to start analysing malware. Now all I needed was samples to play around with. On the 16th of March 2021 I saw the following tweet which seemed like a good starting point:

![](../assets/Abusing-Internet-Archive/twitter-post.png)

Following on the above tweet I've decided to try to quickly download the files from the server and start analysing them.

# Objectives

- Improve analysis skills
- See how long it takes me to analyse
- Identify shortcomings with the toolset available in the lab
- See how far I can go without any prior knowledge of the sample
- Identify gaps in my analysis methodology
- Document the process

# Part 1 - Initial assessment
## Aquiring the samples

Using the web browser I typed the URL link and downloaded all files hosted on the website to a virtual machine. The list of file extensions included txt, xml, sqlite and torrent files from existing directories named atomic1 and eset_202102:

![](../assets/Abusing-Internet-Archive/file-list.png)

### Lessons learned:

- Lab needs to provide a way to automate downloading files recursively from a website. At the moment contenders available in the lab include wget and curl.

## A quick first look at the samples

While comparing the files the only difference was in the file name, each file was numbered in ascending order. Another observation made was, the files were simple text files:

![](../assets/Abusing-Internet-Archive/differences.png)

The first text file analysed was named "detect1.txt". This file contained a PowerShell script with the purpose of finding the Windows Defender status by using <span class="highlight-green">[Get-MpComputerStatus](https://docs.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus?view=windowsserver2019-ps&viewFallbackFrom=win10-ps)</span>. 

Depending on the result, one of two files will be downloaded. 

Unfortunately, I made a beginner's mistake at this point. I completely forgot to download the file "amsi1.txt" for further investigation (insert face palm here!). By now I'm thinking it's better to pick another tweet, but as I already have files to look at, instead I decide to carry on with the investigation.

![](../assets/Abusing-Internet-Archive/detect-script.png)

The new file to be downloaded is named "eset1.txt", and also contains a PowerShell script:

![](../assets/Abusing-Internet-Archive/eset1-1st-look.png)

Depending on the result of <span class="highlight-green">if (-not ([System.Management.Automation.PSTypeName]"BP.AMS").Type)</span> the script can take a different execution path. 

In case the condition is false, it executes <span class="highlight-green">[BP.AMS]::Disable()</span> to disable the Anti Malware Scan program. And in case the condition is true, it defines 2 variables.

![](../assets/Abusing-Internet-Archive/eset1-logic.png)

The first variable contains previously xor'd byte values, while the second variable contains what it seems like a key. Following is a block of instructions that apply the logic for every value in the byte values variable, it xor's the value against the predefined key value containing 5 elements according to the cursor position and its counter value. As an example, the first 6 operations of this logic return the following values:

1. ByteArray[0] = 121 xor 52 = 77
2. ByteArray[1] = 12 xor 86 = 90
3. ByteArray[2] = 210 xor 66 = 144
4. ByteArray[3] = 23 xor 23 = 0
5. ByteArray[4] = 62 xor 61 = 3
6. ByteArray[5] = 52 xor 52 = 0

I've decided to generate the file for further analysis without executing it. I've added the following lines on to the PowerShell script:

<pre><code class="bash">$string = [System.Text.Encoding]::Unicode.GetString($byteArray)
Write-Output $string | Out-File -FilePath c:\output\eset1.bin</code></pre>

The first line converts the variable to Unicode string (using 16 bits), while the second line allows to the results to a file.

To prevent the code injection operation I have commented a few lines to be able to safely generate the encoded file:

![](../assets/Abusing-Internet-Archive/eset1-commented-1.png)

![](../assets/Abusing-Internet-Archive/eset1-commented-2.png)

The final operation this script performs, is to download a new file named "atomic1.txt" using the <span class="highlight-green">IEX (New-Object Net.WebClient).DownloadString</span> command. 

This file is another PowerShell script, a longer one than the one in the previous file. The top of the file:

![](../assets/Abusing-Internet-Archive/atomic1-first-look.png)

And the end of the file:

![](../assets/Abusing-Internet-Archive/atomic1-endofscript.png)

To be able to increase the readability of the file, I've searched for any occurrence of ";" and added 3 empty lines just to increase visibility of the various instructions on the file.

![](../assets/Abusing-Internet-Archive/atomic1-eoi.png)

This allows me to quickly use the mini-map to spot large spaces between large variable texts:

![](../assets/Abusing-Internet-Archive/atomic1-minimap.png)

The file contains a condition, depending on if it can find the "aspnet_compiler.exe" file in directory "C:\Windows\Microsoft.NET\Framework\v2.0.50727\" two files are generated. At this point I decided to break the file in two files named "atomic1" and "atomic2", one to generate the positive condition files and another one to generate the negative condition files.

To be able to generate the files contained in the two variables defined in the file, I've commented the ```if([System.IO.File]::Exists("C:\Windows\Microsoft.NET\Framework\v2.0.50727\aspnet_compiler.exe"))``` line:

![](../assets/Abusing-Internet-Archive/atomic1-commented-1.png)

And commented out the injection code ```[Reflection.Assembly]::Load($Cli555).GetType('RunPe.RunPe').GetMethod('Run').Invoke($null,[object[]] ('C:\Windows\Microsoft.NET\Framework\v2.0.50727\aspnet_compiler.exe',$Cli444))``` line:

![](../assets/Abusing-Internet-Archive/atomic1-commented-2.png)

The same commented lines in "atomic1":

![](../assets/Abusing-Internet-Archive/atomic2-commented-1.png)

The same commented lines in "atomic2":

![](../assets/Abusing-Internet-Archive/atomic2-commented-2.png)

## Generating the payloads

Using the same approach as with "eset1.txt" file, I've converted each of the variables to Unicode and used ```Write-Output``` and ```Out-File``` to execute the files in a sand-box virtual machine to generate the files:

![](../assets/Abusing-Internet-Archive/generated-files.png)

## Basic static analysis

After noticing the same file size of a few of the files, I've opened the files in HashMyFiles application to see the hashes of each individual file. Regardless of ```if([System.IO.File]::Exists("C:\Windows\Microsoft.NET\Framework\v2.0.50727\aspnet_compiler.exe"))``` being installed of not, the files generated are the same:

![](../assets/Abusing-Internet-Archive/same-hashes.png)

With that out of way, it was time to identify the files. Each file came up as "Unknown" in Detect It Easy:

![](../assets/Abusing-Internet-Archive/die.png)

After opening the files in a Hex Editor program, I noticed the first 2 bytes in each file were preventing them from being identified as PE files: 

![](../assets/Abusing-Internet-Archive/first-two-bytes.png)

With the first 2 bytes removed from the files, Detect It Easy had no problem in identifying the files. The file generated from "eset1.txt" was identified as a .NET(v4.0.30319) file, and the linker used was Microsoft Linker(11.0)[ DLL32 ]:

![](../assets/Abusing-Internet-Archive/eset1_fixed.png)

File "atomic-cli555" reported that a .NET obfuscator named Confuser(1.X) had been used to obfuscate the .NET (v2.0.50727) file:

![](../assets/Abusing-Internet-Archive/atomic1-555_fixed.png)

Going back to the "eset1" file, using DNSpy decompiler to take a look at the source code:

![](../assets/Abusing-Internet-Archive/eset1-decompiled.png)

After further online investigation, the source code that served as inspiration for the author of the script was found in this blog post [Citadel Cyber Security](https://www.citadel.co.il/Home/Blog/1008):

![](../assets/Abusing-Internet-Archive/file_eset1_citadel.png)

File "atomic1-cli444", Detect It Easy only reported the linker information, it seems this file needs to be further worked on:

![](../assets/Abusing-Internet-Archive/atomic1-444_fixed.png)

Another note was that file "atomic1-cli444" had its .reloc section being flagged as packed in Detect It Easy:

![](../assets/Abusing-Internet-Archive/atomic1-444_fixed-packed-section.png)

By opening "atomic1-cli555" in DNSpy I was left confused, as expected:

![](../assets/Abusing-Internet-Archive/atomic1-555_confused.png)

After searching GitHub for an unpacker that could get rid of the obfuscation that has been applied to the file, I found [https://github.com/ViRb3/de4dot-cex/releases](https://github.com/ViRb3/de4dot-cex/releases), which was able to deobfuscate some parts of file before throwing an error message:

![](../assets/Abusing-Internet-Archive/atomic1-555_fixed-deconfused.png)

By having part of the deobfuscated file I was able to better understand it by searching online and it seems the functionality in the script is derived from a .NET C# script dating back from 2015 named "Aeonhack RunPE C#" (VB version is also available) in [pastebin](https://pastebin.com/Dzhad8rB):

![](../assets/Abusing-Internet-Archive/atomic1-555_possible-source.png)

The following image shows the code available in the Pastebin script, side by side with the deobfuscated file:

![](../assets/Abusing-Internet-Archive/file_atomic1-555_pastebin.png)

It was then time to take the then fixed PE files hashes and search online for information about them:

![](../assets/Abusing-Internet-Archive/sha-256.png)

Quickly searching Hybdrid-Analysis, OTX.AlienVault and VirusTotal returned no results.

![](../assets/Abusing-Internet-Archive/osint-otx-av-hybridanalysis.png)

So I decided to upload the files and see if anything would come up:

![](../assets/Abusing-Internet-Archive/vt-eset1.png)
![](../assets/Abusing-Internet-Archive/vt-atomic444.png)
![](../assets/Abusing-Internet-Archive/vt-atomic555.png)

# Part 2 - A deeper look