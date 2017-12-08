```
package com.yalantis.ucrop.model;

/**
 * Created by Oleksii Shliama [https://github.com/shliama] on 6/21/16.
 */
// exif 정보에 관한 함수(exif는 교환 이미지 파일 형식, 디지털 카메라에서 이용되는 이미지 파일 포맷)
public class ExifInfo {

    private int mExifOrientation;
    private int mExifDegrees;
    private int mExifTranslation;

    public ExifInfo(int exifOrientation, int exifDegrees, int exifTranslation) {
        mExifOrientation = exifOrientation;	// 방향
        mExifDegrees = exifDegrees;			// 각도
        mExifTranslation = exifTranslation; // 이동
    }
    // 이하 3개, 각 멤버변수를 반환하는 함수 
    public int getExifOrientation() {
        return mExifOrientation;
    }

    public int getExifDegrees() {
        return mExifDegrees;
    }

    public int getExifTranslation() {
        return mExifTranslation;
    }

    // 이하 3개, 각 멤버변수 값을 수정하는 함수
    public void setExifOrientation(int exifOrientation) {
        mExifOrientation = exifOrientation;
    }

    public void setExifDegrees(int exifDegrees) {
        mExifDegrees = exifDegrees;
    }

    public void setExifTranslation(int exifTranslation) {
        mExifTranslation = exifTranslation;
    }

    // 이미지의 exif 정보가 저장된 exif 정보와 동일한지 검사하는 함수
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        ExifInfo exifInfo = (ExifInfo) o;

        if (mExifOrientation != exifInfo.mExifOrientation) return false;
        if (mExifDegrees != exifInfo.mExifDegrees) return false;
        return mExifTranslation == exifInfo.mExifTranslation;

    }

    // 임의의 데이터를 고정된 길이의 데이터로 매핑하는 함수
    @Override
    public int hashCode() {
        int result = mExifOrientation;
        result = 31 * result + mExifDegrees;
        result = 31 * result + mExifTranslation;
        return result;
    }

}

```
