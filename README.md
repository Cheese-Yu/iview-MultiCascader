# iview-MultiCascader 多选级联选择组件

* 基于[iView-Cascader](https://www.iviewui.com/components/cascader)的多选级联选择组件

### 预览

![预览](https://github.com/Cheese-Yu/iview-MultiCascader/blob/master/img.jpeg)

### 使用
1. 将MultiCascader.vue的文件复制到你的项目中
2. 在要使用的文件内添加以下代码
```
import MultiCascader from './MultiCascader.vue(文件路径)';
```
3. 使用
```
<MultiCascader ref="addMulti" v-bind="config" @son="changeSon" hasValue :firstOpen="open" v-model="formChange.scope" @on-change="setChangeScope"></MultiCascader>

export default {
  components: {
    MultiCascader
  },
  data () {
    return {
      config: {
        clearable: true,
        multiple: true,
        data: [],
        placeholder: '请选择投放范围',
        style: "width:200px"
      },
      formChange: {
        id: '',
        status: false,
        scope: [],
        end_date: ''
      },
      open: false
    }
  },
  methods: {
    setChangeScope (value) {
      let self = this;
      let data = [];
      if (value) {
        for (let i = 0; i < value.length; i++) {
          data.push(value[i].value)
        }
        self.formChange.scope = data
      }
    },
    changeSon () {
      this.open = false
    }
  }
}
```
