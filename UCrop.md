####com.yalantis.ycrop\view\

package com.yalantis.ucrop;

import android.annotation.TargetApi;
import android.app.Activity;
import android.app.Fragment;
import android.content.Context;
import android.content.Intent;
import android.graphics.Bitmap;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.os.Parcelable;
import android.support.annotation.ColorInt;
import android.support.annotation.DrawableRes;
import android.support.annotation.FloatRange;
import android.support.annotation.IntRange;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;

import com.yalantis.ucrop.model.AspectRatio;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Locale;


 **Created by Oleksii Shliama (https://github.com/shliama).**
 **인텐트 설정을 쉽게 하기 위한 Builder Class**
 
 
	public class UCrop {

    public static final int REQUEST_CROP = 69;
    public static final int RESULT_ERROR = 96;

    private static final String EXTRA_PREFIX = BuildConfig.APPLICATION_ID;

    public static final String EXTRA_INPUT_URI = EXTRA_PREFIX + ".InputUri";
    public static final String EXTRA_OUTPUT_URI = EXTRA_PREFIX + ".OutputUri";
    public static final String EXTRA_OUTPUT_CROP_ASPECT_RATIO = EXTRA_PREFIX + ".CropAspectRatio";
    public static final String EXTRA_OUTPUT_IMAGE_WIDTH = EXTRA_PREFIX + ".ImageWidth";
    public static final String EXTRA_OUTPUT_IMAGE_HEIGHT = EXTRA_PREFIX + ".ImageHeight";
    public static final String EXTRA_OUTPUT_OFFSET_X = EXTRA_PREFIX + ".OffsetX";
    public static final String EXTRA_OUTPUT_OFFSET_Y = EXTRA_PREFIX + ".OffsetY";
    public static final String EXTRA_ERROR = EXTRA_PREFIX + ".Error";

    public static final String EXTRA_ASPECT_RATIO_X = EXTRA_PREFIX + ".AspectRatioX";
    public static final String EXTRA_ASPECT_RATIO_Y = EXTRA_PREFIX + ".AspectRatioY";

    public static final String EXTRA_MAX_SIZE_X = EXTRA_PREFIX + ".MaxSizeX";
    public static final String EXTRA_MAX_SIZE_Y = EXTRA_PREFIX + ".MaxSizeY";

    private Intent mCropIntent;
    private Bundle mCropOptionsBundle;

    /**
     * 새로운 인덴트 빌더를 작성하고 소스 및 대상 이미지 Uri를 설정한다.
     *
     * @param source      이미지를 자르기 위한 Uri
     * @param destination 자른 이미지를 저장하기 위한 Uri
     */
    public static UCrop of(@NonNull Uri source, @NonNull Uri destination) {
        return new UCrop(source, destination);
    }

    private UCrop(@NonNull Uri source, @NonNull Uri destination) {
        mCropIntent = new Intent();
        mCropOptionsBundle = new Bundle();
        mCropOptionsBundle.putParcelable(EXTRA_INPUT_URI, source);
        mCropOptionsBundle.putParcelable(EXTRA_OUTPUT_URI, destination);
    }

    /**
     * 자르기 경계의 종횡비를 설정한다.
     * 사용자는 다른 비율 옵션이 있는 메뉴를 볼 수 없다.
     *
     * @param x X의 종횡비
     * @param y Y의 종횡비
     */
    public UCrop withAspectRatio(float x, float y) {
        mCropOptionsBundle.putFloat(EXTRA_ASPECT_RATIO_X, x);
        mCropOptionsBundle.putFloat(EXTRA_ASPECT_RATIO_Y, y);
        return this;
    }

    /**
     * 소스 이미지의 높이와 너비에서 계산된 자르기 경계의 종횡비를 설정한다.
     * 사용자는 다른 비율 옵션이 있는 메뉴를 볼 수 없다.
     */
    public UCrop useSourceImageAspectRatio() {
        mCropOptionsBundle.putFloat(EXTRA_ASPECT_RATIO_X, 0);
        mCropOptionsBundle.putFloat(EXTRA_ASPECT_RATIO_Y, 0);
        return this;
    }

    /**
     * 잘린 이미지 결과의 최대값을 설정한다.
     *
     * @param width  잘린 이미지 너비의 최대값
     * @param height 잘린 이미지 높이의 최대값
     */
    public UCrop withMaxResultSize(@IntRange(from = 100) int width, @IntRange(from = 100) int height) {
        mCropOptionsBundle.putInt(EXTRA_MAX_SIZE_X, width);
        mCropOptionsBundle.putInt(EXTRA_MAX_SIZE_Y, height);
        return this;
    }

    public UCrop withOptions(@NonNull Options options) {
        mCropOptionsBundle.putAll(options.getOptionBundle());
        return this;
    }

    /**
     * Activity로부터 받은 자르기 인텐트를 전송한다.
     *
     * @param activity 결과를 받기 위한 Activity
     */
    public void start(@NonNull Activity activity) {
        start(activity, REQUEST_CROP);
    }

    /**
     * 커스텀 요청 코드와 함께 Activity에서 자르기 인텐트를 전송한다.
     *
     * @param activity    결과를 받기 위한 Activity
     * @param requestCode 결과에 대한 resultCode
     */
    public void start(@NonNull Activity activity, int requestCode) {
        activity.startActivityForResult(getIntent(activity), requestCode);
    }

    /**
     * Fragment에서 자르기 인텐트를 전송한다.
     *
     * @param fragment 결과를 받기 위한 Fragment
     */
    public void start(@NonNull Context context, @NonNull Fragment fragment) {
        start(context, fragment, REQUEST_CROP);
    }

    /**
     * 지원하는 library Fragment로부터 온 자르기 인텐트를 전송한다.
     *
     * @param fragment 결과를 받기 위한 Fragment
     */
    public void start(@NonNull Context context, @NonNull android.support.v4.app.Fragment fragment) {
        start(context, fragment, REQUEST_CROP);
    }

    /**
     * 커스텀 요청 코드로 자르기 인텐트를 전송한다.
     *
     * @param fragment    결과를 받을 Fragment
     * @param requestCode 결과에 대한 requestCode
     */
    @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    public void start(@NonNull Context context, @NonNull Fragment fragment, int requestCode) {
        fragment.startActivityForResult(getIntent(context), requestCode);
    }

    /**
     * 커스텀 요성 코드로 자르기 인텐트를 전송한다.
     *
     * @param fragment    결과를 받을 Fragment
     * @param requestCode 결과에대한 requestCode
     */
    public void start(@NonNull Context context, @NonNull android.support.v4.app.Fragment fragment, int requestCode) {
        fragment.startActivityForResult(getIntent(context), requestCode);
    }

    /**
     * {@link UCropActivity}를 시작하기 위해 인텐트를 가져온다.
     *
     * @return {@link UCropActivity}의 인텐트
     */
    public Intent getIntent(@NonNull Context context) {
        mCropIntent.setClass(context, UCropActivity.class);
        mCropIntent.putExtras(mCropOptionsBundle);
        return mCropIntent;
    }

    /**
     * 결과 인텐트에서 자른 이미지 Uri를 검색한다.
     *
     * @param intent 자르기 결과 인텐트
     */
    @Nullable
    public static Uri getOutput(@NonNull Intent intent) {
        return intent.getParcelableExtra(EXTRA_OUTPUT_URI);
    }

    /**
     * 자른 이미지의 너비를 검색한다.
     *
     * @param intent 자르기 결과 인텐트
     */
    public static int getOutputImageWidth(@NonNull Intent intent) {
        return intent.getIntExtra(EXTRA_OUTPUT_IMAGE_WIDTH, -1);
    }

    /**
     * 자른 이미지의 높이를 검색한다.
     *
     * @param intent 자르기 결과 인텐트
     */
    public static int getOutputImageHeight(@NonNull Intent intent) {
        return intent.getIntExtra(EXTRA_OUTPUT_IMAGE_HEIGHT, -1);
    }

    /**
     * 결과 인텐트에서 자른 이미지 종횡비를 검색한다.
     *
     * @param intent 자르기 결과 인텐트
     * @return 종횡비를 부동 소수점 값(x:y)로 지정 - 1:1에서는 1, 4:3에서는 4/3이 된다.
     */
    public static float getOutputCropAspectRatio(@NonNull Intent intent) {
        return intent.getParcelableExtra(EXTRA_OUTPUT_CROP_ASPECT_RATIO);
    }

    /**
     * 결과 인텐트에서 오류를 검색한다.
     *
     * @param result 자르기 결과 인텐트
     * @return 이미지 처리중에 발생할 수 있는 Throwable
     */
    @Nullable
    public static Throwable getError(@NonNull Intent result) {
        return (Throwable) result.getSerializableExtra(EXTRA_ERROR);
    }


    /**
     * 일반적으로 사용되지 않는 고급 구성을 설정하는데 도움이 되는 클래스
     * {@link #withOptions(Options)} 메소드와 함께 사용하라.
     */
    public static class Options {

        public static final String EXTRA_COMPRESSION_FORMAT_NAME = EXTRA_PREFIX + ".CompressionFormatName";
        public static final String EXTRA_COMPRESSION_QUALITY = EXTRA_PREFIX + ".CompressionQuality";

        public static final String EXTRA_ALLOWED_GESTURES = EXTRA_PREFIX + ".AllowedGestures";

        public static final String EXTRA_MAX_BITMAP_SIZE = EXTRA_PREFIX + ".MaxBitmapSize";
        public static final String EXTRA_MAX_SCALE_MULTIPLIER = EXTRA_PREFIX + ".MaxScaleMultiplier";
        public static final String EXTRA_IMAGE_TO_CROP_BOUNDS_ANIM_DURATION = EXTRA_PREFIX + ".ImageToCropBoundsAnimDuration";

        public static final String EXTRA_DIMMED_LAYER_COLOR = EXTRA_PREFIX + ".DimmedLayerColor";
        public static final String EXTRA_CIRCLE_DIMMED_LAYER = EXTRA_PREFIX + ".CircleDimmedLayer";

        public static final String EXTRA_SHOW_CROP_FRAME = EXTRA_PREFIX + ".ShowCropFrame";
        public static final String EXTRA_CROP_FRAME_COLOR = EXTRA_PREFIX + ".CropFrameColor";
        public static final String EXTRA_CROP_FRAME_STROKE_WIDTH = EXTRA_PREFIX + ".CropFrameStrokeWidth";

        public static final String EXTRA_SHOW_CROP_GRID = EXTRA_PREFIX + ".ShowCropGrid";
        public static final String EXTRA_CROP_GRID_ROW_COUNT = EXTRA_PREFIX + ".CropGridRowCount";
        public static final String EXTRA_CROP_GRID_COLUMN_COUNT = EXTRA_PREFIX + ".CropGridColumnCount";
        public static final String EXTRA_CROP_GRID_COLOR = EXTRA_PREFIX + ".CropGridColor";
        public static final String EXTRA_CROP_GRID_STROKE_WIDTH = EXTRA_PREFIX + ".CropGridStrokeWidth";

        public static final String EXTRA_TOOL_BAR_COLOR = EXTRA_PREFIX + ".ToolbarColor";
        public static final String EXTRA_STATUS_BAR_COLOR = EXTRA_PREFIX + ".StatusBarColor";
        public static final String EXTRA_UCROP_COLOR_WIDGET_ACTIVE = EXTRA_PREFIX + ".UcropColorWidgetActive";

        public static final String EXTRA_UCROP_WIDGET_COLOR_TOOLBAR = EXTRA_PREFIX + ".UcropToolbarWidgetColor";
        public static final String EXTRA_UCROP_TITLE_TEXT_TOOLBAR = EXTRA_PREFIX + ".UcropToolbarTitleText";
        public static final String EXTRA_UCROP_WIDGET_CANCEL_DRAWABLE = EXTRA_PREFIX + ".UcropToolbarCancelDrawable";
        public static final String EXTRA_UCROP_WIDGET_CROP_DRAWABLE = EXTRA_PREFIX + ".UcropToolbarCropDrawable";

        public static final String EXTRA_UCROP_LOGO_COLOR = EXTRA_PREFIX + ".UcropLogoColor";

        public static final String EXTRA_HIDE_BOTTOM_CONTROLS = EXTRA_PREFIX + ".HideBottomControls";
        public static final String EXTRA_FREE_STYLE_CROP = EXTRA_PREFIX + ".FreeStyleCrop";

        public static final String EXTRA_ASPECT_RATIO_SELECTED_BY_DEFAULT = EXTRA_PREFIX + ".AspectRatioSelectedByDefault";
        public static final String EXTRA_ASPECT_RATIO_OPTIONS = EXTRA_PREFIX + ".AspectRatioOptions";

        public static final String EXTRA_UCROP_ROOT_VIEW_BACKGROUND_COLOR = EXTRA_PREFIX + ".UcropRootViewBackgroundColor";


        private final Bundle mOptionBundle;

        public Options() {
            mOptionBundle = new Bundle();
        }

        @NonNull
        public Bundle getOptionBundle() {
            return mOptionBundle;
        }

        /**
         * 결과 비트맵을 저장하는데 사용할 {@link android.graphics.Bitmap.CompressFormat} 중 하나를 설정한다.
         * */
        public void setCompressionFormat(@NonNull Bitmap.CompressFormat format) {
            mOptionBundle.putString(EXTRA_COMPRESSION_FORMAT_NAME, format.name());
        }

        /**
         * 결과 비트맵을 저장하는데 사용될 압축 품질 [0-100]을 설정한다.
         */
        public void setCompressionQuality(@IntRange(from = 0) int compressQuality) {
            mOptionBundle.putInt(EXTRA_COMPRESSION_QUALITY, compressQuality);
        }

        /**
         * 각 탭에서 사용할 제스처 집합을 선택하라. - 있는 경우
         */
        public void setAllowedGestures(@UCropActivity.GestureTypes int tabScale,
                                       @UCropActivity.GestureTypes int tabRotate,
                                       @UCropActivity.GestureTypes int tabAspectRatio) {
            mOptionBundle.putIntArray(EXTRA_ALLOWED_GESTURES, new int[]{tabScale, tabRotate, tabAspectRatio});
        }

        /**
         * 이미지 범위의 최소값에서 최대값을 계산하는데 사용되는 승수를 설정한다.
         * @param maxScaleMultiplier - (범위의 최소값 * 최대 범위의 승수) = 범위의 최대값
         */
        public void setMaxScaleMultiplier(@FloatRange(from = 1.0, fromInclusive = false) float maxScaleMultiplier) {
            mOptionBundle.putFloat(EXTRA_MAX_SCALE_MULTIPLIER, maxScaleMultiplier);
        }

        /**
         * 이미지의 자르기 경계를 채우기 위한 애니메이션의 지속시간을 설정한다.
         *
         * @param durationMillis - 기간 (milliseconds 단위)
         */
        public void setImageToCropBoundsAnimDuration(@IntRange(from = 100) int durationMillis) {
            mOptionBundle.putInt(EXTRA_IMAGE_TO_CROP_BOUNDS_ANIM_DURATION, durationMillis);
        }

        /**
         * 입력 Uri에서 디코드되어 뷰에서 사용되는 비트맵의 너비와 높이에 대한 최대값을 설정한다.
         * @param maxBitmapSize - 크기 (pixels 단위)
         */
        public void setMaxBitmapSize(@IntRange(from = 100) int maxBitmapSize) {
            mOptionBundle.putInt(EXTRA_MAX_BITMAP_SIZE, maxBitmapSize);
        }

        /**
         * @param color - 자르기 범위 주변의 희미한 영역의 원하는 색상
         */
        public void setDimmedLayerColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_DIMMED_LAYER_COLOR, color);
        }

        /**
         * @param isCircle - 희미한 레이어의 내부에 원이 있도록 하려면 true로 설정하라.
         */
        public void setCircleDimmedLayer(boolean isCircle) {
            mOptionBundle.putBoolean(EXTRA_CIRCLE_DIMMED_LAYER, isCircle);
        }

        /**
         * @param show - 이미지 위에 자르기 프레임 사각형을 표시하려면 true로 설정하라.
         */
        public void setShowCropFrame(boolean show) {
            mOptionBundle.putBoolean(EXTRA_SHOW_CROP_FRAME, show);
        }

        /**
         * @param color - 원하는 자르기 프레임의 색상
         */
        public void setCropFrameColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_CROP_FRAME_COLOR, color);
        }

        /**
         * @param width - 원하는 자르기 프레임 선의 너비(pixels 단위)
         */
        public void setCropFrameStrokeWidth(@IntRange(from = 0) int width) {
            mOptionBundle.putInt(EXTRA_CROP_FRAME_STROKE_WIDTH, width);
        }

        /**
         * @param show - 이미지 위의 자르기 그리드/가이드라인을 보려면 true로 설정하라.
         */
        public void setShowCropGrid(boolean show) {
            mOptionBundle.putBoolean(EXTRA_SHOW_CROP_GRID, show);
        }

        /**
         * @param count - 자르기 그리드의 행 개수
         */
        public void setCropGridRowCount(@IntRange(from = 0) int count) {
            mOptionBundle.putInt(EXTRA_CROP_GRID_ROW_COUNT, count);
        }

        /**
         * @param count - 자르기 그리드의 열 개수
         */
        public void setCropGridColumnCount(@IntRange(from = 0) int count) {
            mOptionBundle.putInt(EXTRA_CROP_GRID_COLUMN_COUNT, count);
        }

        /**
         * @param color - 원하는 자르기 그리드/가이드라인의 색상
         */
        public void setCropGridColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_CROP_GRID_COLOR, color);
        }

        /**
         * @param width - 원하는 자르기 그리드 선의 너비(pixels 단위)
         */
        public void setCropGridStrokeWidth(@IntRange(from = 0) int width) {
            mOptionBundle.putInt(EXTRA_CROP_GRID_STROKE_WIDTH, width);
        }

        /**
         * @param color - 툴바의 원하는 해상도 색상
         */
        public void setToolbarColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_TOOL_BAR_COLOR, color);
        }

        /**
         * @param color - 상태바의 원하는 채워진 색상
         */
        public void setStatusBarColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_STATUS_BAR_COLOR, color);
        }

        /**
         * @param color - 활성화 및 선택된 위젯과 프로그레스 휠 중간 라인의 원하는 채워진 색상(기본값은 주황색)
         */
        public void setActiveWidgetColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_UCROP_COLOR_WIDGET_ACTIVE, color);
        }

        /**
         * @param color - 툴바 텍스트 및 버튼의 원하는 해상도 색상(기본값은 진한 주황색)
         */
        public void setToolbarWidgetColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_UCROP_WIDGET_COLOR_TOOLBAR, color);
        }

        /**
         * @param text - 툴바 타이틀에 대한 원하는 텍스트
         */
        public void setToolbarTitle(@Nullable String text) {
            mOptionBundle.putString(EXTRA_UCROP_TITLE_TEXT_TOOLBAR, text);
        }

        /**
         * @param drawable - desired drawable for the Toolbar left cancel icon
         */
        public void setToolbarCancelDrawable(@DrawableRes int drawable) {
            mOptionBundle.putInt(EXTRA_UCROP_WIDGET_CANCEL_DRAWABLE, drawable);
        }

        /**
         * @param drawable - 툴바 오른쪽 자르기 아이콘에 필요한 drawable
         */
        public void setToolbarCropDrawable(@DrawableRes int drawable) {
            mOptionBundle.putInt(EXTRA_UCROP_WIDGET_CROP_DRAWABLE, drawable);
        }

        /**
         * @param color - 원하는 채워진 로고 색상(기본값은 어두운 회색)
         */
        public void setLogoColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_UCROP_LOGO_COLOR, color);
        }

        /**
         * @param hide - 하단 컨트롤을 숨기려면 true로 설정하라.(기본적으로 표시됨)
         */
        public void setHideBottomControls(boolean hide) {
            mOptionBundle.putBoolean(EXTRA_HIDE_BOTTOM_CONTROLS, hide);
        }

        /**
         * @param enabled - 자르기 경계의 크기를 사용자가 조정할 수 있게 하려면 true로 설정하라.(기본값적으로 비활성화  됨)
         */
        public void setFreeStyleCropEnabled(boolean enabled) {
            mOptionBundle.putBoolean(EXTRA_FREE_STYLE_CROP, enabled);
        }

        /**
         * 사용자가 사용할 수 있도록 원하는 종횡비의 정렬된 목록을 전달한다.
         *
         * @param selectedByDefault - 기본값으로 선택된 종횡비 인덱스 옵션(0부터 시작).
         * @param aspectRatio       - 사용자가 사용할 수 있는 종횡비 옵션 목록
         */
        public void setAspectRatioOptions(int selectedByDefault, AspectRatio... aspectRatio) {
            if (selectedByDefault > aspectRatio.length) {
                throw new IllegalArgumentException(String.format(Locale.US,
                        "Index [selectedByDefault = %d] cannot be higher than aspect ratio options count [count = %d].",
                        selectedByDefault, aspectRatio.length));
            }
            mOptionBundle.putInt(EXTRA_ASPECT_RATIO_SELECTED_BY_DEFAULT, selectedByDefault);
            mOptionBundle.putParcelableArrayList(EXTRA_ASPECT_RATIO_OPTIONS, new ArrayList<Parcelable>(Arrays.asList(aspectRatio)));
        }

        /**
         * @param color - 루트 뷰에 적용되어야 하는 원하는 배경색
         */
        public void setRootViewBackgroundColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_UCROP_ROOT_VIEW_BACKGROUND_COLOR, color);
        }

        /**
         * 자르기 경계의 종횡비를 설정한다.
         * 사용자는 다른 비율 옵션이 있는 메뉴를 볼 수 없다.
         *
         * @param x X의 종횡비
         * @param y Y의 종횡비
         */
        public void withAspectRatio(float x, float y) {
            mOptionBundle.putFloat(EXTRA_ASPECT_RATIO_X, x);
            mOptionBundle.putFloat(EXTRA_ASPECT_RATIO_Y, y);
        }

        /**
         * 원본 이미지 너비와 높이를 기준으로 계산된 자르기 범위의 종횡비를 설정한다.
         * 사용자는 다른 비율 옵션이 있는 메뉴를 볼 수 없다.
         */
        public void useSourceImageAspectRatio() {
            mOptionBundle.putFloat(EXTRA_ASPECT_RATIO_X, 0);
            mOptionBundle.putFloat(EXTRA_ASPECT_RATIO_Y, 0);
        }

        /**
         * 잘린 이미지 결과의 최대 사이즈를 설정한다.
         *
         * @param width  잘린 이미지 너비의 최대값
         * @param height 잘린 이미지 높이의 최대값
         */
        public void withMaxResultSize(@IntRange(from = 100) int width, @IntRange(from = 100) int height) {
            mOptionBundle.putInt(EXTRA_MAX_SIZE_X, width);
            mOptionBundle.putInt(EXTRA_MAX_SIZE_Y, height);
        }

    }

}
