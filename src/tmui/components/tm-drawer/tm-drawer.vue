<template>
  <tm-overlay
    :inContent="props.inContent"
    ref="overlayAni"
    :duration="props.duration + 80"
    @open="OverLayOpen"
    @close="overclose"
    :zIndex="props.zIndex"
    :transprent="!props.mask"
    @click="overlayClickFun"
    :align="align_rp"
    :overlayClick="false"
    v-model:show="_show"
  >
    <tm-translate
      :reverse="reverse_rp"
      :width="anwidth"
      :height="anheight"
      ref="drawerANI"
      :auto-play="false"
      :name="aniname"
      :duration="props.duration"
    >
      <view
        @click.stop="$event.stopPropagation()"
        :style="[
          { width: anwidth, height: anheight },
          !props.transprent ? tmcomputed.borderCss : '',
          !props.transprent ? tmcomputed.backgroundColorCss : '',
          !props.transprent ? tmcomputed.shadowColor : '',
          customCSSStyle,
        ]"
        :class="[round_rp, 'flex flex-col overflow ', customClass]"
      >
        <view
          v-if="!props.closeable && !props.hideHeader"
          class="flex flex-row flex-row-center-center flex-between px-24"
          style="height: 44px"
        >
          <view class="flex-4 flex-shrink">
            <tm-text
              v-if="!props.hideCancel"
              @click="cancel"
              :label="props.cancelText"
            ></tm-text>
          </view>
          <view class="flex-8 px-32 flex-center">
            <slot name="title">
              <tm-text _class="text-overflow-1 opacity-7" :label="props.title"></tm-text>
            </slot>
          </view>
          <view class="flex-4 flex-shrink flex-row flex-row-center-end">
            <tm-text
              :color="okColor"
              @click="ok"
              v-if="!ok_loading"
              :dark="props.dark"
              :label="props.okText"
            ></tm-text>
            <tm-icon
              :color="okColor"
              v-if="ok_loading"
              :spin="ok_loading"
              :dark="isDark"
              :_class="isDark !== true ? 'opacity-4' : ''"
              :fontSize="34"
              :name="ok_loading ? 'tmicon-jiazai_dan' : 'tmicon-times-circle-fill'"
            ></tm-icon>
          </view>
        </view>
        <view
          v-if="props.closeable && !props.hideHeader"
          class="flex flex-row flex-row-center-center flex-between px-24"
          style="height: 44px"
        >
          <view class="flex-9 pr-32">
            <slot name="title">
              <tm-text
                _class="text-overflow-1 opacity-7"
                :dark="props.dark"
                :label="props.title"
              >
              </tm-text>
            </slot>
          </view>
          <view class="flex-3 flex-shrink flex-row flex-row-center-end">
            <tm-icon
              @click="cancel"
              :dark="props.dark"
              :_class="isDark !== true ? 'opacity-3' : ''"
              :fontSize="36"
              name="tmicon-times-circle-fill"
            ></tm-icon>
          </view>
        </view>
        <scroll-view
          :scroll-y="!props.disabbleScroll"
          :style="[{ height: contentHeight }]"
          class="overflow"
        >
          <slot name="default"></slot>
        </scroll-view>
        <slot name="foot">
          <view v-if="props.footHeight>0" class="flex">
          </view>
        </slot>
      </view>
    </tm-translate>
  </tm-overlay>
</template>

<script lang="ts" setup>
/**
 * ??????
 * @description ??????poup????????????????????????????????????????????????????????????
 */
