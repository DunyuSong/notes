# JMeterIntegrateInfluxdb2

# JMeter Install

1.  建议使用JMeter5.4版本[JMeterDownload](https://jmeter.apache.org/download_jmeter.cgi)。

## Influxdb2 install

1. 官网下载及安装方法：https://portal.influxdata.com/downloads/ 

    ```
    wget https://dl.influxdata.com/influxdb/releases/influxdb2-2.0.4.x86_64.rpm
    sudo yum localinstall influxdb2-2.0.4.x86_64.rpm
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.18.0-1.x86_64.rpm
    sudo yum localinstall telegraf-1.18.0-1.x86_64.rpm
    ```

2. github及安装方法：官网如果下载不了从github上下载

    1. https://github.com/influxdata/influxdb/releases/tag/v2.0.3
    2. https://github.com/influxdata/telegraf/releases/tag/v1.18.0

    ```
    wget https://dl.influxdata.com/influxdb/releases/influxdb2-2.0.3.x86_64.rpm
    rpm -iv influxdb2-2.0.3.x86_64.rpm
    wget https://dl.influxdata.com/telegraf/releases/telegraf-1.18.0-1.x86_64.rpm
    rpm -iv telegraf-1.18.0-1.x86_64.rpm
    
    ```

3. 启动服务：sudo service influxdb start

4. 访问：http://192.168.163.11:8086/  第一次初始化需要输入账号、密码、组织、bucket等信息。

5. 如果需要通过CLI（command line interface），[官网](https://docs.influxdata.com/influxdb/v2.0/get-started/)建议我们做一个设置以避免每次都需要进行身份验证。

    ```
     influx config create -n default \
        -u http://192.168.163.11:8086 \
        -o MillerOrg \
        -t ripoNMQhBlscArM7Nr_mGnjroq_dltBUxAqf5eI4zXsSIRion_eu0_f-DYIeZIFQ5Rc5zztVD34FEUNQQETNLQ== \
        -a
    ```

### Influxdb 整合 JMeter

1. JMeter数据写入Influxdb需要在influxdb中生成一个token用于外部系统写入数据库。
2. 在Influxdb中选择 **Data > Tokens**, 点击 **Generate > Read**/**Write Token** .
3. 
4. *特别注意*：influxDB 2.0版本相对1.x版本改动较大，尤其是语法方面的改动，2.0版本的语法使用的是*JavaScript*，1.x使用的是*sql*。
5. 使用 ：https://blog.csdn.net/qq_29648139/article/details/104071486



















