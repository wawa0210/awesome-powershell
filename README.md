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
