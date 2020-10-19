
# Jenkins部署笔记



## Ant自动测试

### 第一份可以用的build.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="test1" default="test" basedir=".">
	<property name="src_dir" value="/home/user/ant/"/>
	<target name="build">
		<mkdir dir="${src_dir}"/>
		<echo message="create work dir succeed!"/>

		<exec  executable="/home/user/test_bin" failonerror="true"/>
		<!-- exec dir="/home/user/" executable="shell.sh" failonerror="true"/-->
	</target>
</project>

```

### 公司用的build.xml模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="test1" default="run" basedir=".">
	<property name="src_dir" value="/home/share/"/>

	<target name="run">
		<antcall target="build" />
	</target>
	<target name="build">
		<!--mkdir dir="./ppp" /-->
		<echo message="start build succeed!"/>
		<exec executable="${src_dir}/test/test_mcore" output="mcore.log" failonerror="true"/>
		<exec executable="${src_dir}/test/test_mdps" failonerror="true"/>
		<exec executable="svn" dir="${src_dir}/svnproject/" output="./svnproject/log.xml" failonerror="true">
			<arg line="log -v -l 2 --xml  " />
		</exec>
	</target>	
</project>	
```

###使用中的build.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project name="auto" default="run" basedir=".">
    
    <property name="src_dir" value="/root/project/src/platforms/linux-x86/bin"/>
    
    <target name="run">
        <antcall target="find_file" />
        <antcall target="build" />
    </target>

     <target name="find_file">
        <condition property="is.exist">
            <and>
                <available file="result.js" /> 
            </and>
        </condition>
        <antcall target="delete_file" />
    </target>

    <target name="delete_file" if="is.exist">
        <echo message="file exist!" />
        <delete file="result.js" />
    </target>
    
    <target name="build">
        <echo message="start build!" />
        <exec executable="${src_dir}/com.mining.app.test_mfsdk" failonerror="true" />
        <!--copy todir="./" file="${src_dir}/com.mining.app.test_mfsdk" /-->
        <!--copy todir="./" file="${src_dir}/result.js" /-->
    </target>

</project>
```



## 修改mtest框架，生成报告

### 第一步：规范化
尝试将mcore的单个测试函数改成mtest的规范型，规定好输入与输出。

### 第二步：转json

将mtest中获取到的测试数据转化为json格式。

#### 设计json数据框架

```json
[
    {
        "suite_name":"test_sample",
        "test_case":[
            {	"name":"add",
                "result":"successful",
                "time":10
            },
            {	"name":"sub",
                "result":"failed",
                "time":20
            },
            {
                "name":"sum",
                "result":"successful",
                "time":0
            },
            .......
        ],
        "total":10,
        "success":3,
        "failure":7
    },
    {
        "suite_name":"test_mcore",
        "test_case":[
			{
				"name":"test mcore md5 ex enctypt",
				"result":"success",
				"time":0
			},
			{
				"name":"test mcore base64",
				"result":"success",
				"time":0
			}, 	   
			{
				"name":"test mcore mparams",
				"result":"success",
				"time":0
			},
            .......
		],
        "total":42,
		"success":22,
		"failure":11
    },
    .......
]
```



#### 自己~~写的json数据~~

```json
{
    NB:{
        "one":"shenzhen",
        "two":1980
    },
    XB:{
        "one":"guangzhou",
        "two":5588
    }
}  
```

#### 在mtest.c中~~取到的json数据~~

```json
{
	"":{
		"name":"add",
		"result":"success",
		"time":0
		},
	"":{
		"name":"sub",
		"result":"success",
		"time":0
		},
	"":{
		"name":"sum",
		"result":"success",
		"time":0
		}
}
```

#### 为了方便html调用，先将json数据文件转为js文件（即对象）---result.js

```js
var JSONObject= 
[
    {
        "suite_name":"test_sample",
        "test_case":[
            {	"name":"add",
                "result":"successful",
                "time":10
            },
            {	"name":"sub",
                "result":"failed",
                "time":20
            },
            {
                "name":"sum",
                "result":"successful",
                "time":0
            }
        ],
        "total":10,
        "success":3,
        "failure":7
    },
    {
        "suite_name":"test_mcore",
        "test_case":[
			{
				"name":"test mcore md5 ex enctypt",
				"result":"success",
				"time":0
			},
			{
				"name":"test mcore base64",
				"result":"success",
				"time":0
			}, 	   
			{
				"name":"test mcore mparams",
				"result":"success",
				"time":0
			}
		],
        "total":42,
		"success":22,
		"failure":11
    },
    "......."
]
```



### 第三步：html获取json数据

#### 调用js对象

在html中加入script标签

`<script src="result.js"> </script>`

####根据json数组成员生成列表

```javascript
<script type="text/javascript" >
    
function addtr(start_line,name,result,time)
{
    var tradd=tab.insertRow(start_line);//插入1行
        
	//插入3列，给内容赋值并添加类名
	var td1 =tradd.insertCell(0);
    td1.innerHTML=name;
	td1.classList.add("cls_name");
    var td2 =tradd.insertCell(1);
    td2.innerHTML=result;
    td2.classList.add("cls_result");
    var td3 =tradd.insertCell(2);
    td3.innerHTML=time;
    td3.classList.add("cls_time");
}
  
var headerid = "header";
for(var j=0;j<JSONObject.length-1;j++)
{
	var count = document.createElement("p");
	var count_1 = document.createElement("div");
	var count_2  = document.createElement("div");
	var count_3  = document.createElement("div");
	var count_4  = document.createElement("div");

	//加一个新count块在上一个count元素后
	document.getElementById(headerid).insertAdjacentElement('afterend',count);
	count.appendChild(count_1);//加4个小模块跟随在本count块内
	count.appendChild(count_2);
	count.appendChild(count_3);
	count.appendChild(count_4);

	//给4个模块添加类名：class="cls_count"
	count_1.classList.add("cls_count");
	count_2.classList.add("cls_count");
	count_3.classList.add("cls_count");
	count_4.classList.add("cls_count");

	//填入数据
	count_1.innerHTML="测试模块:"+JSONObject[j].suite_name;
	count_2.innerHTML="总个数:"+JSONObject[j].total;
	count_3.innerHTML="成功个数:"+JSONObject[j].success;
	count_4.innerHTML="失败个数:"+JSONObject[j].failure;

	//添加表格
	var tab = document.createElement("table");
	count.insertAdjacentElement('beforeend',tab);//将表格加在本count块里，最后一个元素后面
	
	//加一行表头
	addtr(0,"名称","结果","用时(ms)");
	
	tab.rows[0].cells[0].classList.add("clshead_name");
	tab.rows[0].cells[1].classList.add("clshead_result");
	tab.rows[0].cells[2].classList.add("clshead_time");	

	//将每个测试函数的数据写入表格
	for(var i=0;i<JSONObject[j].test_case.length;i++)
		addtr(i+1,
			JSONObject[j].test_case[i].name,
			JSONObject[j].test_case[i].result,
			JSONObject[j].test_case[i].time);
	
	//为了让下个测试套能找到上一个测试套的id，从而在其下方生成数据
	headerid = "count"+(j);
	count.id = headerid;
	console.log(count.id);

}

</script>
```

![测试报告初步模板](file:///C:/Users/Admin/Pictures/测试报告demo.png)



### 第四步：Ant调用build.xml

#### 默认规范

将report.html、build.xml都放在一个指定目录

> ~/workspace/jobs_name/html/

#### 自动运行、调用

Ant按照build.xml的命令自动运行后，在此目录下生成result.js。

于是report.html调用当前目录下的result.js，便能读取出数据并生成表格。




