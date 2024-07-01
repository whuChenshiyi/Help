# i2Nav 导航数据文件说明

## i2Nav_FootMount
### 1. Steps-Align-MIMU.DAT
   - 文件说明：Foot-INS后处理结果，正向滤波+反向平滑，并提取脚步点出的导航数据结果
   - 列数：4。Time(s), Latitude(deg), longitude(deg), Height(m)
   - 可将其转化为txt文件，命令为 
   - > python3 ReadBin.py Steps-Align-MIMU.DAT 4

### 2.sensors.txt
   - 文件说明：i2Nav_FootMount软件采集的传感器数据
   - 默认频率：50HZ
   - 传感器轴向：前右下
   - 列数：5。label，Time(s)，x，y，z
   - label：1，陀螺仪（deg/s）\
   &emsp;&emsp;&emsp; 2，加速度计（m/s^2）\
   &emsp;&emsp;&emsp; 3，扣除零偏的磁力计（μT）\
   &emsp;&emsp;&emsp; 14，未扣除零偏的磁力计（μT）\
   &emsp;&emsp;&emsp; 4，气压计（Pa）

### 3.imupos.txt
   - 文件说明：手机传感器时间与模块IMU时间对齐数据
   - 列数：4。PhoneTime(s)，IMUTime(s)，



## i2NavDataLogger
### 1.sensors.txt
   - 文件说明：i2NavDataLogger软件采集的传感器数据
   - 默认频率：200HZ
   - 传感器轴向：右前上
   - 列数：5。label，Time(s)，x，y，z
   - label：1，陀螺仪（deg/s）\
   &emsp;&emsp;&emsp; 2，加速度计（m/s^2）\
   &emsp;&emsp;&emsp; 3，磁力计（μT），其中磁力计的输出扣除了一个初始零偏值\
   &emsp;&emsp;&emsp; 4，气压计（Pa）

### 2.ntp_time.txt
   - 文件说明：手机传感器与连接WiFi内网设备的时间对齐文件
   - 列数：5。