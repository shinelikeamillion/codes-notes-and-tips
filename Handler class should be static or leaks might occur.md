Repost from: http://blog.csdn.net/wuleihenbang/article/details/17126371

This Handler class should be static or leaks might occur
Handler类应该定义成静态类，否则可能导致内存泄露。

Issue: Ensures that Handler classes do not hold on to a reference to an outer class

In Android, Handler classes should be static or leaks might occur. Messages enqueued on the application thread's MessageQueue also retain their target Handler. If the Handler is an inner class, its outer class will be retained as well. To avoid leaking the outer class, declare the Handler as a static nested class with a WeakReference to its outer class.

大体翻译如下:
Handler 类应该应该为static类型，否则有可能造成泄露。
在程序消息队列中排队的消息保持了对目标Handler类的应用。如果Handler是个内部类，那 么它也会保持它所在的外部类的引用。为了避免泄露这个外部类，应该将Handler声明为static嵌套类，并且使用对外部类的弱应用。

To deal with it：

```
static class MyHandler extends Handler {  
                WeakReference<PopupActivity> mActivity;  

                MyHandler(PopupActivity activity) {  
                        mActivity = new WeakReference<PopupActivity>(activity);  
                }  

                @Override  
                public void handleMessage(Message msg) {  
                        PopupActivity theActivity = mActivity.get();  
                        switch (msg.what) {  
                        case 0:  
                                theActivity.popPlay.setChecked(true);  
                                break;  
                        }  
                }  
 }  
 ```
