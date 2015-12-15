```
private Bitmap compress (Bitmap bm) {
		
	//获取这个图片的宽和高
	float bWidth = bm.getWidth();
	float bHeight = bm.getHeight();
	
	//创建操作图片用的matrix对象
	Matrix matrix = new Matrix();
	
	float scale;
	
	double maxSize = 100.00;// 设定图片压缩的最大值为100kb
	ByteArrayOutputStream baos = new ByteArrayOutputStream();
	bm.compress(CompressFormat.PNG, 100, baos);
	byte[] b = baos.toByteArray();
	//将字节转换成KB
	double mid = b.length / 1024;
	if (mid > maxSize) {
		double i = mid / maxSize;
		scale =(float)( bm.getWidth() / Math.sqrt(i) ) / bWidth;
		//缩放图片的动作
		matrix.postScale(scale, scale);
		//计算宽高缩放率
		bm = Bitmap.createBitmap(bm, 0, 0, (int)bWidth, (int)bHeight, matrix, true);
	}
	
	return bm;
}
```