/*
 * Copyright (C) 2008 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.yalantis.ucrop.util;

import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.ColorFilter;
import android.graphics.Paint;
import android.graphics.PixelFormat;
import android.graphics.drawable.Drawable;

public class FastBitmapDrawable extends Drawable {

    private final Paint mPaint = new Paint(Paint.FILTER_BITMAP_FLAG);   // mPaint 오브젝트 생성

    private Bitmap mBitmap;
    private int mAlpha;
    private int mWidth, mHeight;

    public FastBitmapDrawable(Bitmap b) {
        mAlpha = 255;
        setBitmap(b);
    }

    @Override
    public void draw(Canvas canvas) {   // mBitmap 불러오기
        if (mBitmap != null && !mBitmap.isRecycled()) {
            canvas.drawBitmap(mBitmap, null, getBounds(), mPaint);
        }
    }

    @Override
    public void setColorFilter(ColorFilter cf) {    // 이미지 필터
        mPaint.setColorFilter(cf);
    }

    @Override
    public int getOpacity() {   // 투명효과
        return PixelFormat.TRANSLUCENT;
    }

    public void setFilterBitmap(boolean filterBitmap) { // 비트맵 필터 설정
        mPaint.setFilterBitmap(filterBitmap);
    }

    public int getAlpha() { // Alpha 값 얻기
        return mAlpha;
    }

    @Override
    public void setAlpha(int alpha) {   // Alpha 값 설정
        mAlpha = alpha;
        mPaint.setAlpha(alpha);
    }

    @Override
    public int getIntrinsicWidth() {    // 고유폭 얻기
        return mWidth;
    }

    @Override
    public int getIntrinsicHeight() {   // 고유높이 얻기
        return mHeight;
    }

    @Override
    public int getMinimumWidth() {  // 최소 폭 얻기
        return mWidth;
    }

    @Override
    public int getMinimumHeight() { // 최소 높이 얻기
        return mHeight;
    }

    public Bitmap getBitmap() { // 비트맵 얻기
        return mBitmap;
    }

    public void setBitmap(Bitmap b) {
        mBitmap = b;
        if (b != null) {    // 폭과 넓이 할당
            mWidth = mBitmap.getWidth();
            mHeight = mBitmap.getHeight();
        } else {
            mWidth = mHeight = 0;
        }
    }

}
