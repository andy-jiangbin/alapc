<template>
  <div v-if="async" :style="'background:'+viewModel.setting.background+';'">
    <div v-for="(widget,index) in viewModel.widgets" :key="index" :style="widget.style && widget.style.css" :class="widget.border?widget.border.class:''+ ' '+ widget.blockList">
      <template v-if="widget.status !== 'small'">
        <div v-if="widget.border&& widget.border.show===true">
          <div class="border-header">
            <x-icon class="border-header-icon" :color="widget.border.fontColor" v-if="widget.border.icon" :icon="widget.border.icon"></x-icon>
            <div class="border-header-title" :style="'color:'+widget.border.fontColor">{{widget.border.title}}</div>
            <div class="border-header-desc" :style="'color:'+widget.border.fontColor">{{widget.border.intro}}</div>
            <div class="border-header-tools"></div>
          </div>
          <div class="border-body">
            <component :is="widget.name" :widget="widget" :widgetData="widget" :title="widget.title" ref="childComponent" />
          </div>
        </div>
        <component v-else :is="widget.name" :widget="widget" :widgetData="widget" :title="widget.title" ref="childComponent" />
      </template>
      <div v-if="widget.layout==='tab-layer'">
        <el-tabs v-model="tabName">
          <el-tab-pane :label="columns.option.name" :name="columnsIndex" v-for="(columns,columnsIndex) in widget.columns" :key="columnsIndex">
            <div v-for="(tabWidget,tabWidgetIndex) in columns.widgets" :key="tabWidgetIndex">
              <component :is="tabWidget.name" :title="tabWidget.title" :widgetData="tabWidget" :widget="tabWidget" ref="childComponent" />
            </div>
          </el-tab-pane>
        </el-tabs>
      </div>
      <!-- grid-proutelayer -->
      <div class="grid-layout" v-if="widget.layout==='grid-layer'">
        <el-row :gutter="Number(widget.value.gutter)">
          <el-col :span="Number(widget.value.cols[layoutIndex].span)" v-for="(tablayout,layoutIndex) in widget.columns" :key="layoutIndex">
            <div v-if="tablayout.columns !== null">
              <div v-for="(tabWidget, index) in tablayout.columns" :key="index">
                <component :is="tabWidget.name" :title="tabWidget.title" :widgetData="tabWidget" :widget="tabWidget" ref="childComponent" />
              </div>
            </div>
          </el-col>
        </el-row>
      </div>
      <div class="fullsreen-layout" style="width:100%" v-if="widget.layout==='fullsreen-layer'">
      </div>
    </div>
  </div>
</template>

<script>
  import service from './service'
  import theme from '@/service/core/theme'
  export default {
    name: 'x-widget',
    props: {
      type: {} // 类型包括Front,Admin,City，Store等
    },
    data () {
      return {
        viewModel: {},
        tabName: 0,
        async: false
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      async init () {
        this.async = false
        this.viewModel = await theme.page(this.$route, this.type)
        if (this.type === 'admin') {
          if (!this.viewModel) {
            this.viewModel = service.setWidget(this)
          }
          if (!this.viewModel) {
            this.$admin.message('您输入的网址不正确,页面不存在', 'danger', '页面不存在')
            return
          } else {
            await service.checkAdminRole(this.viewModel)
          }
        }
        this.async = true
        if (!this.$base.isBuild()) {
          console.info('页面信息:', this.viewModel)
        }
        this.watchWidget(this.viewModel)
      },
      // 监听子组件中的数据，如果需要触发可以在子组件中定义watchWidget方法
      // 具体的用法可以参考admin-auto-table admin-product-list等组件的实现
      watchWidget (val) {
        this.$nextTick(() => {
          if (this.$refs.childComponent) {
            this.$refs.childComponent.forEach(element => {
              if (element.watchWidget) {
                element.watchWidget(val)
              }
            })
          }
        })
      },
      watchRoute () {
        this.init()
      }
    },
    watch: {
      $route: 'watchRoute'
    }
  }
</script>
