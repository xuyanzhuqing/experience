<!--
  入门级流程弹框
-->
<script>
import step1 from './step/step1.vue'
import step2 from './step/step2.vue'
import step3 from './step/step3.vue'
import process from '../index.vue'

const steps = [{
  name: 'step1',
  title: '开始'
}, {
  name: 'step2',
  title: '配置',
}, {
  name: 'step3',
  title: '结束'
}]

export default {
  extends: process,
  components: {
    step1,
    step2,
    step3
  },
  data () {
    return {
      steps,
      currentView: 'step1'
    }
  }
}
</script>