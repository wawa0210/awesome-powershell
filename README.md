## powershell使用过程中的最佳实践 [Powershell best practices](powershell_en.md)


### CPU

* 获取主机 cpu 使用率

    ```
    Get-WmiObject win32_processor | select LoadPercentage  | fl
    ```

* 指定间隔获取一次 CPU 占有率，并保存到文件中

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

* 获取进程cpu使用率

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

* cpu 压力测试
    ```
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

* 获取内存使用率

  ```
  $ComputerMemory = Get-WmiObject -ComputerName localhost -Class win32_operatingsystem -ErrorAction Stop
  $Memory = ((($ComputerMemory.TotalVisibleMemorySize - $ComputerMemory.FreePhysicalMemory)*100)/ $ComputerMemory.TotalVisibleMemorySize)
  $RoundMemory = [math]::Round($Memory, 2)
  ```

* 间隔时间获取内存使用率，并保存到文件中
  
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
### 文件系统

* 快速创建指定大小的文件(fsutil)

  ```
  fsutil file createnew 1.log 1073741824

  #单位换算如下(文件大小基本单位为字节)
  1 MB = 1024 * 1024 bytes
  100 MB = 100 * 1024 * 1024  bytes
  1 GB = 1024 * 1024 * 1024 bytes
  1 TB = 1024 * 1024 * 1024 * 1024 bytes
  ```

* 快速创建指定大小的文件(使用 powershell)

  ```
  $content = "f" * 501MB
  $content | Out-File -FilePath test.txt
  ```

### 微软官方文档:

- [Microsoft Docs](https://docs.microsoft.com/en-us/windows/wsl/about)
- [Release Notes](https://docs.microsoft.com/en-us/windows/wsl/release-notes)
- [WSL Blog](https://blogs.msdn.microsoft.com/wsl) (Historical)
- [Command Line Blog](https://blogs.msdn.microsoft.com/commandline/) (Active)

### 社区最佳实践:

- Stack Overflow: https://stackoverflow.com/questions/tagged/wsl
- Ask Ubuntu: https://askubuntu.com/questions/tagged/wsl
- reddit: https://www.reddit.com/r/bashonubuntuonwindows
- List of programs that work and don't work
- https://github.com/ethanhs/WSL-Programs
- https://github.com/davatron5000/can-i-subsystem-it
- Awesome WSL: https://github.com/sirredbeard/Awesome-WSL
- Tips and guides for new bash users: https://github.com/abergs/ubuntuonwindows