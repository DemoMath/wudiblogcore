---
layout: post
title:  "Android头部视差效果的实现方式"
date:   2016-07-13 
categories: [Android,UI]
tags: [Android]
---
实现了类似qq空间头部的图片弹性效果，手指向下滑动头部图片展示出更多部分 
废话不说，上代码： 
自定义的ParallaxListView
	
	public class ParallaxListView extends ListView {

	    private ImageView parallaxImageView;
	    private int maxHeight;
	    private int originalHeight;

	    public ParallaxListView(Context context) {
	        this(context, null);
	    }
	
	    public ParallaxListView(Context context, AttributeSet attrs) {
	        this(context, attrs, 0);
	    }
	
	    public ParallaxListView(Context context, AttributeSet attrs, int defStyle) {
	        super(context, attrs, defStyle);
	    }
	
	    /**
	     * 对外提供一个传入图片的方式
	     * 
	     * @param parallaxImageView
	     */
	    public void setParallaxImageView(final ImageView parallaxImageView) {
	        this.parallaxImageView = parallaxImageView;
	        // 设置最大高度为图片的真实高度
	        maxHeight = parallaxImageView.getDrawable().getIntrinsicHeight();
	        // 利用视图树获取最初的高度
	        parallaxImageView.getViewTreeObserver().addOnGlobalLayoutListener(
	                new OnGlobalLayoutListener() {
	
	                    /**
	                     * 该方法在完成布局的时候调用
	                     */
	                    @Override
	                    public void onGlobalLayout() {
	                        // 一般用完立即移除，因为只要有宽高变化，就会重新布局，会引起onGlobalLayout重新调用
	                        parallaxImageView.getViewTreeObserver()
	                                .removeGlobalOnLayoutListener(this);
	                        originalHeight = parallaxImageView.getHeight();
	
	                    }
	                });
	
	    }
	
	    /**
	     * 重新overScrollBy方法 该方法是在listview滑动到头的时候调用，并且可以在该方法中获取滑动的距离 deltaY: 继续滑动的距离
	     * 正值：表示底部到头 负值：顶部到头
	     */
	    @SuppressLint("NewApi")
	    @Override
	    protected boolean overScrollBy(int deltaX, int deltaY, int scrollX,
	            int scrollY, int scrollRangeX, int scrollRangeY,
	            int maxOverScrollX, int maxOverScrollY, boolean isTouchEvent) {
	        // 如果是顶部到头，并且是手指拖动到头，才让ImageView的高度增加
	        if (deltaY < 0 && isTouchEvent) {
	            // 得到ImageView的新高度
	            int newHeight = parallaxImageView.getHeight() - deltaY / 3;
	            // 对newHeight高度进行限制
	            if (newHeight > maxHeight)
	                newHeight = maxHeight;
	            // 将新高度设置给imageview
	            android.view.ViewGroup.LayoutParams params = parallaxImageView
	                    .getLayoutParams();
	            params.height = newHeight;
	            parallaxImageView.setLayoutParams(params);
	
	        }
	
	        return super.overScrollBy(deltaX, deltaY, scrollX, scrollY,
	                scrollRangeX, scrollRangeY, maxOverScrollX, maxOverScrollY,
	                isTouchEvent);
	    }
	
	    @SuppressLint("NewApi")
	    @Override
	    public boolean onTouchEvent(MotionEvent ev) {
	        // 当手指抬起的时候执行一些逻辑
	        if (ev.getAction() == MotionEvent.ACTION_UP) {
	            // 让ImageView高度缓慢恢复到初始设置的120
	            final ValueAnimator animator = ValueAnimator.ofInt(
	                    parallaxImageView.getHeight(), originalHeight);
	            animator.addUpdateListener(new AnimatorUpdateListener() {
	
	                @Override
	                public void onAnimationUpdate(ValueAnimator animation) {
	                    // 获取动画当前的值
	                    int value = (Integer) animator.getAnimatedValue();
	                    android.view.ViewGroup.LayoutParams params = parallaxImageView
	                            .getLayoutParams();
	                    params.height = value;
	                    parallaxImageView.setLayoutParams(params);
	
	                }
	            });
	            animator.setInterpolator(new OvershootInterpolator());
	            animator.setDuration(400);
	
	            animator.start();
	
	        }
	
	        return super.onTouchEvent(ev);
	    }
	}

ParallaxAdapter：

	public class ParallaxAdapter extends BaseAdapter {

	    private Context context;
	
	    public ParallaxAdapter(Context context) {
	        this.context = context;
	    }
	
	    @Override
	    public int getCount() {
	        return 30;
	    }
	
	    @Override
	    public Object getItem(int position) {
	        return null;
	    }
	
	    @Override
	    public long getItemId(int position) {
	        return 0;
	    }
	
	    @Override
	    public View getView(int position, View convertView, ViewGroup parent) {
	        TextView tv = new TextView(context);
	        tv.setText("哈哈哈"+position);
	        tv.setTextSize(30);
	        tv.setTextColor(Color.BLACK);
	        return tv;
	    }
	}


Activity中使用：

	public class MainActivity extends Activity {
	
	    private ParallaxListView mPlv;
	    private View headerView;
	    private ImageView parallaxImageView;
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	
	        initView();
	        initData();
	    }
	    /**
	     * 添加数据
	     */
	    private void initData() {
	        mPlv.addHeaderView(headerView);
	        mPlv.setParallaxImageView(parallaxImageView);
	        mPlv.setAdapter(new ParallaxAdapter(getApplicationContext()));
	    }
	    /**
	     * 添加布局
	     */
	    @SuppressLint("NewApi") private void initView() {
	        mPlv = (ParallaxListView) findViewById(R.id.lv);
	        //去掉阴影
	        mPlv.setOverScrollMode(AbsListView.OVER_SCROLL_NEVER);
	        headerView = View.inflate(getApplicationContext(), R.layout.item_header, null);
	        parallaxImageView = (ImageView) headerView.findViewById(R.id.iv);
	    }
	
	}
