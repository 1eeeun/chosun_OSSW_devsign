```
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

/**
 * Created by Oleksii Shliama (https://github.com/shliama).
 * <p/>
 * Builder class to ease Intent setup.
 */
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
     * 새로운 인텐트 빌더 작성, 소스 및 대상 이미지 경로 설정
     *
     * @param source      자르는 이미지를 위한 경로
     * @param destination 자른 이미지를 저장하기 위한 경로
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
     * 자르기 경계의 비율을 설정
     * User는 다른 비율 옵션이 있는 메뉴를 볼 수 없음
     *
     * @param x aspect ratio X
     * @param y aspect ratio Y
     */
    public UCrop withAspectRatio(float x, float y) {
        mCropOptionsBundle.putFloat(EXTRA_ASPECT_RATIO_X, x);
        mCropOptionsBundle.putFloat(EXTRA_ASPECT_RATIO_Y, y);
        return this;
    }


    // 원본 이미지의 너비와 높이를 기준으로 계산 된 자르기 범위의 비율을 설정
    public UCrop useSourceImageAspectRatio() {
        mCropOptionsBundle.putFloat(EXTRA_ASPECT_RATIO_X, 0);
        mCropOptionsBundle.putFloat(EXTRA_ASPECT_RATIO_Y, 0);
        return this;
    }

    /**
     * 잘린 이미지 최대 크기 설정
     *
     * @param width  잘린 이미지의 최대 폭
     * @param height 잘린 이미지의 최대 높이
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
     * 액티비티에서 crop 인텐트를 보냄
     *
     * @param activity 결과를 받을 액티비티
     */
    public void start(@NonNull Activity activity) {
        start(activity, REQUEST_CROP);
    }

    /**
     * 액티비티에서 crop 인텐트를 커스텀 요청 코드와 함께 보냄
     *
     * @param activity    결과를 받을 액티비티
     * @param requestCode requestCode for result
     */
    public void start(@NonNull Activity activity, int requestCode) {
        activity.startActivityForResult(getIntent(activity), requestCode);
    }

    /**
     * Fragment에서 crop 인텐트를 보냄
     *
     * @param fragment 결과를 받을 Fragment
     */
    public void start(@NonNull Context context, @NonNull Fragment fragment) {
        start(context, fragment, REQUEST_CROP);
    }

    /**
     * 지원 라이브러리 Fragment에서 crop 인텐트를 보냄
     *
     * @param fragment 결과를 받을 Fragment
     */
    public void start(@NonNull Context context, @NonNull android.support.v4.app.Fragment fragment) {
        start(context, fragment, REQUEST_CROP);
    }

    /**
     * custom request code로 인텐트를 보냄
     *
     * @param fragment    결과를 받을 Fragment
     * @param requestCode requestCode for result
     */
    @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    public void start(@NonNull Context context, @NonNull Fragment fragment, int requestCode) {
        fragment.startActivityForResult(getIntent(context), requestCode);
    }

    public void start(@NonNull Context context, @NonNull android.support.v4.app.Fragment fragment, int requestCode) {
        fragment.startActivityForResult(getIntent(context), requestCode);
    }

    /**
     * UCropActivity를 실행하기 위한 인텐트를 받음
     *
     * @return UCropActivity를 위한 인텐트를 반환
     */
    public Intent getIntent(@NonNull Context context) {
        mCropIntent.setClass(context, UCropActivity.class);
        mCropIntent.putExtras(mCropOptionsBundle);
        return mCropIntent;
    }

    /**
     * 결과 인텐트에서 자른 이미지 경로 검색
     *
     * @param intent 자르기 결과 인텐트
     */
    @Nullable
    public static Uri getOutput(@NonNull Intent intent) {
        return intent.getParcelableExtra(EXTRA_OUTPUT_URI);
    }

    /**
     * 잘린 이미지의 너비 검색
     *
     * @param intent 자르기 결과 인텐트
     */
    public static int getOutputImageWidth(@NonNull Intent intent) {
        return intent.getIntExtra(EXTRA_OUTPUT_IMAGE_WIDTH, -1);
    }

    /**
     * 잘린 이미지의 높이 검색
     *
     * @param intent 자르기 결과 인텐트
     */
    public static int getOutputImageHeight(@NonNull Intent intent) {
        return intent.getIntExtra(EXTRA_OUTPUT_IMAGE_HEIGHT, -1);
    }

    /**
     * 잘린 이미지의 비율 검색
     *
     * @param intent 자르기 결과 인텐트
     * @return 비율은 부동 소수점 값으로 반환
     */
    public static float getOutputCropAspectRatio(@NonNull Intent intent) {
        return intent.getParcelableExtra(EXTRA_OUTPUT_CROP_ASPECT_RATIO);
    }

    /**
     * 오류 검사
     *
     * @param result 자르기 결과 인텐트
     * @return 이미지 프로세싱 중에 무슨 일이 일어났는지 반환
     */
    @Nullable
    public static Throwable getError(@NonNull Intent result) {
        return (Throwable) result.getSerializableExtra(EXTRA_ERROR);
    }


    /**
     * 일반적으로 사용하지 않는 고급 구성을 설정하는데 도움이 되는 클래스
     * #withOptions(Options) 메소드와 함게 사용
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
         * android.graphics.Bitmap.CompressFormat 처리된 비트맵을 저장할 때 사용
         */
        public void setCompressionFormat(@NonNull Bitmap.CompressFormat format) {
            mOptionBundle.putString(EXTRA_COMPRESSION_FORMAT_NAME, format.name());
        }

        /**
         * Set compression quality [0-100] 처리된 비트맵을 저장할 때 사용
         */
        public void setCompressionQuality(@IntRange(from = 0) int compressQuality) {
            mOptionBundle.putInt(EXTRA_COMPRESSION_QUALITY, compressQuality);
        }

        /**
         * 각 탭에서 사용할 제스처 집합이 있는 경우 선택
         */
        public void setAllowedGestures(@UCropActivity.GestureTypes int tabScale,
                                       @UCropActivity.GestureTypes int tabRotate,
                                       @UCropActivity.GestureTypes int tabAspectRatio) {
            mOptionBundle.putIntArray(EXTRA_ALLOWED_GESTURES, new int[]{tabScale, tabRotate, tabAspectRatio});
        }

        /**
         * 최소 이미지 스케일에서 최대 이미지 스케일을 계산하는데 사용되는 승수를 설정하는 메소드
         *
         * @param maxScaleMultiplier - (minScale * maxScaleMultiplier) = maxScale
         */
        public void setMaxScaleMultiplier(@FloatRange(from = 1.0, fromInclusive = false) float maxScaleMultiplier) {
            mOptionBundle.putFloat(EXTRA_MAX_SCALE_MULTIPLIER, maxScaleMultiplier);
        }

        /**
         * 이미지의 자르기 경계를 래핑하기 위한 애니메이션 지속 시간을 설정하는 메소드
         *
         * @param durationMillis - 기간(milliseconds)
         */
        public void setImageToCropBoundsAnimDuration(@IntRange(from = 100) int durationMillis) {
            mOptionBundle.putInt(EXTRA_IMAGE_TO_CROP_BOUNDS_ANIM_DURATION, durationMillis);
        }

        /**
         * 입력 경로에서 디코드되어 뷰에서 사사용되는 비트맵의 너비와 높이에 대한 최대 크기 설정
         *
         * @param maxBitmapSize - 픽셀의 사이즈
         */
        public void setMaxBitmapSize(@IntRange(from = 100) int maxBitmapSize) {
            mOptionBundle.putInt(EXTRA_MAX_BITMAP_SIZE, maxBitmapSize);
        }

        /**
         * 자르기 범위 주변의 희미한 영역의 색을 설정 
         *
         * @param color - 자르기 범위 주변의 희미한 영역의 색(원하는 색)
         */
        public void setDimmedLayerColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_DIMMED_LAYER_COLOR, color);
        }

        /**
         * 흐리게 처리 된 레이어 내부 원 여부 설정
         * 
         * @param isCircle - 흐리게 처리 된 레이어의 내부에 원이 있도록 하려면 true로 설정
         */
        public void setCircleDimmedLayer(boolean isCircle) {
            mOptionBundle.putBoolean(EXTRA_CIRCLE_DIMMED_LAYER, isCircle);
        }

        /**
         * 이미지 위 자르기 프레임 사각형 표시 유무 설정
         * 
         * @param show - 이미지 위에 자르기 프레임 사각형을 표시하려면 true로 설정
         */
        public void setShowCropFrame(boolean show) {
            mOptionBundle.putBoolean(EXTRA_SHOW_CROP_FRAME, show);
        }

        /**
         * 자르기 프레임의 색 설정
         * 
         * @param color - 자르기 프레임의 색(원하는 색)
         */
        public void setCropFrameColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_CROP_FRAME_COLOR, color);
        }

        /**
         * 자르기 프레임 선의 너비 설정
         *
         * @param width - 자르기 프레임 선의 너비(픽셀 단위)
         */
        public void setCropFrameStrokeWidth(@IntRange(from = 0) int width) {
            mOptionBundle.putInt(EXTRA_CROP_FRAME_STROKE_WIDTH, width);
        }

        /**
         * 그리드/가이드라인 유무 설정
         *
         * @param show - 이미지 위에 자르기 그리드/가이드라인을 보이게 하고 싶으면 ture로 설정
         */
        public void setShowCropGrid(boolean show) {
            mOptionBundle.putBoolean(EXTRA_SHOW_CROP_GRID, show);
        }

        /**
         * 자르기 그리드의 row 개수 설정
         *
         * @param count - 자르기 그리의 row 개수
         */
        public void setCropGridRowCount(@IntRange(from = 0) int count) {
            mOptionBundle.putInt(EXTRA_CROP_GRID_ROW_COUNT, count);
        }

        /**
         * 자르기 그리드의 column 개수 설정
         *
         * @param count - 자르기 그리의 column 개수
         */
        public void setCropGridColumnCount(@IntRange(from = 0) int count) {
            mOptionBundle.putInt(EXTRA_CROP_GRID_COLUMN_COUNT, count);
        }

        /**
         * 그리드/가이드라인 색상 설정
         *
         * @param color - 그리드/가이드라인 색상
         */
        public void setCropGridColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_CROP_GRID_COLOR, color);
        }

        /**
         * 그리드 선의 너비 설정
         *
         * @param width - 원하는 자르기 그리드 선의 너비(픽셀 단위)
         */
        public void setCropGridStrokeWidth(@IntRange(from = 0) int width) {
            mOptionBundle.putInt(EXTRA_CROP_GRID_STROKE_WIDTH, width);
        }

        /**
         * 툴바의 색상 설정
         *
         * @param color - 원하는 툴바의 색상
         */
        public void setToolbarColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_TOOL_BAR_COLOR, color);
        }

        /**
         * 상태바의 색상 설정
         *
         * @param color - 원하는 상태바의 색상
         */
        public void setStatusBarColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_STATUS_BAR_COLOR, color);
        }

        /**
         * 액티브, 위젯, 미들라인의 색 설정
         *
         * @param color - 원하는 액티브, 위젯, 미들라인의 색(기본 : 오렌지)
         */
        public void setActiveWidgetColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_UCROP_COLOR_WIDGET_ACTIVE, color);
        }

        /**
         * 툴바의 텍스트와 버튼 색 설정
         *
         * @param color - 원하는 툴바 텍스트와 버튼 색(기본 : 다크 오렌지)
         */
        public void setToolbarWidgetColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_UCROP_WIDGET_COLOR_TOOLBAR, color);
        }

        /**
         * 툴바 타이들의 텍스트 설정
         *
         * @param text - 원하는 툴바 타이틀
         */
        public void setToolbarTitle(@Nullable String text) {
            mOptionBundle.putString(EXTRA_UCROP_TITLE_TEXT_TOOLBAR, text);
        }

        /**
         * 툴바 왼쪽의 취소 아이콘 설정
         *
         * @param drawable - 툴바 왼쪽의 취소 아이콘
         */
        public void setToolbarCancelDrawable(@DrawableRes int drawable) {
            mOptionBundle.putInt(EXTRA_UCROP_WIDGET_CANCEL_DRAWABLE, drawable);
        }

        /**
         * 툴바 오른쪽의 자르기 아이콘 설정
         *
         * @param drawable - 툴바 오른쪽의 자르기 아이콘
         */
        public void setToolbarCropDrawable(@DrawableRes int drawable) {
            mOptionBundle.putInt(EXTRA_UCROP_WIDGET_CROP_DRAWABLE, drawable);
        }

        /**
         * 로고를 채울 색 설정
         *
         * @param color - 로고를 채울 색(기본 : 다크 그레이)
         */
        public void setLogoColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_UCROP_LOGO_COLOR, color);
        }

        /**
         * 컨트롤 버튼 유무 설정
         *
         * @param hide - 컨트롤 버튼을 보이게 하려면 true로 설정(기본 : true)
         */
        public void setHideBottomControls(boolean hide) {
            mOptionBundle.putBoolean(EXTRA_HIDE_BOTTOM_CONTROLS, hide);
        }

        /**
         * 사용자가 자르기 범위 크기를 조정할 수 있는지 없는지 설정
         *
         * @param enabled - 사용자가 자르기 범위의 크기를 조정할 수 있게 하려면 true로 설정(기본 : false)
         */
        public void setFreeStyleCropEnabled(boolean enabled) {
            mOptionBundle.putBoolean(EXTRA_FREE_STYLE_CROP, enabled);
        }

        /**
         * 사용자가 사용할 수 있도록 원하는 가로, 세로 비율의 정렬 된 목록을 전달
         *
         * @param selectedByDefault - 디폴트로부터 선택되는 비율의 인덱스(0부터 시작)
         * @param aspectRatio       - 사용자가 사용할 수 잇는 비율 옵션 목ㄹ혹
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
         * root view에 적용되는 백그라운드 컬러 설정
         *
         * @param color - root view에 적용될 백그라운드 컬러
         */
        public void setRootViewBackgroundColor(@ColorInt int color) {
            mOptionBundle.putInt(EXTRA_UCROP_ROOT_VIEW_BACKGROUND_COLOR, color);
        }

        /**
         * 자르기 경계의 비율을 설정
         * User는 다른 비율 옵션이 있는 메뉴를 볼 수 없음
         *
         * @param x aspect ratio X
         * @param y aspect ratio Y
         */
        public void withAspectRatio(float x, float y) {
            mOptionBundle.putFloat(EXTRA_ASPECT_RATIO_X, x);
            mOptionBundle.putFloat(EXTRA_ASPECT_RATIO_Y, y);
        }

        /**
         * 원본 이미지의 너비와 높이를 기준으로 계산 된 자르기 범위의 비율 설정
         * User는 다른 비율 옵션이 있는 메뉴를 볼 수 없음
         */
        public void useSourceImageAspectRatio() {
            mOptionBundle.putFloat(EXTRA_ASPECT_RATIO_X, 0);
            mOptionBundle.putFloat(EXTRA_ASPECT_RATIO_Y, 0);
        }

        /**
         * 잘린 이미지의 최대 사이즈 설정
         *
         * @param width  잘린 이미지의 최대 너비
         * @param height 잘린 이미지의 최대 높이
         */
        public void withMaxResultSize(@IntRange(from = 100) int width, @IntRange(from = 100) int height) {
            mOptionBundle.putInt(EXTRA_MAX_SIZE_X, width);
            mOptionBundle.putInt(EXTRA_MAX_SIZE_Y, height);
        }

    }

}

```
