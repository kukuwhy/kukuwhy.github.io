---
layout: post
title:  "child gating"
tags: frida
---

# 사용되는 주요 함수 정리

1. hook.py

```python
# -*- coding: utf-8 -*-
from __future__ import print_function

import threading

import frida
from frida_tools.application import Reactor

js_script = open('hookKill.js', 'r').read()

class Application(object):
    def __init__(self):
        self._stop_requested = threading.Event()
        self._reactor = Reactor(run_until_return=lambda reactor: self._stop_requested.wait())

        self._device = frida.get_usb_device(1)
        # self._device.enable_spawn_gating()
        self._sessions = set()
        self._device.on("spawn-added", lambda child: self._reactor.schedule(lambda: self._on_child_added(child)))
        self._device.on("spawn-removed", lambda child: self._reactor.schedule(lambda: self._on_child_removed(child)))
        self._device.on("child-added", lambda child: self._reactor.schedule(lambda: self._on_child_added(child)))
        self._device.on("child-removed", lambda child: self._reactor.schedule(lambda: self._on_child_removed(child)))
        self._device.on("output", lambda pid, fd, data: self._reactor.schedule(lambda: self._on_output(pid, fd, data)))

    def run(self):
        self._reactor.schedule(lambda: self._start())
        self._reactor.run()

    def _start(self):
        pid = self._device.spawn("com.kjbank.asb.cbanking")
        self._instrument(pid)

    def _stop_if_idle(self):
        if len(self._sessions) == 0:
            self._stop_requested.set()

    def _instrument(self, pid):
        try:
            print("✔ attach(pid={})".format(pid))
            session = self._device.attach(pid)
            session.on("detached",
                       lambda reason: self._reactor.schedule(lambda: self._on_detached(pid, session, reason)))
            print("✔ enable_child_gating()")
            session.enable_child_gating()
            print("✔ create_script()")
            global js_script
            script = session.create_script(js_script, runtime='v8')
            script.on("message", lambda message, data: self._reactor.schedule(lambda: self._on_message(pid, message)))
            print("✔ load()")
            script.load()
            print("✔ resume(pid={})".format(pid))
            self._device.resume(pid)
            self._sessions.add(session)
        except Exception as e:
            print(e)
            print("Something's wrong! resuming...")
            self._device.resume(pid)

    def _on_child_added(self, child):
        print("⚡ child_added: {}".format(child))
        if child.origin == 'exec':
            print('not instrumenting exec shit')
            self._device.resume(child.pid)
            return
        self._instrument(child.pid)

    def _on_child_removed(self, child):
        print("⚡ child_removed: {}".format(child))

    def _on_output(self, pid, fd, data):
        print("⚡ output: pid={}, fd={}, data={}".format(pid, fd, repr(data)))

    def _on_detached(self, pid, session, reason):
        print("⚡ detached: pid={}, reason='{}'".format(pid, reason))
        self._sessions.remove(session)
        self._reactor.schedule(self._stop_if_idle, delay=0.5)

    def _on_message(self, pid, message):
        try:
            print("⚡ message: pid={}, payload={}".format(pid, message["payload"]))
        except:
            print(message)

app = Application()
app.run()
```

1. hookKill.js

```jsx
//Interceptor.attach(Module.findExportByName(null, "ptrace"), {
//    onEnter: function (args) {
//        console.log("ptrace called!");
//        send("ptrace");
//        //console.log(args);
//    }
//});
//
//Interceptor.attach(Module.findExportByName(null, "exit"), {
//    onEnter: function (args) {
//        console.log("exit called!");
//        send("exit");
//        //console.log(args);
//    }
//});
//
//Interceptor.attach(Module.findExportByName(null, "abort"), {
//    onEnter: function (args) {
//        console.log("abort called!");
//        send("abort");
//        //console.log(args);
//    }
//});
//
//Interceptor.attach(Module.findExportByName(null, "fork"), {
//    onEnter: function (args) {
//        console.log("fork called!");
//        send("fork");
//        //console.log(args);
//    }
//});
//
//Interceptor.attach(Module.findExportByName(null, "vfork"), {
//    onEnter: function (args) {
//        console.log("vfork called!");
//        send("vfork");
//        //console.log(args);
//    }
//});
//
//Interceptor.attach(Module.findExportByName(null, "_end"), {
//    onEnter: function (args) {
//        console.log("fork called!");
//        send("fork");
//        //console.log(args);
//    }
//});
//
//Interceptor.attach(Module.findExportByName(null, "_exit"), {
//    onEnter: function (args) {
//        console.log("_exit called!");
//        send("_exit");
//        //console.log(args);
//    }
//});

Interceptor.replace(Module.findExportByName(null, "kill"), new NativeCallback(function(pid, sig){
    console.log("Tried to kill pid:",pid,"sig:",sig);
    send("kill sent")
    return 0;
},'int',['int','int']));

send("Started shit in this pid");
setImmediate( function() {
    Java.perform( function() {
        console.log("");
        console.log("[.] Debug check bypass");
        const ini = Java.use("kotlin.jvm.internal.Intrinsics");
        var stringClass = Java.use("java.lang.String");
        var stringInstance = stringClass.$new("qa");
        const Class = Java.use("java.lang.Class");
        const nat = Java.use("com.goggles.Native");
        const pureApp = Java.use("com.kjbank.asb.cbanking.ui.MainActivity$startPureApp$1");
        const pu = Java.use("com.kjbank.asb.cbanking.thirdparty.ahope.PureApp");

        pu.startPureApp.implementation = function() {
            console.log("[+] startPure hi");
        }

//        ini.areEqual.implementation = function(argv1, argv2) {
//            console.log("[+] PureApp Disable hooking");
//            if(stringInstance.equals(argv1))
//            {
//                console.log("[+] DEV == QA Hooking");
//                return true
//            }
//            return this.areEqual(argv1, argv2);
//        }

//
//        nat.a.overload('java.lang.String').implementation = function(argv) {
//            console.log("[+] NATIVE PRINT!!");
//            console.log(this.a("00004df66064683829db382c72e3c1a295627c8a4a3bf01c065cb8dc28f61ddedab261ccbf320c86e12cad3ab58a0871ec86e840818e9baad194f2a0fa95cb1d702bbd0c"));
//            return this.a(argv);
//        };

        var Debug = Java.use('android.os.Debug');
        Debug.isDebuggerConnected.implementation = function() {
            console.log('isDebuggerConnected Bypassed !');
            return false;
        }

        var process = Java.use('android.os.Process');
        process.killProcess.overload('int').implementation = function(pid) {
            console.log('Shit happens killprocess',pid);
            return;
        }

        var system = Java.use('java.lang.System');
        system.exit.overload('int').implementation = function (pid) {
            console.log('Exit called!');
            send("system.exit")
        }

        var runtime = Java.use('java.lang.Runtime');
        runtime.halt.overload('int').implementation = function(var_0) {
            console.log('halt called');
            send('halt called');
        }
    })
})
```