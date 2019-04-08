<template>
  <div
    v-bind:class="{ collapse, collapsing, show: showCollapse }"
    v-bind:style="styleObject"
  >
    <slot></slot>
  </div>
</template>

<script>
import _ from 'lodash';

function transformTransitionDuration(stringDuration) {
  // always get the first duration (same as bootstrap)
  const durationArr = _.split(stringDuration, ',');
  const timeInSeconds = _.toNumber(_.trimEnd(_.toLower(durationArr[0]), 's'));
  if (_.isNaN(timeInSeconds)) {
    return 0;
  }
  return timeInSeconds * 1000;
}

// modified from bootstrap file bootstrap.bundle.js
function getTransitionDurationFromElement(element) {
  if (!element) {
    return 0;
  }
  const elementStyle = window.getComputedStyle(element, null);
  const animationDuration = elementStyle.getPropertyValue('animation-duration');
  const transitionDuration = elementStyle.getPropertyValue('transition-duration');
  const durationArr = [];
  durationArr.push(transformTransitionDuration(animationDuration));
  durationArr.push(transformTransitionDuration(transitionDuration));
  return _.max(durationArr);
}


function handleShowChange(newShowValue) {
  if (newShowValue) {
    this.handleShowCollapse();
  } else {
    this.handleHideCollapse();
  }
}

function handleShowCollapse() {
  if (this.showCollapse || this.collapsing) {
    return;
  }
  this.$emit('show');
  const dimension = this.getDimension();
  const scrollSize = `scroll${_.upperFirst(dimension)}`;
  this.collapsing = true;
  this.collapse = false;
  this.$nextTick(() => {
    const collapsingTransitionDuration = getTransitionDurationFromElement(this.$el);
    setTimeout(() => {
      this.collapsing = false;
      this.collapse = true;
      this.showCollapse = true;
      this.styleObject[dimension] = '';
      this.$emit('shown');
    }, collapsingTransitionDuration);
    this.styleObject[dimension] = `${this.$el[scrollSize]}px`;
  });
}

function handleHideCollapse() {
  if (!this.showCollapse) {
    return;
  }
  this.$emit('hide');
  const dimension = this.getDimension();
  this.styleObject[dimension] = `${this.$el.getBoundingClientRect()[dimension]}px`;
  this.collapsing = true;
  this.collapse = false;
  this.showCollapse = false;
  this.$nextTick(() => {
    this.styleObject[dimension] = '';
    const collapsingTransitionDuration = getTransitionDurationFromElement(this.$el);
    setTimeout(() => {
      this.collapsing = false;
      this.collapse = true;
      this.$emit('hidden');
    }, collapsingTransitionDuration);
  });
}

function getDimension() {
  if (this.width) {
    return 'width';
  }
  return 'height';
}

export default {
  name: 'bs-collapse',
  props: {
    show: Boolean,
    height: {
      type: Boolean,
      default: true,
    },
    width: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      collapse: true,
      collapsing: false,
      showCollapse: false,
      styleObject: {
        width: '',
        height: '',
      },
    };
  },
  watch: {
    show: handleShowChange,
  },
  methods: {
    handleShowCollapse,
    handleHideCollapse,
    getDimension,
  },
};
</script>
