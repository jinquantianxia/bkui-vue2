<!--
 * Tencent is pleased to support the open source community by making
 * 蓝鲸智云PaaS平台社区版 (BlueKing PaaS Community Edition) available.
 *
 * Copyright (C) 2021 THL A29 Limited, a Tencent company.  All rights reserved.
 *
 * 蓝鲸智云PaaS平台社区版 (BlueKing PaaS Community Edition) is licensed under the MIT License.
 *
 * License for 蓝鲸智云PaaS平台社区版 (BlueKing PaaS Community Edition):
 *
 *
 * Terms of the MIT License:
 * ---------------------------------------------------
 * Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated
 * documentation files (the "Software"), to deal in the Software without restriction, including without limitation
 * the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and
 * to permit persons to whom the Software is furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in all copies or substantial portions of
 * the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT
 * THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF
 * CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
 * IN THE SOFTWARE.
-->

<template>
  <div class="bk-cascade"
    :class="{
      'is-focus': showCascade,
      'is-disabled': disabled,
      'is-unselected': isUnselected,
      'is-default-trigger': !$scopedSlots.trigger,
      'is-custom-trigger': $scopedSlots.trigger
    }"
    tabindex="0"
    :data-placeholder="placeContent"
    @keydown.enter.prevent="showTippyInstance"
    @keydown.tab="handleDropdownHide"
    @keydown.esc.stop.prevent="handleDropdownHide">
    <i v-if="!$scopedSlots.trigger" class="bk-cascade-angle bk-icon icon-angle-down"></i>
    <i class="bk-cascade-clear bk-icon icon-close"
      v-if="clearable && !disabled && selectedName && !$scopedSlots.trigger"
      @click.prevent.stop="clearData">
    </i>
    <bk-popover
      style="max-width: 100%;"
      class="bk-cascade-dropdown"
      ref="cascadeDropdown"
      trigger="click"
      placement="bottom-start"
      theme="light bk-cascade-dropdown"
      animation="slide-toggle"
      :distance="12"
      :on-show="handleDropdownShow"
      :on-hide="handleDropdownHide"
      :tippy-options="{ ...{ offset: '0 3' }, ...popoverOptions }">
      <slot name="trigger" :selected="selectedList" :is-show="showCascade">
        <template v-if="multiple">
          <div class="bk-cascade-name" :title="selectedName" v-if="limitOneLine">
            <span>{{selectedName}}</span>
          </div>
          <section v-else>
            <div class="bk-cascade-tag-list" v-if="multipleSelectedList.length || !filterable">
              <span v-for="(item, index) in multipleSelectedList"
                :key="index"
                class="bk-cascade-tag-item" style="padding-right: 18px;">
                <span class="bk-cascade-item-name" :title="item.name">{{item.name}}</span>
                <a href="javascript:void(0)" class="remove-key" @click.stop="removeTag(item, index)" tabindex="-1">
                  <i class="bk-icon icon-close"></i>
                </a>
              </span>
            </div>
          </section>
          <div class="bk-cascade-name" v-if="filterable">
            <input
              class="bk-cascade-search-input"
              ref="searchInput"
              type="text"
              :placeholder="t('bk.select.searchPlaceholder')"
              @input="handleSearchInput"
              v-model="searchContent">
          </div>
        </template>
        <div class="bk-cascade-name" :title="selectedName" v-else>
          {{/*  changeImmediately 开启时，要求 filterable 为 ture，且未选择； 默认情况下只要 filterable 为 true，就打开搜索，不破坏原有逻辑  */}}
          <input
            v-if="changeImmediately ? filterable && currentList.length === 0 : filterable"
            class="bk-cascade-search-input"
            ref="searchInput"
            type="text"
            :placeholder="t('bk.select.searchPlaceholder')"
            @input="handleSearchInput"
            v-model="searchContent">
          <span v-else>{{selectedName}}</span>
        </div>
      </slot>
      <div slot="content"
        :class="['bk-cascade-dropdown-content',extPopoverCls]"
        v-show="!disabled"
        :style="dropdownContentStyle"
        ref="refDropContentWrapper"
      >
        <div class="bk-cascade-options-wrapper">
          <div class="bk-cascade-options" :style="dropdownOptionsStyle">
            <div class="bk-cascade-panel" v-if="filterable && !!searchContent && filterableStatus">
              <ul class="bk-cascade-panel-ul" style="width: 100%">
                <li
                  class="bk-option"
                  v-for="(item, index) in searchList"
                  :key="index"
                  @click.prevent.stop="handleSelectItem(item, index)"
                  :class="{
                    'is-selected': item.isSelected,
                    'is-disabled': item.disabled
                  }"
                >
                  <div class="bk-option-content">
                    <slot>
                      <div class="bk-option-content-default" :title="item.name">
                        <span class="bk-option-name">{{ item.name }}</span>
                      </div>
                    </slot>
                  </div>
                </li>
                <li class="bk-option-none" v-if="!searchList.length">
                  <span>{{ emptyText || t("bk.select.searchEmpty") }}</span>
                </li>
              </ul>
            </div>
            <bk-caspanel
              v-show="!filterableStatus"
              ref="refDropdownContent"
              :list="cascadeList"
              :trigger="trigger"
              :scroll-width="scrollWidth"
              :disabled="disabled"
              :check-any-level="checkAnyLevel"
              :filterable="filterable"
              :multiple="multiple"
              :is-remote="isRemote"
              :remote-method="remoteMethod"
              :allow-disabled-expand="allowDisabledExpand"
              @updateSelectedList="updateSelectedList"
            >
              <template slot="option" slot-scope="{ node }">
                <slot name="option" :node="node">{{ node.name }}</slot>
              </template>
              <template slot="prepend" slot-scope="{ node }">
                <slot name="prepend" :node="node"></slot>
              </template>
            </bk-caspanel>
          </div>
        </div>
        <div class="bk-cascade-extension" v-if="$slots.extension">
          <slot name="extension"></slot>
        </div>
      </div>
    </bk-popover>
  </div>
