# Activity 启动流程

1. `startActivity`的内部会通过`Instrumentation`来尝试启动`Activity`，这是一个跨进程的过程。
2. 它会调用`AMS`的`startActivity`方法，在`AMS`校验`Activity`的合法性，判断栈顶是否有`Activity`，决定应该冷启动还是热启动
3. 以上准备工作完成后会通过`ApplicationThread`回调到`app`所在的进程，这又是一次跨进程调用，`ApplicationThread`其实就是一个`Binder`，`Binder`中的逻辑是在工作线程中执行，因此`Binder`内部调用`Handler`发送消息到UI线程
4. 如果是冷启动，首先会通过`Zygote`进程`fork`出一个新的进程，然后反射执行`ActivityThread`的`main`方法对其进行初始化
5. 如果是热启动，则`pause`栈顶`Activity`，再调用`handleLanchActivity`创建和启动目标`Activity`，接着执行`onCreate`和`onResume` ，这样`Activity`就显示到界面上。
6. 上述流程都执行完毕后，会去执行栈顶`Activity`的`onStop`