```

/**
 * ͼƬ������
 * @author 10000_hours
 * ����ģʽ��������
 * 	1.��һ�Σ�û����ͬ���Ĵ������ж϶�����û�н��й�ʵ���������Թ��˴󲿷ִ���
 * 	2.�ڶ��Σ�û����synchronized ��ͬ���Ĵ�������Զ���û�н��й�ʵ������������������߳�ͬʱ����if�����棬������Ҫ��ͬ�����̻߳������һЩ
 *  ���ֱ����synchronizedͬ��getInstance�����ĺô��ǻ����Ч�ʣ���Ϊ����ÿһ�ζ�ͬ������
 */
public class ImageLoader {
	
	private static ImageLoader mInstance;
	
	/**
	 * ͼƬ����ĺ��Ķ���(��������ͼƬ��ռ�õ��ڴ�)
	 */  
	private LruCache<String, Bitmap> mLruCache;
	
	/**
	 * �̳߳�
	 */
	private ExecutorService mThreadPool;
	private static final int DEFAULT_THREAD_COUNT = 1;
	
	/**
	 * ���еĵ��ȷ�ʽ
	 */
	private Type mType = Type.LIFO;
	
	/**
	 * �������
	 */
	private LinkedList<Runnable> mTaskQueue;
	
	/**
	 * ��̨��ѯ�߳�
	 */
	private Thread mPoolThread;
	private Handler mPoolThreadHandler;
	
	/**
	 * UI �߳��е�Handler
	 */
	private Handler mUIHandler;
	
	
	/**
	 * java �в��������ź�������ͬ��
	 */
	private Semaphore mSemaphorePoolThreadHandler = new Semaphore(0);
	
	/**
	 * �����ź����������������
	 */
	private Semaphore mSemaphoreThreadPool;
	
	/**
	 * ͼƬ�Ƚ��ȳ����ߺ���ȳ���ö�ٱ�ʶ
	 */
	public enum Type {
		FIFO, LIFO
	}
	
	// �ѹ��췽��˽�л���ʹ���û�а취ͨ��new������
	private ImageLoader (int threadCount, Type type) {
		init(threadCount, type);
	}
	
	/**
	 * ���»�����
	 * @param threadCount
	 * @param type
	 */
	private void init(int threadCount, Type type) {
		//��̨��ѯ�߳�
		mPoolThread = new Thread(){
			@Override
			public void run() {
				Looper.prepare();
				mPoolThreadHandler = new Handler() {
					@Override
					public void handleMessage(Message msg) {
						//�̳߳�ȡ��һ������ȥִ��
						mThreadPool.execute(getTask());
						
						try {
							mSemaphoreThreadPool.acquire();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				};
				// �ͷ�һ���ź���
				mSemaphorePoolThreadHandler.release();
				Looper.loop();
			};
		};
		mPoolThread.start();
		
		// TODO �����ź������ƶ���-->�����ȡ������ֵ
		
		// ��ȡӦ�õ�����ʹ�õ��ڴ�
		int maxMemory = (int) Runtime.getRuntime().maxMemory();
		int cacheMemmory = maxMemory / 8;
		
		mLruCache = new LruCache<String, Bitmap>(cacheMemmory) {
			@Override
			protected int sizeOf(String key, Bitmap value) {
				// ����ÿһ��ͼƬ��ռ�ݵ��ڴ棨ÿ��ռ���ֽ������߶ȣ�
				return value.getRowBytes() * value.getHeight();
			}
		};
		
		// �����̳߳�
		mThreadPool = Executors.newFixedThreadPool(threadCount);
		mTaskQueue = new LinkedList<Runnable>();
		mType = type;
		
		//�����߳�����ʼ���������
		mSemaphoreThreadPool = new Semaphore(threadCount);
	}
	
	/**
	 * ���������ȡ��һ������
	 * @return
	 */
	private Runnable getTask() {
		if (Type.FIFO == mType) {
			return mTaskQueue.removeFirst();
		} else if (Type.LIFO == mType) {
			return mTaskQueue.removeLast();
		}
		return null;
	}

	private static ImageLoader getInstance () {
		
		if (null == mInstance) {
			
			synchronized (ImageLoader.class) {
				if (null == mInstance) {
					mInstance = new ImageLoader(DEFAULT_THREAD_COUNT, Type.LIFO);
				}
			}
		}
		
		return mInstance;
	}
	/**
	 * ����pathΪimageView ����ͼƬ
	 * @param path
	 * @param imageView
	 */
	public void loadImage (final String path, final ImageView imageView) {
		imageView.setTag(path);
		
		if (null == mUIHandler) {
			mUIHandler = new Handler() {
				@Override
				public void handleMessage(Message msg) {
					//��ȡ�õ���ͼƬ��ΪimageView�ص�����ͼƬ
					ImgBeanHolder holder = (ImgBeanHolder) msg.obj;
					Bitmap bm = holder.bitmap;
					ImageView imgView = holder.imageView;
					String path = holder.path;
					
					// ��path��getTag�洢·�����бȽ�
					if (imgView.getTag().toString().equals(path)) {
						imgView.setImageBitmap(bm);
					}
				}
			};
		}
		
		// ����path�ڻ����л�ȡbitmap
		Bitmap bm = getBitmapFromLruCache(path);
		
		if (bm != null) {
			refreshBitmap(path, imageView, bm);
		} else {
			addTask(new Runnable() {
				
				@Override
				public void run() {
					// ����ͼƬ
					// ͼƬ��ѹ��
					// 1.���ͼƬ��Ҫ��ʾ�Ĵ�С
					ImageSize imageVeiwSize = getImageVeiwSize(imageView);
					// 2.ѹ��ͼƬ
					Bitmap bm = decodeSampleBitmapFromPath(path , imageVeiwSize.Width, imageVeiwSize.Height);
					// 3.��ͼƬ���뵽����
					addBitmapToLruCache(path, bm);
					
					refreshBitmap(path, imageView, bm);
					
					mSemaphoreThreadPool.release();
				}
			});
		}
	}
	
	
	private void refreshBitmap(final String path, final ImageView imageView,
			Bitmap bm) {
		Message message = Message.obtain();
		ImgBeanHolder holder = new ImgBeanHolder();
		holder.bitmap = bm;
		holder.path = path;
		holder.imageView = imageView;
		message.obj = holder;
		mUIHandler.sendMessage(message);
	}
	
	/**
	 * ��ͼƬ����LruCache(����)
	 * @param path
	 * @param bm
	 */
	protected void addBitmapToLruCache(String path, Bitmap bm) {
		if (getBitmapFromLruCache(path) == null) {
			if (bm != null) {
				mLruCache.put(path, bm);
			}
		}
	}

	/**
	 * ����ͼƬ��Ҫ��ʾ�Ŀ�͸ߣ���ͼƬ����ѹ��
	 * @param path
	 * @param width
	 * @param height
	 * @return
	 */
	protected Bitmap decodeSampleBitmapFromPath(String path, int width,
			int height) {
		
		// ��ȡͼƬ�Ŀ�͸ߣ����ǲ�����ͼƬ���ص��ڴ浱��
		BitmapFactory.Options options = new Options();
		options.inJustDecodeBounds = true;//���ý�ͼƬ���ص��ڴ�
		BitmapFactory.decodeFile(path, options);
		
		options.inSampleSize = caculateInSample(options, width, height);
		
		// ʹ�û�õ���InSampleSize�ٴν���ͼƬ
		options.inJustDecodeBounds = false;
		Bitmap bitmap = BitmapFactory.decodeFile(path, options);
		
		return bitmap;
	}
	/**
	 * ��������Ŀ�͸��Լ�ͼƬʵ�ʵĿ�͸߼���SampleSize
	 * @param options
	 * @param width
	 * @param height
	 * @return
	 */
	private int caculateInSample(Options options, int reqWidth, int reqHeight) {
		int width = options.outWidth;
		int height = options.outHeight;
		
		int inSampleSize = 1;
		
		if (width > reqWidth || height > reqHeight) {
			int widthRadio = Math.round(width*1.0f / reqWidth);
			int heightRadio = Math.round(height*1.0f / reqHeight);
			
			inSampleSize = Math.max(widthRadio, heightRadio);
		}
		
		return inSampleSize;
	}

	private class ImageSize {
		int Width;
		int Height;
	}
	
	// ����imageView��ȡ�ʵ���ѹ���Ŀ�͸�
	@SuppressLint("NewApi")
	private ImageSize getImageVeiwSize(ImageView imageView) {
		ImageSize imageSize = new ImageSize();
		LayoutParams layoutParams = imageView.getLayoutParams();
		
		DisplayMetrics displayMetrics = imageView.getContext().getResources().getDisplayMetrics();
		
		int width = imageView.getWidth();//��ȡimageView ��ʵ�ʿ��
		if (width <= 0) {
			width = layoutParams.width;//��ȡimageView��layout�������Ŀ��
		}
		if (width <= 0) {
			width = imageView.getMaxWidth();//������ֵ
		}
		if (width <= 0) {
			width = displayMetrics.widthPixels;
		}
		
		int height = imageView.getHeight();//��ȡimageView ��ʵ�ʿ��
		if (height <= 0) {
			height = layoutParams.height;//��ȡimageView��layout�������Ŀ��
		}
		if (height <= 0) {
			height = imageView.getMaxHeight();//������ֵ
		}
		if (height <= 0) {
			height = displayMetrics.heightPixels;
		}
		
		imageSize.Width = width;
		imageSize.Height = height;
		
		return null;
	}
	
	
	private synchronized void addTask(Runnable runnable) {
		mTaskQueue.add(runnable);
		
		// if (mPoolThreadHandler == null) wait(); 
		try {
			// ��ΪĬ����0������������ڴ������߳� :)
			if (mSemaphorePoolThreadHandler == null) {
				mSemaphorePoolThreadHandler.acquire();
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		mPoolThreadHandler.sendEmptyMessage(0x110);
	}

	/**
	 * ����path�ڻ����л�ȡbitmap
	 * @param key
	 * @return
	 */
	private Bitmap getBitmapFromLruCache(String key) {
		return mLruCache.get(key);
	}

	private class ImgBeanHolder {
		Bitmap bitmap;
		ImageView imageView;
		String path;
	}

}
```