import tmTranslate from "../tm-translate/tm-translate.vue";
import tmText from "../tm-text/tm-text.vue";
import tmIcon from "../tm-icon/tm-icon.vue";
import tmOverlay from "../tm-overlay/tm-overlay.vue";
import {
  getCurrentInstance,
  computed,
  ref,
  provide,
  inject,
  onMounted,
  onUnmounted,
  nextTick,
  watch,
} from "vue";
import { cssstyle, tmVuetify, colorThemeType } from "../../tool/lib/interface";
import {
  custom_props,
  computedTheme,
  computedClass,
  computedStyle,
  computedDark,
} from "../../tool/lib/minxs";
import { useTmpiniaStore } from "../../tool/lib/tmpinia";
const drawerANI = ref<InstanceType<typeof tmTranslate> | null>(null);
const overlayAni = ref<InstanceType<typeof tmOverlay> | null>(null);
const store = useTmpiniaStore();
const props = defineProps({
  ...custom_props,
  //??????????????????
  mask: {
    type: [Boolean, String],
    default: true,
  },
  //?????????????????????
  placement: {
    type: String,
    default: "bottom", //top|left|right|bottom|center
  },
  show: {
    type: [Boolean],
    default: false,
  },
  width: {
    type: Number,
    default: 500,
  },
  height: {
    type: Number,
    default: 600,
  },
  round: {
    type: Number,
    default: 12,
  },
  //???????????????????????????ms.
  duration: {
    type: Number,
    default: 300,
  },
  //??????????????????????????????
  overlayClick: {
    type: Boolean,
    default: true,
  },
  transprent: {
    type: [Boolean, String],
    default: false,
  },
  //?????????????????????????????????????????????????????????????????????
  closeable: {
    type: [Boolean, String],
    default: false,
  },
  color: {
    type: String,
    default: "white",
  },
  title: [String],
  okText: {
    type: [String],
    default: "??????",
  },
  okColor: {
    type: [String],
    default: "primary",
  },
  //true??????????????????????????????????????????
  okLoading: {
    type: [Boolean, String],
    default: false,
  },
  cancelText: {
    type: [String],
    default: "??????",
  },
  hideCancel: {
    type: [Boolean, String],
    default: false,
  },
  //??????????????????????????????????????????
  hideHeader: {
    type: [Boolean, String],
    default: false,
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  zIndex: {
    type: [Number, String],
    default: 401,
  },
  unit: {
    type: String,
    default: "rpx",
  },
  disabbleScroll: {
    type: Boolean,
    default: false,
  },
  /** ????????????????????????????????????????????????????????????????????? */
  inContent: {
    type: Boolean,
    default: false,
  },
  footHeight:{
    type:Number,
    default:0
  }
});
const emits = defineEmits(["click", "open", "close", "update:show", "ok", "cancel"]);
const proxy = getCurrentInstance()?.proxy ?? null;
const sysinfo = inject(
  "tmuiSysInfo",
  computed(() => {
    return {
      bottom: 0,
      height: 750,
      width: uni.upx2px(750),
      top: 0,
      isCustomHeader: false,
      sysinfo: null,
    };
  })
);
// ??????????????????????????????????????????
const tmcfg = computed<tmVuetify>(() => store.tmStore);
//??????????????????
const customCSSStyle = computed(() => computedStyle(props));
//?????????
const customClass = computed(() => computedClass(props));
//?????????????????????
const isDark = computed(() => computedDark(props, tmcfg.value));
//????????????
const tmcomputed = computed<cssstyle>(() =>
  computedTheme(props, isDark.value, tmcfg.value)
);
const syswidth = computed(() => sysinfo.value.width);
const sysheight = computed(() => sysinfo.value.height);
const reverse = ref(true);
const timeid = ref(0);
let timerId:any = NaN;
let timerIdth:any = NaN;
let timerIdth_flas = false;
uni.hideKeyboard();

let _show = ref(props.show);
function debounce(func: Function, wait = 500, immediate = false) {
  // ???????????????
  if (!isNaN(timerId)) clearTimeout(timerId);
  // ??????????????????????????????????????????

  if (immediate) {
    var callNow = !timerId;
    timerId = setTimeout(() => {
      timerId = NaN;
    }, wait);

    if (callNow) typeof func === "function" && func();
  } else {
    // ?????????????????????????????????????????????timeout????????????????????????????????????wait???????????????func????????????
    timerId = setTimeout(() => {
      typeof func === "function" && func();
    }, wait);
  }
}

function throttle(func: Function, wait = 500, immediate = true) {
  if (immediate) {
    if (!timerIdth_flas) {
      timerIdth_flas = true;
      // ??????????????????????????????wait????????????????????????
      typeof func === "function" && func();

      timerIdth = setTimeout(() => {
        timerIdth_flas = false;
      }, wait);
    }
  } else {
    if (!timerIdth_flas) {
      timerIdth_flas = true;
      // ?????????????????????????????????wait???????????????????????????
      timerIdth = setTimeout(() => {
        timerIdth_flas = false;
        typeof func === "function" && func();
      }, wait);
    }
  }
}

timeid.value = uni.$tm.u.getUid(4);
if (_show.value) {
  reverse.value = false;
}
watch(
  () => props.show,
  (val) => {
    if (val) {
      reverse.value = true;
    } else {
      reverse.value = false;
      overlayAni.value?.close();
    }
    if (_show.value !== props.show) {
      nextTick(() => {
        drawerANI.value?.play();
      });
    }
    _show.value = props.show;
  }
);
onMounted(() => {
  if (_show.value) {
    open();
  }
});
const ok_loading = computed(() => props.okLoading);
const round_rp = computed(() => {
  if (aniname.value == "left") return "round-r-" + props.round;
  if (aniname.value == "right") return "round-l-" + props.round;
  if (aniname.value == "up") return "round-b-" + props.round;
  if (aniname.value == "down") return "round-t-" + props.round;
  if (aniname.value == "zoom") return "round-" + props.round;
});
const reverse_rp = computed(() => {
  if (aniname.value != "zoom") return reverse.value;
  return !reverse.value;
});
const aniname = computed(() => {
  if (props.placement == "center") return "zoom";
  if (props.placement == "top") return "up";
  if (props.placement == "bottom") return "down";
  return props.placement;
});
const anwidth = computed(() => {
  if (aniname.value == "zoom") {
    return props.width + props.unit;
  }
  if (props.placement == "left" || props.placement == "right") {
    return props.width + props.unit;
  }
  return syswidth.value + "px";
});
const anheight = computed(() => {
  let wucha = 0;
  if (
    props.placement == "top" ||
    props.placement == "bottom" ||
    aniname.value == "zoom"
  ) {
    return props.height + wucha + props.unit;
  }
  return sysheight.value + "px";
});
const contentHeight = computed(() => {
  let base_height = props.hideHeader ? 0 : 44;
  let _footerHeight = uni.$tm.u.topx(props.footHeight)
  if (
    props.placement == "top" ||
    props.placement == "bottom" ||
    aniname.value == "zoom"
  ) {
    let h = props.height;
    if (props.unit == "rpx") {
      h = uni.upx2px(props.height);
    }
    return (h - base_height - _footerHeight) + "px";
  }
  return (sysheight.value - base_height - _footerHeight) + "px";
});
const align_rp = computed(() => {
  if (aniname.value == "down") {
    return "flex-col-bottom-center";
  }
  if (aniname.value == "up") {
    return "flex-top-custom";
  }
  if (aniname.value == "left") {
    return "flex-row-top-start";
  }
  if (aniname.value == "right") {
    return "flex-row-bottom-start";
  }
  if (aniname.value == "zoom") {
    return "flex-center";
  }
});

function OverLayOpen() {
  // nextTick(function() {
  // 	drawerANI.value?.play();
  // })
  _show.value = true;
  emits("open");
  emits("update:show", true);
}
function overclose() {
  nextTick(() => {
    _show.value = false;
    emits("close");
    emits("update:show", false);
  });
}
function overlayClickFun(e: Event) {
  emits("click", e);
  if (!props.overlayClick || props.disabled || !overlayAni.value) return;
  reverse.value = false;
  throttle(
    () => {
      emits("cancel");
      overlayAni.value?.close();
      drawerANI.value?.play();
    },
    props.duration + 80,
    true
  );
}

function ok() {
  if (props.disabled) return;
  reverse.value = false;
  debounce(
    () => {
      emits("ok");
      overlayAni.value?.close();
      drawerANI.value?.play();
    },
    500,
    true
  );
}

function cancel() {
  if (props.disabled) return;
  reverse.value = false;
  debounce(
    () => {
      emits("cancel");
      overlayAni.value?.close();
      drawerANI.value?.play();
    },
    500,
    true
  );
}

//???????????????
function open() {
  reverse.value = true;
  _show.value = true;
  nextTick(() => {
    drawerANI.value?.play();
  });
}

//??????????????????????????????
function close() {
  reverse.value = false;
  overlayAni.value?.close();
  drawerANI.value?.play();
}

//????????????????????????
defineExpose({
  close: close,
  open: open,
});
</script>

<style scoped>
.flex-left-custom {
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
}

.flex-right-custom {
  display: flex;
  justify-content: flex-start;
  align-items: flex-end;
}

.flex-top-custom {
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
}

.flex-end-custom {
  display: flex;
  justify-content: flex-end;
  align-items: flex-end;
}

.flex-center-custom {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: row;
}
</style>
