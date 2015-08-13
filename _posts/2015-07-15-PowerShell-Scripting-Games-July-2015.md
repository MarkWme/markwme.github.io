---
layout: post
title:  "PowerShell Scripting Games - July 2015 Puzzle"
categories: microsoft powershell
tags: powershell "scripting games"
excerpt_separator: <!--more-->
---
The best way to get better at something is to practice it as often as possible. I've always enjoyed scripting and consider myself something of a VBScript wizard!  But like a lot of VBScript aficionados I've taken my time to switch to PowerShell. Why?  Well, I've got a whole bunch of VBScripts that work just fine. But since Windows Server 2012 arrived with PowerShell v3.0, it's been increasingly difficult to ignore PowerShell. And although I do have a great back catalogue of .vbs files, there's no denying that these days PowerShell could probably make a better job of it.

So, to get better at PowerShell, I look for opportunities to use it as often as possible. It's not just a great tool for automating system administration tasks, it also comes in handy in a wealth of other situations. Like processing the Sales History file that eBay provides to my wife for her online store. Another good way to practice is to take part in the [PowerShell.org Scripting Games][PowerShellOrg]. This used to run as an annual event, but it was recently decided to make this into a regular monthly challenge.

Rather than just post a solution, I thought it might be helpful to others to post a bit about the thought process that I followed to solve July's puzzle. By the way, this is by no means the *right* or the only solution. The great thing about PowerShell is that there are often several *right* ways to solve a problem. Ultimately, the *right* solution is one that works!

<!--more-->

#The July 2015 Puzzle#

**Write a one-liner that produces the following output**

~~~
PSComputerName ServicePackMajorVersion Version  BIOSSerial
-------------- ----------------------- -------  ----------
win81                                0 6.3.9600 VMware-56 4d 09 1 71 dd a9 d0 e6 46 9f
~~~

The puzzle also included some challenges:

* Try to use no more than one semicolon total in the entire one-liner

* Try not to use ForEach-Object or one of its aliases

* Write the command so that it could target multiple computers (no error handling needed) if desired

* Want to go obscure? Feel free to use aliases and whatever other shortcuts you want to produce a teeny-tiny one-liner.

The first thing that jumped out at me was the need to output the BIOS Serial Number and from past experience I knew straight away that means having to use WMI. In current versions of PowerShell, the CIM cmdlets are the preferred way to do this.

So, how do you find out where the BIOS Serial Number is buried in the wealth of data that WMI provides?  Well, one way to do this is with the `Get-CimClass` cmdlet. The `-ClassName` parameter takes wildcards, so we can use that to find anything that mentions BIOS simply by placing asterisks before and after the word BIOS, meaning find anything that has the word BIOS buried somewhere in it.

{% highlight powershell %}
Get-CimClass -ClassName *BIOS*
{% endhighlight %}

~~~
   NameSpace: ROOT/cimv2

