title: 使用WPF加载Unity可执行程序
date: 2019-04-09 11:23:31
tags:
categories: WPF
---
  公司有开发wpf需求,模型主要是由Unity开发,导出后是`.exe`可执行程序,
外层是利用react-native编写界面,主要组件为react-native-windows,于是需要融合二者.
  首先是写入wpf调用外部`exe`组件:

```
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Windows;
using System.Windows.Interop;

namespace Vesal_PC
{
    // GameHost is a FrameworkElement and can be added to controls like so:
    // var gameHost = new GameHost(container.ActualWidth, container.ActualHeight);
    // container.Child = gameHost;
    //
    // Sources:
    // - https://msdn.microsoft.com/en-us/library/ms752055.aspx
    // - Another source I can't remember or find, concerning running a standalone Unity process using parentHWND parameter
    
    internal class GameHost : HwndHost
    {
        private const int WS_CHILD = 0x40000000;
        private const int WS_VISIBLE = 0x10000000;
        private const int HOST_ID = 0x00000002;

        private IntPtr hwndHost;
        private int hostHeight, hostWidth;

        private Process process;
        private IntPtr unityHWND = IntPtr.Zero;

        private const int WM_ACTIVATE = 0x0006;
        private readonly IntPtr WA_ACTIVE = new IntPtr(1);
        private readonly IntPtr WA_INACTIVE = new IntPtr(0);
        public GameHost(double width, double height)
        {
            hostHeight = (int)height;
            hostWidth = (int)width;
        }


        protected override HandleRef BuildWindowCore(HandleRef hwndParent)
        {
            hwndHost = IntPtr.Zero;

            hwndHost = CreateWindowEx(0, "static", "",
                                      WS_CHILD | WS_VISIBLE,
                                      0, 0,
                                      hostWidth, hostHeight,
                                      hwndParent.Handle,
                                      (IntPtr)HOST_ID,
                                      IntPtr.Zero,
                                      0);

            {
                try
                {
                    process = new Process();
                    process.StartInfo.FileName = AppDomain.CurrentDomain.BaseDirectory + "win/vesal.exe";
                    process.StartInfo.Arguments = "-parentHWND " + hwndHost.ToInt32() + " " + Environment.CommandLine;
                    process.StartInfo.UseShellExecute = true;
                    process.StartInfo.CreateNoWindow = true;

                    process.Start();

                    process.WaitForInputIdle();
                    // Doesn't work for some reason ?!
                    //unityHWND = process.MainWindowHandle;
                    EnumChildWindows(hwndHost, WindowEnum, IntPtr.Zero);
                }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message + "GameHost: Could not find game executable.");
                }
            }

            return new HandleRef(this, hwndHost);
        }

        protected override void DestroyWindowCore(HandleRef hwnd)
        {
            try
            {
                process.CloseMainWindow();

                System.Threading.Thread.Sleep(1000);
                while (process.HasExited == false)
                    process.Kill();
            }
            catch (Exception)
            {

            }

            DestroyWindow(hwnd.Handle);
        }

        private void ActivateUnityWindow()
        {
            SendMessage(unityHWND, WM_ACTIVATE, WA_ACTIVE, IntPtr.Zero);
        }

        private void DeactivateUnityWindow()
        {
            SendMessage(unityHWND, WM_ACTIVATE, WA_INACTIVE, IntPtr.Zero);
        }

        protected override IntPtr WndProc(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
        {
            handled = false;
            return IntPtr.Zero;
        }

        private int WindowEnum(IntPtr hwnd, IntPtr lparam)
        {
            unityHWND = hwnd;
            ActivateUnityWindow();
            return 0;
        }

        //PInvoke declarations
        [DllImport("user32.dll", EntryPoint = "CreateWindowEx", CharSet = CharSet.Unicode)]
        internal static extern IntPtr CreateWindowEx(int dwExStyle,
                                                      string lpszClassName,
                                                      string lpszWindowName,
                                                      int style,
                                                      int x, int y,
                                                      int width, int height,
                                                      IntPtr hwndParent,
                                                      IntPtr hMenu,
                                                      IntPtr hInst,
                                                      [MarshalAs(UnmanagedType.AsAny)] object pvParam);

        [DllImport("user32.dll", EntryPoint = "DestroyWindow", CharSet = CharSet.Unicode)]
        internal static extern bool DestroyWindow(IntPtr hwnd);

        [DllImport("User32.dll")]
        static extern bool MoveWindow(IntPtr handle, int x, int y, int width, int height, bool redraw);

        internal delegate int WindowEnumProc(IntPtr hwnd, IntPtr lparam);
        [DllImport("user32.dll")]
        internal static extern bool EnumChildWindows(IntPtr hwnd, WindowEnumProc func, IntPtr lParam);

        [DllImport("user32.dll")]
        static extern int SendMessage(IntPtr hWnd, int msg, IntPtr wParam, IntPtr lParam);
    }
}

```

编写react-native UnityView组件:
> ReactUnityManager.cs

