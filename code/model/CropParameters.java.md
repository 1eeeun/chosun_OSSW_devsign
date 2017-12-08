```
package com.yalantis.ucrop.model;

import android.graphics.Bitmap;

/**
 * Created by Oleksii Shliama [https://github.com/shliama] on 6/21/16.
 */
// 이미지 편집시 사용되는 인자들을 관리하는 클래스
public class CropParameters {

    private int mMaxResultImageSizeX, mMaxResultImageSizeY;

    private Bitmap.CompressFormat mCompressFormat;
    private int mCompressQuality;
    private String mImageInputPath, mImageOutputPath;
    private ExifInfo mExifInfo;


    public CropParameters(int maxResultImageSizeX, int maxResultImageSizeY,
                          Bitmap.CompressFormat compressFormat, int compressQuality,
                          String imageInputPath, String imageOutputPath, ExifInfo exifInfo) {
        mMaxResultImageSizeX = maxResultImageSizeX;	// 이미지 최대 사이즈
        mMaxResultImageSizeY = maxResultImageSizeY;
        mCompressFormat = compressFormat;			// 저장 포맷
        mCompressQuality = compressQuality;			// 저장 퀄리티
        mImageInputPath = imageInputPath;			// 불러온 이미지의 경로
        mImageOutputPath = imageOutputPath;			// 이미지 저장 경로
        mExifInfo = exifInfo;						// exif정보
    }
    // 이하 멤버변수 반환 함수
    public int getMaxResultImageSizeX() {
        return mMaxResultImageSizeX;
    }

    public int getMaxResultImageSizeY() {
        return mMaxResultImageSizeY;
    }
    
    public Bitmap.CompressFormat getCompressFormat() {
        return mCompressFormat;
    }
    
    public int getCompressQuality() {
        return mCompressQuality;
    }
    
    public String getImageInputPath() {
        return mImageInputPath;
    }

    public String getImageOutputPath() {
        return mImageOutputPath;
    }
    /
    public ExifInfo getExifInfo() {
        return mExifInfo;
    }

}

```
