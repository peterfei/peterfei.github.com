title: 记IOS开发之我是如何解决react-native-unity-view插件在Product模式闪退
date: 2019-04-10 09:44:13
tags:
- IOS
- React-native
- react-native-unity-view
categories: IOS
---

>  事件起因:

> 公司在使用react-native开发解剖3D软件,需要和unity进行融合,在组件选择上,使用了react-native-unity-view这款插件,在融合过程中,踩了很多坑,花费精力最大的,是年后IOS升级为12.1后,集成方案集中出现闪退.

##### Issue #79
[iOS crashes on launch in release builds only #79](https://github.com/f111fei/react-native-unity-view/issues/79)

> Cash log 如下

```
Thread 7 name:  com.facebook.react.JavaScript
Thread 7 Crashed:
0   libsystem_kernel.dylib        	0x00000001fbbbd104 __pthread_kill + 8
1   libsystem_pthread.dylib       	0x00000001fbc3ca00 pthread_kill$VARIANT$armv81 + 296
2   libsystem_c.dylib             	0x00000001fbb14d78 abort + 140
3   libsystem_malloc.dylib        	0x00000001fbc11768 _malloc_put + 0
4   libsystem_malloc.dylib        	0x00000001fbc11924 malloc_report + 64
5   libsystem_malloc.dylib        	0x00000001fbc042d0 free + 376
6   libc++.1.dylib                	0x00000001fb1bf120 std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >::~basic_string+ 258336 () + 32
7   UnityTest                     	0x0000000102de23a4 std::__1::__vector_base<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > > >::~__vector_base() + 730020 (vector:451)
8   UnityTest                     	0x0000000103385214 facebook::react::ModuleRegistry::getConfig+ 6640148 (std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&) + 100
9   UnityTest                     	0x0000000103394168 facebook::react::JSCNativeModules::createModule+ 6701416 (std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> > const&, OpaqueJSContext const*) + 292
10  UnityTest                     	0x0000000103393cc8 facebook::react::JSCNativeModules::getModule+ 6700232 (OpaqueJSContext const*, OpaqueJSString*) + 188
11  UnityTest                     	0x000000010338e85c OpaqueJSValue const* (*facebook::react::(anonymous namespace)::exceptionWrapMethod<&(facebook::react::JSCExecutor::getNativeModule(OpaqueJSValue*, OpaqueJSString*))>())(OpaqueJSContext const*, OpaqueJSValue*, OpaqueJSString*, OpaqueJSValue const**)::funcWrapper::call+ 6678620 (OpaqueJSContext const*, OpaqueJSValue*, OpaqueJSString*, OpaqueJSValue const**) + 104
12  JavaScriptCore                	0x0000000203350404 JSC::JSCallbackObject<JSC::JSDestructibleObject>::getOwnPropertySlot+ 574468 (JSC::JSObject*, JSC::ExecState*, JSC::PropertyName, JSC::PropertySlot&) + 340
13  JavaScriptCore                	0x0000000203a354f0 llint_slow_path_get_by_id + 2008
14  JavaScriptCore                	0x0000000203328928 llint_entry + 11528
15  JavaScriptCore                	0x000000020332d134 llint_entry + 29972
16  JavaScriptCore                	0x000000020332d134 llint_entry + 29972
17  JavaScriptCore                	0x000000020332d134 llint_entry + 29972
18  JavaScriptCore                	0x000000020332d134 llint_entry + 29972
19  JavaScriptCore                	0x000000020332d134 llint_entry + 29972
20  JavaScriptCore                	0x0000000203325a1c vmEntryToJavaScript + 300
21  JavaScriptCore                	0x000000020399bfe4 JSC::Interpreter::executeProgram+ 7176164 (JSC::SourceCode const&, JSC::ExecState*, JSC::JSObject*) + 9620
22  JavaScriptCore                	0x0000000203b77218 JSC::evaluate+ 9122328 (JSC::ExecState*, JSC::SourceCode const&, JSC::JSValue, WTF::NakedPtr<JSC::Exception>&) + 316
23  JavaScriptCore                	0x000000020334e634 JSEvaluateScript + 472
24  UnityTest                     	0x000000010336dad0 facebook::react::evaluateScript+ 6544080 (OpaqueJSContext const*, OpaqueJSString*, OpaqueJSString*) + 80
25  UnityTest                     	0x000000010338c918 facebook::react::JSCExecutor::loadApplicationScript+ 6670616 (std::__1::unique_ptr<facebook::react::JSBigString const, std::__1::default_delete<facebook::react::JSBigString const> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >) + 528
26  UnityTest                     	0x0000000103392ba4 std::__1::__function::__func<facebook::react::NativeToJsBridge::loadApplication(std::__1::unique_ptr<facebook::react::RAMBundleRegistry, std::__1::default_delete<facebook::react::RAMBundleRegistry> >, std::__1::unique_ptr<facebook::react::JSBigString const, std::__1::default_delete<facebook::react::JSBigString const> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >)::$_0, std::__1::allocator<facebook::react::NativeToJsBridge::loadApplication(std::__1::unique_ptr<facebook::react::RAMBundleRegistry, std::__1::default_delete<facebook::react::RAMBundleRegistry> >, std::__1::unique_ptr<facebook::react::JSBigString const, std::__1::default_delete<facebook::react::JSBigString const> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >)::$_0>, void (facebook::react::JSExecutor*)>::operator()+ 6695844 (facebook::react::JSExecutor*&&) + 144
27  UnityTest                     	0x0000000103393ba8 std::__1::function<void (facebook::react::JSExecutor*)>::operator()+ 6699944 (facebook::react::JSExecutor*) const + 40
28  UnityTest                     	0x000000010330c9cc facebook::react::tryAndReturnError(std::__1::function<void + 6146508 ()> const&) + 24
29  UnityTest                     	0x0000000103302528 facebook::react::RCTMessageThread::tryFunc(std::__1::function<void + 6104360 ()> const&) + 24
30  CoreFoundation                	0x00000001fbfb6408 __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__ + 20
31  CoreFoundation                	0x00000001fbfb5d08 __CFRunLoopDoBlocks + 272
32  CoreFoundation                	0x00000001fbfb1220 __CFRunLoopRun + 2376
33  CoreFoundation                	0x00000001fbfb05b8 CFRunLoopRunSpecific + 436
34  UnityTest                     	0x00000001032e36a4 +[RCTCxxBridge runRunLoop] + 264
35  Foundation                    	0x00000001fcad73b0 __NSThread__start__ + 1040
36  libsystem_pthread.dylib       	0x00000001fbc412fc _pthread_body + 128
37  libsystem_pthread.dylib       	0x00000001fbc4125c _pthread_start + 48
38  libsystem_pthread.dylib       	0x00000001fbc44d08 thread_start + 4

```
##### 最初的解决方案
- 升级`react-native`版本到0.57.0
- `rm -fr node_modules`
- 升级插件版本`react-native-unity-view": "^1.2.1"`
- 在Xcode里`Setting "Strip Linked Product" to NO`  

成功阻止了编译闪退,但发布后被审核告知闪退.来来回回几次后,终于重新测试发现,居然是打包为`release IPA file`后才开始闪退-_-!!

##### 再次尝试
之后在issue里提问回帖,得到如下方案:

在配置文件里加入


```
COPY_PHASE_STRIP = YES;
ENABLE_BITCODE = NO;
STRIP_INSTALLED_PRODUCT = NO;


```


之后在测试机上运行,没有闪退.于是提交审核,发布了版本.之后有用户反馈在`IPHONE XS Max`上闪退 ``-_-!!!``

##### 最后的尝试
之后继续回帖,在 [JanOwiesniak](https://github.com/JanOwiesniak)和[mtostenson ](https://github.com/mtostenson)的建议下,进行如下改动:


* using XCodes new build system or the legacy build system
* Reinstall node_modues 
* Update Unity from 2018.2.14f1 to 2018.3.6f1
* Build UnityExport
* Patch UnityExport [jiulongw/swift-unity#120 (comment)](https://github.com/jiulongw/swift-unity/pull/120#issuecomment-456315250) [#79 (comment)](https://github.com/f111fei/react-native-unity-view/issues/79#issuecomment-465819733)
* Update react-native-unity-view to 1.3.3
* Update react-native to 0.57+
* Patch react-native moduleNames() [(#79 (comment)](https://github.com/f111fei/react-native-unity-view/issues/79#issuecomment-465110101)
* Add AVKit.framework in Xcode


##### 最终成功解决iphone xs max 12.1兼容

