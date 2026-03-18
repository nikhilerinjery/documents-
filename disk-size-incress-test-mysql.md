## First, confirm that the OS sees the  total disk size. Run:
```Get-Disk:```

## Check the Partition
Now, identify the partition you want to extend (usually the one with the drive letter C). Run:

```Get-Partition ```

## Get the Maximum Supported Size
To ensure you are "appending" all available space, run this command (replace 0 and 2 with your actual Disk and Partition numbers):

```
$MaxSize = (Get-PartitionSupportedSize -DiskNumber 0 -PartitionNumber 1).SizeMax
```

here my partitionnumber is 1


## Resize the Partition
This applies that max size to your C drive.

```
Resize-Partition -DiskNumber 0 -PartitionNumber 1 -Size $MaxSize
```

## Verify the Result
Run this to make sure your C drive now shows the full capacity:

```
Get-Volume -DriveLetter C
```
