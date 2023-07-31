#go-zero学习笔记

##api语法结构

api示例 

	// api语法版本
	syntax = "v1"
	
	// import literal
	import "foo.api"
	
	// import group
	import (
	    "bar.api"
	    "foo/bar.api"
	)
	info(
	    author: "songmeizi"
	    date:   "2020-01-08"
	    desc:   "api语法示例及语法说明"
	)
	
	// type literal
	
	type Foo{
	    Foo int `json:"foo"`
	}
	
	// type group
	
	type(
	    Bar{
	        Bar int `json:"bar"`
	    }
	)
	
	// service block
	@server(
	    jwt:   Auth
	    group: foo
	)
	service foo-api{
	    @doc "foo"
	    @handler foo
	    post /foo (Foo) returns (Bar)
	}

**api语法结构**

* syntax语法声明
* import语法块
* info语法块
* type语法块
* service语法块
* 隐藏通道

**syntax语法说明**
​

* syntax​：固定token，标志一个syntax语法结构的开始
​
* checkVersion​：自定义go方法，检测STRING是否为一个合法的版本号，目前检测逻辑为，STRING必须是满足(?m)"v[1-9][0-9]*"正则。
​
* STRING​：一串英文双引号包裹的字符串，如"v1"
一个api语法文件只能有0或者1个syntax语法声明，如果没有syntax，则默认为v1版本

**import语法说明**

* import​：固定token，标志一个import语法的开始
​
* checkImportValue​：自定义go方法，检测STRING是否为一个合法的文件路径，目前检测逻辑为，STRING必须是满足(?m)"(/?[a-zA-Z0-9_#-])+\.api"正则。
​
* STRING​：一串英文双引号包裹的字符串，如"foo.api"


**info语法说明**


* info​：固定token，标志一个info语法块的开始
​
* checkKeyValue​：自定义go方法，检测VALUE是否为一个合法值。
​
* VALUE​：key对应的值，可以为单行的除'\r','\n','/'后的任意字符，多行请以""包裹，不过强烈建议所有都以""包裹


**type语法说明**

* 保留了golang内置数据类型​bool​,​int​,​int8​,​int16​,​int32​,​int64​,​uint​,​uint8​,​uint16​,​uint32​,​uint64​,​uintptr ​,​float32​,​float64​,​complex64​,​complex128​,​string​,​byte​,​rune​,

* 兼容golang struct风格声明

* 保留golang关键字

**service语法说明**

* serviceSpec​：包含了一个可选语法块atServer和serviceApi语法块，其遵循序列模式（编写service必须要按照顺序，否则会解析出错）
​
* atServer​： 可选语法块，定义key-value结构的server metadata，'@server' 表示这一个server语法块的开始，其可以用于描述serviceApi或者route语法块，其用于描述不同语法块时有一些特殊关键key 需要值得注意，见 atServer关键key描述说明。
* serviceApi​：包含了1到多个serviceRoute语法块
* serviceRoute​：按照序列模式包含了atDoc,handler和route
* atDoc​：可选语法块，一个路由的key-value描述，其在解析后会传递到spec.Spec结构体，如果不关心传递到spec.Spec, 推荐用单行注释替代。
* handler​：是对路由的handler层描述，可以通过atServer指定handler key来指定handler名称， 也可以直接* 用* * atHandler语法块来定义handler名称
* atHandler​：'@handler' 固定token，后接一个遵循正则[_a-zA-Z][a-zA-Z_-]*)的值，用于声明一个handler名称
* route​：路由，有httpMethod、path、可选request、可选response组成，httpMethod是必须是小写。
* body​：api请求体语法定义，必须要由()包裹的可选的ID值
* replyBody​：api响应体语法定义，必须由()包裹的struct、array(向前兼容处理，后续可能会废弃，强烈推荐以struct包裹，不要直接用array作为响应体)
* kvLit​： 同info key-value
* serviceName​: 可以有多个'-'join的ID值
* path​：api请求路径，必须以'/'或者'/:'开头，切不能以'/'结尾，中间可包含ID或者多个以'-'join的ID字符串


​

