####com.yalantis.ucrop\view\OverlayView
```
package com.yalantis.ucrop.view;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.Path;
import android.graphics.RectF;
import android.graphics.Region;
import android.os.Build;
import android.support.annotation.ColorInt;
import android.support.annotation.IntDef;
import android.support.annotation.IntRange;
import android.support.annotation.NonNull;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;

import com.yalantis.ucrop.R;
import com.yalantis.ucrop.callback.OverlayViewChangeListener;
import com.yalantis.ucrop.util.RectUtils;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

**Created by Oleksii Shliama (https://github.com/shliama).**

** 이 뷰는 이미지 위에 오버레이를 그리는 데 사용된다.**
** 프레임, 자르기 가이드라인 및 음영 영역이 있을 수 있다.**
** 제대로 그리려면 LAYER_TYPE_SOFTWARE가 있어야 한다.**


	public class OverlayView extends View {

    public static final int FREESTYLE_CROP_MODE_DISABLE = 0;
    public static final int FREESTYLE_CROP_MODE_ENABLE = 1;
    public static final int FREESTYLE_CROP_MODE_ENABLE_WITH_PASS_THROUGH = 2;

    public static final boolean DEFAULT_SHOW_CROP_FRAME = true;
    public static final boolean DEFAULT_SHOW_CROP_GRID = true;
    public static final boolean DEFAULT_CIRCLE_DIMMED_LAYER = false;
    public static final int DEFAULT_FREESTYLE_CROP_MODE = FREESTYLE_CROP_MODE_DISABLE;
    public static final int DEFAULT_CROP_GRID_ROW_COUNT = 2;
    public static final int DEFAULT_CROP_GRID_COLUMN_COUNT = 2;

    private final RectF mCropViewRect = new RectF();
    private final RectF mTempRect = new RectF();

    protected int mThisWidth, mThisHeight;
    protected float[] mCropGridCorners;
    protected float[] mCropGridCenter;
    
    private int mCropGridRowCount, mCropGridColumnCount;
    private float mTargetAspectRatio;
    private float[] mGridPoints = null;
    private boolean mShowCropFrame, mShowCropGrid;
    private boolean mCircleDimmedLayer;
    private int mDimmedColor;
    private Path mCircularPath = new Path();
    private Paint mDimmedStrokePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    private Paint mCropGridPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    private Paint mCropFramePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    private Paint mCropFrameCornersPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    @FreestyleMode
    private int mFreestyleCropMode = DEFAULT_FREESTYLE_CROP_MODE;
    private float mPreviousTouchX = -1, mPreviousTouchY = -1;
    private int mCurrentTouchCornerIndex = -1;
    private int mTouchPointThreshold;
    private int mCropRectMinSize;
    private int mCropRectCornerTouchAreaLineLength;

    private OverlayViewChangeListener mCallback;

    private boolean mShouldSetupCropBounds;

    {
        mTouchPointThreshold = getResources().getDimensionPixelSize(R.dimen.ucrop_default_crop_rect_corner_touch_threshold);
        mCropRectMinSize = getResources().getDimensionPixelSize(R.dimen.ucrop_default_crop_rect_min_size);
        mCropRectCornerTouchAreaLineLength = getResources().getDimensionPixelSize(R.dimen.ucrop_default_crop_rect_corner_touch_area_line_length);
    }

    public OverlayView(Context context) {
        this(context, null);
    }

    public OverlayView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public OverlayView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init();
    }

    public OverlayViewChangeListener getOverlayViewChangeListener() {
        return mCallback;
    }

    public void setOverlayViewChangeListener(OverlayViewChangeListener callback) {
        mCallback = callback;
    }

    @NonNull
    public RectF getCropViewRect() {
        return mCropViewRect;
    }

    @Deprecated
    /***
     * Freestyle crop mode가 두 개 이상 있으므로 새 메소드 {@link #getFreestyleCropMode()}을 사용하라.
     * */
    public boolean isFreestyleCropEnabled() {
        return mFreestyleCropMode == FREESTYLE_CROP_MODE_ENABLE;
    }

    @Deprecated
    /***
     * Freestyle crop mode가 두 개 이상 있으므로 새 메소드 {@link #setFreestyleCropMode(int)}를 사용하라.
     * */
    public void setFreestyleCropEnabled(boolean freestyleCropEnabled) {
        mFreestyleCropMode = freestyleCropEnabled ? FREESTYLE_CROP_MODE_ENABLE : FREESTYLE_CROP_MODE_DISABLE;
    }

    @FreestyleMode
    public int getFreestyleCropMode() {
        return mFreestyleCropMode;
    }

    public void setFreestyleCropMode(@FreestyleMode int mFreestyleCropMode) {
        this.mFreestyleCropMode = mFreestyleCropMode;
        postInvalidate();
    }

    /**
     * {@link #mCircleDimmedLayer} 변수에 대한 도구
     *
     * @param circleDimmedLayer - 희미한 레이어가 원형으로 되도록 설정하려면 true로 설정한다.
     * */
    public void setCircleDimmedLayer(boolean circleDimmedLayer) {
        mCircleDimmedLayer = circleDimmedLayer;
    }

    /**
     * 자르기 격자 행 수에 대한 도구
     * 더 이상 유효하지 않기 떄문에{@link #mGridPoints}의 변수를 재설정한다.
     */
    public void setCropGridRowCount(@IntRange(from = 0) int cropGridRowCount) {
        mCropGridRowCount = cropGridRowCount;
        mGridPoints = null;
    }

    /**
     * 자르기 격자 열 수에 대한 도구
     * 더 이상 유효하지 않기 때문에 {@link #mGridPoints}의 변수를 재설정한다.
     */
    public void setCropGridColumnCount(@IntRange(from = 0) int cropGridColumnCount) {
        mCropGridColumnCount = cropGridColumnCount;
        mGridPoints = null;
    }

    /**
     * {@link #mShowCropFrame} 변수에 대한 도구
     *
     * @param showCropFrame - 이미지 상단에 자르기 프레임 사각형을 표시하려면 true로 설정한다.
     */
    public void setShowCropFrame(boolean showCropFrame) {
        mShowCropFrame = showCropFrame;
    }

    /**
     * {@link #mShowCropGrid} 변수에 대한 도구
     *
     * @param showCropGrid - 이미지 상단에 자르기 격자를 표시하려면 true로 설정한다.
     */
    public void setShowCropGrid(boolean showCropGrid) {
        mShowCropGrid = showCropGrid;
    }

    /**
     * {@link #mDimmedColor} 변수에 대한 도구
     *
     * @param dimmedColor - 자르기 범위 주변의 흐릿한 색 영역의 원하는 색상
     */
    public void setDimmedColor(@ColorInt int dimmedColor) {
        mDimmedColor = dimmedColor;
    }

    /**
     * 자르기 프레임 선 폭에 대한 도구
     */
    public void setCropFrameStrokeWidth(@IntRange(from = 0) int width) {
        mCropFramePaint.setStrokeWidth(width);
    }

    /**
     * 자르기 격자 선 폭에 대한 도구
     */
    public void setCropGridStrokeWidth(@IntRange(from = 0) int width) {
        mCropGridPaint.setStrokeWidth(width);
    }

    /**
     * 자르기 프레임 색상 설정 도구
     */
    public void setCropFrameColor(@ColorInt int color) {
        mCropFramePaint.setColor(color);
    }

    /**
     * 자르기 격자 색상 설정 도구
     */
    public void setCropGridColor(@ColorInt int color) {
        mCropGridPaint.setColor(color);
    }

    /**
     * 자르기 범위의 종횡비를 설정한다.
     *
     * @param targetAspectRatio - 이미지 자르기의 가로 세율 비율(예: 16:9의 경우 1.77(7)
     */
    public void setTargetAspectRatio(final float targetAspectRatio) {
        mTargetAspectRatio = targetAspectRatio;
        if (mThisWidth > 0) {
            setupCropBounds();
            postInvalidate();
        } else {
            mShouldSetupCropBounds = true;
        }
    }

    /**
     * 주어진 종횡비 및 뷰 크기에 대해 자르기 경계 사각형을 설정한다.
     * {@link #mCropViewRect}은 자르기 경계를 그리는데 사용되며, 패딩을 사용한다.
     */
    public void setupCropBounds() {
        int height = (int) (mThisWidth / mTargetAspectRatio);
        if (height > mThisHeight) {
            int width = (int) (mThisHeight * mTargetAspectRatio);
            int halfDiff = (mThisWidth - width) / 2;
            mCropViewRect.set(getPaddingLeft() + halfDiff, getPaddingTop(),
                    getPaddingLeft() + width + halfDiff, getPaddingTop() + mThisHeight);
        } else {
            int halfDiff = (mThisHeight - height) / 2;
            mCropViewRect.set(getPaddingLeft(), getPaddingTop() + halfDiff,
                    getPaddingLeft() + mThisWidth, getPaddingTop() + height + halfDiff);
        }

        if (mCallback != null) {
            mCallback.onCropRectUpdated(mCropViewRect);
        }

        updateGridPoints();
    }

    private void updateGridPoints() {
        mCropGridCorners = RectUtils.getCornersFromRect(mCropViewRect);
        mCropGridCenter = RectUtils.getCenterFromRect(mCropViewRect);

        mGridPoints = null;
        mCircularPath.reset();
        mCircularPath.addCircle(mCropViewRect.centerX(), mCropViewRect.centerY(),
                Math.min(mCropViewRect.width(), mCropViewRect.height()) / 2.f, Path.Direction.CW);
    }

    protected void init() {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN_MR2 &&
                Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            setLayerType(LAYER_TYPE_SOFTWARE, null);
        }
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        if (changed) {
            left = getPaddingLeft();
            top = getPaddingTop();
            right = getWidth() - getPaddingRight();
            bottom = getHeight() - getPaddingBottom();
            mThisWidth = right - left;
            mThisHeight = bottom - top;

            if (mShouldSetupCropBounds) {
                mShouldSetupCropBounds = false;
                setTargetAspectRatio(mTargetAspectRatio);
            }
        }
    }

    /**
     * 이미지와 함께 음영 처리된 레이어, 자르기 경계 및 자르기 가이드라인이 있어야 한다.
     * */
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        drawDimmedLayer(canvas);
        drawCropGrid(canvas);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (mCropViewRect.isEmpty() || mFreestyleCropMode == FREESTYLE_CROP_MODE_DISABLE) { return false; }

        float x = event.getX();
        float y = event.getY();

        if ((event.getAction() & MotionEvent.ACTION_MASK) == MotionEvent.ACTION_DOWN) {
            mCurrentTouchCornerIndex = getCurrentTouchIndex(x, y);
            boolean shouldHandle = mCurrentTouchCornerIndex != -1;
            if (!shouldHandle) {
                mPreviousTouchX = -1;
                mPreviousTouchY = -1;
            } else if (mPreviousTouchX < 0) {
                mPreviousTouchX = x;
                mPreviousTouchY = y;
            }
            return shouldHandle;
        }

        if ((event.getAction() & MotionEvent.ACTION_MASK) == MotionEvent.ACTION_MOVE) {
            if (event.getPointerCount() == 1 && mCurrentTouchCornerIndex != -1) {

                x = Math.min(Math.max(x, getPaddingLeft()), getWidth() - getPaddingRight());
                y = Math.min(Math.max(y, getPaddingTop()), getHeight() - getPaddingBottom());

                updateCropViewRect(x, y);

                mPreviousTouchX = x;
                mPreviousTouchY = y;

                return true;
            }
        }

        if ((event.getAction() & MotionEvent.ACTION_MASK) == MotionEvent.ACTION_UP) {
            mPreviousTouchX = -1;
            mPreviousTouchY = -1;
            mCurrentTouchCornerIndex = -1;

            if (mCallback != null) {
                mCallback.onCropRectUpdated(mCropViewRect);
            }
        }

        return false;
    }

    /**
     * *
     * 모서리의 순서는 다음과 같다:
     * 0------->1
     * ^        |
     * |   4    |
     * |        v
     * 3<-------2
     */
    private void updateCropViewRect(float touchX, float touchY) {
        mTempRect.set(mCropViewRect);

        switch (mCurrentTouchCornerIndex) {
            // resize rectangle
            case 0:
                mTempRect.set(touchX, touchY, mCropViewRect.right, mCropViewRect.bottom);
                break;
            case 1:
                mTempRect.set(mCropViewRect.left, touchY, touchX, mCropViewRect.bottom);
                break;
            case 2:
                mTempRect.set(mCropViewRect.left, mCropViewRect.top, touchX, touchY);
                break;
            case 3:
                mTempRect.set(touchX, mCropViewRect.top, mCropViewRect.right, touchY);
                break;
            // move rectangle
            case 4:
                mTempRect.offset(touchX - mPreviousTouchX, touchY - mPreviousTouchY);
                if (mTempRect.left > getLeft() && mTempRect.top > getTop()
                        && mTempRect.right < getRight() && mTempRect.bottom < getBottom()) {
                    mCropViewRect.set(mTempRect);
                    updateGridPoints();
                    postInvalidate();
                }
                return;
        }

        boolean changeHeight = mTempRect.height() >= mCropRectMinSize;
        boolean changeWidth = mTempRect.width() >= mCropRectMinSize;
        mCropViewRect.set(
                changeWidth ? mTempRect.left : mCropViewRect.left,
                changeHeight ? mTempRect.top : mCropViewRect.top,
                changeWidth ? mTempRect.right : mCropViewRect.right,
                changeHeight ? mTempRect.bottom : mCropViewRect.bottom);

        if (changeHeight || changeWidth) {
            updateGridPoints();
            postInvalidate();
        }
    }

    /**
     * *
     * float 배열의 모서리 순서는 다음과 같다:
     * 0------->1
     * ^        |
     * |   4    |
     * |        v
     * 3<-------2
     *
     * @return - 드래그 되는 모서리의 인덱스
     */
    private int getCurrentTouchIndex(float touchX, float touchY) {
        int closestPointIndex = -1;
        double closestPointDistance = mTouchPointThreshold;
        for (int i = 0; i < 8; i += 2) {
            double distanceToCorner = Math.sqrt(Math.pow(touchX - mCropGridCorners[i], 2)
                    + Math.pow(touchY - mCropGridCorners[i + 1], 2));
            if (distanceToCorner < closestPointDistance) {
                closestPointDistance = distanceToCorner;
                closestPointIndex = i / 2;
            }
        }

        if (mFreestyleCropMode == FREESTYLE_CROP_MODE_ENABLE && closestPointIndex < 0 && mCropViewRect.contains(touchX, touchY)) {
            return 4;
        }

//        for (int i = 0; i <= 8; i += 2) {
//
//            double distanceToCorner;
//            if (i < 8) { // corners
//                distanceToCorner = Math.sqrt(Math.pow(touchX - mCropGridCorners[i], 2)
//                        + Math.pow(touchY - mCropGridCorners[i + 1], 2));
//            } else { // center
//                distanceToCorner = Math.sqrt(Math.pow(touchX - mCropGridCenter[0], 2)
//                        + Math.pow(touchY - mCropGridCenter[1], 2));
//            }
//            if (distanceToCorner < closestPointDistance) {
//                closestPointDistance = distanceToCorner;
//                closestPointIndex = i / 2;
//            }
//        }
        return closestPointIndex;
    }

    /**
     * 자르기 경계 주위에 흐리게 표시된 영역을 그린다.
     *
     * @param canvas - 유효한 캔버스 객체
     */
    protected void drawDimmedLayer(@NonNull Canvas canvas) {
        canvas.save();
        if (mCircleDimmedLayer) {
            canvas.clipPath(mCircularPath, Region.Op.DIFFERENCE);
        } else {
            canvas.clipRect(mCropViewRect, Region.Op.DIFFERENCE);
        }
        canvas.drawColor(mDimmedColor);
        canvas.restore();

        if (mCircleDimmedLayer) { // Draw 1px stroke to fix antialias
            canvas.drawCircle(mCropViewRect.centerX(), mCropViewRect.centerY(),
                    Math.min(mCropViewRect.width(), mCropViewRect.height()) / 2.f, mDimmedStrokePaint);
        }
    }

    /**
     * 필요한 경우 자르기 경계(빈 사각형)와 자르기 가이드라인(자르기 경계 내 수직 및 수평선)을 그린다.
     * @param canvas - 유효한 캔버스 객체
     */
    protected void drawCropGrid(@NonNull Canvas canvas) {
        if (mShowCropGrid) {
            if (mGridPoints == null && !mCropViewRect.isEmpty()) {

                mGridPoints = new float[(mCropGridRowCount) * 4 + (mCropGridColumnCount) * 4];

                int index = 0;
                for (int i = 0; i < mCropGridRowCount; i++) {
                    mGridPoints[index++] = mCropViewRect.left;
                    mGridPoints[index++] = (mCropViewRect.height() * (((float) i + 1.0f) / (float) (mCropGridRowCount + 1))) + mCropViewRect.top;
                    mGridPoints[index++] = mCropViewRect.right;
                    mGridPoints[index++] = (mCropViewRect.height() * (((float) i + 1.0f) / (float) (mCropGridRowCount + 1))) + mCropViewRect.top;
                }

                for (int i = 0; i < mCropGridColumnCount; i++) {
                    mGridPoints[index++] = (mCropViewRect.width() * (((float) i + 1.0f) / (float) (mCropGridColumnCount + 1))) + mCropViewRect.left;
                    mGridPoints[index++] = mCropViewRect.top;
                    mGridPoints[index++] = (mCropViewRect.width() * (((float) i + 1.0f) / (float) (mCropGridColumnCount + 1))) + mCropViewRect.left;
                    mGridPoints[index++] = mCropViewRect.bottom;
                }
            }

            if (mGridPoints != null) {
                canvas.drawLines(mGridPoints, mCropGridPaint);
            }
        }

        if (mShowCropFrame) {
            canvas.drawRect(mCropViewRect, mCropFramePaint);
        }

        if (mFreestyleCropMode != FREESTYLE_CROP_MODE_DISABLE) {
            canvas.save();

            mTempRect.set(mCropViewRect);
            mTempRect.inset(mCropRectCornerTouchAreaLineLength, -mCropRectCornerTouchAreaLineLength);
            canvas.clipRect(mTempRect, Region.Op.DIFFERENCE);

            mTempRect.set(mCropViewRect);
            mTempRect.inset(-mCropRectCornerTouchAreaLineLength, mCropRectCornerTouchAreaLineLength);
            canvas.clipRect(mTempRect, Region.Op.DIFFERENCE);

            canvas.drawRect(mCropViewRect, mCropFrameCornersPaint);

            canvas.restore();
        }
    }

    /**
     * 스타일 지정된 특성에서 필요한 모든 값을 추출한다.
     * 보기를 구성하는데 사용된다.
     */
    @SuppressWarnings("deprecation")
    protected void processStyledAttributes(@NonNull TypedArray a) {
        mCircleDimmedLayer = a.getBoolean(R.styleable.ucrop_UCropView_ucrop_circle_dimmed_layer, DEFAULT_CIRCLE_DIMMED_LAYER);
        mDimmedColor = a.getColor(R.styleable.ucrop_UCropView_ucrop_dimmed_color,
                getResources().getColor(R.color.ucrop_color_default_dimmed));
        mDimmedStrokePaint.setColor(mDimmedColor);
        mDimmedStrokePaint.setStyle(Paint.Style.STROKE);
        mDimmedStrokePaint.setStrokeWidth(1);

        initCropFrameStyle(a);
        mShowCropFrame = a.getBoolean(R.styleable.ucrop_UCropView_ucrop_show_frame, DEFAULT_SHOW_CROP_FRAME);

        initCropGridStyle(a);
        mShowCropGrid = a.getBoolean(R.styleable.ucrop_UCropView_ucrop_show_grid, DEFAULT_SHOW_CROP_GRID);
    }

    /**
     * 자르기 경계에 대한 객체를 설정한다.
     */
    @SuppressWarnings("deprecation")
    private void initCropFrameStyle(@NonNull TypedArray a) {
        int cropFrameStrokeSize = a.getDimensionPixelSize(R.styleable.ucrop_UCropView_ucrop_frame_stroke_size,
                getResources().getDimensionPixelSize(R.dimen.ucrop_default_crop_frame_stoke_width));
        int cropFrameColor = a.getColor(R.styleable.ucrop_UCropView_ucrop_frame_color,
                getResources().getColor(R.color.ucrop_color_default_crop_frame));
        mCropFramePaint.setStrokeWidth(cropFrameStrokeSize);
        mCropFramePaint.setColor(cropFrameColor);
        mCropFramePaint.setStyle(Paint.Style.STROKE);

        mCropFrameCornersPaint.setStrokeWidth(cropFrameStrokeSize * 3);
        mCropFrameCornersPaint.setColor(cropFrameColor);
        mCropFrameCornersPaint.setStyle(Paint.Style.STROKE);
    }

    /**
     * 자르기 가이드라인에 대한 객체를 설정한다.
     */
    @SuppressWarnings("deprecation")
    private void initCropGridStyle(@NonNull TypedArray a) {
        int cropGridStrokeSize = a.getDimensionPixelSize(R.styleable.ucrop_UCropView_ucrop_grid_stroke_size,
                getResources().getDimensionPixelSize(R.dimen.ucrop_default_crop_grid_stoke_width));
        int cropGridColor = a.getColor(R.styleable.ucrop_UCropView_ucrop_grid_color,
                getResources().getColor(R.color.ucrop_color_default_crop_grid));
        mCropGridPaint.setStrokeWidth(cropGridStrokeSize);
        mCropGridPaint.setColor(cropGridColor);

        mCropGridRowCount = a.getInt(R.styleable.ucrop_UCropView_ucrop_grid_row_count, DEFAULT_CROP_GRID_ROW_COUNT);
        mCropGridColumnCount = a.getInt(R.styleable.ucrop_UCropView_ucrop_grid_column_count, DEFAULT_CROP_GRID_COLUMN_COUNT);
    }


    @Retention(RetentionPolicy.SOURCE)
    @IntDef({FREESTYLE_CROP_MODE_DISABLE, FREESTYLE_CROP_MODE_ENABLE, FREESTYLE_CROP_MODE_ENABLE_WITH_PASS_THROUGH})
    public @interface FreestyleMode {
    }

}
```