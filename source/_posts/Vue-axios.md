---
title: Vue-axios
date: 2022-12-19 18:08:31
tags: vue
categories: 知识点
---
众所周知，现在开发项目大多采用前后端分离的模式，个人采用vue做前端项目比较多一点。
说到前端，那前端肯定只是负责渲染展示数据的，那么这里有一个问题，数据从哪里来呢？
在开发阶段，大多会采用mock模拟数据，做一些假数据来满足暂时的需求，但是最终生产上的数据肯定是从后端接口获取而来。
如何在vue项目上从后端获取数据呢？这就是我们今天要说的内容啦～

# 一、前端从后端获取数据的方式通常采用http/https的方式
方法通常有GET、POST、PUT、DELETE、PATCH这五种；
GET==>用来获取数据，
POST==> 是用来新增数据表单提交或文件上传
DELETE==>是用来删除数据
PUT==>是用来更新数据（所有数据推送到后端）
PATCH==>是用来更新数据（只将修改的数据推送到后端）
# 二、从前端请求后端接口获取数据格式：
GET方法：
## 1、axios.get(url,config).then(res=>{数据处理逻辑}).catch(err=>{错误处理逻辑})
## 2、axios({method:'get',url:'xxxxx',config}).then(res=>{数据处理逻辑}).catch(err=>{错误处理逻辑})
可以在config中设置基础URL，超时时间、传参方式、请求头等信息，但是传参方式一般为params，请求参数在url中。

## POST方法（appcation/json或者form-data）：
### 1、①appcation/json方式
```
let data={id:12}
axios.post(url,data,config).then(res=>{数据处理逻辑}).catch(err=>{错误处理逻辑})
```

### ②appcation/json方式：
```
let data={id:12}
axios({method:'post',url,data:data,config}).then(res=>{数据处理逻辑}).catch(err=>{错误处理逻辑})
```

### 2、①form-data方式
```
let data={id:12}
let formData = new FormData()
for(let key in data){
formData.append(key,data[key])
}
axios.post(url,formData,config).then(res=>{数据处理逻辑}).catch(err=>{错误处理逻辑})
```
### ②form-data方式
```
let data={id:12}
let formData = new FormData()
for(let key in data){
formData.append(key,data[key])
}
axios({method:'post',url,formData:formData,config}).then(res=>{数据处理逻辑}).catch(err=>{错误处理逻辑})
```
post请求同样可以在在config中设置基础URL，超时时间、传参方式、请求头等信息，但是传参方式一般用data，参数在请求体中。
## PUT和PATCH方式：
put和patch跟post一样，就方法不一样而已，参考post方法。
## DELETE方式：
类似get方式，就方法不一样而已，参考get方法。
可以在config中设置基础URL，超时时间、传参方式、请求头等信息，
注意请求参数，如果在url中则config中采用params方式传参，如果在请求体中，则config中采用data方式传参。
# 三、axios并发请求
## 并发请求：同时进行多个请求，并统一处理返回值。
比如说：我们需要同时请求用户信息和商品信息，然后将获得的两种信息进行拼接等统一加工到一起。
这涉及到axios的两种方法：axios.all(arr[])和axios.spread()。
## axios.all()和axios.spread()方法
其中axios.all(arr[])这个方法的作用就是同时去请求，它的参数是数组，数组的内容就是请求方式和路径。
比如：arr[] =[axios.get(url),axios.post(url,data,config)]
## 另一个axios.spread((A,B)=>{})
这个方法是用来处理返回的数据的，其中{}中是具体的处理逻辑，A就是第一个请求的返回值，B就是第二个请求的返回值。
具体用法：
```
axios.all([axios.get(url),axios.post(url,data,config)]).then(axios.spread((A,B)=>{}))
```
# 四、axios实例
你会不会有这样一个疑问，就是为什么要用axios实例呢？
那可能是因为在你的项目中会涉及到多个接口地址，而且每种接口的超时时间等等设置都不一样，
那么这个时候，你就可以针对某个接口地址某种超时时间来创建独立的axios实例，后期你就可以直接使用这种特殊axios实例去进行数据请求了。
具体用法：
```
let instance = axios.create(config);
instance.get(url).then(res=>{数据处理逻辑}).catch(err=>{错误处理逻辑})
```
# 五、关于请求中的config
```
config的格式为：
{
baseURL:'http:/xxxxxx', //基础url
timeout:6000, //超时时间
url:xxxxxx, //具体url
method:'get/post/put/patch/delete', //请求方式
headers:{token:'xxxxx'等}, //请求头设置
params:{}, //请求参数对象，它会将请求参数拼接到url上
data:{} //请求参数对象，它会将请求参数放到请求体中
}
```
config应用场景
## 1、全局配置
```
axios.defaults.timeout = 1000
axios.defaults.baseURL = 'http://XXXXX'
```
## 2、实例配置
在axios创建实例中配置
```
let instance = axios.create();
instance.defaults.timeout = 1000
```
## 3、请求配置
在请求中配置
```
axios.get(url,config).then(res=>{数据处理逻辑}).catch(err=>{错误处理逻辑})
```
其中，配置优先级为：3>2>1
# 六、axios拦截器
## 什么是拦截器？
拦截器就是在请求之前和响应之后能进行一些额外操作的功能实现。
一般分为请求拦截器和响应拦截器两种。
## 请求拦截器
请求拦截器顾名思义就是在请求之前进行拦截并进行一些额外操作。
```
service.interceptors.request.use(
config => {
//在发送请求前的额外处理
return config
},
)
```
## 响应拦截器
响应拦截器顾名思义就是在响应之后进行拦截并进行一些额外操作。
```
service.interceptors.response.use(
res => {
//响应之后做一些额外操作
return res
},
error => {
//在发生错误后的额外处理
return Promise.reject(error)
}
)
注意：不管是请求拦截器还是响应拦截器，在发生错误后都会进入到请求的catch方法中，如：
axios.get(url,config).then(res=>{数据处理逻辑}).catch(err=>{错误处理逻辑})
```
例子：发送请求前，在请求头中添加token，就可以用拦截器来实现
```
let instance = axios.create(config);
instance.interceptors.request.use(confit=>{
config.headers.token="sssssss"
return config
},
error => {
//在发生错误后的额外处理
return Promise.reject(error)
}
)
```
## 取消拦截器
顾名思义就是取消掉已经配置的拦截器

例子：
```
let instance = axios.create(config);
instance.interceptors.request.use(
config=>{
config.headers.token="sssssss"
return config
},
error => {
//在发生错误后的额外处理
return Promise.reject(error)
}
)
//取消拦截器操作：
axios.interceptors.request.eject(instance)
```
# 七、取消请求
取消请求实际中用的到不多，主要使用场景比如：用户正在批量查询一个比较耗时的数据，发起请求后用户不想查了，
这时候就可以取消这个请求,主要用到：axios.CancelToken.source()方法
例子：
```
axios.CancelToken.source()
axios.get(url,{CancelToken:source,token}).then(res=>{数据处理逻辑}).catch(err=>{错误处理逻辑})
//触发取消请求：
source.cancel('错误信息')
```
就可以了



