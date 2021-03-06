<template>
<popover
  align='center'
  transition='fade'
  class='c-day-popover'
  :content-offset='popoverContentOffset'
  :visibility='popoverVisibility'
  :content-style='popoverContentStyle'
  :is-interactive='popoverIsInteractive'
  toggle-visible-on-click>
  <!-- Background layers -->
  <transition-group
    name='background'
    tag='div'
    class='c-day-backgrounds'>
    <div
      v-for='background in backgrounds'
      :key='background.key'
      :class='background.wrapperClass'
      :style='dayOpacityStyle'>
      <div
        class='c-day-background'
        :style='background.style'>
      </div>
    </div>
  </transition-group>
  <!-- Content layer -->
  <div
    class='c-day-layer c-day-box-center-center'
    :style='dayOpacityStyle'>
    <div
      ref='dayContent'
      class='c-day-content'
      :style='contentStyle_'
      @click='click'
      @mouseenter='mouseenter'
      @mouseover='mouseover'
      @mouseleave='mouseleave'>
      {{ label }}
    </div>
  </div>
  <!-- Dots layer -->
  <div
    class='c-day-layer c-day-box-center-bottom'
    :style='dayOpacityStyle'
    v-if='hasDots'>
    <div
      class='c-day-dots'
      :style='dotsStyle'>
      <span
        v-for='dot in dots'
        :key='dot.key'
        class='c-day-dot'
        :style='dot.style'>
      </span>
    </div>
  </div>
  <!-- Bars layer -->
  <div
    class='c-day-layer c-day-box-center-bottom'
    :style='dayOpacityStyle'
    v-if='hasBars'>
    <div
      class='c-day-bars'
      :style='barsStyle'>
      <span
        v-for='bar in bars'
        :key='bar.key'
        class='c-day-bar'
        :style='bar.style'>
      </span>
    </div>
  </div>
  <!-- Popover content -->
  <div
    class='c-day-popover-content'
    slot='popover-content'>
    <!-- Header slot -->
    <slot
      name='popover-header'
      :day-info='dayInfo'
      :attributes='attributesList'>
    </slot>
    <!-- Content row slots -->
    <calendar-day-popover-row
      v-for='popover in popovers'
      :key='popover.key'
      :attribute='popover.attribute'
      :hide-indicator='popover.hideIndicator'>
      <slot
        :name='popover.slot'
        :attribute='popover.attribute'
        :custom-data='popover.attribute.customData'
        :day-info='dayInfo'>
        <span
          class='popover-label'
          :style='popover.labelStyle'
          :key='popover.key'>
          {{ popover.label }}
        </span>
      </slot>
    </calendar-day-popover-row>
  </div>
</popover>
</template>

<script>
import Popover from './Popover';
import CalendarDayPopoverRow from './CalendarDayPopoverRow';
import { arrayHasItems, objectFromArray } from '../utils/helpers';
import { isFunction } from '../utils/typeCheckers';

