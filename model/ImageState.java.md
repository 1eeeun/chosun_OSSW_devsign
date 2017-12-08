```
package com.yalantis.ucrop.model;

import android.graphics.RectF;

/**
 * Created by Oleksii Shliama [https://github.com/shliama] on 6/21/16.
 */
// 이미지 상태를 반환하는 클래스
public class ImageState {

    private RectF mCropRect;
    private RectF mCurrentImageRect;

    private float mCurrentScale, mCurrentAngle;

    public ImageState(RectF cropRect, RectF currentImageRect, float currentScale, float currentAngle) {
        mCropRect = cropRect;                   // 자르기 상자
        mCurrentImageRect = currentImageRect;   // 이미지
        mCurrentScale = currentScale;           // 현재 이미지 크기
        mCurrentAngle = currentAngle;           // 현재 이미지 각도
    }
    
    // 이하 각 속성 반환 함수
    public RectF getCropRect() {
        return mCropRect;
    }

    public RectF getCurrentImageRect() {
        return mCurrentImageRect;
    }

    public float getCurrentScale() {
        return mCurrentScale;
    }

    public float getCurrentAngle() {
        return mCurrentAngle;
    }
}
```