CimClassName                        CimClassMethods      CimClassProperties
------------                        ---------------      ------------------
CIM_BIOSElement                     {}                   {Caption, Description, InstallDate, Name...}
Win32_BIOS                          {}                   {Caption, Description, InstallDate, Name...}
CIM_VideoBIOSElement                {}                   {Caption, Description, InstallDate, Name...}
Win32_SMBIOSMemory                  {SetPowerState, R... {Caption, Description, InstallDate, Name...}
CIM_VideoBIOSFeature                {}                   {Caption, Description, InstallDate, Name...}
CIM_BIOSFeature                     {}                   {Caption, Description, InstallDate, Name...}
Win32_SystemBIOS                    {}                   {GroupComponent, PartComponent}
CIM_VideoBIOSFeatureVideoBIOSEle... {}                   {GroupComponent, PartComponent}
CIM_BIOSFeatureBIOSElements         {}                   {GroupComponent, PartComponent}
CIM_BIOSLoadedInNV                  {}                   {Antecedent, Dependent, EndingAddress, StartingAddress}
~~~

Ok, so quite a few things to choose from here, so let's see what information each one contains until we hit what we're looking for. We can use `Get-CimInstance` to see what data is returned.  Let's start with the top entry - `CIM_BIOSElement`.

`Get-CimInstance -ClassName CIM_BIOSElement`

~~~
SMBIOSBIOSVersion : A08
Manufacturer      : Dell Inc.
Name              : A08
SerialNumber      : 1Y47X11
Version           : DELL - 1072009
~~~
Ok, so we have a BIOS serial number. But, we also need a computer name, version and service pack version. These are all things that relate to the Operating System. Let's turn to `Get-CimClass` again and see if we can find anything that relates to Operating Systems.

`Get-CimClass -ClassName *OperatingSystem*`

~~~
   NameSpace: ROOT/cimv2

CimClassName                        CimClassMethods      CimClassProperties
------------                        ---------------      ------------------
CIM_OperatingSystem                 {Reboot, Shutdown}   {Caption, Description, InstallDate, Name...}
Win32_OperatingSystem               {Reboot, Shutdown... {Caption, Description, InstallDate, Name...}
Win32_OperatingSystemAutochkSetting {}                   {Element, Setting}
Win32_SystemOperatingSystem         {}                   {GroupComponent, PartComponent, PrimaryOS}
CIM_OperatingSystemSoftwareFeature  {}                   {GroupComponent, PartComponent}
Win32_OperatingSystemQFE            {}                   {Antecedent, Dependent}
~~~

As before, a few options to choose from. Let's start at the beginning with `CIM_OperatingSystem`. We'll pass that to `Get-CimInstance` and see what it finds.

`Get-CimInstance -ClassName CIM_OperatingSystem`

~~~
SystemDirectory     Organization BuildNumber RegisteredUser   SerialNumber            Version 
---------------     ------------ ----------- --------------   ------------            ------- 
C:\Windows\system32              9600        user             00230-00905-02714-AB338 6.3.9600
~~~

So that give us a version. What about the other values we need to obtain? By default, PowerShell makes use of predefined Views and Default Display Property Sets to limit the output from a command to what is expected to be the most useful data. Often commands can return a lot more information, so you'll need to dig a little deeper to see what else is available. Here we can make use of Get-Member

`Get-CimInstance -ClassName CIM_OperatingSystem | Get-Member`

To save some space on this post, here's a modfied version of the output from that command. But please do try running it on your own computer to see the full result.

~~~
   TypeName: Microsoft.Management.Infrastructure.CimInstance#root/cimv2/Win32_OperatingSystem

Name                                      MemberType  Definition
----                                      ----------  ----------
...
PSComputerName                            Property    string PSComputerName {get;}
...
ServicePackMajorVersion                   Property    uint16 ServicePackMajorVersion {get;}
...
Version                                   Property    string Version {get;}
~~~

Ok, so we have three of the values we're looking for in `CIM_OperatingSystem` and one in `CIM_BIOSElement`. Now we have to get them into one table. The first three values can simply be picked from `CIM_OperatingSystem` using `Select-Object`.

`Get-CimInstance -ClassName CIM_OperatingSystem | Select-Object PSComputerName, ServicePackMajorVersion, Version`

~~~
PSComputerName ServicePackMajorVersion Version 
-------------- ----------------------- ------- 
                                     0 6.3.9600
~~~

The PSComputerName value is blank, but we will fix that later when we address the part of the challenge around working with multiple computer names. How do we get the BIOS Version number in there too? We can use the nifty *custom properties* feature of PowerShell. Custom properties use a hash table to append additional information to an object. The general format of the hash table for a custom property is as follows:

`@{name="keyname"; expression={script block}}`

`Name` will become the name of our new column heading. The interesting part though is `{script block}` because that means we can put a script in there!  So:

`@{name="BIOSSerial"; expression={Get-CimInstance -ClassName CIM_BIOSElement | Select-Object SerialNumber}}`

So, if we now append that to our Select-Object above, we end up with the following

`Get-CimInstance -ClassName CIM_OperatingSystem | Select-Object PSComputerName, ServicePackMajorVersion, Version, @{name="BIOSSerial"; expression={Get-CimInstance -ClassName CIM_BIOSElement | Select-Object SerialNumber | Select-Object -ExpandProperty SerialNumber}}`

~~~
PSComputerName ServicePackMajorVersion Version  BIOSSerial             
-------------- ----------------------- -------  ----------             
                                     0 6.3.9600 @{SerialNumber=1Y47X11}

~~~

Dammit! Not quite what we wanted. We need to get at that SerialNumber key so we can display just the value. We can do that with the `ExpandProperty` option of `Select-Object`

`Get-CimInstance -ClassName CIM_OperatingSystem | Select-Object PSComputerName, ServicePackMajorVersion, Version, @{name="BIOSSerial"; expression={Get-CimInstance -ClassName CIM_BIOSElement | Select-Object SerialNumber | Select-Object -ExpandProperty SerialNumber}}`

~~~
PSComputerName ServicePackMajorVersion Version  BIOSSerial
-------------- ----------------------- -------  ----------
                                     0 6.3.9600 1Y47X11   
~~~
Ta da!  Ok, nearly there. Just a couple of things to address. First off, how do we get this to work with multiple computers?  We need to provide Get-CimInstance with a list of computer names. We can do that via the pipeline like so:

`"CFLAB-SQL-01", "CFLAB-DC-01", "CFLAB-VMM-02" | Get-CimInstance -ClassName CIM_OperatingSystem | Select-Object PSComputerName, ServicePackMajorVersion, Version, @{name="BIOSSerial"; expression={Get-CimInstance -ClassName CIM_BIOSElement -ComputerName $_.PSCOMPUTERNAME | Select-Object SerialNumber | Select-Object -ExpandProperty SerialNumber}}`

And that works!  Also, note that we had to add the `-ComputerName $_.PSCOMPUTERNAME` parameter to the expression in the hash table, otherwise we'd be returning the BIOSSerial for the computer we're running the script from.

~~~
PSComputerName ServicePackMajorVersion Version    BIOSSerial
-------------- ----------------------- -------    ----------
CFLAB-DC-01                          0 10.0.10074 4463-1604-3039-0490-0901-9323-05
CFLAB-SQL-01                         0 10.0.10074 0952-3787-0397-1880-8637-3656-89
CFLAB-VMM-02                         0 10.0.10074 3355-1262-2901-1114-9446-3275-17
~~~

Ah, but you're not getting off that easily!  Why does it work?  Well, we can figure that out. Let's take a look at the help for Get-CimInstance:

`Get-Help Get-CimInstance`

~~~
SYNTAX
Get-CimInstance [-ClassName] <String> [-ComputerName <String[]>] [-Filter <String>] [-KeyOnly] [-Namespace <String>] [-OperationTimeoutSec <UInt32>] [-Property <String[]>] [-QueryDialect 
<String>] [-Shallow] [<CommonParameters>]

Get-CimInstance [-Namespace <String>] [-OperationTimeoutSec <UInt32>] [-QueryDialect <String>] [-ResourceUri <Uri>] [-Shallow] -CimSession <CimSession[]> -Query <String> [<CommonParameters>]

Get-CimInstance [-InputObject] <CimInstance> [-OperationTimeoutSec <UInt32>] [-ResourceUri <Uri>] -CimSession <CimSession[]> [<CommonParameters>]

Get-CimInstance [-ClassName] <String> [-Filter <String>] [-KeyOnly] [-Namespace <String>] [-OperationTimeoutSec <UInt32>] [-Property <String[]>] [-QueryDialect <String>] [-Shallow] -CimSession 
<CimSession[]> [<CommonParameters>]

Get-CimInstance [-Filter <String>] [-KeyOnly] [-Namespace <String>] [-OperationTimeoutSec <UInt32>] [-Property <String[]>] [-Shallow] -CimSession <CimSession[]> -ResourceUri <Uri> 
[<CommonParameters>]

Get-CimInstance [-ComputerName <String[]>] [-Filter <String>] [-KeyOnly] [-Namespace <String>] [-OperationTimeoutSec <UInt32>] [-Property <String[]>] [-Shallow] -ResourceUri <Uri> 
[<CommonParameters>]

Get-CimInstance [-ComputerName <String[]>] [-Namespace <String>] [-OperationTimeoutSec <UInt32>] [-QueryDialect <String>] [-ResourceUri <Uri>] [-Shallow] -Query <String> [<CommonParameters>]

Get-CimInstance [-InputObject] <CimInstance> [-ComputerName <String[]>] [-OperationTimeoutSec <UInt32>] [-ResourceUri <Uri>] [<CommonParameters>]
~~~
Ugh!  I remember when I first started working with PowerShell that I always struggled to make sense of the *SYNTAX* section of the help command's output!  The above are known as parameter sets. Whichever parameters you provide determine which parameter set you will end up using. In our case, we are providing the `-ClassName` parameter. So, the only parameter sets of interest are those which include the `-ClassName` parameter.

~~~
Get-CimInstance [-ClassName] <String> [-ComputerName <String[]>] [-Filter <String>] [-KeyOnly] [-Namespace <String>] [-OperationTimeoutSec <UInt32>] [-Property <String[]>] [-QueryDialect 
<String>] [-Shallow] [<CommonParameters>]

Get-CimInstance [-ClassName] <String> [-Filter <String>] [-KeyOnly] [-Namespace <String>] [-OperationTimeoutSec <UInt32>] [-Property <String[]>] [-QueryDialect <String>] [-Shallow] -CimSession 
<CimSession[]> [<CommonParameters>]
~~~

As well as the `-ClassName` parameter, we're also providing our list of servers from the pipeline. So, we're providing one additional item of information on top of our `-ClassName` parameter. Out of the two parameter sets that we're now left with, the first one has parameters which are all optional. However, the second one has a mandatory parameter `-CimSession`. PowerShell attempts to see if the values we've provided can be coerced into being `CimSession` objects. It turns out, they can. If you try:

`Get-CimInstance -ClassName CIM_OperatingSystem -CimSession CFLAB-SQL-01`

~~~
SystemDirectory     Organization BuildNumber RegisteredUser SerialNumber            Version    PSComputerName
---------------     ------------ ----------- -------------- ------------            -------    --------------
C:\Windows\system32              10074       Windows User   00123-32500-01495-AB120 10.0.10074 CFLAB-SQL-01 
~~~

You can see that it works just fine. So, feeding our list of server names into the pipeline at the start of the command results in them being passed to the `-CimSession` parameter of `Get-CimInstance`.

Almost finished. Our command line is looking a little on the long side!  The last of the four challenges was to "go obscure". Well, I'm not sure about that, but we could certainly do with making this a little shorter. So, let's have a go at that.

We're using the `Get-CimInstance` and `Select-Object` cmdlets a couple of times, so let's use the `-Definition` parameter of the `Get-Alias` cmdlet to discover if they have an alias

`Get-Alias -Definition Get-CimInstance, Select-Object`

~~~
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           gcim -> Get-CimInstance
Alias           select -> Select-Object
~~~

They do. So, here's our first stab at shortening our command line.

`"CFLAB-SQL-01", "CFLAB-DC-01", "CFLAB-VMM-02" | gcim -ClassName CIM_OperatingSystem | select PSComputerName, ServicePackMajorVersion, Version, @{name="BIOSSerial"; expression={gcim -ClassName CIM_BIOSElement -ComputerName $_.PSCOMPUTERNAME | select SerialNumber | select -ExpandProperty SerialNumber}}`

The next thing we can target is the parameters. From our earlier look at the parameter sets for `Get-CimInstance` we can see that we can omit the `-ClassName` parameter altogether as the parameter name is enclosed in square brackets, indicating that it's optional and positional. This means that the first thing we type after `Get-CimInstance`, if no parameter name is specified, is assumed to be a ClassName. In the second `Get-CimInstance` query within our hash table, we're also specifying the `-ComputerName` parameter, so let's see if we can find an alias for that. The following command will help us:

`(Get-Command Get-CimInstance | Select-Object -ExpandProperty parameters).ComputerName.Aliases`

~~~
CN
ServerName
~~~

In fact, there are two aliases. `CN` and `ServerName`. `CN` is the shorter of those two, so let's use that. The above command can be modified to discover if the same is true about the `Select-Object` `ExpandProperty` parameter. However, to save time I'll tell you that it doesn't have an alias. But we can make us of abbreviated parameters here. All you need to do is type enough of the parameter name for PowerShell to be able to determine your intended parameter. In the case of `Select-Object`, it has both `ExpandProperty` and `ExcludeProperty` parameters, so we need to type at least `-Exp` to differentiate between the two. Now let's revisit our command line with our positional, abbreviated and aliased parameters:

`"CFLAB-SQL-01", "CFLAB-DC-01", "CFLAB-VMM-02" | gcim CIM_OperatingSystem | select PSComputerName, ServicePackMajorVersion, Version, @{name="BIOSSerial"; expression={gcim CIM_BIOSElement -CN $_.PSCOMPUTERNAME | select SerialNumber | select -Exp SerialNumber}}`

Next let's tackle the objects we're selecting in the pipeline. We can use wildcards here, so with a little experimentation we can figure out the minimal amount of characters to type in order to filter out only the object we require.

~~~
PSComputerName --> PSC*
ServicePackMajorVersion --> *j*
Version --> v*
SerialNumber --> Se*
SerialNumber --> *
~~~
Notice that the second time we encounter `SerialNumber`, we can reduce it to simply `*` as we have previously filtered out all other objects, so only the one remains.

`"CFLAB-SQL-01", "CFLAB-DC-01", "CFLAB-VMM-02" | gcim CIM_OperatingSystem | select PSC*, *j*, v*, @{name="BIOSSerial"; expression={gcim CIM_BIOSElement -CN $_.PSCOMPUTERNAME | select Se* | select -Exp *}}`

And finally, we can take the `name` and `expression` in the hash table and abbreviate them to simply `n` and `e`

`"CFLAB-SQL-01", "CFLAB-DC-01", "CFLAB-VMM-02" | gcim CIM_OperatingSystem | select PSC*, *j*, v*, @{n="BIOSSerial"; e={gcim CIM_BIOSElement -CN $_.PSCOMPUTERNAME | select Se* | select -Exp *}}`

And I *think* that's about as short as you can get it, but let me know if I missed something!  And let's run that command one final time to make sure it's all looking good.

~~~
PSComputerName ServicePackMajorVersion Version    BIOSSerial
-------------- ----------------------- -------    ----------
CFLAB-VMM-02                         0 10.0.10074 3355-1262-2901-1114-9446-3275-17
CFLAB-SQL-01                         0 10.0.10074 0952-3787-0397-1880-8637-3656-89
CFLAB-DC-01                          0 10.0.10074 4463-1604-3039-0490-0901-9323-05
~~~

And we're done!

[PowerShellOrg]:	http://powershell.org/wp/2015/07/04/2015-july-scripting-games-puzzle/