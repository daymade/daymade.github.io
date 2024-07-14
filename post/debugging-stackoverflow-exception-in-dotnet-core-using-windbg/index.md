
<!--more-->


## Platform

- dotnet core runing on Windows
- lldb

## Tools

- adplus
- windbg(x64)
- DebugDiag 2 Analysis

## Progress

### Using adplus to create a dump when process crashed 

```
adplus.exe -crash -pn dotnet.exe -o d:\ -quiet
```

### load the dump 
```shell
Microsoft (R) Windows Debugger Version 10.0.14321.1024 AMD64
Copyright (c) Microsoft Corporation. All rights reserved.


Loading Dump File [C:\Users\xxx\Desktop\MINIDUMP_FirstChance_sov_StackOverflow_dotnet.exe__0134_2018-04-20_17-10-29-460_1420.dmp]
Comment: 'FirstChance_sov_StackOverflow'
User Mini Dump File: Only registers, stack and portions of memory are available

Symbol search path is: srv*
Executable search path is: 
Windows 8 Version 9200 MP (4 procs) Free x64
Product: Server, suite: TerminalServer SingleUserTS
Machine Name:
Debug session time: Fri Apr 20 17:10:42.000 2018 (UTC + 8:00)
System Uptime: not available
Process Uptime: 0 days 2:54:11.000
................................................................
................................................................
................................................................
..........................................
This dump file has an exception of interest stored in it.
The stored exception information can be accessed via .ecxr.
(1420.9f4): Stack overflow - code c00000fd (first/second chance not available)
coreclr!AllocateString+0x2 [inlined in coreclr!StringObject::NewString+0x20]:
00007ffa`efef9a44 e85715faff      call    coreclr!SlowAllocateString (00007ffa`efe9afa0)
```
### set symbols path
```
0:000> .sympath cache*c:\LocalSymbolsCache;C:\tempServices\Services;srv*c:\ServerSymbolsCache*https://msdl.microsoft.com/download/symbols

Symbol search path is: cache*c:\LocalSymbolsCache;C:\tempServices\Services;srv*c:\ServerSymbolsCache*https://msdl.microsoft.com/download/symbols
Expanded Symbol search path is: cache*c:\localsymbolscache;c:\tempservices\services;srv*c:\serversymbolscache*https://msdl.microsoft.com/download/symbols

************* Symbol Path validation summary **************
Response                         Time (ms)     Location
Deferred                                       cache*c:\LocalSymbolsCache
OK                                             C:\tempServices\Services
Deferred                                       srv*c:\ServerSymbolsCache*https://msdl.microsoft.com/download/symbols

```

### load sos extensions for windbg

```
0:000> .load C:\Program Files\dotnet\shared\Microsoft.NETCore.App\2.0.7\sos.dll

----------------------------------------------------------------------------
The user dump currently examined is a minidump. Consequently, only a subset
of sos.dll functionality will be available. If needed, attaching to the live
process or debugging a full dump will allow access to sos.dll's full feature
set.
To create a full user dump use the command: .dump /ma <filename>
----------------------------------------------------------------------------
```

```
0:000> .symfix
0:000> .reload
```

### try to analyze problems automatically

```
0:000> !analyze -v

*******************************************************************************
*                                                                             *
*                        Exception Analysis                                   *
*                                                                             *
*******************************************************************************

*** WARNING: Unable to verify timestamp for libuv.dll
*** ERROR: Module load completed but symbols could not be loaded for libuv.dll
GetUrlPageData2 (WinHttp) failed: 12002.

...remove lines

EXCEPTION_RECORD:  (.exr -1)
ExceptionAddress: 00007ffaefef9a44 (coreclr!AllocateString+0x0000000000000002)
   ExceptionCode: c00000fd (Stack overflow)
  ExceptionFlags: 00000000
NumberParameters: 2
   Parameter[0]: 0000000000000001
   Parameter[1]: 000000fadfbc3ff8

DEFAULT_BUCKET_ID:  STACK_OVERFLOW_DATA

PROCESS_NAME:  dotnet.exe

MANAGED_ENGINE_MODULE:  coreclr

MANAGED_ANALYSIS_PROVIDER:  SOS

MANAGED_THREAD_ID: 9f4

STACK_OVERFLOW
    Tid    [0x31]
    Frame  [0x00]

DATA
    Tid    [0x9f4]
    Failure Bucketing

...remove lines

```

Tid 0x31 (49 for decimal) has an stack overflow exception, which caused the application to exit abnormally.

### print all threads 

```
0:000> !threads
ThreadCount:      57
UnstartedThread:  0
BackgroundThread: 53
PendingThread:    0
DeadThread:       3
Hosted Runtime:   no
                                                                                                        Lock  
       ID OSID ThreadOBJ           State GC Mode     GC Alloc Context                  Domain           Count Apt Exception
   0    1 1824 0000028d2fccfb90    2a020 Preemptive  0000000000000000:0000000000000000 0000028d2feb19f0 1     Ukn 
   2    2 146c 0000028d2fcc8fb0    2b220 Preemptive  0000028D35170278:0000028D35172220 0000028d2feb19f0 0     Ukn (Finalizer) 
   5    3  b40 0000028d2fd650c0  1029220 Preemptive  0000000000000000:0000000000000000 0000028d2feb19f0 0     Ukn (Threadpool Worker) r) 
  48   57 1120 0000028d4ae2b9b0  1029220 Preemptive  0000000000000000:0000000000000000 0000028d2feb19f0 0     Ukn (Threadpool Worker) 
  49   43  9f4 0000028d4a33e910  1029220 Cooperative 0000028D354EB360:0000028D354EBFD0 0000028d2feb19f0 0     Ukn (Threadpool Worker) 
  50   59 1a10 0000028d4ae4ca80  1029220 Preemptive  0000000000000000:0000000000000000 0000028d2feb19f0 0     Ukn (Threadpool Worker) 
XXXX   20    0 0000028d4ac519f0    39820 Preemptive  0000000000000000:0000000000000000 0000028d2feb19f0 0     Ukn 
XXXX   64    0 0000028d4a341020    39820 Preemptive  0000000000000000:0000000000000000 0000028d2feb19f0 0     Ukn 
```

### switch to specical thread context

```
0:000> ~~[9f4]s
coreclr!AllocateString+0x2 [inlined in coreclr!StringObject::NewString+0x20]:
00007ffa`efef9a44 e85715faff      call    coreclr!SlowAllocateString (00007ffa`efe9afa0)
```

### print clr managed stack
```
0:049> !clrstack
OS Thread Id: 0x9f4 (49)
        Child SP               IP Call Site
000000fadfbc4458 00007ffaefef9a44 [HelperMethodFrame_PROTECTOBJ: 000000fadfbc4458] System.Number.FormatInt64(Int64, System.String, System.Globalization.NumberFormatInfo)
000000fadfbc4590 00007ffa91992cc6 xxx.LinePackageSalePrice+c__DisplayClass4_1.b__1(xxx.BookResourceInfo)
000000fadfbc45d0 00007ffaf992b16f System.Linq.Enumerable.TryGetFirst[[System.__Canon, System.Private.CoreLib]](System.Collections.Generic.IEnumerable`1, System.Func`2, Boolean ByRef)
000000fadfbc4640 00007ffa914fd0d2 xxx.LinePackageSalePrice.IsPackageB(xxx.GetLinePackageSalePriceRequest, System.Collections.Generic.List`1, System.Collections.Generic.List`1, Int64, Int32)
000000fadfbc4730 00007ffa914fd121 xxx.LinePackageSalePrice.IsPackageB(xxx.GetLinePackageSalePriceRequest, System.Collections.Generic.List`1, System.Collections.Generic.List`1, Int64, Int32)
... removed 6k lines duplicated
000000fadfd3d080 00007ffa914fd121 xxx.LinePackageSalePrice.IsPackageB(xxx.GetLinePackageSalePriceRequest, System.Collections.Generic.List`1, System.Collections.Generic.List`1, Int64, Int32)
000000fadfd3d170 00007ffa914fd121 xxx.LinePackageSalePrice.IsPackageB(xxx.GetLinePackageSalePriceRequest, System.Collections.Generic.List`1, System.Collections.Generic.List`1, Int64, Int32)
000000fadfd3d260 00007ffa914fca97 xxx.LinePackageSalePrice.IsPackage(xxx.GetLinePackageSalePriceRequest, System.Collections.Generic.List`1, System.Collections.Generic.List`1 ByRef, Int32 ByRef)
000000fadfd3d3c0 00007ffa914fb5a1 xxx.LinePackageSalePrice.GetLinePackageSalePrice(xxx.GetLinePackageSalePriceRequest)
000000fadfd3d5b0 00007ffa90f16efe xxx.InitResponse[[System.__Canon, System.Private.CoreLib],[System.__Canon, System.Private.CoreLib]](System.Func`2>, System.__Canon, System.String, System.String, System.String, System.String, System.String)
000000fadfd3d690 00007ffa914fb154 xxx.GetLinePackageSalePrice(xxx.GetLinePackageSalePriceRequest)
000000fadfd3d890 00007ffaeff735f3 [DebuggerU2MCatchHandlerFrame: 000000fadfd3d890] 
000000fadfd3dc28 00007ffaeff735f3 [HelperMethodFrame_PROTECTOBJ: 000000fadfd3dc28] System.RuntimeMethodHandle.InvokeMethod(System.Object, System.Object[], System.Signature, Boolean)
000000fadfd3e770 00007ffae07e7299 System.Threading.ThreadPoolWorkQueue.Dispatch() [E:\A\_work\805\s\src\mscorlib\src\System\Threading\ThreadPool.cs @ 582]
000000fadfd3ec70 00007ffaeff735f3 [DebuggerU2MCatchHandlerFrame: 000000fadfd3ec70] 
```
### print clr managed stack(with parameters information)
```
0:049> !clrstack -p
OS Thread Id: 0x9f4 (49)
        Child SP               IP Call Site
000000fadfbc4458 00007ffaefef9a44 [HelperMethodFrame_PROTECTOBJ: 000000fadfbc4458] System.Number.FormatInt64(Int64, System.String, System.Globalization.NumberFormatInfo)
000000fadfbc4590 00007ffa91992cc6 xxx.LinePackageSalePrice+c__DisplayClass4_1.b__1(xxx.BookResourceInfo)
    PARAMETERS:
        this = <no data>
        x = <no data>

000000fadfbc45d0 00007ffaf992b16f System.Linq.Enumerable.TryGetFirst[[System.__Canon, System.Private.CoreLib]](System.Collections.Generic.IEnumerable`1, System.Func`2, Boolean ByRef)
    PARAMETERS:
        source = <no data>

000000fadfbc4640 00007ffa914fd0d2 xxx.LinePackageSalePrice.IsPackageB(xxx.GetLinePackageSalePriceRequest, System.Collections.Generic.List`1, System.Collections.Generic.List`1, Int64, Int32)
    PARAMETERS:
        this (<CLR reg>) = 0x0000028d3513d7f0
        request (<CLR reg>) = 0x0000028d3513d478

000000fadfd3d080 00007ffa914fd121 xxx.LinePackageSalePrice.IsPackageB(xxx.GetLinePackageSalePriceRequest, System.Collections.Generic.List`1, System.Collections.Generic.List`1, Int64, Int32)
    PARAMETERS:
        this = <no data>
        request = <no data>
		
... removed many lines duplicated

000000fadfd3d080 00007ffa914fd121 xxx.LinePackageSalePrice.IsPackageB(xxx.GetLinePackageSalePriceRequest, System.Collections.Generic.List`1, System.Collections.Generic.List`1, Int64, Int32)
    PARAMETERS:
        this = <no data>
        request = <no data>

000000fadfd3d170 00007ffa914fd121 xxx.LinePackageSalePrice.IsPackageB(xxx.GetLinePackageSalePriceRequest, System.Collections.Generic.List`1, System.Collections.Generic.List`1, Int64, Int32)
    PARAMETERS:
        this = <no data>
        request = <no data>

000000fadfd3d260 00007ffa914fca97 xxx.LinePackageSalePrice.IsPackage(xxx.GetLinePackageSalePriceRequest, System.Collections.Generic.List`1, System.Collections.Generic.List`1 ByRef, Int32 ByRef)
    PARAMETERS:
        this = <no data>
        request = <no data>

000000fadfd3d3c0 00007ffa914fb5a1 xxx.LinePackageSalePrice.GetLinePackageSalePrice(xxx.GetLinePackageSalePriceRequest)
    PARAMETERS:
        this (<CLR reg>) = 0x0000028d3513d7f0
        request (<CLR reg>) = 0x0000028d3513d478

000000fadfd3d5b0 00007ffa90f16efe xxx.InitResponse[[System.__Canon, System.Private.CoreLib],[System.__Canon, System.Private.CoreLib]](System.Func`2>, System.__Canon, System.String, System.String, System.String, System.String, System.String)
    PARAMETERS:

000000fadfd3d690 00007ffa914fb154 xxx.LineService.GetLinePackageSalePrice(xxx.GetLinePackageSalePriceRequest)
    PARAMETERS:
        this = <no data>
        request = <no data>

000000fadfd3e700 00007ffae071b64e System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext, System.Threading.ContextCallback, System.Object) [E:\A\_work\805\s\src\mscorlib\shared\System\Threading\ExecutionContext.cs @ 145]
    PARAMETERS:
        executionContext = <no data>
        callback = <no data>
        state = <no data>

000000fadfd3e770 00007ffae07e7299 System.Threading.ThreadPoolWorkQueue.Dispatch() [E:\A\_work\805\s\src\mscorlib\src\System\Threading\ThreadPool.cs @ 582]

000000fadfd3ec70 00007ffaeff735f3 [DebuggerU2MCatchHandlerFrame: 000000fadfd3ec70] 
```

### dump object by address

```
0:049> !do 0x0000028d3513d478
<Note: this object has an invalid CLASS field>
Invalid object
```
failed, 

### dump value type by method table and address

```
0:049> !dumpvc 000000fadfbc4640 0x0000028d3513d478
Not a managed object
```
failed

### see what's on the heap

```
0:049> !dumpheap -stat
Error requesting heap segment 0000028d417e0000
Failed to retrieve segments for gc heap
Unable to build snapshot of the garbage collector state
```

...Ok we still have a full dump 

### load full dump file

```
0:000> .sympath cache*c:\LocalSymbolsCache;C:\tempServices\Services;srv*c:\ServerSymbolsCache*https://msdl.microsoft.com/download/symbols
Symbol search path is: cache*c:\LocalSymbolsCache;C:\tempServices\Services;srv*c:\ServerSymbolsCache*https://msdl.microsoft.com/download/symbols
Expanded Symbol search path is: cache*c:\localsymbolscache;c:\tempservices\services;srv*c:\serversymbolscache*https://msdl.microsoft.com/download/symbols

************* Symbol Path validation summary **************
Response                         Time (ms)     Location
Deferred                                       cache*c:\LocalSymbolsCache
OK                                             C:\tempServices\Services
Deferred                                       srv*c:\ServerSymbolsCache*https://msdl.microsoft.com/download/symbols
```

```
0:000> .loadby sos coreclr
0:000> .reload
................................................................
................................................................
................................................................
..........................................
Loading unloaded module list
```

```
0:000> !dumpheap -stat
Statistics:
              MT    Count    TotalSize Class Name
00007ffae0b4f9a8        1           24 System.Collections.Generic.GenericComparer`1[[System.UInt64, System.Private.CoreLib]]
00007ffa918de1c0        1           24 xxx.ZBYLineProductSpecialPricePolicyModel[]
00007ffa918dd6e0        1           24 xxx.Line.ResourcePolicyUseInfo[]
00007ffa908b81c0        1           72 xxx.Line.GetLinePackageSalePriceRequest
...removed lines 
00007ffae0ab5300      117        14336 System.Type[]
00007ffa916cebb0     6432       257280 System.Collections.Generic.List`1[[xxx.ZBYLineProductPolicyRelationModel, xxx]]
00007ffa91711ef0     6433       360200 xxx.ZBYLineProductPolicyRelationModel[]
0000028d2fd88890    25244     18863634      Free
Total 356948 objects
Fragmented blocks larger than 0.5 MB:
            Addr     Size      Followed by
0000028d3511ada8    0.6MB 0000028d351ac668 System.IntPtr[]
0000028d351ac690    0.9MB 0000028d352918c8 System.IntPtr[]
0000028d35291918    2.4MB 0000028d354f8110 System.Byte[]
```

### dumpheap -mt can extract the address of an object from method table

```
0:000> !DumpHeap /d -mt 00007ffa908b81c0
         Address               MT     Size
0000028d34e053c0 00007ffa908b81c0       72     

Statistics:
              MT    Count    TotalSize Class Name
00007ffa908b81c0        1           72 xxx.Line.GetLinePackageSalePriceRequest
Total 1 objects
```

```
0:000> !do 0000028d34e053c0
Name:        xxx.Line.GetLinePackageSalePriceRequest
MethodTable: 00007ffa908b81c0
EEClass:     00007ffa908dbeb0
Size:        72(0x48) bytes
File:        C:\app\Services\xxx.dll
Fields:
              MT    Field   Offset                 Type VT     Attr            Value Name
00007ffae0ae8020  4000006        8        System.String  0 instance 0000028d34e05670 <OperCode>k__BackingField
00007ffae0ae8020  4000007       10        System.String  0 instance 0000028d34e05690 <OperName>k__BackingField
00007ffae0ae8020  400076c       18        System.String  0 instance 0000028d34e056b0 <ChannelKey>k__BackingField
00007ffae0ae8020  4000513       20        System.String  0 instance 0000028d34e05408 <LineId>k__BackingField
00007ffae01d7168  4000514       28 ...Private.CoreLib]]  0 instance 0000028d34e05430 <LinePackageIds>k__BackingField
00007ffae0aeb5f0  4000515       38      System.DateTime  1 instance 0000028d34e053f8 <BookDate>k__BackingField
00007ffa90d58ab8  4000516       30 ...frontend.Entity]]  0 instance 0000028d34e054e0 <BookResourceInfos>k__BackingField
```

```
0:000> !do 0000028d34e05408
Name:        System.String
MethodTable: 00007ffae0ae8020
EEClass:     00007ffae027a250
Size:        36(0x24) bytes
File:        C:\Program Files\dotnet\shared\Microsoft.NETCore.App\2.0.7\System.Private.CoreLib.dll
String:      70277
Fields:
              MT    Field   Offset                 Type VT     Attr            Value Name
00007ffae0b066d8  4000218        8         System.Int32  1 instance                5 m_stringLength
00007ffae0ae8268  4000219        c          System.Char  1 instance               37 m_firstChar
00007ffae0ae8020  400021a       78        System.String  0   shared           static Empty
                                 >> Domain:Value  0000028d2feb19f0:NotInit  <<
```

```
0:000> !do 0000028d34e05430
Name:        System.Collections.Generic.List`1[[System.String, System.Private.CoreLib]]
MethodTable: 00007ffae01d7168
EEClass:     00007ffae022d868
Size:        40(0x28) bytes
File:        C:\Program Files\dotnet\shared\Microsoft.NETCore.App\2.0.7\System.Private.CoreLib.dll
Fields:
              MT    Field   Offset                 Type VT     Attr            Value Name
00007ffae0ab3018  4001682        8     System.__Canon[]  0 instance 0000028d34e05480 _items
00007ffae0b066d8  4001683       18         System.Int32  1 instance                2 _size
00007ffae0b066d8  4001684       1c         System.Int32  1 instance                2 _version
00007ffae0aeb370  4001685       10        System.Object  0 instance 0000000000000000 _syncRoot
00007ffae0ab3018  4001686        0     System.__Canon[]  0   shared           static _emptyArray
                                 >> Domain:Value  0000028d2feb19f0:NotInit  <<
```

### dump array should be used when the object is an array

```
0:000> !da 0000028d34e05480
Name:        System.String[]
MethodTable: 00007ffae0ab2d20
EEClass:     00007ffae0272a00
Size:        56(0x38) bytes
Array:       Rank 1, Number of elements 4, Type CLASS
Element Methodtable: 00007ffae0ae8020
[0] 0000028d34e05458
[1] 0000028d34e054b8
[2] null
[3] null
```

```
0:000> !do 0000028d34e05458
Name:        System.String
MethodTable: 00007ffae0ae8020
EEClass:     00007ffae027a250
Size:        40(0x28) bytes
File:        C:\Program Files\dotnet\shared\Microsoft.NETCore.App\2.0.7\System.Private.CoreLib.dll
String:      1671849
Fields:
              MT    Field   Offset                 Type VT     Attr            Value Name
00007ffae0b066d8  4000218        8         System.Int32  1 instance                7 m_stringLength
00007ffae0ae8268  4000219        c          System.Char  1 instance               31 m_firstChar
00007ffae0ae8020  400021a       78        System.String  0   shared           static Empty
                                 >> Domain:Value  0000028d2feb19f0:NotInit  <<
```

### datetime is a little difficult to calculate
```
0:000> !dumpvc 00007ffae0aeb5f0  0000028d34e053f8
Name:        System.DateTime
MethodTable: 00007ffae0aeb5f0
EEClass:     00007ffae027b688
Size:        24(0x18) bytes
File:        C:\Program Files\dotnet\shared\Microsoft.NETCore.App\2.0.7\System.Private.CoreLib.dll
Fields:
              MT    Field   Offset                 Type VT     Attr            Value Name
00007ffae0b12568  400025c        0        System.UInt64  1 instance 636628896000000000 _dateData
00007ffae0ab4590  400024d       98       System.Int32[]  0   shared           static s_daysToMonth365
                                 >> Domain:Value  0000028d2feb19f0:NotInit  <<
00007ffae0ab4590  400024e       a0       System.Int32[]  0   shared           static s_daysToMonth366
                                 >> Domain:Value  0000028d2feb19f0:NotInit  <<
00007ffae0aeb5f0  400024f       88      System.DateTime  1   shared           static MinValue
                                 >> Domain:Value  0000028d2feb19f0:NotInit  <<
00007ffae0aeb5f0  4000250       90      System.DateTime  1   shared           static MaxValue
                                 >> Domain:Value  0000028d2feb19f0:NotInit  <<
								 
0:000> ?0n636628896000000000 & 0x3FFFFFFFFFFFFFFF
Evaluate expression: 636628896000000000 = 08d5c29b`9fd44000

0:000> .formats 08d5c29b`9fd44000
Evaluate expression:
  Hex:     08d5c29b`9fd44000
  Decimal: 636628896000000000
  Octal:   0043256051563765040000
  Binary:  00001000 11010101 11000010 10011011 10011111 11010100 01000000 00000000
  Chars:   ......@.
  Time:    Sat May 26 08:00:00.000 3618 (UTC + 8:00)
  Float:   low -8.98914e-020 high 1.28652e-033
  Double:  4.2178e-266
```

Now we have extracted entities which the stack overflow exception caused by:

```
{
	LineId: 70277
	LinePackageIds: [1671849,1483157]
	BookDate: Sat May 26 08:00:00.000 3618 (UTC + 8:00)
	BookResourceInfos: [{ResourceId:"232055",PolicyId:"516654841997590528", Date:"Sat May 26 08:00:00.000 3618 (UTC + 8:00)"},{232055, 516654841997590528, Sat May 26 08:00:00.000 3618 (UTC + 8:00)}]
}
```

and the stack-overflowed function:

```
xxx.LinePackageSalePrice.IsPackageB
```

Done!

## Concolusion:

- .sympath cache*c:\LocalSymbolsCache;C:\tempServices\Services;srv*c:\ServerSymbolsCache*https://msdl.microsoft.com/download/symbols
- .load C:\Program Files\dotnet\shared\Microsoft.NETCore.App\2.0.7\sos.dll
- .loadby sos coreclr 
- .symfix
- .reload
- !analyze -v
- !threads
- ~~[9f4]s
- ~49s
- !clrstack
- !clrstack -p 
- !dumpheap -stat
- !DumpHeap /d -mt 00007ffa908b81c0
- !do 0000028d34e05408
- !da 0000028d34e05480
- !dumpvc 00007ffae0aeb5f0  0000028d34e053f8

### others command may be useful

- !eestack 

### tricks

- Ctrl + Break can interrupt `*BUSY*` operations
- ESC can clear your current input 
- remove srv* from sympath after symbols cached will speed up your symbols finding.

## References

- https://blogs.msdn.microsoft.com/paullou/2011/06/28/debugging-managed-code-memory-leak-with-memory-dump-using-windbg/
- https://theartofdev.com/windbg-cheat-sheet/
- https://docs.microsoft.com/en-us/dotnet/framework/tools/sos-dll-sos-debugging-extension
- http://www.cnblogs.com/gaochundong/p/windbg_cheat_sheet.html
