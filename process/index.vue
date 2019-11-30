<!--
1. 流程展示
2. 跳转步骤
3. 共享数据

注意点： 1. 数据全部由modal保存，内部尽量不保存状态
        2. 用之前请查看master文件,防止覆盖变量或者方法名称
        3. 尽量在组件外业务代码

封版日期：2019/11/29
-->
<template>
  <Modal
      v-model="show"
      :width="width"
      :mask-closable="false"
      class="dnt-process-model"
      @on-cancel="cancel">
      <!-- 顶部 -->
      <h5 slot="header">
        <slot name="title">
          {{title}}
        </slot>
      </h5>

      <!-- 内容部分 -->
      <section :class="{vertical, horizontal: !vertical}">
        <dntStep v-model="currentView" :direction="direction" :steps="steps" :link="link" :oldView="oldView"/>
        <content class="dnt-step-content">
          <keep-alive :include="aliveInclude">
            <component v-if="aliveInclude.length" ref="com" :is="currentView" v-model="value" :oldView="oldView"></component>
          </keep-alive>
        </Content>
      </section>

      <!-- 底部 -->
      <template slot="footer">
        <div v-if="steps.length">
          <Button v-if="defaluts(steps[current].preview, true) && this.current !== 0" type="primary"
                @click="qaPreview">上一步</Button>
          <Button v-if="defaluts(steps[current].next, true) && this.current !== this.steps.length - 1"
                  type="primary"
                  :disabled="defaluts(steps[current].disabled, false)"
                  @click="qaNext">下一步</Button>
          <Button type="primary"
                  v-if="defaluts(steps[current].submit, false)"
                  @click="qaSubmit">{{submitText}}</Button>
          <Button type="primary"
                  v-if="defaluts(steps[current].exit, true)"
                  @click="forceExit">{{exitText}}</Button>
        </div>
        <div v-else></div>
      </template>
  </Modal>
</template>
<script>

import dntStep from './com/dnt-step.vue'
let timer = null

export default {
  name: 'process-modal',
  components: {
    dntStep
  },
  provide () {
    return {
      modal: this.modal,
      preview: this.preview,
      next: this.next,
      jump: this.jump,
      broadcast: this.broadcast,
      steps: this.steps,
      close: this.close,
      safeClose: this.safeClose
    }
  },
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    value: {
      required: false
    },
    title: {
      type: String,
      default: 'Process Modal'
    },
    width: {
      type: Number,
      default: 800
    },
    // 是否竖排
    vertical: {
      type: Boolean,
      default: true
    },
    link: {
      type: Boolean,
      default: true
    },
    submitText: {
      type: String,
      default: '提交'
    },
    exitText: {
      type: String,
      default: '退出'
    }
  },
  data () {
    return {
      // 共享数据源
      modal: {},
      orginalModal: '',
      steps: [],
      orginalSteps: '',
      // 旧视图
      oldView: '',
      // 当前视图
      currentView: '',
      // 当前活动的组件
      aliveInclude: []
    }
  },
  computed: {
    show: {
      get () {
        return this.visible
      },
      set (v) {
        this.$emit('update:visible', v)
      }
    },
    direction () {
      const direction = ['horizontal', 'vertical']
      const index = Number(this.vertical)
      return direction[index]
    },
    // 当前步骤
    current () {
      return this.steps.findIndex(v => v.name === this.currentView)
    }
  },
  watch: {
    show (n) {
      // 第一次打开时缓存状态
      if (!this.orginalSteps) {
        this.orginalSteps = JSON.stringify(this.steps)
      }
      if (!this.orginalModal) {
        this.orginalModal = JSON.stringify(this.modal)
      }
      if (n) {
        this.postCache()
        this.reload()
      } else {
        this.reset()
      }
    },
    currentView (n, o = '') {
      this.oldView = o
    }
  },
  methods: {
    // 弹框打开时触发
    reload () {},
    // 弹框关闭触发
    reset () {},
    // 强制退出
    forceExit () {
      this.resetModal()
      this.exit()
    },
    // 重置modal
    resetModal () {
      const modal = JSON.parse(this.orginalModal)
      const keys = Object.keys(modal)
      // 还原默认值
      for (let key in modal) {
        this.$set(this.modal, key, modal[key])
      }
      // 清空后期绑定上去的数据
      for (let key in this.modal) {
        if (!keys.includes(key)) {
          delete this.modal[key]
        }
      }
    },
    // 加入alive缓存
    postCache () {
      const include = this.steps.slice(0).map(v => v.name)
      this.aliveInclude.push(...include)
    },
    // 清空alive缓存 step1数据 状态
    clearCache () {
      while (this.aliveInclude.length) {
        this.aliveInclude.pop()
      }
    },
    cancel () {
      this.close()
    },
    close () {
      this.show = false
    },
    exit () {
      // 还原步骤数据
      this.$set(this, 'steps', JSON.parse(this.orginalSteps))
      // 清空组建数据状态
      this.clearCache()
      // 跳转页面到第一步
      this.safeClose()
    },
    // 安全显示
    safeClose () {
      this.close()

      timer && window.clearTimeout(timer)

      // 防止页面闪烁
      timer = window.setTimeout(() => {
        this.jump(0)
      }, 1000)
    },
    // 上一步
    preview () {
      this.jump(this.current - 1)
    },
    // 下一步
    next () {
      this.jump(this.current + 1)
    },
    // 根据序号 or name直接跳转到某一步骤
    jump (index = 0) {
      const curr = this.steps[index]
      if (!curr) {
        index = this.steps.findIndex(v => v.name === index)
      }
      this.currentView = this.steps[index].name
    },
    // 凡是带有qa的方法均需要满足某一步骤的状态
    qaPreview () {
      if (this.proxy('qaPreview')) return
      this.preview()
    },
    qaNext () {
      if (this.proxy('qaNext')) return
      this.next()
    },
    // 提交按钮
    qaSubmit () {
      this.proxy('qaSubmit')
    },
    // 代理执行子组内的方法
    proxy (func, ...parmas) {
      const com = this.$refs.com
      com[func] && com[func](...parmas)
      return !!com[func]
    },
    defaluts (val, defaluts) {
      return typeof val === 'undefined' ? defaluts : val
    }
  }
}
</script>
<style lang="less">
.dnt-process-model {
  // 水平样式
  .horizontal {
    padding-top: 10px;
  }

  // 垂直样式
  .vertical {
    display: flex;
    .ivu-steps {
      display: flex;
      flex-direction: column;
      justify-content: center;
      width: 120px;
    }
  }

  /* 内容公共样式 */
  .dnt-step-content {
    width: 100%;
    padding: 4px;
  }
}
</style>