```
// Copyright (c) Microsoft Corporation. All rights reserved.
// Portions derived from React Native:
// Copyright (c) 2015-present, Facebook, Inc.
// Licensed under the MIT License.

using Newtonsoft.Json.Linq;
using ReactNative.Collections;
using ReactNative.Modules.Image;
using ReactNative.UIManager;
using ReactNative.UIManager.Annotations;
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Threading;
using System.Windows.Controls;

namespace Vesal_PC
{
    /// <summary>
    /// The view manager responsible for rendering native images.
    /// </summary>
    public class ReactUnityManager : SimpleViewManager<Border>
    {
        private readonly ConcurrentDictionary<Border, List<KeyValuePair<string, double>>> _imageSources =
            new ConcurrentDictionary<Border, List<KeyValuePair<string, double>>>();
        Border view;

        /// <summary>
        /// The view manager name.
        /// </summary>
        public override string Name
        {
            get
            {
                return "RCTUnityView";
            }
        }

        /// <summary>
        /// The view manager event constants.
        /// </summary>
        public override JObject CustomDirectEventTypeConstants
        {
            get
            {
                return new JObject
                {
                    {
                        "topLoadStart",
                        new JObject
                        {
                            { "registrationName", "onLoadStart" }
                        }
                    },
                    {
                        "topLoad",
                        new JObject
                        {
                            { "registrationName", "onLoad" }
                        }
                    },
                    {
                        "topLoadEnd",
                        new JObject
                        {
                            { "registrationName", "onLoadEnd" }
                        }
                    },
                    {
                        "topError",
                        new JObject
                        {
                            { "registrationName", "onError" }
                        }
                    },
                };
            }
        }

        

        /// <summary>
        /// Called when view is detached from view hierarchy and allows for 
        /// additional cleanup.
        /// </summary>
        /// <param name="reactContext">The React context.</param>
        /// <param name="view">The view.</param>
        public override void OnDropViewInstance(ThemedReactContext reactContext, Border view)
        {
            base.OnDropViewInstance(reactContext, view);

            _imageSources.TryRemove(view, out _);
        }

        [ReactPropGroup(
            ViewProps.BorderWidth,
            ViewProps.BorderLeftWidth,
            ViewProps.BorderRightWidth,
            ViewProps.BorderTopWidth,
            ViewProps.BorderBottomWidth,
            DefaultDouble = double.NaN)]

         
        public void SetBorderWidth(Border view, int index, double width)
        {
            var _width = width;
            //view.SetBorderWidth(ViewProps.BorderSpacingTypes[index], width);
        }
        /// <summary>
        /// Creates the image view instance.
        /// </summary>
        /// <param name="reactContext">The React context.</param>
        /// <returns>The image view instance.</returns>
        protected override Border CreateViewInstance(ThemedReactContext reactContext)
        {

           
            var border = new Border
            {
                
            };
            var gameHost = new GameHost((int)MainWindow.mw.ActualWidth, (int)MainWindow.mw.ActualHeight-60);
            // container.Child = gameHost;
            border.Child = gameHost;
            var a = view;
            return border;
        }

        
    }
}

```
注册组件:
>MyReactPage.cs

```
using ReactNative.Bridge;
using ReactNative.Modules.Core;
using ReactNative.UIManager;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Vesal_PC.MyModel;

namespace Vesal_PC
{
    public class MyReactPage : IReactPackage
    {
        public static ReactContext context;
        public IReadOnlyList<INativeModule> CreateNativeModules(ReactContext reactContext)
        {
            context = reactContext;
            MyDialogModel mdm = new MyDialogModel(reactContext);
        
            return new List<INativeModule>
        {
               mdm,
        };
        }

        public IReadOnlyList<IViewManager> CreateViewManagers(ReactContext reactContext)
        {

            return new List<IViewManager> {
                    new ReactUnityManager()
            };

        }
    }
}

```
>AppReactPage.cs

```
using ReactNative;
using ReactNative.Bridge;
using ReactNative.Modules.Core;
using ReactNative.Shell;
using ReactNative.UIManager;
using System;
using System.Collections.Generic;
using ReactNative.Modules.Dialog;
namespace Vesal_PC
{
    internal class AppReactPage : ReactPage
    {
        public override string MainComponentName => "Vesal_PC";

        public override string JavaScriptMainModuleName => "index";

#if BUNDLE
        public override string JavaScriptBundleFile => AppDomain.CurrentDomain.BaseDirectory + "ReactAssets/index.wpf.bundle";
        
#endif

        public override List<IReactPackage> Packages => new List<IReactPackage>
        {
            new MainReactPackage(),
            new MyReactPage(),
        };

        public override bool UseDeveloperSupport
        {
            get
            {
#if !BUNDLE || DEBUG
                return true;
#else
                return false;
#endif
            }
        }

        

    }


}

```
前端:

>UnityView.js 

```
import { PropTypes, Component } from "react";
import { requireNativeComponent, View, ViewPropTypes } from "react-native";

class UnityView extends Component {
  render() {
    return <RCTUnityView {...this.props} />;
  }
}
module.exports = requireNativeComponent("RCTUnityView", UnityView);

```

使用:
```
    <UnityView style={{width:1680,height:1050}}/>
```