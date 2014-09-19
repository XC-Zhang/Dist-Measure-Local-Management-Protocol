####测距仪控制程序与本地服务器通信协议
--------

#####1.数据通信格式


```javascript
//JSON Schema
{
	"description": "Exchange Data(数据交换格式)",
	"type": "object",
	"properties": {
		"Command": {
			"type": "string",
			"description": "用来定义每次通信的具体内容"
		},
		"Para": {
			"type": "string",
			"description": "本次通信的具体参数或结果,根据

Command的不同有不同的格式"
		}
	}
}
//以上是对本次协议中的通讯数据符合JSON Schema格式的描述.
```

-----------

#####2.数据通信具体实现

在C#控制程序里,通过PubSub类库来交换通信数据,每次传入的数据为上述的*Exchange 

Data*格式的序列化字符串.

C#控制程序应自行将其序列化成符合*Exchange Data*格式的Class后再进行处理.

C#客户端通过Subsribe侦听命令,通过Publish发送结果.

侦听的```Channel```是在 **DistMeasureManage:ToController** 频道上,具体的命令操作

发布的```Channel```是在 **DistMeasureManage:FromController** 频道上,具体的命令操作

通过解析后的Command来获取.

根据不同的Command再解析不同的Para,来最终履行协议.

-----------
#####3.测量一个点
```javascript
//Subsribe ToController
{
    Command: "Measure",
    Para: {
        "title": "要测量的仪器编号",
        "type": "integer",
    }
}
//示例
{
    Command: "Measure",
    Para: 32
}
```

```javascript
//Publish FromController
{
    Command: "Measure",
    Para: {
        "description": "测量结果数据",
        "type": "object",
        "properties": {
            "Device": {
                "description": "仪器的编号",
                "type": "integer"
            },
            "Value": {
                "description": "测量返回的距离",
                "type": "float"
            }
        }
    }
}
//示例
{
    Command: "Measure",
    Para: {
    	Device:33,
    	Value:5.321
    }
}
```

------
#####4.打开或关闭激光
```javascript
//Subsribe
{
    Command: "ToggleLaser",
    Para: {
        "description": "数据信息",
        "type": "object",
        "properties": {
            "Device": {
                "description": "仪器的编号",
                "type": "integer"
            },
            "Toggle": {
                "description": "开关指示,1是开,0是关",
                "type": "integer"
            }
        }
    }
}
//示例
{
    Command: "ToggleLaser",
    Para: {
        Device: 43,
        Toggle: 1
    }
}
```

```javascript
//Publish
//格式同上,不赘述
//示例
{
    Command: "ToggleLaser",
    Para: {
        Device: 43,
        Toggle: 1//打开成功
    }
}
```

#####5.测量一组仪器
```javascript
//Subsribe
{
    Command: "MeasureGroup",
    Para: {
        "description": "测量数据信息",
        "type": "object",
        "properties": {
            "StartDevice": {
                "description": "起始仪器编号",
                "type": "integer"
            },
            "EndDevice": {
                "description": "终止仪器编号",
                "type": "integer"
            }
        }
    }
}
//示例
{
    Command: "MeasureGroup",
    Para: {
        StartDevice: 3,
        EndDevice: 22
    }
}//从3号测量到22号仪器

//Publish
//格式同Measure,每完成一个仪器测量,发布一个结果
```
