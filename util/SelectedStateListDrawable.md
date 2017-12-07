package com.yalantis.ucrop.util;

import android.graphics.PorterDuff;
import android.graphics.drawable.Drawable;
import android.graphics.drawable.StateListDrawable;

/**
 * Hack class to properly support state drawable back to Android 1.6
 */
public class SelectedStateListDrawable extends StateListDrawable {

    private int mSelectionColor;

    public SelectedStateListDrawable(Drawable drawable, int selectionColor) {   // Drawalbe의 상태 반환
        super();
        this.mSelectionColor = selectionColor;
        addState(new int[]{android.R.attr.state_selected}, drawable);   // state_selected의 특정 상태값 리턴시 drrawable 이라는 Drawable 객체 사용
        addState(new int[]{}, drawable);
    }

    @Override
    protected boolean onStateChange(int[] states) { // 상태가 변하게 되었을 경우
        boolean isStatePressedInArray = false;
        for (int state : states) {
            if (state == android.R.attr.state_selected) {
                isStatePressedInArray = true;
            }
        }
        if (isStatePressedInArray) {
            super.setColorFilter(mSelectionColor, PorterDuff.Mode.SRC_ATOP);    // 해당 영역에 포함되지 않은 원본 픽셀을 삭제
        } else {
            super.clearColorFilter();   // 필터 삭제
        }
        return super.onStateChange(states);
    }

    @Override
    public boolean isStateful() {
        return true;
    }

}
