```
private Bitmap compress (Bitmap bm) {
		
	//��ȡ���ͼƬ�Ŀ�͸�
	float bWidth = bm.getWidth();
	float bHeight = bm.getHeight();
	
	//��������ͼƬ�õ�matrix����
	Matrix matrix = new Matrix();
	
	float scale;
	
	double maxSize = 100.00;// �趨ͼƬѹ�������ֵΪ100kb
	ByteArrayOutputStream baos = new ByteArrayOutputStream();
	bm.compress(CompressFormat.PNG, 100, baos);
	byte[] b = baos.toByteArray();
	//���ֽ�ת����KB
	double mid = b.length / 1024;
	if (mid > maxSize) {
		double i = mid / maxSize;
		scale =(float)( bm.getWidth() / Math.sqrt(i) ) / bWidth;
		//����ͼƬ�Ķ���
		matrix.postScale(scale, scale);
		//������������
		bm = Bitmap.createBitmap(bm, 0, 0, (int)bWidth, (int)bHeight, matrix, true);
	}
	
	return bm;
}
```