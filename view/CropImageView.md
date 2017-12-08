####com.yalantis.ucrop\view\CropImageView


```
package com.yalantis.ucrop.view;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Bitmap;
import android.graphics.Matrix;
import android.graphics.RectF;
import android.graphics.drawable.Drawable;
import android.support.annotation.IntRange;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.util.AttributeSet;

import com.yalantis.ucrop.R;
import com.yalantis.ucrop.callback.BitmapCropCallback;
import com.yalantis.ucrop.callback.CropBoundsChangeListener;
import com.yalantis.ucrop.model.CropParameters;
import com.yalantis.ucrop.model.ImageState;
import com.yalantis.ucrop.task.BitmapCropTask;
import com.yalantis.ucrop.util.CubicEasing;
import com.yalantis.ucrop.util.RectUtils;

import java.lang.ref.WeakReference;
import java.util.Arrays;

**Created by Oleksii Shliama (https://github.com/shliama).**

 ** 이 클래스는 자르기 기능, 자르기 지침을 작성하고 이미지를 올바른 상태로 유지하는 방법을 추가한다.**
 ** 또한 확장을 위한 검사를 추가하기 위해 부모 클래스 메소드를 확장한다.**
 
 
	public class CropImageView extends TransformImageView {

    public static final int DEFAULT_MAX_BITMAP_SIZE = 0;
    public static final int DEFAULT_IMAGE_TO_CROP_BOUNDS_ANIM_DURATION = 500;
    public static final float DEFAULT_MAX_SCALE_MULTIPLIER = 10.0f;
    public static final float SOURCE_IMAGE_ASPECT_RATIO = 0f;
    public static final float DEFAULT_ASPECT_RATIO = SOURCE_IMAGE_ASPECT_RATIO;

    private final RectF mCropRect = new RectF();

    private final Matrix mTempMatrix = new Matrix();

    private float mTargetAspectRatio;
    private float mMaxScaleMultiplier = DEFAULT_MAX_SCALE_MULTIPLIER;

    private CropBoundsChangeListener mCropBoundsChangeListener;

    private Runnable mWrapCropBoundsRunnable, mZoomImageToPositionRunnable = null;

    private float mMaxScale, mMinScale;
    private int mMaxResultImageSizeX = 0, mMaxResultImageSizeY = 0;
    private long mImageToWrapCropBoundsAnimDuration = DEFAULT_IMAGE_TO_CROP_BOUNDS_ANIM_DURATION;

    public CropImageView(Context context) {
        this(context, null);
    }

    public CropImageView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CropImageView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    /**
     * 모든 최신 애니메이션을 취소하고 '자르기 영역'을 채우기 위해 이미지를 설정(애니메이션 없이)
     * 적절한 매개변수를 사용하여 {@link BitmapCropTask}를 생성하고 실행한다.*/
    public void cropAndSaveImage(@NonNull Bitmap.CompressFormat compressFormat, int compressQuality,
                                 @Nullable BitmapCropCallback cropCallback) {
        cancelAllAnimations();
        setImageToWrapCropBounds(false);

        final ImageState imageState = new ImageState(
                mCropRect, RectUtils.trapToRect(mCurrentImageCorners),
                getCurrentScale(), getCurrentAngle());

        final CropParameters cropParameters = new CropParameters(
                mMaxResultImageSizeX, mMaxResultImageSizeY,
                compressFormat, compressQuality,
                getImageInputPath(), getImageOutputPath(), getExifInfo());

        new BitmapCropTask(getViewBitmap(), imageState, cropParameters, cropCallback).execute();
    }


    /**
     * 현재 이미지와 자르기 비율의 최대 범위 값을 반환한다.
     */
    public float getMaxScale() {
        return mMaxScale;
    }


    /**
     * 현재 이미지와 자르기 비율의 최소 범위 값을 반환한다.
     */
    public float getMinScale() {
        return mMinScale;
    }

    /**
     *자르기 범위의 종횡비를 반환
     */
    public float getTargetAspectRatio() {
        return mTargetAspectRatio;
    }

    /**
     * 현재 자르기 사각형을 지정된 것으로 업데이트한다.
     * 이미지 특성을 다시 계산하고 새로운 자르기 사각형에 맞춰 위치를 지정한다.
     */
    public void setCropRect(RectF cropRect) {
        mTargetAspectRatio = cropRect.width() / cropRect.height();
        mCropRect.set(cropRect.left - getPaddingLeft(), cropRect.top - getPaddingTop(),
                cropRect.right - getPaddingRight(), cropRect.bottom - getPaddingBottom());
        calculateImageScaleBounds();
        setImageToWrapCropBounds();
    }


    /**
     * 자르기 범위의 종횡비를 설정한다.
     * {@link #SOURCE_IMAGE_ASPECT_RATIO}의 값이 pass이면
     * 종횡비는 현재 이미지 너비와 높이를 기반으로 계산된다.
     * @param targetAspectRatio - 이미지 자르기의 가로 세로 비율 (예 - 16:9의 경우 1.77(7))
     */
    public void setTargetAspectRatio(float targetAspectRatio) {
        final Drawable drawable = getDrawable();
        if (drawable == null) {
            mTargetAspectRatio = targetAspectRatio;
            return;
        }

        if (targetAspectRatio == SOURCE_IMAGE_ASPECT_RATIO) {
            mTargetAspectRatio = drawable.getIntrinsicWidth() / (float) drawable.getIntrinsicHeight();
        } else {
            mTargetAspectRatio = targetAspectRatio;
        }

        if (mCropBoundsChangeListener != null) {
            mCropBoundsChangeListener.onCropAspectRatioChanged(mTargetAspectRatio);
        }
    }

    @Nullable
    public CropBoundsChangeListener getCropBoundsChangeListener() {
        return mCropBoundsChangeListener;
    }

    public void setCropBoundsChangeListener(@Nullable CropBoundsChangeListener cropBoundsChangeListener) {
        mCropBoundsChangeListener = cropBoundsChangeListener;
    }



    /**
     * 잘린 이미지의 최대 너비 설정 - X축
     * @param maxResultImageSizeX - 크기(픽셀 단위)
     */
     public void setMaxResultImageSizeX(@IntRange(from = 10) int maxResultImageSizeX) {
        mMaxResultImageSizeX = maxResultImageSizeX;
    }


    /**
     * 잘린 이미지의 최대 너비 설정 - Y축
     * @param maxResultImageSizeY - 크기(픽셀 단위)
     * */

    public void setMaxResultImageSizeY(@IntRange(from = 10) int maxResultImageSizeY) {
        mMaxResultImageSizeY = maxResultImageSizeY;
    }


    /**
     *자르기 경계를 감싸기 위해 이미지의 애니메이션 지속 시간을 설정한다.
     * @param imageToWrapCropBoundsAnimDuration - 기간(밀리세컨드 단위)
     * */

    public void setImageToWrapCropBoundsAnimDuration(@IntRange(from = 100) long imageToWrapCropBoundsAnimDuration) {
        if (imageToWrapCropBoundsAnimDuration > 0) {
            mImageToWrapCropBoundsAnimDuration = imageToWrapCropBoundsAnimDuration;
        } else {
            throw new IllegalArgumentException("Animation duration cannot be negative value.");
        }
    }


    /**
     *배율은 최소 이미지 범위에서 최대 이미지 범위 계산하는데 사용된다.
     * @param maxScaleMultiplier - (최소 범위 * 최대 범위)
     */
    public void setMaxScaleMultiplier(float maxScaleMultiplier) {
        mMaxScaleMultiplier = maxScaleMultiplier;
    }

    /**
     * 이미지 중심과 관련된 주어진 값에 대해 이미지의 크기를 조절(축소)한다.
     */
    public void zoomOutImage(float deltaScale) {
        zoomOutImage(deltaScale, mCropRect.centerX(), mCropRect.centerY());
    }

    /**
     * 주어진 값 (x,y)에 관련된 주어진 값에 대해 이미지의 크기를 조절(축소)한다.
     */
    public void zoomOutImage(float scale, float centerX, float centerY) {
        if (scale >= getMinScale()) {
            postScale(scale / getCurrentScale(), centerX, centerY);
        }
    }

    /**
     * 이미지 중심과 관련된 주어진 값에 대해 이미지의 크기를 조절(확대)한다.
     */
    public void zoomInImage(float deltaScale) {
        zoomInImage(deltaScale, mCropRect.centerX(), mCropRect.centerY());
    }

    /**
     * 주어진 값 (x,y)에 관련된 주어진 값에 대해 이미지의 크기를 조절(확대)한다.
     */
    public void zoomInImage(float scale, float centerX, float centerY) {
        if (scale <= getMaxScale()) {
            postScale(scale / getCurrentScale(), centerX, centerY);
        }
    }

    /**
     * 결과의 범위값이 최소/최대의 경계 내에 있는 경우에만,
     * 점 (px, py)와 관련된 주어진 값에 대한 이미지 범위를 변경한다.
     *
     *
     * @param deltaScale - 값의 범위
     * @param px         - X 중심의 범위
     * @param py         - Y 중심의 범위
     */
    public void postScale(float deltaScale, float px, float py) {
        if (deltaScale > 1 && getCurrentScale() * deltaScale <= getMaxScale()) {
            super.postScale(deltaScale, px, py);
        } else if (deltaScale < 1 && getCurrentScale() * deltaScale >= getMinScale()) {
            super.postScale(deltaScale, px, py);
        }
    }

    /**
     * 이미지 중심과 관련된 특정 각도로 이미지를 회전한다.
     *
     * @param deltaAngle - 회전할 각도
     */
    public void postRotate(float deltaAngle) {
        postRotate(deltaAngle, mCropRect.centerX(), mCropRect.centerY());
    }

    /**
     * 애니메이션을 나타내는 현재 실행 가능한 모든 객체를 취소한다.
     */
    public void cancelAllAnimations() {
        removeCallbacks(mWrapCropBoundsRunnable);
        removeCallbacks(mZoomImageToPositionRunnable);
    }

    public void setImageToWrapCropBounds() {
        setImageToWrapCropBounds(true);
    }

    /**
     * 이미지가 자르기 범위를 채우지 않으면 해당 영역을 채우기 위해 이미지를 올바르게 변환하고
     * 크기를 조정해야 한다. 따라서 이 메소드는 delta x, y 및 범위 값을 계산하고 이미지를
     * 애니메이션으로 애니메이션으로 확대하는 {@link WrapCropBoundsRunnable}로 전달.
     * 크기 값은 이미지가 자르기 경계 사각형 중심으로 변환된 후 자르기 경계를 채우지 않는 경우에만
     * 계산되어야 한다. 임시 변수를 사용하여 이 방법을 확인한다.
     */
    public void setImageToWrapCropBounds(boolean animate) {
        if (mBitmapLaidOut && !isImageWrapCropBounds()) {

            float currentX = mCurrentImageCenter[0];
            float currentY = mCurrentImageCenter[1];
            float currentScale = getCurrentScale();

            float deltaX = mCropRect.centerX() - currentX;
            float deltaY = mCropRect.centerY() - currentY;
            float deltaScale = 0;

            mTempMatrix.reset();
            mTempMatrix.setTranslate(deltaX, deltaY);

            final float[] tempCurrentImageCorners = Arrays.copyOf(mCurrentImageCorners, mCurrentImageCorners.length);
            mTempMatrix.mapPoints(tempCurrentImageCorners);

            boolean willImageWrapCropBoundsAfterTranslate = isImageWrapCropBounds(tempCurrentImageCorners);

            if (willImageWrapCropBoundsAfterTranslate) {
                final float[] imageIndents = calculateImageIndents();
                deltaX = -(imageIndents[0] + imageIndents[2]);
                deltaY = -(imageIndents[1] + imageIndents[3]);
            } else {
                RectF tempCropRect = new RectF(mCropRect);
                mTempMatrix.reset();
                mTempMatrix.setRotate(getCurrentAngle());
                mTempMatrix.mapRect(tempCropRect);

                final float[] currentImageSides = RectUtils.getRectSidesFromCorners(mCurrentImageCorners);

                deltaScale = Math.max(tempCropRect.width() / currentImageSides[0],
                        tempCropRect.height() / currentImageSides[1]);
                deltaScale = deltaScale * currentScale - currentScale;
            }

            if (animate) {
                post(mWrapCropBoundsRunnable = new WrapCropBoundsRunnable(
                        CropImageView.this, mImageToWrapCropBoundsAnimDuration, currentX, currentY, deltaX, deltaY,
                        currentScale, deltaScale, willImageWrapCropBoundsAfterTranslate));
            } else {
                postTranslate(deltaX, deltaY);
                if (!willImageWrapCropBoundsAfterTranslate) {
                    zoomInImage(currentScale + deltaScale, mCropRect.centerX(), mCropRect.centerY());
                }
            }
        }
    }

    /**
     * 첫번째로, 이미지 회전 및 자르기 직사각형을 회전한다.
     * 두번째로, 꼭지점 사이의 delta를 계산한다.
     * 세번째로, delta(부호)에 따라 배열 안에 0 또는 delta를 넣는다.
     *네번째로, Matrix를 사용하여 그 점들을 다시 회전시킨다.
     *
     * @return - 이미지 들여쓰기의 float 배열 (4 float) - [left, top, right, bottom]순으로
     */
    private float[] calculateImageIndents() {
        mTempMatrix.reset();
        mTempMatrix.setRotate(-getCurrentAngle());

        float[] unrotatedImageCorners = Arrays.copyOf(mCurrentImageCorners, mCurrentImageCorners.length);
        float[] unrotatedCropBoundsCorners = RectUtils.getCornersFromRect(mCropRect);

        mTempMatrix.mapPoints(unrotatedImageCorners);
        mTempMatrix.mapPoints(unrotatedCropBoundsCorners);

        RectF unrotatedImageRect = RectUtils.trapToRect(unrotatedImageCorners);
        RectF unrotatedCropRect = RectUtils.trapToRect(unrotatedCropBoundsCorners);

        float deltaLeft = unrotatedImageRect.left - unrotatedCropRect.left;
        float deltaTop = unrotatedImageRect.top - unrotatedCropRect.top;
        float deltaRight = unrotatedImageRect.right - unrotatedCropRect.right;
        float deltaBottom = unrotatedImageRect.bottom - unrotatedCropRect.bottom;

        float indents[] = new float[4];
        indents[0] = (deltaLeft > 0) ? deltaLeft : 0;
        indents[1] = (deltaTop > 0) ? deltaTop : 0;
        indents[2] = (deltaRight < 0) ? deltaRight : 0;
        indents[3] = (deltaBottom < 0) ? deltaBottom : 0;

        mTempMatrix.reset();
        mTempMatrix.setRotate(getCurrentAngle());
        mTempMatrix.mapPoints(indents);

        return indents;
    }

    /**
     * 이미지를 배치할 때, 현재 자르기 범위에 맞게 적절히 중심을 맞추어야 한다.
     */
    @Override
    protected void onImageLaidOut() {
        super.onImageLaidOut();
        final Drawable drawable = getDrawable();
        if (drawable == null) {
            return;
        }

        float drawableWidth = drawable.getIntrinsicWidth();
        float drawableHeight = drawable.getIntrinsicHeight();

        if (mTargetAspectRatio == SOURCE_IMAGE_ASPECT_RATIO) {
            mTargetAspectRatio = drawableWidth / drawableHeight;
        }

        int height = (int) (mThisWidth / mTargetAspectRatio);
        if (height > mThisHeight) {
            int width = (int) (mThisHeight * mTargetAspectRatio);
            int halfDiff = (mThisWidth - width) / 2;
            mCropRect.set(halfDiff, 0, width + halfDiff, mThisHeight);
        } else {
            int halfDiff = (mThisHeight - height) / 2;
            mCropRect.set(0, halfDiff, mThisWidth, height + halfDiff);
        }

        calculateImageScaleBounds(drawableWidth, drawableHeight);
        setupInitialImagePosition(drawableWidth, drawableHeight);

        if (mCropBoundsChangeListener != null) {
            mCropBoundsChangeListener.onCropAspectRatioChanged(mTargetAspectRatio);
        }
        if (mTransformImageListener != null) {
            mTransformImageListener.onScale(getCurrentScale());
            mTransformImageListener.onRotate(getCurrentAngle());
        }
    }

    /**
     * 현재 이미지가 자르기 경계를 채우는지 여부를 점검한다.
     */
    protected boolean isImageWrapCropBounds() {
        return isImageWrapCropBounds(mCurrentImageCorners);
    }

    /**
     * 4개의 모서리 점(8 float)으로 표현되는 사각형이 자르기 경계 사각형을 채우는지 여부를 점검한다.
     *
     * @param imageCorners - 사각형의 모서리
     * @return - 자르기 범위를 감싸는 경우 true, 그렇지 않으면 false
     */
    protected boolean isImageWrapCropBounds(float[] imageCorners) {
        mTempMatrix.reset();
        mTempMatrix.setRotate(-getCurrentAngle());

        float[] unrotatedImageCorners = Arrays.copyOf(imageCorners, imageCorners.length);
        mTempMatrix.mapPoints(unrotatedImageCorners);

        float[] unrotatedCropBoundsCorners = RectUtils.getCornersFromRect(mCropRect);
        mTempMatrix.mapPoints(unrotatedCropBoundsCorners);

        return RectUtils.trapToRect(unrotatedImageCorners).contains(RectUtils.trapToRect(unrotatedCropBoundsCorners));
    }

    /**
     *주어진 중심(x, y)에 관련된 이미지 범위(주어진 시간동안 애니메이션 확대/축소)를 변경한다.
     *
     * @param scale      - 목표 범위
     * @param centerX    - 중심 X의 범위
     * @param centerY    - 중심 Y의 범위
     * @param durationMs - 애니메이션 지속 시간
     */
    protected void zoomImageToPosition(float scale, float centerX, float centerY, long durationMs) {
        if (scale > getMaxScale()) {
            scale = getMaxScale();
        }

        final float oldScale = getCurrentScale();
        final float deltaScale = scale - oldScale;

        post(mZoomImageToPositionRunnable = new ZoomImageToPosition(CropImageView.this,
                durationMs, oldScale, deltaScale, centerX, centerY));
    }

    private void calculateImageScaleBounds() {
        final Drawable drawable = getDrawable();
        if (drawable == null) {
            return;
        }
        calculateImageScaleBounds(drawable.getIntrinsicWidth(), drawable.getIntrinsicHeight());
    }

    /**
     * 현재 {@link #mCropRect}에 대한 이미지의 최소값과 최대값을 계산한다.
     *
     * @param drawableWidth  - 이미지 너비
     * @param drawableHeight - 이미지 높이
     */
    private void calculateImageScaleBounds(float drawableWidth, float drawableHeight) {
        float widthScale = Math.min(mCropRect.width() / drawableWidth, mCropRect.width() / drawableHeight);
        float heightScale = Math.min(mCropRect.height() / drawableHeight, mCropRect.height() / drawableWidth);

        mMinScale = Math.min(widthScale, heightScale);
        mMaxScale = mMinScale * mMaxScaleMultiplier;
    }

    /**
     * 초기 이미지 위치를 계산하여 올바르게 배치한다.
     * 그런 다음 해당 값을 현재 이미지 행렬에 설정한다.
     *
     * @param drawableWidth  - 이미지 너비
     * @param drawableHeight - 이미지 높이
     */
    private void setupInitialImagePosition(float drawableWidth, float drawableHeight) {
        float cropRectWidth = mCropRect.width();
        float cropRectHeight = mCropRect.height();

        float widthScale = mCropRect.width() / drawableWidth;
        float heightScale = mCropRect.height() / drawableHeight;

        float initialMinScale = Math.max(widthScale, heightScale);

        float tw = (cropRectWidth - drawableWidth * initialMinScale) / 2.0f + mCropRect.left;
        float th = (cropRectHeight - drawableHeight * initialMinScale) / 2.0f + mCropRect.top;

        mCurrentImageMatrix.reset();
        mCurrentImageMatrix.postScale(initialMinScale, initialMinScale);
        mCurrentImageMatrix.postTranslate(tw, th);
        setImageMatrix(mCurrentImageMatrix);
    }

    /**
     * 스타일 지정된 특성에서 필요한 모든 값을 추출한다.
     * 보기를 구성하는데 사용된다.
     */
    @SuppressWarnings("deprecation")
    protected void processStyledAttributes(@NonNull TypedArray a) {
        float targetAspectRatioX = Math.abs(a.getFloat(R.styleable.ucrop_UCropView_ucrop_aspect_ratio_x, DEFAULT_ASPECT_RATIO));
        float targetAspectRatioY = Math.abs(a.getFloat(R.styleable.ucrop_UCropView_ucrop_aspect_ratio_y, DEFAULT_ASPECT_RATIO));

        if (targetAspectRatioX == SOURCE_IMAGE_ASPECT_RATIO || targetAspectRatioY == SOURCE_IMAGE_ASPECT_RATIO) {
            mTargetAspectRatio = SOURCE_IMAGE_ASPECT_RATIO;
        } else {
            mTargetAspectRatio = targetAspectRatioX / targetAspectRatioY;
        }
    }

    /**
     * 이 Runnable은 이미지를 움직이게 하여 자르기 범위를 완전히 채울 수 있다.
     * 주어진 값은 애니메이션 시간동안 채워진다.
     * Runnable은 {@link #cancelAllAnimations()}메소드 혹은 {@link WrapCropBoundsRunnable} 메소드 내의
     * 특정 조건이 트리거 될 때 종료될 수 있다.
     * */
    private static class WrapCropBoundsRunnable implements Runnable {

        private final WeakReference<CropImageView> mCropImageView;

        private final long mDurationMs, mStartTime;
        private final float mOldX, mOldY;
        private final float mCenterDiffX, mCenterDiffY;
        private final float mOldScale;
        private final float mDeltaScale;
        private final boolean mWillBeImageInBoundsAfterTranslate;

        public WrapCropBoundsRunnable(CropImageView cropImageView,
                                      long durationMs,
                                      float oldX, float oldY,
                                      float centerDiffX, float centerDiffY,
                                      float oldScale, float deltaScale,
                                      boolean willBeImageInBoundsAfterTranslate) {

            mCropImageView = new WeakReference<>(cropImageView);

            mDurationMs = durationMs;
            mStartTime = System.currentTimeMillis();
            mOldX = oldX;
            mOldY = oldY;
            mCenterDiffX = centerDiffX;
            mCenterDiffY = centerDiffY;
            mOldScale = oldScale;
            mDeltaScale = deltaScale;
            mWillBeImageInBoundsAfterTranslate = willBeImageInBoundsAfterTranslate;
        }

        @Override
        public void run() {
            CropImageView cropImageView = mCropImageView.get();
            if (cropImageView == null) {
                return;
            }

            long now = System.currentTimeMillis();
            float currentMs = Math.min(mDurationMs, now - mStartTime);

            float newX = CubicEasing.easeOut(currentMs, 0, mCenterDiffX, mDurationMs);
            float newY = CubicEasing.easeOut(currentMs, 0, mCenterDiffY, mDurationMs);
            float newScale = CubicEasing.easeInOut(currentMs, 0, mDeltaScale, mDurationMs);

            if (currentMs < mDurationMs) {
                cropImageView.postTranslate(newX - (cropImageView.mCurrentImageCenter[0] - mOldX), newY - (cropImageView.mCurrentImageCenter[1] - mOldY));
                if (!mWillBeImageInBoundsAfterTranslate) {
                    cropImageView.zoomInImage(mOldScale + newScale, cropImageView.mCropRect.centerX(), cropImageView.mCropRect.centerY());
                }
                if (!cropImageView.isImageWrapCropBounds()) {
                    cropImageView.post(this);
                }
            }
        }
    }

    /**
     * 이 Runnable은 이미지 확대/축소에 애니메이션을 적용하는데 사용된다.
     * 주어진 값은 애니메이션 시간동안 채워진다.
     * Runnable은 {@link #cancelAllAnimations()} 메소드로 종료할수도 있고,
     * {@link ZoomImageToPosition#run()}메소드 내의 특정 조건이 트리거 되었을 떄 종료할 수 있다.
     **/
    private static class ZoomImageToPosition implements Runnable {

        private final WeakReference<CropImageView> mCropImageView;

        private final long mDurationMs, mStartTime;
        private final float mOldScale;
        private final float mDeltaScale;
        private final float mDestX;
        private final float mDestY;

        public ZoomImageToPosition(CropImageView cropImageView,
                                   long durationMs,
                                   float oldScale, float deltaScale,
                                   float destX, float destY) {

            mCropImageView = new WeakReference<>(cropImageView);

            mStartTime = System.currentTimeMillis();
            mDurationMs = durationMs;
            mOldScale = oldScale;
            mDeltaScale = deltaScale;
            mDestX = destX;
            mDestY = destY;
        }

        @Override
        public void run() {
            CropImageView cropImageView = mCropImageView.get();
            if (cropImageView == null) {
                return;
            }

            long now = System.currentTimeMillis();
            float currentMs = Math.min(mDurationMs, now - mStartTime);
            float newScale = CubicEasing.easeInOut(currentMs, 0, mDeltaScale, mDurationMs);

            if (currentMs < mDurationMs) {
                cropImageView.zoomInImage(mOldScale + newScale, mDestX, mDestY);
                cropImageView.post(this);
            } else {
                cropImageView.setImageToWrapCropBounds();
            }
        }

    }
}
```