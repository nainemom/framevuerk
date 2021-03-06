<template>
  <div :class="$style.slider">
    <div class="labels">
      <slot name="label" />
    </div>
    <div class="contents">
      <div
        ref="inner"
        class="inner"
      >
        <slot name="content" />
      </div>
      <fvButton
        v-if="showButtons"
        fab
        class="btn prev"
        invert
        css-color="primary"
        css-border="no"
        css-shadow="no"
        @click="moveCurrent(-1)"
        v-text="$theme.direction.prevChar"
      />
      <fvButton
        v-if="showButtons"
        fab
        class="btn next"
        invert
        css-color="primary"
        css-border="no"
        css-shadow="no"
        @click="moveCurrent(1)"
        v-text="$theme.direction.nextChar"
      />
    </div>
  </div>
</template>

<doc>
@prop current @type Any @default undefined @description Current selected slide on viewport. Use this with .sync modifier.
@prop slidesPerPage @type Number @default 1 @description How many slides should visible on each view.
@prop showButtons @type Boolean @default false @description Show next/prev buttons at sides?
@prop swipeEvents @type Boolean @default true @description Handle swipe geastures to move slide to left/right.

@event update:current @params newValue @description Triggers when selected slide changes.

@slot label @description Use fvSlideLabel with this slot name in top of component content. Each `label` slot counts as one label/slide.
@slot content @description Use fvSlideContent with this slot name in component content just after defining `label`s.. Each `content` slot counts as one content/slide.
</doc>

<example>
@config title 'Default'
@config state true
@config example true

@data val = '1'
<fvSlider :current.sync="val" show-buttons>
  <fvSlideLabel slot="label" name="1"> One </fvSlideLabel>
  <fvSlideLabel slot="label" name="2"> Two </fvSlideLabel>
  <fvSlideLabel slot="label" name="3"> Three </fvSlideLabel>
  <fvSlideLabel slot="label" name="4"> Four </fvSlideLabel>

  <fvSlideContent slot="content" name="1" css-padding-y="xl" css-text-align="center" css-color="footer">
    <h2> Hiiii </h2>
    <h5> is this ok?! </h5>
  </fvSlideContent>
  <fvSlideContent slot="content" name="2" css-padding-y="xl" css-text-align="center" css-color="danger">
    <h2> Byeee </h2>
    <h5> is this not ok?! </h5>
  </fvSlideContent>
  <fvSlideContent slot="content" name="3" css-padding-y="xl" css-text-align="center" css-color="gray">
    <h2> Dumba </h2>
    <h5> dishdi dum?! </h5>
  </fvSlideContent>
  <fvSlideContent slot="content" name="4" css-padding-y="xl" css-text-align="center" css-color="background">
    <h2> Yayyy </h2>
    <h5> my eyes. my eyeeees! </h5>
  </fvSlideContent>
</fvSlider>
</example>

<script>
import { moveIndex } from '../utility/utils';
import Swipe from '../utility/swipe';

export default {
  inject: ['$theme'],
  props: {
    // eslint-disable-next-line vue/require-prop-types
    current: {
      default: undefined,
    },
    slidesPerPage: {
      type: Number,
      default: 1,
    },
    showButtons: {
      type: Boolean,
      default: false,
    },
    swipeEvents: {
      type: Boolean,
      default: true,
    },
  },
  provide() {
    return {
      $slider: this,
    };
  },
  data() {
    return {
      swipe: null,
    };
  },
  computed: {
    numberOfTabs() {
      return this.$slots.content.length;
    },
    numberOfSlides() {
      return Math.ceil(this.numberOfTabs / this.slidesPerPage);
    },
    eachTabSize() {
      return this.numberOfTabs / (this.numberOfSlides * 100);
    },
    currentIndex() {
      return this.current ? this.$slots.content.findIndex((content) => content.componentOptions.propsData.name === this.current) : 0;
    },
    currentTransform() {
      const factor = this.$theme.direction.leftFactor * -1;
      return factor * (this.currentIndex * (100 / this.numberOfTabs));
    },
  },
  watch: {
    current() {
      this.resetTransform();
    },
    swipeEvents: {
      handler(value) {
        this.setSwipeEvents(value);
      },
      immediate: true,
    },
  },
  beforeDestroy() {
    this.setSwipeEvents(false);
  },
  methods: {
    setSwipeEvents(active) {
      this.$nextTick(() => {
        if (active) {
          this.swipe = new Swipe(this.$refs.inner);
          this.swipe.before(this.beforeSwipe);
          this.swipe.while(this.whileSwipe);
          this.swipe.after(this.afterSwipe);
          this.swipe.on();
        } else if (this.swipe) {
          this.swipe.off();
        }
      });
    },
    resetTransform() {
      this.$refs.inner.style.transform = `translateX(${this.currentTransform}%)`;
    },
    moveIndex(offset) {
      const max = this.numberOfTabs - (this.slidesPerPage - 1);
      return moveIndex(this.currentIndex + offset, max);
    },
    beforeSwipe() {
      this.$refs.inner.style.transitionDuration = '0s';
    },
    whileSwipe(pos, diff) {
      this.$refs.inner.style.transform = `translateX(calc(${this.currentTransform}% + ${diff.x}px))`;
    },
    afterSwipe(pos, diff) {
      this.$refs.inner.style.transitionDuration = null;
      // eslint-disable-next-line no-nested-ternary
      const offsetIndex = this.$theme.direction.leftFactor * (diff.x < -100 ? 1 : (diff.x > 100 ? -1 : 0));
      const newIndex = this.moveIndex(offsetIndex);
      const newValue = this.$slots.content[newIndex].componentOptions.propsData.name;
      if (newIndex === this.currentIndex) {
        this.resetTransform();
      }
      this.setCurrent(newValue);
    },
    setCurrent(value) {
      this.$emit('update:current', value);
    },
    moveCurrent(offset) {
      this.$emit('update:current', this.$slots.content[this.moveIndex(offset)].componentOptions.propsData.name);
    },
  },
  style({ className }) {
    return [
      className('slider', {
        '& > .labels': {
          position: 'relative',
          display: 'flex',
          flexDirection: 'row',
          borderBottomWidth: '1px',
          borderBottomColor: this.$theme.colors.background.shade(-13),
        },
        '& > .contents': {
          position: 'relative',
          overflow: 'hidden',
          '& > .btn': {
            position: 'absolute',
            top: '50%',
            transform: 'translateY(-50%)',
            '&.next': {
              [this.$theme.direction.end]: 0,
            },
            '&.prev': {
              [this.$theme.direction.start]: 0,
            },
          },
          '& > .inner': {
            display: 'flex',
            flexDirection: 'row',
            width: `${this.numberOfSlides * 100}%`,
            transition: 'transform 230ms',
            willChange: 'transform',
          },
        },
      }),
    ];
  },
};
</script>
