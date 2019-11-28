
awesome-powershell | [Powershell 最佳实践](README.md)
===================

**This article mainly records some `powershell` usage tips. Through these tips, we can more easily and quickly solve practical problems. It is inevitable that there are currently negligence. Corrections are welcome. At the same time, like-minded partners are welcome to improve together and build `powershell` best practices.**

### CPU

* Get cpu usage

    ```cpu
    Get-WmiObject win32_processor | select LoadPercentage  | fl
    ```

* Obtain the cpu usage rate at the specified interval and save it to the file.

    ```cpu_percent
    if (-not (Test-Path "c:\cpu_useage.log")){
    "" > c:\cpu_useage.log
    }
    while ($true) {
        $cpu_useage_obj = Get-WmiObject win32_processor | select LoadPercentage | fl
        $cpu_useage_str = ($cpu_useage_obj | out-string)
        # remove Line break
        $cpu_useage = $cpu_useage_str.Trim(" .-`t`n`r").Split(':')[1]
        Add-Content -Path "c:\cpu_useage.log" -Value ((Get-Date -Format "yyyy-mm-dd HH:mm:ss") + "|"+$cpu_useage)
        Start-Sleep 2
    }
    ```

* Get process cpu useage by process name

    ```cpu_useage_by_process_name
    function GetCpuUsageByProcessName {
        param (
            [string] $processName
        )
        $CurrentProcessCpuUseage = (get-process $processName  | Select-Object -expand CPU | Measure-Object -Sum | Select-Object -expand Sum)/1000/4
        $TotalCpuCore = (Get-CimInstance Win32_ComputerSystem).NumberOfLogicalProcessors

        return $CurrentProcessCpuUseage / $TotalCpuCore
    }

    ```
* Cpu pressure test

    ```cpu_pressure
    #可以平均分配到每个 cpu 核心中进行压测
    $cpu_cores = 4
    foreach ($loopnumber in 1..$cpu_cores){
        Start-Job -ScriptBlock{ 
            foreach ($loopnumber in 1..2147483647) {
                $result=1;
                foreach ($number in 1..2147483647) {
                    $result = $result * $number};$result
                } 
            }
        }
    ```

### Memory

* Get memory usage

  ```memory_useage
  $ComputerMemory = Get-WmiObject -ComputerName localhost -Class win32_operatingsystem -ErrorAction Stop
  $Memory = ((($ComputerMemory.TotalVisibleMemorySize - $ComputerMemory.FreePhysicalMemory)*100)/ $ComputerMemory.TotalVisibleMemorySize)
  $RoundMemory = [math]::Round($Memory, 2)
  ```

* Obtain the memory usage rate at the specified interval and save it to the file.
  
  ```memory_useage_time_span
  if (-not (Test-Path "c:\memory_useage.log")){
    "" > c:\memory_useage.log
  }
  while ($true) {
    $ComputerMemory = Get-WmiObject -ComputerName localhost -Class win32_operatingsystem -ErrorAction Stop
    $Memory = ((($ComputerMemory.TotalVisibleMemorySize - $ComputerMemory.FreePhysicalMemory)*100)/ $ComputerMemory.TotalVisibleMemorySize)
    $RoundMemory = [math]::Round($Memory, 2)
    Add-Content -Path "c:\memory_useage.log" -Value ((Get-Date -Format "yyyy-mm-dd HH:mm:ss") + "|"+$RoundMemory)
    Start-Sleep 2
  }
  ```

### File System

* Quickly create large file(fsutil)

  ```fsutil
  fsutil file createnew 1.log 1073741824

  #Some common file sizes to save you from math(File size is in bytes)
  1 MB = 1024 * 1024 bytes
  100 MB = 100 * 1024 * 1024  bytes
  1 GB = 1024 * 1024 * 1024 bytes
  1 TB = 1024 * 1024 * 1024 * 1024 bytes
  ```

* Quickly create large file(using powershell)

  ```create_file
  #Beyond short integer range will overflow
  $content = "f" * 501MB
  $content | Out-File -FilePath test.txt
  ```

### Microsoft Links:

- [Microsoft Docs](https://docs.microsoft.com/en-us/windows/wsl/about)
- [Release Notes](https://docs.microsoft.com/en-us/windows/wsl/release-notes)
- [WSL Blog](https://blogs.msdn.microsoft.com/wsl) (Historical)
- [Command Line Blog](https://blogs.msdn.microsoft.com/commandline/) (Active)

### Community Links:

- Stack Overflow: https://stackoverflow.com/questions/tagged/wsl
- Ask Ubuntu: https://askubuntu.com/questions/tagged/wsl
- reddit: https://www.reddit.com/r/bashonubuntuonwindows
- List of programs that work and don't work
- https://github.com/ethanhs/WSL-Programs
- https://github.com/davatron5000/can-i-subsystem-it
- Awesome WSL: https://github.com/sirredbeard/Awesome-WSL
- Tips and guides for new bash users: https://github.com/abergs/ubuntuonwindows