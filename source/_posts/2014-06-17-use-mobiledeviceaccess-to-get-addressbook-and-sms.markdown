---
layout: post
title: "使用MobileDeviceAccess获取iOS的短信和联系人"
date: 2014-06-17 15:53:10 +0800
comments: true
categories: MobileDeviceAccess OSX

---

### 目标  
使用MobileDeviceAccess来从iOS系统中读取以下内容：    
  
* 短信
* 联系人
* 已安装的软件

### 资源  
  
* [Github上的iTools例子](https://github.com/heiby/iTools) 该Demo已经实现了如下功能：      

    * 设备的检测。  
    * 设备信息读取。  
    * 已经安装的App管理。  
    * 用户文件管理。
* [关于MobileDeviceAccess的网站](www.libimobiledevice.org)   

### 获取短信、联系人  

MobileDeviceAccess里面封装了很多类，只需要使用某些类去获取自己想要的内容即可，下面记录一下过程：  
  
* 第一步：谷歌了一下iOS设备的联系人、短信的存储位置：`/private/var/mobile/Library/`
* 第二步：浏览了一下MobileDeviceAccess的各个类，发现了`AMFileRelay`这个类是用来读取文件集合的。
* 第三步：整理了一下`AMFileRelay`的说明，大概意思是该类定义了几个集合名称，而这些集合名称对应着iOS设备下不同的目录，经整理后的集合名称-目录对应关系如下：
<TABLE>
        <TR><TH>Set Name</TH><TH>File/Directory</TH></TR>
        <TR><TD>AppleSupport</TD><TD>/private/var/logs/AppleSupport</TD></TR>
        <TR><TD>Caches</TD><TD>/private/var/mobile/Library/Caches</TD></TR>
        <TR><TD>CrashReporter</TD><TD>/Library/Logs/CrashReporter<BR>/private/var/mobile/Library/Logs/CrashReporter</TD></TR>
        <TR><TD>MobileWirelessSync</TD><TD>/private/var/mobile/Library/Logs/MobileWirelessSync</TD></TR>
        
        <TR><TD>Lockdown</TD>
            <TD>/private/var/root/Library/Lockdown/activation_records
                <BR>/private/var/root/Library/Lockdown/data_ark.plist
                    <BR>/private/var/root/Library/Lockdown/pair_records
                    <BR>/Library/Logs/lockdownd.log
            </TD>
        </TR>
 
        <TR><TD>MobileInstallation</TD>
            <TD>/var/mobile/Library/Logs/MobileInstallation
                    <BR>/var/mobile/Library/Caches/com.apple.mobile.installation.plist
                   <BR>/var/mobile/Library/MobileInstallation/ArchivedApplications.plist
                   <BR>/var/mobile/Library/MobileInstallation/ApplicationAttributes.plist
                     <BR>/var/mobile/Library/MobileInstallation/SafeHarbor.plist
            </TD>
        </TR>
 
        <TR><TD>SafeHarbor</TD><TD>/var/mobile/Library/SafeHarbor</TD></TR>
        
        <TR><TD>Network</TD>
        <TD>/private/var/log/ppp
                <BR>/private/var/log/racoon.log
                <BR>/var/log/eapolclient.en0.log
        </TD>
        </TR>
 
        <TR><TD>SystemConfiguration</TD><TD>/Library/Preferences/SystemConfiguration</TD></TR>
        
        <TR><TD>UserDatabases</TD>
            <TD>/private/var/mobile/Library/AddressBook
                    <BR>/private/var/mobile/Library/Calendar
                    <BR>/private/var/mobile/Library/CallHistory
                    <BR>/private/var/mobile/Library/Mail/Envelope Index
                    <BR>/private/var/mobile/Library/SMS
            </TD>
        </TR>
        
        <TR><TD>VPN</TD><TD>/private/var/log/racoon.log</TD></TR>
        <TR><TD>WiFi</TD><TD>/var/log/wifimanager.log
 <BR>/var/log/eapolclient.en0.log</TD></TR>
        <TR><TD>tmp</TD><TD>/private/var/tmp</TD></TR>
</TABLE>
所以，需要使用的文件集合名称就是`UserDatabases`    

* 第四步：使用`AMFileRelay`读取文件集合，具体代码如下：   
 
```      
-(void)setDevice:(AMDevice *)device{
    if (![_device.udid isEqualToString:device.udid]) {
        _device=device;
        
        [_deviceInfo setObject:@([_device newAFCRootDirectory]!=nil) forKey:@"isJailbreak"];
        
        /**
         *  添加的代码
         *  作为测试，我们在获取设备信息的时候直接将文件读出来存储到桌面
         */
        AMFileRelay *relay = [device newAMFileRelay];
        NSOutputStream *output = [NSOutputStream outputStreamToFileAtPath:@"/Users/zhouguoyong/Desktop/UserDatabases.cpio" append:NO];
        [relay getFileSet:@"UserDatabases" into:output];
        /**
         *  添加的代码
         */

        id value=[_device deviceValueForKey:@"BatteryCurrentCapacity" inDomain:@"com.apple.mobile.battery"];
        if(value)[_deviceInfo setObject:value forKey:@"BatteryCurrentCapacity"];
        
        float total=[[_device deviceValueForKey:@"TotalDiskCapacity" inDomain:@"com.apple.disk_usage"] longLongValue]/1048576.f;
        float dataA=[[_device deviceValueForKey:@"TotalDataAvailable" inDomain:@"com.apple.disk_usage"] longLongValue]/1048576.f;
        float systemA=[[_device deviceValueForKey:@"TotalSystemAvailable" inDomain:@"com.apple.disk_usage"] longLongValue]/1048576.f;
        value=[NSString stringWithFormat:@"%.1fGB/%.1fGB",(dataA+systemA)/1024.f,total/1024.f];
        if(value)[_deviceInfo setObject:value forKey:@"DiskUsage"];
    }
}
```   
 
* 第五步：根据`AMFileRelay`的说明，读取出来的文件集合是[cpio](http://baike.baidu.com/view/2524122.htm)格式的，需要将其解压。~~解压命令：cpio -idmv < UserDatabases.cpio~~ 经过测试，OSX的归档文件应该是支持解压的，所以直接双击解压就可以了。
* 第六步：可以看到解压出来的文件都是sqlite的数据库文件，剩下的就是数据库操作了。    

### 问题  
* 在mac上的iTools工具只有在iCould关闭通讯录的时候才能读取到联系人，而使用上述方法获取联系人的时候，在开启和关闭iCould通讯录的时候都可以拿到数据，这一点让我比较奇怪，猜测是在iCould关闭的时候iOS设备上就下载了iCould上的内容，所有我后来的关闭iCould通讯录实际上都无伤大雅了。  
