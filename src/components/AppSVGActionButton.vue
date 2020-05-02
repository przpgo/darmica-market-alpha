<template>
  <q-btn
    class="q-pa-sm svg-button"
    :aria-label="ariaLabel"
    :rounded="true"
    :color="color"
    :size="size"
    @click="$emit('click')"
  >
    <svg
      xmlns:svg="http://www.w3.org/2000/svg"
      xmlns="http://www.w3.org/2000/svg"
      viewBox="0 0 250 250"
      width="50"
      version="1.1"
      fill="url(#linearGradient)"
    >
      <g
        fill="#FF16D7"
      >
        <g
          transform="translate(85 66)"
          fill="#FFE547"
        >
          <ellipse
            opacity=".244"
            cx="66.5"
            cy="67.5"
            rx="66.5"
            ry="67"
          />
          <circle
            cx="67"
            cy="67.5"
            r="34"
          />
        </g>
        <g
          transform="translate(0 67)"
          fill="#FF16D7"
        >
          <ellipse
            opacity=".244"
            cx="66.5"
            cy="66.5"
            rx="66.5"
            ry="67"
          />
          <circle
            cx="67"
            cy="66.5"
            r="34"
          />
        </g>
        <g
          transform="translate(42)"
          fill="#86D600"
        >
          <ellipse
            opacity=".244"
            cx="66.5"
            cy="67.5"
            rx="66.5"
            ry="67"
          />
          <circle
            cx="67"
            cy="67.5"
            r="34"
          />
        </g>
      </g>
    </svg>
  </q-btn>
</template>

<script>
export default {
  props: {
    color: {
      type: String,
      default: 'primaryColor',
    },
    size: {
      type: String,
      default: 'md',
    },
    rounded: {
      type: Boolean,
      default: false,
    },
    label: {
      type: String,
      default: ''
    }
  },
  data () {
    return {}
  },
  computed: {
    ariaLabel () {
      return this.label || this.$t({ id: 'navigation.search' })
    }
  }
}
</script>

<style lang="stylus" scoped>
</style>
