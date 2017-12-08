```
package com.yalantis.ucrop.model;

import android.os.Parcel;
import android.os.Parcelable;
import android.support.annotation.Nullable;

/**
 * Created by Oleksii Shliama [https://github.com/shliama] on 6/24/16.
 */
// 이미지 비율에 관한 클래스
public class AspectRatio implements Parcelable {

    @Nullable
    private final String mAspectRatioTitle;
    private final float mAspectRatioX;
    private final float mAspectRatioY;

    // 비율을 저장하는 함수
    public AspectRatio(@Nullable String aspectRatioTitle, float aspectRatioX, float aspectRatioY) {
        mAspectRatioTitle = aspectRatioTitle;	// 비율 이름
        mAspectRatioX = aspectRatioX;			// x 비율
        mAspectRatioY = aspectRatioY;			// y 비율
    }

    // 현재 dataPosition의 값을 각 속성에 저장하는 함수
    protected AspectRatio(Parcel in) {
        mAspectRatioTitle = in.readString();
        mAspectRatioX = in.readFloat();
        mAspectRatioY = in.readFloat();
    }

    // 저장된 dataPosition의 값으로 Parcel로 전개
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(mAspectRatioTitle);
        dest.writeFloat(mAspectRatioX);
        dest.writeFloat(mAspectRatioY);
    }

    @Override
    public int describeContents() {
        return 0;
    }

    // 새로운 Parcel을 만드는 함수
    public static final Creator<AspectRatio> CREATOR = new Creator<AspectRatio>() {
        @Override
        public AspectRatio createFromParcel(Parcel in) {
            return new AspectRatio(in);
        }

        @Override
        public AspectRatio[] newArray(int size) {
            return new AspectRatio[size];
        }
    };

    // 이하 멤버변수들을 반환하는 함수
    @Nullable
    public String getAspectRatioTitle() {
        return mAspectRatioTitle;
    }

    public float getAspectRatioX() {
        return mAspectRatioX;
    }

    public float getAspectRatioY() {
        return mAspectRatioY;
    }

}
```
