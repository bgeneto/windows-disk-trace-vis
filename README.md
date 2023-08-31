# Windows Disk Trace Visualizer

[Windows Disk Trace Visualizer](https://windows-disk-trace-vis.streamlit.app/) is a simple but useful web application designed to streamline the process of analyzing Event Trace Log (ETL) files,
specifically focusing on Disk I/O activities. This tool offers an intuitive and user-friendly interface to effortlessly navigate
through complex ETL data, providing insightful summaries and statistics for a thorough understanding of your disk performance requirements.
Key Features:

- **Data Analysis:** Uncover the most frequently requested data size, the percentage of random reading or writing, the average access time, and more.
- **Performance:** Identify the average throughput and IOPS for each disk, and compare the performance of sequential and random requests.
- **Filter Capability:** Filter data by process name or request size.

All this info is valuable for understanding where the performance of your disk/ssd matters most for your desired/traced workload/application.

# Usage 

The webapp is hosted [here](https://windows-disk-trace-vis.streamlit.app). You will need my custom
[profile](https://raw.githubusercontent.com/bgeneto/windows-disk-trace-vis/main/DiskIO.wpaProfile) to convert the trace log to csv format. 

First, you need to record a trace of your 'Disk I/O activity' with WPRUI.exe. You can trace your Windows boot process or a specific application/workload.
Then you need to convert the saved `.etl` file to a `.csv` file using the `wpaexporter.exe` tool that comes with [Windows ADK](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install).
The `wpaexporter.exe` tool is typically located in the `C:\\Program Files (x86)\\Windows Kits\\10\\Windows Performance Toolkit` folder, an example usage follows:
```cmd
wpaexporter.exe -i boottrace.etl -profile DiskIO.wpaProfile -delimiter ;
```

You can also trace your boot process as follows:

- Open an elevated command prompt and run:

   ```cmd
   wpr -addboot GeneralProfile.Light -filemode -recordtempto D:\\Temp
   ```

After this, the trace will start automatically at the early stage of the next (re)boot.

- The command syntax to save the boot trace (.etl file) is the following:

   ```cmd
   wpr -stopboot D:\\Temp\\boottrace.etl
   ```

Now you can upload the (compressed) `.csv` file to this page. The page script &mdash; written in Python{python_svg}using Pandas{pandas_svg} &mdash; will analyze the single trace record using the provided
[profile](https://raw.githubusercontent.com/bgeneto/windows-disk-trace-vis/main/DiskIO.wpaProfile) and show you the results in tabular and graphical formats.
> **Note:** tested with WPR v10.0.25931.1000 on Windows 11 Pro version 22H2.

# Example Output 

Here is an excerpt from the application's output, which demonstrates a standard result from a Windows 11 boot trace conducted on a system that includes both an HDD and an SSD.

| Metric             | Value  |
| ------------------ | ------ |
| Percent READ       | 90.34% |
| Percent WRITE      | 9.66%  |
| Percent RANDOM     | 89.55% |
| Percent SEQUENTIAL | 10.45% |

 
![image](https://github.com/bgeneto/windows-disk-trace-vis/assets/473074/5b4e9baa-d508-4281-88b9-2ec147a5b0ea)
Note the predominant number of 4KB random read requests (60%), followed by 16KB requests (19%).

![image](https://github.com/bgeneto/windows-disk-trace-vis/assets/473074/552796b4-f46d-490a-9721-22ab88a51687)
Note how much faster is an SSD vs HDD in terms of latency/access time.

![image](https://github.com/bgeneto/windows-disk-trace-vis/assets/473074/a3fd3754-5add-48d7-876f-e323fdded137)
Random vs Sequential requests by IO Type.

# About WPR

Windows Performance Recorder ([WPR](https://learn.microsoft.com/en-us/windows-hardware/test/wpt/windows-performance-recorder))
is a performance recording tool that is based on Event Tracing for Windows ([ETW](https://learn.microsoft.com/en-us/windows/win32/etw/about-event-tracing)).
It records system and application events that you can analyze by parsing the generated Event Trace Log ([ETL](https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/trace-log)) file.
The ETL file format is a proprietary binary file format used by Microsoft Windows for collecting
and storing event traces generated by various components of the operating system. Since it is a poorly documented and
proprietary format, we need to use the [WPA Exporter](https://learn.microsoft.com/en-us/windows-hardware/test/wpt/exporter)
tool in order to convert the .etl file to a .csv file and then perform the desired trace analysis via csv file. Both WPR and WPA Exporter are part of the
Windows Assessment and Deployment Kit [Windows ADK](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install).
