1. mcu -> elevator:  
[1,s,c]\n  
每一位代表：floor state door  
state: u up, d down, s stop  
door:c closed, o opened, i closing/opening    
错误代码：  
[ERROR_W]：字头错误  
[ERROR-S]：长度不对，目前是6bytes  
[ERROR-C]：crc检验错误  

2. elevator -> mcu  
struct cmd  
{  
    unsigned char header = 0xFE;  
    unsigned char cmd;   
    short int floor=0;  
    unsigned short int u16CRC;  
}  
cmd: 1 预约电梯 2 开门  3 关门  4 取消开门  5 取消关门  
现在也改为json格式:  
呼梯[1,floor_num], e.g. [1,5]按5楼  
操作门：开门[2] 关门[3] 取消开门[4] 取消关门[5]  

    
3. elevator -> cloud   pub:/cti/ele/state   
{"ID":"", "floorNum_r":"", "state":"", "door":""}  
floorNum_r: 相对楼层，有负楼层    
state: up, down, stop  
door: opened closed  
ID: 电梯唯一ID 00001   
lora通道增加查询命令：T是UNIX时间戳为毫秒1574773000652  
所有走lora通道的都是半双工通信，必须一问一答之后再进行下一次通信    
lora通信加上3位的request {"R":"xxxx"} 任意4个字符    
lora发送命令最后加\n换行符  
{"I":"00097","C":"CH","T":"1574773000652","R":"2sw2"}  
返回：  
{"E":"O","I":"00097", "F":"-2", "S":"U/D/S", "D":"O/C"，"T":"1574773000652","R":"2sw2"}  

 

4. cloud -> elevator  pub:/cti/ele/cmd  
{"ID":"", "requestID":"", "cmd":"call", "floorNum_r":""}  
{"ID":"", "requestID":"", "cmd":"close", "duration":""}    
{"ID":"", "requestID":"", "cmd":"open", "duration":""}    
{"ID":"", "requestID":"", "cmd":"cancelclose"}    
{"ID":"", "requestID":"", "cmd":"cancelopen"}   
floorNum:绝对楼层从1开始    
duration:表示要按住几秒钟，30表示30s    
requestID:表示指令的唯一性  
Lora通道：  
I：电梯ID C：cmd F：楼层  T：时间戳 R：requestID   
CA：call呼梯 O：open开门 CO：cancelopen取消开门 CL：close关门 CC：cancelclose取消开门  
D：duration开关门时间  
呼梯    {"I":"00097","C":"CA","F":"-3","T":"1574771219","R":"2122"}  
开门    {"I":"00097","C":"O","D":"10","T":"1574771219","R":"1892"}  
取消开门 {"I":"00097","C":"CO","T":"1574771219","R":"9922"}  
关门    {"I":"00097","C":"CL","D":"10","T":"1574771219","R":"2221"}  
取消关门 {"I":"00097","C":"CC","T":"1574771219","R":"2236"}  


5. elevator -> cloud   pub:/cti/ele/cmd-rsp   
{"requestID":"", "ret":"", "msg":""}  
ret: OK FAIL  
msg:   floor num is wrong / duration can't be small than 1  
lora通道：  
E:返回错误 M:msg 具体错误信息 T：时间戳 R：requestID  
错误：{"E":"F","M":"","T":"1574771219","R":"2022"}    
正确：{"E":"O","T":"1574771219","R":"999s"}      
M：f num is w / d small than 1  



#### 楼层映射
位于梯控侧  
| 云端 | -3 | -2 |-1  |1 |3 |5 |7 |7M |9 |
| -------- | -------- | -------- |-------- |-------- |-------- |-------- |-------- |-------- |-------- |
| 梯控 | -3 | -2  |-1 |1 |2 |3 |4 |5  |6|




