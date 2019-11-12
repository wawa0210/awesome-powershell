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

### File

* Quickly create large file
  ```
  fsutil file createnew 1.log 1073741824

  #Some common file sizes to save you from math
  1 MB = 1048576 bytes
  100 MB = 104857600 bytes
  1 GB = 1073741824 bytes
  10 GB = 10737418240 bytes
  100 GB =107374182400 bytes
  1 TB = 1099511627776 bytes
  10 TB = 10995116277760 bytes
  ```