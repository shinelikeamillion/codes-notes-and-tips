
  Android判断网络连接的方法
```
//要获取网络状态的权限<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/> 
ConnectivityManager cm=(ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);  
NetworkInfo info=cm.getActiveNetworkInfo();  
 if(info!=null){  
    Toast.makeText(MainActivity.this, "连网正常"+info.getTypeName(), Toast.LENGTH_SHORT).show();  
}else{  
    Toast.makeText(MainActivity.this, "未连网", Toast.LENGTH_SHORT).show();  
}
```
