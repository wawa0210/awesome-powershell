### CPU

* Get cpu usage

    ```
    Get-WmiObject win32_processor | select LoadPercentage  | fl
    ```

* Obtain the cpu usage rate at the specified interval and save it to the file.

    ```
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
* Get process cpu useage
    ```
    function GetCpuUsageByProcessName {
        param (
            [string] $processName
        )
        $CurrentProcessCpuUseage = (get-process $processName  | Select-Object -expand CPU | Measure-Object -Sum | Select-Object -expand Sum)/1000/4
        $TotalCpuCore = (Get-CimInstance Win32_ComputerSystem).NumberOfLogicalProcessors

        return $CurrentProcessCpuUseage / $TotalCpuCore
    }

    ```

### Memory

* Get memory usage

  ```
  $ComputerMemory = Get-WmiObject -ComputerName localhost -Class win32_operatingsystem -ErrorAction Stop
  $Memory = ((($ComputerMemory.TotalVisibleMemorySize - $ComputerMemory.FreePhysicalMemory)*100)/ $ComputerMemory.TotalVisibleMemorySize)
  $RoundMemory = [math]::Round($Memory, 2)
  ```

* Obtain the memory usage rate at the specified interval and save it to the file.
  
  ```
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
  ```
  fsutil file createnew 1.log 1073741824

  #Some common file sizes to save you from math(File size is in bytes)
  1 MB = 1024 * 1024 bytes
  100 MB = 100 * 1024 * 1024  bytes
  1 GB = 1024 * 1024 * 1024 bytes
  1 TB = 1024 * 1024 * 1024 * 1024 bytes
  ```

* Quickly create large file(using powershell)

  ```
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