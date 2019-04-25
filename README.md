# unity-iOS-errors
***本仓库主要放了unity项目整合到iOS原生项目中的教程及记录一下在进行unity项目整合到iOS原生项目中的错误及解决办法，会陆续更新遇到的错误及解决办法。***

## 一
**在应用了第三方(微信、微博、QQ、U盟)静态SDK时，在TARGETS -> Build Phases -> Link Binary With Libraries中删除相应的静态连接，可以直接引用(在桥接文件中import一下)**

## 二
如果Unity版本比较低，可能会遇到xxx was compiled with optimization - stepping may behave oddly; variables may not be available的问题
**请在building setting-> Other C Flags 中添加:-DRUNTIME_IL2CPP=1，前面加的也要保留。然后把Optimization Level ，debug的部分都改成none**

## 三
遇到clang编译错误时可能是没有添加UnityCallBack方法导致的
**在Classes -> UnityAppController.mm的init方法前加入**
```
#ifdef __cplusplus
extern "C"{
#endif
void UnityCallBack(const char *jsonStr);
#ifdef __cplusplus

}
#endif
void UnityCallBack(const char *jsonStr)
{
NSString *str = [NSString stringWithFormat:@"%s",jsonStr];

NSData *jsonData = [str dataUsingEncoding:NSUTF8StringEncoding];
NSError *err;
NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:jsonData options:NSJSONReadingMutableContainers error:&err];
//    NSLog(@"-------%s",jsonStr);
if (err) {
NSLog(@"-------解析失败");
return;
}
if (dic != nil && dic.count) {
//        MGDealUnityMessage *dealMessage = [[MGDealUnityMessage alloc] init];
//        [dealMessage setDataWithData:dic];
// 一下为你的原生方法接收Unity回调消息的类。我使用通知传值(因为使用的Swift，所以通知更方便)
[[NSNotificationCenter defaultCenter] postNotificationName:@"unityCallBack" object:dic];
}
}
```

## 四
最近在整合项目时总是编译出错在.cpp文件中缺少 or 找不到某个类的属性、方法。unity版本(2018.2.0f2)、Xcode(10.2.1)
解决办法: 
**试试在Other C Flags 中加入 
-fno-strict-overflow
-DINIT_SCRIPTING_BACKEND=1 
-DNET_4_0 
-DRUNTIME_IL2CPP=1
C Language Dialect 设置为GUN99**

## 五
not found image错误。 
**将log信息中的库在Link Binary With Libraries中改为Optional**