export default {
  components: {
    Popover,
    CalendarDayPopoverRow,
  },
  props: {
    dayInfo: { type: Object, required: true },
    attributes: Object,
    popoverContentOffset: { type: String, default: '7px' },
    styles: Object,
  },
  data() {
    return {
      backgrounds: [],
      dots: [],
      bars: [],
      popovers: [],
      contentStyle: null,
      contentHoverStyle: null,
      popoverVisibility: 'hidden',
      popoverIsInteractive: false,
      isHovered: false,
    };
  },
  computed: {
    label() {
      return this.dayInfo.label;
    },
    dateTime() {
      return this.dayInfo.dateTime;
    },
    inMonth() {
      return this.dayInfo.inMonth;
    },
    dayOpacityStyle() {
      const dayCell = this.styles.dayCell;
      const dayCellNotInMonth = this.styles.dayCellNotInMonth;
      // Assign default opacity
      let opacity = (dayCell && dayCell.opacity) || 1;
      // Reassign to 'not in month' opacity if it was specified
      if (!this.inMonth && Object.prototype.hasOwnProperty.call(dayCellNotInMonth, 'opacity')) opacity = dayCellNotInMonth.opacity;
      return {
        opacity,
      };
    },
    dayCellStyle() {
      // Merge 'not in month' style if needed
      const style = this.inMonth ? this.styles.dayCell : {
        ...this.styles.dayCell,
        ...this.styles.dayCellNotInMonth,
      };
      // Due to some rendering glitches in Chrome, we need to remove opacity from top level style
      delete style.opacity;
      return style;
    },
    contentStyle_() {
      let style = this.contentStyle;
      if (this.isHovered) style = { ...style, ...this.contentHoverStyle };
      return style;
    },
    attributesList() {
      return this.attributes.find(this.dayInfo);
    },
    attributesMap() {
      return objectFromArray(this.attributesList);
    },
    hasBackgrounds() {
      return arrayHasItems(this.backgrounds);
    },
    hasDots() {
      return arrayHasItems(this.dots);
    },
    dotsStyle() {
      return this.styles.dots;
    },
    hasBars() {
      return arrayHasItems(this.bars);
    },
    barsStyle() {
      return this.styles.bars;
    },
    highlights() {
      return this.hasBackgrounds ?
        this.backgrounds.map(b => b.highlight) :
        [];
    },
    hasPopovers() {
      return !!arrayHasItems(this.popovers);
    },
    popoverContentStyle() {
      return this.styles.dayPopoverContent;
    },
    eventDayInfo() {
      return {
        ...this.dayInfo,
        el: this.$refs.dayContent,
      };
    },
  },
  watch: {
    attributes() {
      this.processAttributes();
    },
    styles() {
      this.processAttributes();
    },
    popovers(val) {
      let visibility = '';
      let isInteractive = false;
      let content;
      val.forEach((popover) => {
        if (!visibility && popover.visibility) visibility = popover.visibility;
        isInteractive = isInteractive || popover.isInteractive;
        content = content || popover.label || popover.slot;
      });
      this.popoverVisibility = visibility || (content && 'hover') || 'hidden';
      this.popoverIsInteractive = isInteractive;
    },
  },
  created() {
    this.processAttributes();
  },
  methods: {
    click() {
      this.$emit('dayselect', this.dayInfo, this.attributesMap);
    },
    mouseenter() {
      this.$emit('daymouseenter', this.dayInfo, this.attributesMap);
    },
    mouseover() {
      this.isHovered = true;
      this.$emit('daymouseover', this.dayInfo, this.attributesMap);
    },
    mouseleave() {
      this.isHovered = false;
      this.$emit('daymouseleave', this.dayInfo, this.attributesMap);
    },
    processAttributes() {
      const backgrounds = [];
      const dots = [];
      const bars = [];
      const popovers = [];
      const contentStyles = [];
      const contentHoverStyles = [];
      // Get the day attributes
      this
        .attributesList
        .forEach((attribute) => {
          // Add background for highlight if needed
          if (attribute.highlight) backgrounds.push(this.getBackground(attribute));
          // Add dot if needed
          if (attribute.dot) dots.push(this.getDot(attribute));
          // Add bar if needed
          if (attribute.bar) bars.push(this.getBar(attribute));
          // Add popover if needed
          if (attribute.popover) popovers.unshift(this.getPopover(attribute));
          // Add content style if needed
          if (attribute.contentStyle) contentStyles.push(attribute.contentStyle);
          // Add content hover style if needed
          if (attribute.contentHoverStyle) contentHoverStyles.push(attribute.contentHoverStyle);
        });
      // Assign day attributes
      this.backgrounds = backgrounds;
      this.dots = dots;
      this.bars = bars;
      this.popovers = popovers;
      this.contentStyle = Object.assign({}, this.styles.dayContent, ...contentStyles);
      this.contentHoverStyle = Object.assign({}, this.styles.dayContentHover, ...contentHoverStyles);
    },
    getBackground(attribute) {
      // Initialize the background object
      const highlight = attribute.highlight;
      const dateInfo = attribute.targetDate;
      const background = {
        key: attribute.key,
        highlight,
        style: {
          width: highlight.height,
          height: highlight.height,
          backgroundColor: highlight.backgroundColor,
          borderColor: highlight.borderColor,
          borderWidth: highlight.borderWidth,
          borderStyle: highlight.borderStyle,
          borderRadius: highlight.borderRadius,
          opacity: highlight.opacity,
        },
      };
      if (dateInfo.isDate) {
        background.wrapperClass = `c-day-box-center-center${highlight.animated ? ' c-day-scale-enter c-day-scale-leave' : ''}`;
      } else {
        const onStart = dateInfo.startTime === this.dateTime;
        const onEnd = dateInfo.endTime === this.dateTime;
        const borderWidth = background.style.borderWidth;
        const borderRadius = background.style.borderRadius;
        const endWidth = '95%';
        // Is the day date on the highlight start and end date
        if (onStart && onEnd) {
          background.wrapperClass = `c-day-box-center-center${highlight.animated ? ' c-day-scale-enter c-day-scale-leave' : ''}`;
          background.style.width = endWidth;
          background.style.borderWidth = borderWidth;
          background.style.borderRadius = `${borderRadius} ${borderRadius} ${borderRadius} ${borderRadius}`;
        // Is the day date on the highlight start date
        } else if (onStart) {
          background.wrapperClass = `c-day-box-right-center shift-right${highlight.animated ? ' c-day-slide-left-enter' : ''}`;
          background.style.width = endWidth;
          background.style.borderWidth = `${borderWidth} 0 ${borderWidth} ${borderWidth}`;
          background.style.borderRadius = `${borderRadius} 0 0 ${borderRadius}`;
        // Is the day date on the highlight end date
        } else if (onEnd) {
          background.wrapperClass = `c-day-box-left-center shift-left${highlight.animated ? ' c-day-slide-right-enter' : ''}`;
          background.style.width = endWidth;
          background.style.borderWidth = `${borderWidth} ${borderWidth} ${borderWidth} 0`;
          background.style.borderRadius = `0 ${borderRadius} ${borderRadius} 0`;
        // Is the day date between the highlight start/end dates
        } else {
          background.wrapperClass = 'c-day-box-center-center shift-left-right';
          background.style.width = '100%';
          background.style.borderWidth = `${borderWidth} 0`;
          background.style.borderRadius = '0';
        }
      }
      return background;
    },
    getDot(attribute) {
      return {
        key: attribute.key,
        dot: attribute.dot,
        style: {
          width: attribute.dot.diameter,
          height: attribute.dot.diameter,
          backgroundColor: attribute.dot.backgroundColor,
          borderColor: attribute.dot.borderColor,
          borderWidth: attribute.dot.borderWidth,
          borderStyle: attribute.dot.borderStyle,
          borderRadius: attribute.dot.borderRadius,
          opacity: attribute.dot.opacity,
        },
      };
    },
    getBar(attribute) {
      return {
        key: attribute.key,
        bar: attribute.bar,
        style: {
          height: attribute.bar.height,
          backgroundColor: attribute.bar.backgroundColor,
          borderColor: attribute.bar.borderColor,
          borderWidth: attribute.bar.borderWidth,
          borderStyle: attribute.bar.borderStyle,
          opacity: attribute.bar.opacity,
        },
      };
    },
    getPopover(attribute) {
      const {
        label,
        labelStyle,
        hideIndicator,
        slot,
        visibility,
        isInteractive,
      } = attribute.popover;
      return {
        key: attribute.key,
        customData: attribute.customData,
        attribute,
        label: isFunction(label) ? label(attribute, this.dayInfo) : label,
        labelStyle: isFunction(labelStyle) ? labelStyle(attribute, this.dayInfo) : labelStyle,
        hideIndicator,
        slot,
        visibility,
        isInteractive: isInteractive !== undefined ? isInteractive : !!slot,
      };
    },
  },
};
</script>