</template>

<script>
import bkPopover from '@/components/popover'
import bkCaspanel from './caspanel.vue'
import cascadeInfo from './cascade.js'
import locale from 'bk-magic-vue/lib/locale'

export default {
  name: 'bkCascade',
  components: {
    bkPopover,
    bkCaspanel
  },
  mixins: [cascadeInfo, locale.mixin],
  props: {
    list: {
      type: Array,
      default: () => []
    },
    value: {
      type: Array,
      default: () => []
    },
    changeImmediately: {
      type: Boolean,
      default: false
    },
    placeholder: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    },
    scrollWidth: {
      type: Number,
      default: 160
    },
    scrollHeight: {
      type: Number,
      default: 216
    },
    clearable: {
      type: Boolean,
      default: false
    },
    checkAnyLevel: {
      type: Boolean,
      default: false
    },
    showCompleteName: {
      type: Boolean,
      default: true
    },
    filterable: {
      type: Boolean,
      default: false
    },
    multiple: {
      type: Boolean,
      default: false
    },
    limitOneLine: {
      type: Boolean,
      default: false
    },
    separator: {
      type: String,
      default: ' / '
    },
    trigger: {
      validator (value) {
        return ['click', 'hover'].includes(value)
      },
      default: 'click'
    },
    emptyText: {
      type: String,
      default: ''
    },
    isRemote: {
      type: Boolean,
      default: false
    },
    remoteMethod: {
      type: Function
    },
    popoverOptions: {
      type: Object,
      default: () => ({})
    },
    options: {
      type: Object,
      default: () => ({})
    },
    extPopoverCls: {
      type: String,
      default: ''
    },
    maxWidth: {
      type: [String, Number],
      default: '100%'
    },
    allowDisabledExpand: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      // popover需要展开的宽度
      popoverWidth: 1,
      historyPopover: 0,
      showCascade: false,
      defaultWidth: 0,
      // 搜索内容
      searchContent: '',
      // 搜索List
      searchList: [],
      // 数据
      cascadeList: [],
      // 单选选中数据
      selectedList: [],
      // 每次点击返回的数据
      tmpSelected: [],
      // 单选选中的idList
      currentList: [],
      // 多选选中的数据
      multipleSelectedList: [],
      multipleCurrentList: [],
      // 搜索弹窗参数
      filterableStatus: false,
      filterRestoreStatus: false,
      remoteSearchLoading: false
    }
  },
  computed: {
    selectedName () {
      let label = []
      if (this.multiple) {
        label = this.multipleSelectedList.map((item) => item.name)
      } else {
        if (!this.showCompleteName && this.selectedList.length) {
          label = [this.selectedList[this.selectedList.length - 1].name]
        } else {
          label = this.selectedList.map((item) => item.name)
        }
      }
      return this.multiple ? label.join(' , ') : label.join(this.separator)
    },
    isUnselected () {
      return this.selectedName === ''
    },
    placeContent () {
      return this.placeholder ? this.placeholder : this.t('bk.select.pleaseselect')
    },
    nodeOptions () {
      const nodeOptions = {
        idKey: 'id',
        nameKey: 'name',
        childrenKey: 'children'
      }
      return Object.assign(nodeOptions, this.options)
    },
    dropdownContentWidth () {
      return (this.filterable && !!this.searchContent && this.filterableStatus && !this.searchList.length) || !this.cascadeList.length
        ? this.defaultWidth
        : this.scrollWidth * this.popoverWidth + 2
    },
    dropdownContentStyle () {
      const maxWidth = /^\d+$/.test(this.maxWidth) ? `${this.maxWidth}px` : this.maxWidth
      const width = this.dropdownContentWidth + 'px'
      return {
        maxWidth,
        width,
        overflowX: 'auto'
      }
    },
    dropdownOptionsStyle () {
      return {
        width: this.dropdownContentWidth - 2 + 'px',
        height:
          ((this.filterable && !!this.searchContent && this.filterableStatus && !this.searchList.length) || !this.cascadeList.length
            ? 32
            : this.scrollHeight) + 'px'
      }
    }
  },
  watch: {
    showCascade (val) {
      if (!val) {
        // 通知表单组件，可用于实时验证
        this.dispatch('bk-form-item', 'form-blur')
      }
      this.$emit('toggle', val)
      this.filterableStatus = this.searchContent !== ''
      if (val) {
        this.popoverWidth = 1
        if (this.currentList.length) {
          this.updateSelected()
        }
      } else {
        if (this.filterable && !this.multiple) {
          if (this.searchContent !== this.selectedName) {
            this.filterRestoreStatus = true
          }
          this.searchContent = this.selectedName
        }
        this.tippyInstance()
      }
      this.filterableStatus = false
    },
    searchContent (val) {
      if (this.filterRestoreStatus) {
        this.filterRestoreStatus = false
        return
      }
      if (val) {
        this.filterableStatus = true
        this.$refs.cascadeDropdown.$refs.reference._tippy.show()
        this.handleDropdownShow()
        this.getSearchList()
      } else {
        this.filterableStatus = false
        if (this.multiple) {
          // 改变选框的选中态
          this.recursiveList(this.cascadeList, this.multipleCurrentList, 'search')
        }
      }
    },
    list: {
      handler (newValue) {
        // watch list变化改变cascadeList的值
        this.cascadeList = this.recurrenceNodes(JSON.parse(JSON.stringify(newValue)))
        // 当list发生变化时，显示的数据也要做相应的改变
        if (this.multiple) {
          this.recursiveList(this.cascadeList, this.multipleCurrentList, 'init')
          this.checkListStatus(this.multipleCurrentList)
          const newList = this.changeList()
          this.multipleSelectedList = newList
            .filter((item) => item.isSelected)
            .map((item) => {
              return {
                id: item.id,
                name: item.name
              }
            })
        } else {
          // 通过递归找到对应的项(如果不存在对应的项，则清空当前的数据)
          if (this.currentList.length) {
            let findStatus = false
            const checkInfo = (arr) => {
              arr.forEach((itemArr) => {
                if (itemArr.id === this.currentList[this.currentList.length - 1]) {
                  findStatus = true
                }
                if (itemArr.children && itemArr.children.length) {
                  checkInfo(itemArr.children)
                }
              })
            }
            checkInfo(this.cascadeList)
            if (!findStatus) {
              this.currentList = []
              this.selectedList = []
              this.tmpSelected = []
            }
          }
        }
      },
      deep: true
    },
    value: {
      handler () {
        this.initValue()
      },
      deep: true
    }
  },
  created () {
    this.cascadeList = this.recurrenceNodes(JSON.parse(JSON.stringify(this.list)))
    // 单选
    this.$on('on-id-change', (params) => {
      const { isLast, checkAnyLevel, fromInit } = params

      if (isLast || checkAnyLevel || fromInit) {
        const oldIdList = JSON.parse(JSON.stringify(this.currentList))
        this.selectedList = this.tmpSelected

        const newIdList = this.selectedList.map((item) => item.id)

        if (!fromInit) {
          this.currentList = newIdList
          this.exposedId(this.currentList, oldIdList)
        }
      }
      if (isLast && !fromInit) {
        this.tippyInstance()
      }
    })
    // 多选
    this.$on('on-multiple-change', (params) => {
      const { checkAnyLevel, fromInit } = params
      this.selectedList = this.tmpSelected
      const newIdList = this.selectedList.map((item) => item.id)
      if (!fromInit) {
        this.currentList = newIdList
      }
      // 多选触发cascade选中的状态
      if (!checkAnyLevel) {
        this.cascadeList.forEach((item) => {
          if (item.children && item.children.length) {
            item.isSelected = item.children.every((child) => child.isSelected || child.disabled)
            item.isIndeterminate = item.children.some((child) => child.isIndeterminate || child.isSelected)
          }
        })
      }
      const oldIdList = JSON.parse(JSON.stringify(this.multipleCurrentList))
      this.exposeMultiple(oldIdList)
    })
    // cascade的层级（popoverWidth的值控制）
    this.$on('on-popover-width', (params) => {
      const currentItem = params.item
      this.popoverWidth = this.tmpSelected.length
      if (currentItem.children && currentItem.children.length) {
        this.popoverWidth += 1
      }
    })

    this.$on('on-cascade-click', () => {
      this.handleScrollToRight()
    })
  },
  mounted () {
    this.initValue()
    // 暂时将远程加载中的搜索去掉
    if (this.isRemote) {
      this.filterable = false
    }
    // 开启了filterable，单选下的display与srachContent是一样的。因此初始化要回填
    if (this.filterable && !this.multiple) {
      this.searchContent = this.selectedName
      this.$nextTick(() => {
        this.tippyInstance() // 回填不需要打开下拉，关闭
      })
    }
  },
  methods: {
    recurrenceNodes (list) {
      if (!Array.isArray(list) || !list.length) {
        return []
      }
      const { idKey, nameKey, childrenKey } = this.nodeOptions
      // 默认赋值
      list.forEach((item, index) => {
        // 修复数字类型 多选不展示问题：因为整个组件 基于  join 与split 方法，如：changeList 方法
        // item.id = item[idKey]
        item.id = this.multiple ? String(item[idKey]) : item[idKey]
        // 这里不可以使用 `item[nameKey] || ''`， 因为如果是 0，将变为空字符串无法渲染； 同时因暂不支持`??`语法，而暂时如此改之
        item.name = (item[nameKey] !== null && item[nameKey] !== undefined) ? item[nameKey] : ''
        const children = item[childrenKey]
        if (Array.isArray(children)) {
          item.children = this.recurrenceNodes(children)
        }
      })
      return list
    },
    initValue () {
      // 如果是多选内置需要的数据
      if (this.multiple) {
        this.multipleCurrentList = this.value
        this.changeValueList()
      } else {
        this.currentList = this.value
        if (this.value.length) {
          this.updateSelected()
        } else {
          this.clearData()
        }
      }
    },
    // 清空数据
    clearData () {
      if (this.disabled) return
      if (this.multiple) {
        const oldId = JSON.parse(JSON.stringify(this.multipleCurrentList))
        this.multipleCurrentList = []
        this.multipleSelectedList = []
        this.exposedId(this.multipleCurrentList, oldId)
        // 递归修改选中和半选的状态
        const changeCheckStatus = (arr) => {
          arr.forEach((item) => {
            item.isSelected = false
            item.isIndeterminate = false
            if (item.children && item.children.length) {
              changeCheckStatus(item.children)
            }
          })
        }
        changeCheckStatus(this.cascadeList)
      } else {
        const oldId = JSON.parse(JSON.stringify(this.currentList))
        this.currentList = []
        this.selectedList = []
        this.tmpSelected = []
        this.exposedId(this.currentList, oldId)
        this.searchContent = ''
      }
      this.broadcast('bkCaspanel', 'on-clear')
      // 关闭下拉面板
      this.tippyInstance()
    },
    // popover面板的展开收起回调
    handleDropdownShow () {
      this.popoverWidth = 1
      this.defaultWidth = this.$el.offsetWidth
      if (this.currentList.length) {
        this.updateSelected()
      }
      if (this.disabled) return
      this.showCascade = true
      if (!this.currentList.length) {
        this.broadcast('bkCaspanel', 'on-clear')
      }
    },
    handleDropdownHide () {
      this.showCascade = false
    },
    tippyInstance () {
      if (this.$refs.cascadeDropdown) {
        this.$refs.cascadeDropdown.$refs.reference._tippy.hide()
      }
    },
    showTippyInstance () {
      if (this.$refs.cascadeDropdown) {
        this.$refs.cascadeDropdown.$refs.reference._tippy.show()
      }
    },
    // 当数据发生变化是触发change事件
    exposedId (newId, oldId) {
      if (JSON.stringify(newId) !== JSON.stringify(oldId)) {
        const selectedList = this.multiple ? this.multipleSelectedList : this.selectedList
        this.$emit('input', newId)
        this.$emit('change', newId, oldId, JSON.parse(JSON.stringify(selectedList)))
        // 通知表单组件，可用于实时验证
        this.dispatch('bk-form-item', 'form-change')
      }
    },
    exposeMultiple (oldId) {
      const newList = this.changeList()
      this.multipleSelectedList = []
      this.multipleCurrentList = []
      // 使用一次循环过滤元素
      for (let i = 0; i < newList.length; i++) {
        if (newList[i].isSelected) {
          this.multipleSelectedList.push({
            id: newList[i].id,
            name: newList[i].name
          })
        }
      }
      for (let i = 0; i < newList.length; i++) {
        if (newList[i].isSelected) {
          this.multipleCurrentList.push(newList[i].id)
        }
      }
      // this.multipleSelectedList = newList.filter(item => item.isSelected).map(item => {
      //     return {
      //         id: item.id,
      //         name: item.name
      //     }
      // })
      // this.multipleCurrentList = newList.filter(item => item.isSelected).map(item => item.id)

      // 抛出选中事件
      this.exposedId(this.multipleCurrentList, oldId)
    },
    // 更新选中的数据
    updateSelectedList (selectedList) {
      this.tmpSelected = selectedList
    },
    // 将数据赋值
    updateSelected () {
      this.broadcast('bkCaspanel', 'change-selected', {
        idList: this.currentList
      })
    },
    // 搜索功能
    getSearchList () {
      const selections = this.changeList()
      this.searchList = selections.filter((item) => {
        return item.name && item.name.indexOf(this.searchContent) > -1
      })
      if (!this.multiple) {
        this.searchList.forEach((item) => {
          item.isSelected = this.currentList.join(',') === item.id.join(',')
        })
      }
    },
    changeList () {
      const listInfo = JSON.parse(JSON.stringify(this.cascadeList))
      const selections = []
      const getSelections = (arr, id, name) => {
        arr.forEach((item) => {
          item.id = id ? id + this.separator + item.id : item.id
          item.name = name ? name + this.separator + item.name : item.name

          if (this.checkAnyLevel) {
            selections.push({
              id: String(item.id).split(this.separator),
              name: item.name,
              disabled: !!item.disabled,
              isSelected: !!item.isSelected
            })
            if (item.children && item.children.length) {
              getSelections(item.children, item.id, item.name)
            }
          } else {
            if (item.children && item.children.length) {
              getSelections(item.children, item.id, item.name)
              delete item.id
              delete item.name
            } else {
              if (!item.id.split) {
                return false
              }
              selections.push({
                id: item.id.split(this.separator),
                name: item.name,
                disabled: !!item.disabled,
                isSelected: !!item.isSelected
              })
            }
          }
        })
      }
      getSelections(listInfo)
      return selections
    },
    changeValueList () {
      // 赋值操作
      if (this.isRemote) {
        this.recursiveList(this.cascadeList, this.value, 'search')
      } else {
        this.recursiveList(this.cascadeList, this.value, 'init')
      }
      this.checkListStatus(this.value)

      const newList = this.changeList()
      this.multipleSelectedList = newList
        .filter((item) => item.isSelected)
        .map((item) => {
          return {
            id: item.id,
            name: item.name
          }
        })
    },
    recursiveList (arr, valueList, type, path = []) {
      arr.forEach((arrItem) => {
        if (type === 'init') {
          this.$set(arrItem, 'isSelected', false)
          this.$set(arrItem, 'isIndeterminate', false)
        }

        // 更新path数组，添加当前节点的id
        const newPath = path.concat(arrItem.id)

        valueList.forEach((item) => {
          if (Array.isArray(item)) {
            // 比较每一项是否和路径数组完全匹配
            if (item.length === newPath.length && item.every((child, index) => child === newPath[index])) {
              arrItem.isSelected = true
            }

            // 如果是中间状态的话，只匹配 item 中的一部分和路径的前缀
            if (!this.checkAnyLevel && item.length > newPath.length && item.slice(0, newPath.length).every((child, index) => child === newPath[index])) {
              item.forEach((child) => {
                if (child === arrItem.id) {
                  arrItem.isIndeterminate = true
                }
              })
            }
          }
        })

        if (arrItem.children && arrItem.children.length) {
          // 传递当前路径到子节点
          this.recursiveList(arrItem.children, valueList, type, newPath)
        }
      })
    },
    // 对比数据选中态
    checkListStatus (currentValue) {
      if (!this.checkAnyLevel) {
        const valueList = []
        currentValue.forEach((item) => {
          Array.isArray(item) && valueList.push(item.length)
        })
        const value = Math.max(...valueList)
        for (let i = 0; i <= value; i++) {
          const checkInfo = (arr) => {
            arr.forEach((arrItem) => {
              if (arrItem.children && arrItem.children.length) {
                arrItem.isSelected = arrItem.children.every((child) => child.isSelected || child.disabled)
                arrItem.isIndeterminate = arrItem.children.some((child) => child.isIndeterminate || child.isSelected)
                checkInfo(arrItem.children)
              }
            })
          }
          checkInfo(this.cascadeList)
        }
      }
    },
    handleSelectItem (item, index) {
      if (item.disabled) {
        return
      }
      if (this.multiple) {
        const oldId = JSON.parse(JSON.stringify(this.multipleCurrentList))
        item.isSelected = !item.isSelected
        if (item.isSelected) {
          this.multipleCurrentList.push(item.id)
          this.multipleSelectedList.push({
            id: item.id,
            name: item.name
          })
        } else {
          this.multipleCurrentList = this.multipleCurrentList.filter((node) => {
            return node.join('/') !== item.id.join('/')
          })
          this.multipleSelectedList = this.multipleSelectedList.filter((node) => {
            return node.id.join('/') !== item.id.join('/')
          })
        }
        this.recursiveList(this.cascadeList, this.multipleCurrentList, 'search')
        // 同步数据的选中态
        this.broadcast('bkCaspanel', 'multiple-selected', {
          idList: item.id,
          isSelected: item.isSelected
        })
        // 抛出事件
        this.exposeMultiple(oldId)
      } else {
        const oldVal = JSON.parse(JSON.stringify(this.currentList))
        this.currentList = item.id
        this.tmpSelected = []
        this.selectedList = []
        this.updateSelected()
        setTimeout(() => {
          this.searchContent = this.selectedName
          this.exposedId(this.currentList, oldVal)
          this.tippyInstance()
        }, 0)
      }
    },
    handleScrollToRight () {
      if (/^\d+(px|vh|vm|re?m)?$/.test(this.maxWidth)) {
        if (this.$refs.refDropdownContent && this.$refs.refDropdownContent.$el) {
          this.$nextTick(() => {
            const wrapper = this.$refs.refDropContentWrapper
            const target = this.$refs.refDropdownContent.$el
            const scrollWidth = target.clientWidth
            const parentWidth = wrapper.clientWidth
            const left = scrollWidth > parentWidth ? scrollWidth - parentWidth : 0
            wrapper.scrollTo({
              top: 0,
              left
            })
          })
        }
      }
    },
    // 多选删除数据
    removeTag (item, index) {
      if (this.disabled) {
        return
      }
      const oldId = JSON.parse(JSON.stringify(this.multipleCurrentList))
      this.multipleCurrentList = this.multipleCurrentList.filter((itemInfo) => itemInfo.join(',') !== item.id.join(','))
      this.multipleSelectedList = this.multipleSelectedList.filter((itemInfo) => itemInfo.id.join(',') !== item.id.join(','))
      const itemId = item.id[item.id.length - 1]
      // 同步数据的选中态
      const changeCheckStatus = (arr) => {
        arr.forEach((item) => {
          if (`${itemId}` === `${item.id}`) {
            item.isSelected = false
            item.isIndeterminate = false
          }
          if (item.children && item.children.length) {
            changeCheckStatus(item.children)
          }
        })
      }
      changeCheckStatus(this.cascadeList)
      this.checkListStatus(this.multipleCurrentList)
      // 抛出事件
      this.exposeMultiple(oldId)
    },
    // 搜索输入触发search事件
    handleSearchInput (event) {
      const value = event.target.value
      this.$emit('search', value, event)
    }
  }
}
</script>
<style>
@import "../../ui/cascade.css";
</style>
