<style lang="less">
  .MultiCascader {
    width: 100%;
    position: relative;
  }
  .placeholder {
    color: #cccccc
  }
  .selection-container {
    margin-bottom: 10px;
    border: 1px solid #dddee1;
    width: 100%;
    border-radius:  4px;
    min-height: 40px;
    padding: 5px;
    position: relative;
    cursor: pointer;
    background-color: white;
    transition: border 0.2s ease-in-out, background 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
  }
  .selection {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    width: 100%;
    padding-right: 5px;
  }
</style>


<template>
  <div :class="classes">
    <div
        :class="selectionCls"
        ref="reference">
      <slot name="input">
        <input type="hidden" :name="name" :value="model">
        <Cascader :data="data"
                  style="display: inline-block; width: 100%"
                  v-model="cascader"
                  change-on-select
                  @on-visible-change="toggleMenu"
                  @on-change="(value, selectedData) => handleChange(value, selectedData)" transfer>
          <span v-show="noSel && !filterable" :class="[prefixCls + '-placeholder']" style="display: inline">{{ placeholder }}</span>
          <div v-if="noSel" class="ivu-tag" style="background: none;"></div>
          <div class="ivu-tag" v-for="(item, index) in selData" :key="item.value" style="display: inline-block;background: #efefef">
            <span class="ivu-tag-text">{{ item.name }}</span>
            <span class="ivu-tag-text">{{ item.label }}</span>
            <Icon type="ios-close-empty" @click.native.stop="removeTag(index)"></Icon>
          </div>
        </Cascader>
        <Icon type="ios-close" :class="[prefixCls + '-arrow']" v-show="showCloseIcon" @click.native.stop="clearSelect"></Icon>
        <Icon type="arrow-down-b" :class="[prefixCls + '-arrow']" v-if="!remote"></Icon>
      </slot>
    </div>
  </div>
</template>

<script>
  const prefixCls = 'ivu-select';
  export default {
    props: {
      data: {
        type    : Array,
        default : ()=>[],
      },
      value: {
        type: [String, Number, Array],
        default: ''
      },
      valueField: {
        type: String,
        default: 'value',
      },

      label: {
        type: [String, Number, Array],
        default: ''
      },
      multiple: {
        type: Boolean,
        default: true
      },
      disabled: {
        type: Boolean,
        default: false
      },
      clearable: {
        type: Boolean,
        default: false
      },
      placeholder: {
        type: String
      },
      filterable: {
        type: Boolean,
        default: false
      },
      filterMethod: {
        type: Function
      },
      remote: {
        type: Boolean,
        default: false
      },
      remoteMethod: {
        type: Function
      },
      loading: {
        type: Boolean,
        default: false
      },
      loadingText: {
        type: String
      },
      size: {
        // validator (value) {
        //     return oneOf(value, ['small', 'large', 'default']);
        // }
      },
      labelInValue: {
        type: Boolean,
        default: false
      },
      firstOpen: {
        type: Boolean,
        default: true
      },
      notFoundText: {
        type: String
      },
      placement: {
        // validator (value) {
        //     return oneOf(value, ['top', 'bottom']);
        // },
        default: 'bottom'
      },
      transfer: {
        type: Boolean,
        default: false
      },
      hasValue: {
        type: Boolean,
        default: false
      },
      // Use for AutoComplete
      autoComplete: {
        type: Boolean,
        default: false
      },
      name: {
        type: String
      },
      elementId: {
        type: String
      },
      renderFormat: {
        type: Function,
        default: (labels) => labels.join(' / ')
      }
    },
    data () {
      return {
        cascader: [],
        selData: [],
        propData: [],
        noSel: true,
        addTag: true,
//        firstOpen: true,
        // 转换后的选项
        prefixCls: prefixCls,
        visible: false,
        options: [],
        optionInstances: [],
        selectedSingle: '',    // label
        // selectedMultiple: [],
        focusIndex: 0,
        query: '',
        lastQuery: '',
        selectToChangeQuery: false,    // when select an option, set this first and set query, because query is watching, it will emit event
        inputLength: 20,
        notFound: false,
        slotChangeDuration: false,    // if slot change duration and in multiple, set true and after slot change, set false
        model: this.value,
        currentLabel: this.label
      }
    },
    watch: {
      value: function value(val) {
        if (this.hasValue) {
          if (this.firstOpen) {
            this.setTag()
            this.$emit('son',false);
          }
        }
      }
    },
    computed: {
      classes () {
        return [
          `${prefixCls}`,
          {
            [`${prefixCls}-visible`]: this.visible,
            [`${prefixCls}-disabled`]: this.disabled,
            [`${prefixCls}-multiple`]: this.multiple,
            [`${prefixCls}-single`]: !this.multiple,
            [`${prefixCls}-show-clear`]: this.showCloseIcon,
            [`${prefixCls}-${this.size}`]: !!this.size
          }
        ];
      },
      selectionCls () {
        return {
          [`${prefixCls}-selection`]: !this.autoComplete
        };
      },
      showCloseIcon () {
        return this.multiple && this.clearable;
//        return this.multiple && this.clearable && !this.noSel;
      }
    },
    methods:{
      dispatch(componentName, eventName, params) {
        var parent = this.$parent || this.$root;
        var name = parent.$options.name;

        while (parent && (!name || name !== componentName)) {
          parent = parent.$parent;

          if (parent) {
            name = parent.$options.name;
          }
        }
        if (parent) {
          parent.$emit.apply(parent, [eventName].concat(params));
        }
      },
      setTag() {
        let self = this;
        let value = self.value;
        let data = [];
        if (value) {
          for (let i = 0; i < value.length; i++) {
            let ob = {value: value[i].id, label: value[i].name}
            data.push(ob);
          }
        }
        if (data.length > 0) {
          self.noSel = false;
        }
        else self.noSel = true;
        self.selData = data.concat()
        self.propData = data.concat()
        self.setValue(self.propData);
      },
      // 设置值
      setValue(val) {
        this.$nextTick(() => {
          this.$emit('input', val);
          this.$emit('on-change', val);
          this.dispatch('FormItem', 'on-form-change', val);
        });
      },
      clearSelect() {
        let self = this;
        self.selData =[]
        self.propData =[]
        self.noSel = true;
        self.setValue([])
      },
      getValue() {
        let self = this;
        return self.selData
      },
      removeTag(index) {
        let self = this;
        self.selData.splice(index, 1);
        self.propData.splice(index, 1);
        self.setValue(self.propData)
        if (self.selData.length == 0) {
          self.noSel = true;
        }
      },
      isInArray (arr,value){
        for(var i = 0; i < arr.length; i++){
          if(value.value == arr[i].value){
            return true;
          }
        }
        return false;
      },
      handleChange (value, selectedData) {
        let self = this;
        let val = selectedData[selectedData.length-1];
        let _val = {value:val.value, label: val.label};
        if (!self.isInArray(self.selData, val)) {
          if (self.addTag) {
            self.addTag = false;
            self.noSel = false;
            self.selData.push(val);
            self.propData.push(_val);
            self.setValue(self.propData);
          }
          else {
            self.selData[self.selData.length-1] = val
            self.propData[self.propData.length-1] = _val
            self.setValue(self.propData);
          }
        }
      },
      toggleMenu (visible) {
        let self = this;
        if (this.disabled || this.autoComplete) {
          return false;
        }
        // 关闭
        if (!visible) {
          self.addTag = true;
          self.cascader = []
        }
        this.visible = !this.visible;
      },

    }
  }
</script>