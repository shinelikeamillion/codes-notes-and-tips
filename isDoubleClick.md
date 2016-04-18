```java
// 检查双击 & 防止用户连续点击
	public static long lastTime;
public boolean isDouble () {
	boolean flag = false;
	long time = System.currentTimeMillis() - lastTime;
	
	if (time < 300) {
		flag = true;
	}
	
	lastTime = System.currentTimeMills();
	return flag;
} 
```