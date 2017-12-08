```
package com.yalantis.ucrop.util;

public final class CubicEasing {

    public static float easeOut(float time, float start, float end, float duration) {
        // 0의 속도에서 모션을 시작한 다음 실행하면서 점점 빨리지게 합니다.
        return end * ((time = time / duration - 1.0f) * time * time + 1.0f) + start;
    }

    public static float easeIn(float time, float start, float end, float duration) {
        // 모션을 빠르게 시작한 다음 실행하면서 0까지 점점 느려지게 합니다.
        return end * (time /= duration) * time * time + start;
    }

    public static float easeInOut(float time, float start, float end, float duration) {
        // 모션을 결합하여 모션을 속도 0에서 시작하고 가속한 다음 0으로 속도를 늦춥니다.
        return (time /= duration / 2.0f) < 1.0f ? end / 2.0f * time * time * time + start : end / 2.0f * ((time -= 2.0f) * time * time + 2.0f) + start;
    }

}
```