<template>
  <div class="el-view-select" :class="selectClass">
    <el-select
      ref="select"
      v-model="innerValues"
      popper-class="view-select-option"
      style="width: 100%;"
      multiple
      clearable
      @remove-tag="_selectRemoveTag"
      @clear="_selectClearFun"
      @focus="_popoverShowFun"
    >
      <el-option
        v-for="item in selectOptions"
        :key="item.id"
        :label="item.name"
        :value="item.id"
      />
    </el-select>

    

    
    <el-dialog
      :visible="dialogShow"
      :show-close="false"
      class="dialog-css"
      :fullscreen="true"
    >
      <div ref="contaninerDiv" :style="{'height': panelHeight + 'px'}" v-if="dialogShow && viewLoaded">
        <Preview
          :component-data="componentData"
          :canvas-style-data="canvasStyleData"
          :panel-info="panelInfo"
          :show-position="showPosition"
        />
      </div>
      
      <div slot="title" class="dialog-footer title-text">
        <span style="font-size: 14px;">
          选择视图
        </span>
        <span style="float: right;">
          <el-button type="primary" size="mini" @click="closeDialog()">{{ $t('commons.confirm') }}</el-button>
          <el-button size="mini" @click="cancelDialog()">{{ $t('commons.cancel') }}</el-button>
        </span>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import Preview from '@/components/canvas/components/Editor/Preview'
import { findOne } from '@/api/panel/panel'
import { viewOptions } from '@/api/chart/chart'
import { panelDataPrepare } from '@/components/canvas/utils/utils'
export default {
  name: 'DeViewSelect',
  components: { Preview },
  props: {
    value: {
      type: Array,
      default: () => []
    },
    panelId: {
      type: String,
      default: null
    }

  },
  data() {
    return {
      visible: false,
      placement: 'bottom',
      transition: 'el-zoom-in-top',
      width: 500,
      selectClass: 'my-top-class',
      innerValues: [],
      panelHeight: 450,
      showPosition: 'email-task',
      viewLoaded: false,
      selectOptions: [],
      dialogShow: false,
      idsBeforeOpen: []
    }
  },
  computed: {
    popperClass() {
      const _c = 'el-view-select-popper ' + this.popoverClass
      return this.disabled ? _c + ' disabled ' : _c
    },
    selectedViews() {
      return this.$store.getters.panelViews[this.panelId]
    }
  },
  watch: {
    value(val, old) {
      this.innerValues = val
    },
    innerValues(val, old) {
      if (val !== old) {
        this.$emit('input', val)
      }
    },
    panelId(val, old) {
      if (val !== old) {
        this.innerValues = []
        this.loadView()
      }
    },
    selectedViews: {
      handler(val) {
        if (!val || !JSON.stringify(val)) {
          this.innerValues = []
          return
        }
        const viewIds = JSON.parse(JSON.stringify(val))

        this.innerValues = JSON.parse(JSON.stringify(viewIds))
      },
      deep: true
    }
  },
  mounted() {
    this._updateH()
    
  },
  beforeDestroy() {
    this._selectClearFun()
    
  },
  created() {
    this.loadView()
  },
  methods: {
    loadView() {
      this._selectClearFun()
      this.innerValues = this.value
      this.viewLoaded = false
      this.panelId && findOne(this.panelId).then(response => {
        this.panelInfo = {
          id: response.data.id,
          name: response.data.name,
          privileges: response.data.privileges,
          sourcePanelName: response.data.sourcePanelName,
          status: response.data.status,
          createBy: response.data.createBy,
          createTime: response.data.createTime,
          updateBy: response.data.updateBy,
          updateTime: response.data.updateTime
        }
        this.$store.dispatch('panel/setPanelInfo', this.panelInfo)
        panelDataPrepare(JSON.parse(response.data.panelData), JSON.parse(response.data.panelStyle), rsp => {
          this.viewLoaded = true
          this.componentData = rsp.componentData
          this.canvasStyleData = rsp.componentStyle
          this.loadOptions()
        })
      })
    },

    loadOptions() {
      this.panelId && viewOptions(this.panelId).then(res => {
        this.selectOptions = res.data
        this.init()
      })
    },
    _updateH() {
      this.$nextTick(() => {
        if(this.$refs.contaninerDiv) {
          this.width = this.$refs.contaninerDiv.clientWidth
          this.panelHeight = this.width * 9 / 16
        }
        
      })
    },
    _popoverShowFun(val) {
      this.openDialog()
      this._updateH()
      this.$emit('onFoucs')
    },
    
    
    _selectRemoveTag(viewId) {
      this.selectedViews.forEach(item => {
        if (item === viewId) {
          this.$store.dispatch('task/delView', { 'panelId': this.panelId, 'viewId': item })
        }
      })
    },
    _selectClearFun() {
      this.$store.dispatch('task/delPanelViews', this.panelId)
    },
    init() {
      if (this.value && this.value.length) {
        const viewIds = JSON.parse(JSON.stringify(this.value))
        this.$store.dispatch('task/initPanelView', { 'panelId': this.panelId, 'viewIds': viewIds })
      }
    },
    openDialog() {
      if (this.value && this.value.length) {
        this.idsBeforeOpen =  JSON.parse(JSON.stringify(this.value))
      }
      this.dialogShow = true
    },
    closeDialog() {
      this.dialogShow = false
    },
    cancelDialog() {
      this.innerValues = JSON.parse(JSON.stringify(this.idsBeforeOpen))
      const viewIds = JSON.parse(JSON.stringify(this.innerValues))
      this.$store.dispatch('task/initPanelView', { 'panelId': this.panelId, 'viewIds': viewIds })
      this.closeDialog()
    }
  }

}
</script>

<style lang="scss" scoped>
.el-view-select .view-select-option {
  display: none !important;
}

.my-top-class {
  width: 100%;
}
.dialog-css ::v-deep .el-dialog__title {
  font-size: 14px;
}

.dialog-css ::v-deep .el-dialog__header {
  padding: 20px 20px 0;
}

.dialog-css ::v-deep .el-dialog__body {
  padding: 10px 20px 20px;
}
</style>
