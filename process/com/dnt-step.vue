<!--
1. 跳转
2. 禁止选中文本
-->
<template>
  <Steps
    :current="current"
    :direction="direction"
    size="small"
    class="dnt-steps">
    <Step v-for="(value, index) in steps"
          :title="value.title"
          :key="index"
          :class="{link: link && index < current && !skipRange.includes(index)}"
          @click.native="jump(value, index, $event)"/>
  </Steps>
</template>

<script>
export default {
  name: 'dnt-step',
  props: {
    value: {
      required: true,
      type: String,
      default: ''
    },
    steps: {
      required: true,
      type: Array,
      default: () => ([])
    },
    direction: {
      required: false,
      type: String,
      validator (v) {
        const direction = ['horizontal', 'vertical']
        return direction.includes(v)
      },
      default: 'horizontal'
    },
    link: {
      require: false,
      type: Boolean,
      default: true
    },
    oldView: {
      required: false,
      type: String,
      default: ''
    }
  },
  computed: {
    current () {
      return this.steps.findIndex(v => v.name === this.value)
    },
    // 计算被跳过的步骤
    skipRange () {
      const oldIndex = this.steps.findIndex(v => v.name === this.oldView)
      let result = []

      if (oldIndex > -1) {
        const skipDis = Math.abs(this.current - oldIndex) > 1
        if (skipDis) {
          result.push(this.current, oldIndex)
          const [min, max] = result.sort()
          result = this.range(min, max)
        }
      }

      return result
    }
  },
  methods: {
    canLink (index) {
      const {link, current} = this
      return link && index < current
    },
    range (start, end, step = 1) {
      const result = []
      const len = end - start - 1
      let basic = start + step

      while (basic < end) {
        result.push(basic)
        basic += step
      }
      return result
    },
    jump (value, index) {
      if (!this.canLink(index) || this.skipRange.includes(index)) {
        return
      }
      const view = value.name || ''
      this.$emit('input', view)
    }
  }
}
</script>

<style lang="less">
  .dnt-steps {
    .ivu-steps-title {
      max-width: 96px;
      user-select: none;
      overflow: hidden;/*超出部分隐藏*/
      text-overflow:ellipsis;/* 超出部分显示省略号 */
      white-space: nowrap;/*规定段落中的文本不进行换行 */
    }
    .link .ivu-steps-title {
      text-decoration: underline;
      color: #2d8cf0;
      cursor: pointer;
    }
  }
</style>