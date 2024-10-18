<template>
    <div class="container">
        <div class="outer-circle">
            <div class="cursor" :style="{'--angle': cursorPosition}"></div>
            <div class="aim" :class="{'intersecting': isIntersecting, 'game-over': gameOver}" :style="{'--angle': aimUpperSidePosition, '--delta': skewDelta}"></div>
            <div class="inner-circle"></div>
        </div>
        <div class="score">
            <div v-for="i in 6" :key="i" class="score-dot" :class="{'success': i <= successStrikes}"></div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue';

const props = defineProps<{difficulty: 'easy' | 'hard'}>();
const emit = defineEmits(['success', 'fail']);

const gameStarted = ref(false);
const timerStarted = ref(false);
const cursorPosition = ref(180);
const startTime = ref(0);
const lastFrameTime = ref(0);
const aimUpperSidePosition = ref(0);
const successStrikes = ref(0);
const gameOver = ref(false);
const isPaused = ref(false);
const FULL_ROUND_TIME = 1250;

const difficultyMultiplier = computed(() => {
    return props.difficulty === 'easy' ? 0.4 : 0.8;
})

const hitRangeDelta = computed(() => {
    return 120 / (1 + (Math.min(successStrikes.value, 5) * difficultyMultiplier.value));
})

const skewDelta = computed(() => {
    return (90 - hitRangeDelta.value) / 2;
})

const twoAnglesDiff = (a: number, b: number) => {
    return Math.abs((a - b + 540) % 360 - 180);
}

const isIntersecting = computed(() => {
    return twoAnglesDiff(cursorPosition.value, aimUpperSidePosition.value) < hitRangeDelta.value / 2;
})

const startTimer = () => {
    timerStarted.value = true;
    startTime.value = Date.now();
    lastFrameTime.value = Date.now();
    requestAnimationFrame(tickCursor);
}

const stopTimer = () => {
    timerStarted.value = false;
}

const tickCursor = () => {
    if (!timerStarted.value) {
        return;
    }
    const currentTime = Date.now();
    const lastFrameDelta = (currentTime - lastFrameTime.value);

    cursorPosition.value += lastFrameDelta / FULL_ROUND_TIME * 360;
    cursorPosition.value %= 360;
    lastFrameTime.value = currentTime;

    requestAnimationFrame(tickCursor);
}

const proceedPlayerInteraction = () => {
    if (isPaused.value) {
        return;
    }

    if (gameOver.value) {
        gameOver.value = false;
        successStrikes.value = 0;
        aimUpperSidePosition.value = Math.random() * 360;
        cursorPosition.value = 180;
        timerStarted.value = false;
    } else if (timerStarted.value) {
        if (isIntersecting.value) {
            successStrikes.value += 1;
            isPaused.value = true;
            stopTimer();
            if (successStrikes.value >= 6) {
                emit('success');
            } else {
                setTimeout(() => {
                    isPaused.value = false;
                    startTimer();
                }, 1000);
            }
        } else {
            stopTimer();
            gameOver.value = true;
            emit('fail');
        }
    } else {
        startTimer();
    }
}

const onMouseMove = (e: MouseEvent) => {
    if (!gameStarted.value) {
        gameStarted.value = true;
    }

    proceedPlayerInteraction();
}

const onKeyDown = (e: KeyboardEvent) => {
    if (!gameStarted.value) {
        gameStarted.value = true;
        proceedPlayerInteraction();
    } else if (e.key === 'e') {
        proceedPlayerInteraction();
    }
}

onMounted(() => {
    aimUpperSidePosition.value = Math.random() * 60 - 30;
    document.addEventListener('click', onMouseMove);
    document.addEventListener('keydown', onKeyDown);
})

onUnmounted(() => {
    document.removeEventListener('click', onMouseMove);
    document.removeEventListener('keydown', onKeyDown);
})

</script>
<style scoped>
.container {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
}
.outer-circle {
    width: 160px;
    height: 160px;
    border-radius: 50%;
    border: 2px dotted black;
    position: relative;
    overflow: hidden;
}

.inner-circle {
    width: 80px;
    height: 80px;
    border-radius: 50%;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    overflow: hidden;
    background-color: green;
    z-index: 3;
}

.cursor {
    width: 3px;
    height: 80px;
    background-color: black;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: rotate(calc(var(--angle) * 1deg));
    transform-origin: 0% 0%;
    z-index: 2;
}

.aim {transform-origin: 0% 0%;
    position: absolute;
    width: 160px;
    height: 160px;
    background-color: gray;
    top: 50%;
    left: 50%;
    transform:
        rotate(calc(45deg + var(--angle) * 1deg))
        skew(calc(var(--delta) * 1deg), calc(var(--delta) * 1deg));

    &.intersecting {
        background-color: lightgreen;
    }
    &.game-over {
        background-color: red;
    }
}

.score {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: row;
    margin-top: 8px;
    padding: 6px;
    border-radius: 12px;
    background-color: black;
    gap: 4px;
}

.score-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: #FFFFFF40;

    &.success {
        background: green;
    }
}

</style>