<style lang='sass' scoped>

@import '../styles/vars.sass'
@import '../styles/mixins.sass'

.c-day-popover
  position: relative
  flex: 1
  height: $day-height

.c-day-layer
  position: absolute
  left: 0
  right: 0
  top: 0
  bottom: 0
  pointer-events: none

.c-day-box-center-center
  +box()
  height: 100%
  transform-origin: 50% 50%

.c-day-box-left-center
  +box(flex-start)
  height: 100%
  transform-origin: 0% 50%

.c-day-box-right-center
  +box(flex-end)
  height: 100%
  transform-origin: 100% 50%

.c-day-box-center-bottom
  +box(center, flex-end)

.c-day-backgrounds
  height: 100%
  overflow: hidden

.c-day-background
  transition: height $background-transition-time, background-color $background-transition-time

.shift-left
  margin-left: -1px

.shift-right
  margin-right: -1px

.shift-left-right
  margin: 0 -1px

.c-day-dots
  +box()

.c-day-dot
  width: $dot-diameter
  height: $dot-diameter
  border-radius: $dot-border-radius
  background-color: $dot-background-color
  transition: all $day-content-transition-time
  &:not(:last-child)
    margin-right: $dot-spacing

.c-day-bars
  +box(flex-start)
  width: $bars-width

.c-day-bar
  flex-grow: 1
  height: $bar-height
  background-color: $bar-background-color
  transition: all $day-content-transition-time

.c-day-content
  +box()
  width: $day-content-width
  height: $day-content-height
  font-size: $day-content-font-size
  font-weight: $day-content-font-weight
  line-height: 1
  border-radius: $day-content-border-radius
  transition: all $day-content-transition-time
  user-select: none
  cursor: default
  pointer-events: all

.c-day-popover-content
  font-size: $day-popover-font-size
  font-weight: $day-popover-font-weight

// TRANSITION ANIMATIONS

.background-enter-active

  &.c-day-fade-enter
    transition: $fade-transition

  &.c-day-slide-right-enter
    animation: $slide-right-enter-animation

  &.c-day-slide-left-enter
    animation: $slide-left-enter-animation

  &.c-day-scale-enter
    animation: $scale-enter-animation

.background-leave-active
  &.c-day-fade-leave
    transition: $fade-transition

  &.c-day-scale-leave
    animation: $scale-leave-animation

.background-enter
  &.c-day-fade-enter
    opacity: 0

.background-leave-to
  &.c-day-fade-leave
    opacity: 0

</style>
