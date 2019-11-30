<!--
  master 级流程弹框
-->
<script>
/**
 *  steps参数配置   类型   默认值
 *  name:     必传    **
 *  title:    必传    **
 *
 *  当前步骤可选按钮配置
 *  exit:     可选    true
 *  submit:   可选    false
 *  preview:  可选    true
 *  next:     可选    true
 *  disabled: 可选    false <下一步enabale状态>
 *
 *
 *  qa方法 父组件可调用子元素的方法<可选>
 *  qaSubmit    提交
 *  qaNext      下一步
 *  qaPreview   上一步
 *
 *
 *  可配置props
 *  visible:    false
 *  value:      <any>
 *  title:      'Process Modal'
 *  width:      800
 *  vertical:   false
 *  link:       false
 *  submitText  提交
 *  exitText    退出
 *
 *  可注入的数据
 *  modal
 *  steps
 *
 *  可注入的方法
 *  preview
 *  next
 *  jump
 *  broadcast
 *  close
 *  safeClose
 */

import step1 from './step/step1.vue'
import step2 from './step/step2.vue'
import step3 from './step/step3.vue'
import process from '../index.vue'

const steps = [{
  name: 'step1',
  title: '开始',
  disabled: true,
  next: true
}, {
  name: 'step2',
  title: '配置',
  // preview: false
}, {
  name: 'step3',
  title: '结束',
  submit: true
}]

export default {
  extends: process,
  components: {
    step1,
    step2,
    step3
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
      default: false
    },
    link: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      steps,
      // 共享数据源
      modal: {
        step1: {
          title: [{name: 'michael'}]
        }
      },
      // 当前视图
      currentView: 'step1'
    }
  },
  methods: {
    // 弹框打开时触发
    reload () {},
    // 强制退出
    forceExit () {
      // 重置数据
      this.resetModal()
      // 时机自行选择
      this.exit()
    },
    // 弹框关闭触发
    reset () {}
  }
}
</script>