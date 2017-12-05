package com.yalantis.ucrop.util;

import android.support.annotation.NonNull;
import android.view.MotionEvent;

public class RotationGestureDetector {

    private static final int INVALID_POINTER_INDEX = -1;

    private float fX, fY, sX, sY;

    private int mPointerIndex1, mPointerIndex2;
    private float mAngle;
    private boolean mIsFirstTouch;

    private OnRotationGestureListener mListener;

    public RotationGestureDetector(OnRotationGestureListener listener) {
        mListener = listener;
        mPointerIndex1 = INVALID_POINTER_INDEX;
        mPointerIndex2 = INVALID_POINTER_INDEX;
    }

    public float getAngle() {
        return mAngle;
    }

    public boolean onTouchEvent(@NonNull MotionEvent event) {
        switch (event.getActionMasked()) {  // 제스처 모션 이벤트
            case MotionEvent.ACTION_DOWN:   // 터치한 경우
                sX = event.getX();  // X 좌표
                sY = event.getY();  // Y 좌표
                mPointerIndex1 = event.findPointerIndex(event.getPointerId(0)); // 실제 포인트의 인덱스
                mAngle = 0;
                mIsFirstTouch = true;
                break;
            case MotionEvent.ACTION_POINTER_DOWN:   // 두 개 이상의 포인트에 대한 터치
                fX = event.getX();  // X 좌표
                fY = event.getY();  // Y 좌표
                mPointerIndex2 = event.findPointerIndex(event.getPointerId(event.getActionIndex()));
                mAngle = 0;
                mIsFirstTouch = true;
                break;
            case MotionEvent.ACTION_MOVE:   // 터치한 상태에서 움직일 경우
                if (mPointerIndex1 != INVALID_POINTER_INDEX && mPointerIndex2 != INVALID_POINTER_INDEX && event.getPointerCount() > mPointerIndex2) {
                    float nfX, nfY, nsX, nsY;

                    // 멀티 터치 환경에서 움직이고 있는 X, Y 좌표값
                    nsX = event.getX(mPointerIndex1);
                    nsY = event.getY(mPointerIndex1);
                    nfX = event.getX(mPointerIndex2);
                    nfY = event.getY(mPointerIndex2);

                    if (mIsFirstTouch) {
                        mAngle = 0;
                        mIsFirstTouch = false;
                    } else {
                        calculateAngleBetweenLines(fX, fY, sX, sY, nfX, nfY, nsX, nsY); // 좌표 간의 거리 계산
                    }

                    if (mListener != null) {
                        mListener.onRotation(this);
                    }
                    fX = nfX;
                    fY = nfY;
                    sX = nsX;
                    sY = nsY;
                }
                break;
            case MotionEvent.ACTION_UP: // 터치한 상태에서 손을 뗄 경우
                mPointerIndex1 = INVALID_POINTER_INDEX;
                break;
            case MotionEvent.ACTION_POINTER_UP: // 두 개 이상의 포인트에서 손을 뗄 경우
                mPointerIndex2 = INVALID_POINTER_INDEX;
                break;
        }
        return true;
    }

    private float calculateAngleBetweenLines(float fx1, float fy1, float fx2, float fy2,
                                             float sx1, float sy1, float sx2, float sy2) {  // 멀티 터치 시 이동 좌표 계산
        return calculateAngleDelta(
                (float) Math.toDegrees((float) Math.atan2((fy1 - fy2), (fx1 - fx2))),
                (float) Math.toDegrees((float) Math.atan2((sy1 - sy2), (sx1 - sx2))));
    }

    private float calculateAngleDelta(float angleFrom, float angleTo) { // 멀티 터치 좌표 각도 계산
        mAngle = angleTo % 360.0f - angleFrom % 360.0f;

        if (mAngle < -180.0f) {
            mAngle += 360.0f;
        } else if (mAngle > 180.0f) {
            mAngle -= 360.0f;
        }

        return mAngle;
    }

    public static class SimpleOnRotationGestureListener implements OnRotationGestureListener {

        @Override
        public boolean onRotation(RotationGestureDetector rotationDetector) {
            return false;
        }
    }

    public interface OnRotationGestureListener {

        boolean onRotation(RotationGestureDetector rotationDetector);
    }

}