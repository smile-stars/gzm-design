<template>
    <div
            :class="[
      `${baseClassName}__slider-wrapper`,
      `${baseClassName}__slider-wrapper--${type}-type`
    ]"
    >
        <template v-if="type === 'alpha'">
            <div
                    :class="`${baseClassName}__slider-padding`"
                    :style="paddingStyle"
            />
        </template>
        <div :class="[`${baseClassName}__slider`, className]" ref="refPanel">
            <div :class="`${baseClassName}__rail`" :style="railStyle"></div>
            <span
                    :class="`${baseClassName}__thumb`"
                    role="slider"
                    tabindex="0"
                    ref="refThumb"
                    :style="styles"
            ></span>
        </div>
    </div>

</template>
<script lang="ts">
import {computed, defineComponent, onBeforeUnmount, onMounted, PropType, reactive, ref} from 'vue';
import {SLIDER_DEFAULT_WIDTH} from '../const';
import {Draggable, Coordinate} from '../utils';
import {useBaseClassName} from '../hooks';
import baseProps from './base-props';

export default defineComponent({
    name: 'ColorSlider',
    props: {
        ...baseProps,
        className: {
            type: String,
            default: '',
        },
        value: {
            type: Number,
            default: 0,
        },
        maxValue: {
            type: Number,
            default: 360,
        },
        railStyle: {
            type: Object as PropType<any>,
        },
        type: {
            type: String as PropType<'hue' | 'alpha'>,
            default: 'hue',
        },
    },
    setup(props) {
        const baseClassName = useBaseClassName();
        const refPanel = ref<HTMLElement>(null);
        const refThumb = ref<HTMLElement>(null);
        const dragInstance = ref<Draggable>(null);
        const panelRect = reactive({
            width: SLIDER_DEFAULT_WIDTH,
        });
        const styles = computed(() => {
            const {width} = panelRect;
            if (!width) {
                return;
            }
            const left = Math.round((props.value / props.maxValue) * width);
            return {
                left: `${left}px`,
                color: props.color.rgb,
            };
        });
        const paddingStyle = computed(() => {
            return {background: `linear-gradient(90deg, rgba(0,0,0,.0) 0%, rgba(0,0,0,.0) 93%, ${props.color.rgb} 93%, ${props.color.rgb} 100%)`,}
        })

        const handleDrag = (coordinate: Coordinate, isEnded?: boolean) => {
            if (props.disabled) {
                return;
            }
            const {width} = panelRect;
            const {x} = coordinate;
            const value = Math.round((x / width) * props.maxValue * 100) / 100;
            props.onChange(value, isEnded);
        };

        const handleDragEnd = (coordinate: Coordinate) => {
            if (props.disabled) {
                return;
            }
            handleDrag(coordinate, true);
        };

        onMounted(() => {
            panelRect.width = refPanel.value.offsetWidth || SLIDER_DEFAULT_WIDTH;
            dragInstance.value = new Draggable(refPanel.value, {
                start: () => {
                    // pop模式下由于是隐藏显示，这个宽度让其每次点击的时候重新计算
                    panelRect.width = refPanel.value.offsetWidth;
                },
                drag: (coordinate: Coordinate) => {
                    handleDrag(coordinate);
                },
                end: handleDragEnd,
            });
        });

        onBeforeUnmount(() => {
            dragInstance.value.destroy();
        });

        return {
            baseClassName,
            refThumb,
            refPanel,
            styles,
            paddingStyle,
        };
    },
})
</script>
<style scoped>

</style>

