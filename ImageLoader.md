```

/**
 * 图片加载类
 * @author 10000_hours
 * 单例模式的两次判
 * 	1.第一次（没有作同步的处理）是判断对象有没有进行过实例化，可以过滤大部分代码
 * 	2.第二次（没有用synchronized 做同步的处理）是针对对象没有进行过实例化的情况，如果多个线程同时到达if体里面，这样需要做同步的线程会相对少一些
 *  相对直接用synchronized同步getInstance方法的好处是会提高效率（因为不用每一次都同步处理）
 */
public class ImageLoader {
	
	private static ImageLoader mInstance;
	
	/**
	 * 图片缓存的核心对象(管理所有图片所占用的内存)
	 */  
	private LruCache<String, Bitmap> mLruCache;
	
	/**
	 * 线程池
	 */
	private ExecutorService mThreadPool;
	private static final int DEFAULT_THREAD_COUNT = 1;
	
	/**
	 * 队列的调度方式
	 */
	private Type mType = Type.LIFO;
	
	/**
	 * 任务队列
	 */
	private LinkedList<Runnable> mTaskQueue;
	
	/**
	 * 后台轮询线程
	 */
	private Thread mPoolThread;
	private Handler mPoolThreadHandler;
	
	/**
	 * UI 线程中的Handler
	 */
	private Handler mUIHandler;
	
	
	/**
	 * java 中并发的类信号量进行同步
	 */
	private Semaphore mSemaphorePoolThreadHandler = new Semaphore(0);
	
	/**
	 * 利用信号量来控制任务队列
	 */
	private Semaphore mSemaphoreThreadPool;
	
	/**
	 * 图片先进先出或者后进先出的枚举标识
	 */
	public enum Type {
		FIFO, LIFO
	}
	
	// 把构造方法私有化，使外界没有办法通过new来构造
	private ImageLoader (int threadCount, Type type) {
		init(threadCount, type);
	}
	
	/**
	 * 出事化操作
	 * @param threadCount
	 * @param type
	 */
	private void init(int threadCount, Type type) {
		//后台轮询线程
		mPoolThread = new Thread(){
			@Override
			public void run() {
				Looper.prepare();
				mPoolThreadHandler = new Handler() {
					@Override
					public void handleMessage(Message msg) {
						//线程池取出一个任务去执行
						mThreadPool.execute(getTask());
						
						try {
							mSemaphoreThreadPool.acquire();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				};
				// 释放一个信号量
				mSemaphorePoolThreadHandler.release();
				Looper.loop();
			};
		};
		mPoolThread.start();
		
		// TODO 利用信号量控制队列-->反射获取宽高最大值
		
		// 获取应用的最大可使用的内存
		int maxMemory = (int) Runtime.getRuntime().maxMemory();
		int cacheMemmory = maxMemory / 8;
		
		mLruCache = new LruCache<String, Bitmap>(cacheMemmory) {
			@Override
			protected int sizeOf(String key, Bitmap value) {
				// 测量每一张图片所占据的内存（每行占据字节数×高度）
				return value.getRowBytes() * value.getHeight();
			}
		};
		
		// 创建线程池
		mThreadPool = Executors.newFixedThreadPool(threadCount);
		mTaskQueue = new LinkedList<Runnable>();
		mType = type;
		
		//跟据线程数初始化任务队列
		mSemaphoreThreadPool = new Semaphore(threadCount);
	}
	
	/**
	 * 从任务队列取出一个方法
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
	 * 根据path为imageView 设置图片
	 * @param path
	 * @param imageView
	 */
	public void loadImage (final String path, final ImageView imageView) {
		imageView.setTag(path);
		
		if (null == mUIHandler) {
			mUIHandler = new Handler() {
				@Override
				public void handleMessage(Message msg) {
					//获取得到的图片，为imageView回调设置图片
					ImgBeanHolder holder = (ImgBeanHolder) msg.obj;
					Bitmap bm = holder.bitmap;
					ImageView imgView = holder.imageView;
					String path = holder.path;
					
					// 将path与getTag存储路径进行比较
					if (imgView.getTag().toString().equals(path)) {
						imgView.setImageBitmap(bm);
					}
				}
			};
		}
		
		// 根据path在缓存中获取bitmap
		Bitmap bm = getBitmapFromLruCache(path);
		
		if (bm != null) {
			refreshBitmap(path, imageView, bm);
		} else {
			addTask(new Runnable() {
				
				@Override
				public void run() {
					// 加载图片
					// 图片的压缩
					// 1.获得图片需要显示的大小
					ImageSize imageVeiwSize = getImageVeiwSize(imageView);
					// 2.压缩图片
					Bitmap bm = decodeSampleBitmapFromPath(path , imageVeiwSize.Width, imageVeiwSize.Height);
					// 3.把图片加入到缓存
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
	 * 将图片加入LruCache(缓存)
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
	 * 根据图片需要显示的宽和高，对图片进行压缩
	 * @param path
	 * @param width
	 * @param height
	 * @return
	 */
	protected Bitmap decodeSampleBitmapFromPath(String path, int width,
			int height) {
		
		// 获取图片的宽和高，但是并不把图片加载到内存当中
		BitmapFactory.Options options = new Options();
		options.inJustDecodeBounds = true;//不用讲图片加载到内存
		BitmapFactory.decodeFile(path, options);
		
		options.inSampleSize = caculateInSample(options, width, height);
		
		// 使用获得到的InSampleSize再次解析图片
		options.inJustDecodeBounds = false;
		Bitmap bitmap = BitmapFactory.decodeFile(path, options);
		
		return bitmap;
	}
	/**
	 * 根据需求的宽和高以及图片实际的宽和高计算SampleSize
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
	
	// 根据imageView获取适当的压缩的宽和高
	@SuppressLint("NewApi")
	private ImageSize getImageVeiwSize(ImageView imageView) {
		ImageSize imageSize = new ImageSize();
		LayoutParams layoutParams = imageView.getLayoutParams();
		
		DisplayMetrics displayMetrics = imageView.getContext().getResources().getDisplayMetrics();
		
		int width = imageView.getWidth();//获取imageView 的实际宽度
		if (width <= 0) {
			width = layoutParams.width;//获取imageView在layout中声明的宽度
		}
		if (width <= 0) {
			width = imageView.getMaxWidth();//检查最大值
		}
		if (width <= 0) {
			width = displayMetrics.widthPixels;
		}
		
		int height = imageView.getHeight();//获取imageView 的实际宽度
		if (height <= 0) {
			height = layoutParams.height;//获取imageView在layout中声明的宽度
		}
		if (height <= 0) {
			height = imageView.getMaxHeight();//检查最大值
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
			// 因为默认是0，所以请求会在此阻塞线程 :)
			if (mSemaphorePoolThreadHandler == null) {
				mSemaphorePoolThreadHandler.acquire();
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		mPoolThreadHandler.sendEmptyMessage(0x110);
	}

	/**
	 * 根据path在缓存中获取bitmap
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
