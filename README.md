# TextStickerViewDemo
可以拖动，旋转，缩放，删除的文本编辑控件，再次点击可以进一步编辑，选择文本颜色
#单击floatButton添加文字
![image](https://github.com/wujie1314520/TextStickerViewDemo/raw/master/screenshot/screenshot1.jpg)
#旋转，缩放，拖动文字
![image](https://github.com/wujie1314520/TextStickerViewDemo/raw/master/screenshot/screenshot2.jpg)
![image](https://github.com/wujie1314520/TextStickerViewDemo/raw/master/screenshot/screenshot3.jpg)
![image](https://github.com/wujie1314520/TextStickerViewDemo/raw/master/screenshot/screenshot4.jpg)
#改变字体颜色
![image](https://github.com/wujie1314520/TextStickerViewDemo/raw/master/screenshot/screenshot5.jpg)
![image](https://github.com/wujie1314520/TextStickerViewDemo/raw/master/screenshot/screenshot6.jpg)
#apk下载地址
https://github.com/wujie1314520/TextStickerViewDemo/raw/master/app-debug.apk

# 840631861个人修改功能
#自定义颜色框，原代码缩放是按照开始比例缩放，修改为按照手势滑动缩放
```

/**
 * 可旋转、随手指缩放（不按比例） 颜色框贴图处理控件
 */
public class BoxStickerView extends View {
    public final int TEXT_SIZE_DEFAULT = getResources().getDimensionPixelSize(R.dimen.fontsize_default);
//    public final int PADDING = getResources().getDimensionPixelSize(R.dimen.font_padding);
    public final int PADDING = 32;
//    public final int STICKER_BTN_HALF_SIZE = getResources().getDimensionPixelSize(R.dimen.sticker_btn_half_size);
    public final int STICKER_BTN_HALF_SIZE = 30;
    private String mText = "  "; //给贴图文本赋个初始值
    private TextPaint mPaint = new TextPaint();
    private Paint debugPaint = new Paint();
    private Paint mHelpPaint = new Paint();

    private RectF mHelpBoxRect = new RectF();
    private Rect mDeleteRect = new Rect();//删除按钮位置
    private Rect mRotateRect = new Rect();//旋转按钮位置

    private RectF mDeleteDstRect = new RectF();
    private RectF mRotateDstRect = new RectF();

    private Bitmap mDeleteBitmap;
    private Bitmap mRotateBitmap;

    private int mCurrentMode = IDLE_MODE;
    //控件的几种模式
    private static final int IDLE_MODE = 2;//正常
    private static final int MOVE_MODE = 3;//移动模式
    private static final int ROTATE_MODE = 4;//旋转模式
    private static final int DELETE_MODE = 5;//删除模式

    private EditText mEditText;//输入控件

    public int layout_x = 0;
    public int layout_y = 0;

    private float last_x = 0;
    private float last_y = 0;

    public float mRotateAngle = 0;
    public float mScaleW = 0;
    public float mScaleH = 0;
    private boolean isInitLayout = true;

    private boolean isShowHelpBtn = true;
    private float dx;
    private float dy;
    private long currentTime;

    //设置字体
    private Typeface type;

    //设置画笔的透明度
    private int mAlpha = 255;

    public BoxStickerView(Context context, Typeface type) {
        super(context);
        initView(context, type);
    }

    public BoxStickerView(Context context, AttributeSet attrs, Typeface type) {
        super(context, attrs);
        initView(context, type);
    }

    public BoxStickerView(Context context, AttributeSet attrs, int defStyleAttr, Typeface type) {
        super(context, attrs, defStyleAttr);
        initView(context, type);
    }

    public void setEditText(EditText textView) {
        this.mEditText = textView;

//        //设置特殊字体
//        mEditText.setTypeface(type);
    }

    private void initView(Context context, Typeface type) {
        debugPaint.setColor(Color.parseColor("#66ff0000"));

        mDeleteBitmap = BitmapFactory.decodeResource(context.getResources(),
                R.mipmap.text_delete);
        mRotateBitmap = BitmapFactory.decodeResource(context.getResources(),
                R.mipmap.text_edit);

        mDeleteRect.set(0, 0, mDeleteBitmap.getWidth(), mDeleteBitmap.getHeight());
        mRotateRect.set(0, 0, mRotateBitmap.getWidth(), mRotateBitmap.getHeight());

        mDeleteDstRect = new RectF(0, 0, STICKER_BTN_HALF_SIZE << 1, STICKER_BTN_HALF_SIZE << 1);
        mRotateDstRect = new RectF(0, 0, STICKER_BTN_HALF_SIZE << 1, STICKER_BTN_HALF_SIZE << 1);

        mPaint.setColor(Color.RED);

        //设置字体
        if (null != type) {
            mPaint.setTypeface(type);
        }

        mPaint.setTextAlign(Paint.Align.CENTER);
        mPaint.setTextSize(TEXT_SIZE_DEFAULT);
        mPaint.setAntiAlias(true);

        //设置画笔的透明度
        mPaint.setAlpha(mAlpha);

        mHelpPaint.setColor(Color.RED);
        mHelpPaint.setStyle(Paint.Style.STROKE);
        mHelpPaint.setAntiAlias(true);
        mHelpPaint.setStrokeWidth(4);

    }

    public void setText(String text) {
        this.mText = text;
        invalidate();
    }

    public String getmText() {
        return mText;
    }

    public void setTextColor(int newColor) {
        mPaint.setColor(newColor);
        invalidate();
    }

    public Typeface getType() {
        return type;
    }

    public void setType(Typeface type) {
        this.type = type;
        mPaint.setTypeface(type);
        invalidate();
    }

    public int getmAlpha() {
        return mAlpha;
    }

    public void setmAlpha(int mAlpha) {
        this.mAlpha = mAlpha;
        mPaint.setAlpha(mAlpha);
        invalidate();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        if (isInitLayout) {
            isInitLayout = false;
            resetView();
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        if (TextUtils.isEmpty(mText))
            return;

        drawContent(canvas);
    }

    private void drawContent(Canvas canvas) {
        setBoxSize(canvas);

        //draw x and rotate button
        int offsetValue = ((int) mDeleteDstRect.width()) >> 1;
        mDeleteDstRect.offsetTo(mHelpBoxRect.left - offsetValue, mHelpBoxRect.top - offsetValue);
        mRotateDstRect.offsetTo(mHelpBoxRect.right - offsetValue, mHelpBoxRect.bottom - offsetValue);

        RectUtil.rotateRect(mDeleteDstRect, mHelpBoxRect.centerX(),
                mHelpBoxRect.centerY(), mRotateAngle);
        RectUtil.rotateRect(mRotateDstRect, mHelpBoxRect.centerX(),
                mHelpBoxRect.centerY(), mRotateAngle);

        canvas.save();
        canvas.rotate(mRotateAngle, mHelpBoxRect.centerX(), mHelpBoxRect.centerY());
        canvas.drawRoundRect(mHelpBoxRect, 10, 10, mHelpPaint);
        canvas.restore();

        if (!isShowHelpBtn) {
            return;
        }

        canvas.drawBitmap(mDeleteBitmap, mDeleteRect, mDeleteDstRect, null);
        canvas.drawBitmap(mRotateBitmap, mRotateRect, mRotateDstRect, null);
        //canvas.drawRect(mRotateDstRect, debugPaint);
        //canvas.drawRect(mDeleteDstRect, debugPaint);
    }

    private void setBoxSize(Canvas canvas) {
        setBoxSize(canvas, layout_x, layout_y, mScaleW, mScaleH, mRotateAngle);
    }

    public void setBoxSize(Canvas canvas, int _x, int _y, float scaleW, float scaleH, float rotate) {
        if (TextUtils.isEmpty(mText)) {
            return;
        }

        int x = _x;
        int y = _y;

        mHelpBoxRect.set(x,y,x+120,y+60);
        RectUtil.scaleRect(mHelpBoxRect, scaleW,scaleH);//自定义缩放

        canvas.save();
        //canvas.scale(scale, scale, mHelpBoxRect.centerX(), mHelpBoxRect.centerY());
        canvas.rotate(rotate, mHelpBoxRect.centerX(), mHelpBoxRect.centerY());
        canvas.restore();
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        boolean ret = super.onTouchEvent(event);// 是否向下传递事件标志 true为消耗

        int action = event.getAction();
        float x = event.getX();
        float y = event.getY();

        switch (action) {
            case MotionEvent.ACTION_DOWN:
                if (mDeleteDstRect.contains(x, y)) {// 删除模式
                    //修改此处，改为在不显示删除按钮的情况下点击框的左上角不会删除框（原来是不现实的情况下点击左上角也会删除）
                    if(isShowHelpBtn)
                        mCurrentMode = DELETE_MODE;
                } else if (mRotateDstRect.contains(x, y)) {// 旋转按钮
                    isShowHelpBtn = true;
                    mCurrentMode = ROTATE_MODE;
                    last_x = mRotateDstRect.centerX();
                    last_y = mRotateDstRect.centerY();
                    ret = true;
                } else if (mHelpBoxRect.contains(x, y)) {// 移动模式 //或者编辑模式
                    isShowHelpBtn = true;
                    mCurrentMode = MOVE_MODE;
                    last_x = x;
                    last_y = y;
                    ret = true;

                    currentTime = System.currentTimeMillis();

                } else {
                    isShowHelpBtn = false;
                    invalidate();
                }// end if

                if (mCurrentMode == DELETE_MODE) {// 删除选定贴图
                    mCurrentMode = IDLE_MODE;// 返回空闲状态
                    clearTextContent();
                    invalidate();
                }// end if
                break;
            case MotionEvent.ACTION_MOVE:
                ret = true;
                if (mCurrentMode == MOVE_MODE) {// 移动贴图
                    mCurrentMode = MOVE_MODE;
                    dx = x - last_x;
                    dy = y - last_y;

                    layout_x += dx;
                    layout_y += dy;

                    invalidate();

                    last_x = x;
                    last_y = y;
                } else if (mCurrentMode == ROTATE_MODE) {// 旋转 缩放文字操作
                    mCurrentMode = ROTATE_MODE;
                    dx = x - last_x;
                    dy = y - last_y;

                    updateRotateAndScale(dx, dy);

                    invalidate();
                    last_x = x;
                    last_y = y;
                }
                break;
            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                //判断是否单击编辑框
                long moveTime = System.currentTimeMillis() - currentTime;//移动时间

                //判断是否继续传递信号
                if (moveTime <= 200 && (dx <= 20 || dy <= 20)) {
                    //点击事件，自己消费
//                    Toast.makeText(getContext(), "aaaaa",Toast.LENGTH_SHORT).show();
//                    isShowHelpBox = false;
//                    invalidate();

                    if (mOnItemClickListener != null) {
                        mOnItemClickListener.onEditClick(this);
                    }

                    return true;
                }
                ret = false;
                mCurrentMode = IDLE_MODE;
                break;
        }// end switch

        return ret;
    }

    public void clearTextContent() {
        if (mEditText != null) {
            mEditText.setText(null);
        }
        //setText(null);
        mText = null;
    }


    /**
     * 旋转 缩放 更新
     *
     * @param dx
     * @param dy
     */
    public void updateRotateAndScale(final float dx, final float dy) {
        float c_x = mHelpBoxRect.centerX();
        float c_y = mHelpBoxRect.centerY();

        float x = mRotateDstRect.centerX();
        float y = mRotateDstRect.centerY();

        float n_x = x + dx;
        float n_y = y + dy;

        float xa = x - c_x;
        float ya = y - c_y;

        float xb = n_x - c_x;
        float yb = n_y - c_y;

        float srcLen = (float) Math.sqrt(xa * xa + ya * ya);
        float curLen = (float) Math.sqrt(xb * xb + yb * yb);

        float scale = curLen / srcLen;// 计算缩放比

        mScaleW += dx;
        mScaleH += dy;
        float newWidth = mHelpBoxRect.width() + mScaleW;

        double cos = (xa * xb + ya * yb) / (srcLen * curLen);
        if (cos > 1 || cos < -1)
            return;
        float angle = (float) Math.toDegrees(Math.acos(cos));
        float calMatrix = xa * yb - xb * ya;// 行列式计算 确定转动方向

        int flag = calMatrix > 0 ? 1 : -1;
        angle = flag * angle;

        mRotateAngle += angle;
    }

    public void resetView() {
        layout_x = getMeasuredWidth() / 2;
        layout_y = getMeasuredHeight() / 2;
        mRotateAngle = 0;
        mScaleW = 0;
        mScaleH = 0;
    }

    public boolean isShowHelpBox() {
        return isShowHelpBtn;
    }

    public void setShowHelpBox(boolean showHelpBox) {
        isShowHelpBtn = showHelpBox;
        invalidate();
    }

    public float getRotateAngle() {
        return mRotateAngle;
    }

    /**
     * 向外部提供监听事件
     */
    private OnEditClickListener mOnItemClickListener;

    public void setOnEditClickListener(OnEditClickListener listener) {
        this.mOnItemClickListener = listener;
    }

    public interface OnEditClickListener {
        void onEditClick(View v);
    }
}//end class
```
#RectUtil 增加缩放指定矩形
```
public static void scaleRect(RectF rect, float scaleW, float scaleY) {
        float w = rect.width();
        float h = rect.height();

        float newW = w + scaleW;
        float newH = h + scaleY;

        if( newW < 50 )
            newW = 50;
        if( newH < 50 )
            newH = 50;

        float dx = (newW - w) / 2;
        float dy = (newH - h) / 2;

        rect.left -= dx;
        rect.top -= dy;
        rect.right += dx;
        rect.bottom += dy;
    }
```

#注意地方
```
if (mDeleteDstRect.contains(x, y)) {// 删除模式
    //修改此处，改为在不显示删除按钮的情况下点击框的左上角不会删除框（原来是不现实的情况下点击左上角也会删除）
    if(isShowHelpBtn)
        mCurrentMode = DELETE_MODE;
}
```
```
RectUtil.scaleRect(mHelpBoxRect, scaleW,scaleH);//自定义缩放
```
修改后的代码 https://download.csdn.net/download/qq_26075861/10768081
