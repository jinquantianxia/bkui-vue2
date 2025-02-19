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
  <div
    :style="{ height: treeHeight }"
    :class="{
      'bk-big-tree': true,
      [extCls]: true,
      'with-virtual-scroll': !!height,
      'bk-big-tree--small': size === 'small'
    }">
    <!-- 虚拟滚动 -->
    <bk-virtual-scroll
      :item-height="nodeHeight"
      ref="virtualScroll"
      v-if="height">
      <tree-item
        slot-scope="{ data: node }"
        :node="node"
        :id="`bk-big-tree-${id}-node-${node.id}`">
        <slot :node="node" :data="node.data" />
      </tree-item>
    </bk-virtual-scroll>
    <div :class="['tree-wrapper', { 'fixed-width': fixedWidth }]" style="height: 100%;" v-else>
      <!-- 非虚拟滚动 -->
      <template v-for="node in nodes">
        <tree-item
          :enable-title-tip="enableTitleTip"
          :node="node"
          :ref="node.id"
          :key="node.id">
          <slot :node="node" :data="node.data" />
        </tree-item>
      </template>
    </div>
    <div class="bk-big-tree-empty" v-if="($slots.empty || useDefaultEmpty) && isSearchEmpty">
      <slot name="empty">
        {{t('bk.bigTree.emptyText')}}
      </slot>
    </div>
  </div>
</template>

<script>
import TreeNode from './tree-node.js'
import { isNullOrUndefined, convertToArray } from './utils.js'
import locale from 'bk-magic-vue/lib/locale'
import bkVirtualScroll from '@/components/virtual-scroll'
import treeItem from './tree-item.vue'

let idSeed = 0

export default {
  name: 'bk-big-tree',
  components: {
    bkVirtualScroll,
    treeItem
  },
  mixins: [locale.mixin],
  props: {
    data: {
      type: Array,
      default () {
        return []
      }
    },
    options: {
      type: Object,
      default () {
        return {}
      }
    },
    fixedWidth: {
      type: Boolean,
      default: false
    },
    lazyMethod: Function,
    lazyDisabled: [Boolean, Function],
    selectable: Boolean,
    showCheckbox: [Boolean, Function],
    checkStrictly: {
      type: Boolean,
      default: true
    },
    checkOnlyAvailableStrictly: Boolean,
    disableStrictly: {
      type: Boolean,
      default: true
    },
    displayMatchedNodeDescendants: Boolean,
    showLinkLine: Boolean,
    expandIcon: {
      type: String,
      default: 'bk-icon icon-down-shape'
    },
    collapseIcon: {
      type: String,
      default: 'bk-icon icon-right-shape'
    },
    loadingClass: {
      type: String,
      default: 'node-loading'
    },
    nodeIcon: {
      type: [String, Function]
    },
    defaultExpandAll: Boolean,
    // 默认展开节点 id
    defaultExpandedNodes: {
      type: Array,
      default () {
        return []
      }
    },
    // 默认 check 节点 id
    defaultCheckedNodes: {
      type: Array,
      default () {
        return []
      }
    },
    // 默认选中节点 id
    defaultSelectedNode: {
      type: [String, Number],
      default: null
    },
    // 默认禁用节点 id
    defaultDisabledNodes: {
      type: Array,
      default () {
        return []
      }
    },
    beforeSelect: Function,
    beforeCheck: Function,
    expandOnClick: {
      type: Boolean,
      default: true
    },
    checkOnClick: Boolean,
    filterMethod: Function,
    nodeWidth: {
      type: [String, Number]
    },
    // 外部设置的 class name
    extCls: {
      type: String,
      default: ''
    },
    useDefaultEmpty: Boolean,
    height: Number, // 设置高度后默认开启虚拟滚动,
    nodeHeight: {
      type: Number,
      default: 32
    },
    configurable: {
      type: Boolean,
      default: true
    },
    padding: {
      type: Number,
      default: 16
    },
    /**
     * 树尺寸
     * 可选项：normal/small
     */
    size: {
      type: String,
      default: 'normal'
    },
    enableTitleTip: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      nodes: [],
      selected: this.defaultSelectedNode,
      needsCalculateNodes: [],
      calculateTimer: null,
      inSearch: false,
      isSearchEmpty: false,
      id: `bk-big-tree-${idSeed++}`
    }
  },
  computed: {
    computedNodeWidth () {
      const parsedWidth = parseInt(this.nodeWidth)
      if (isNaN(parsedWidth)) {
        return null
      }
      return parsedWidth
    },
    nodeOptions () {
      const nodeOptions = {
        idKey: 'id',
        nameKey: 'name',
        childrenKey: 'children'
      }
      return Object.assign(nodeOptions, this.options)
    },
    checkedNodes () {
      return this.nodes.filter(node => node.checked && node.hasCheckbox)
    },
    checked () {
      return this.checkedNodes.map(node => node.id)
    },
    visibleNodes () {
      return this.nodes.filter(node => !!node.visible)
    },
    treeHeight () {
      return this.height ? `${this.height}px` : 'auto'
    },
    hasLine () {
      return !this.height && this.showLinkLine
    }
  },
  watch: {
    needsCalculateNodes () {
      this.handleCalculateLine()
    },
    data (value) {
      this.setData(value)
    },
    hasLine: {
      handler () {
        if (this.hasLine) {
          this.needsCalculateNodes = Object.freeze([...this.needsCalculateNodes, ...this.visibleNodes])
        }
      },
      immediate: true
    }
  },
  created () {
    this.map = {}
  },
  mounted () {
    this.setData(this.data)
  },
  methods: {
    setData (data) {
      const nodes = []
      const map = {}
      this.recurrenceNodes(data, null, nodes, map)
      this.nodes = Object.freeze(nodes)
      this.map = map
      this.initNodeState()
      this.setVirtualScrollList()
      this.registryOptions(this.nodes)
    },
    initNodeState () {
      // 默认展开
      if (!this.defaultExpandAll) {
        const defaultExpandedNodes = [...this.defaultExpandedNodes]

        if (this.defaultSelectedNode) {
          defaultExpandedNodes.push(this.defaultSelectedNode)
        }

        defaultExpandedNodes.forEach(id => {
          const node = this.getNodeById(id)
          if (node) {
            node.expanded = true
          }
        })
      }

      // 默认选中
      this.defaultCheckedNodes.forEach(id => {
        const node = this.getNodeById(id)
        if (node) {
          node.checked = true
        }
      })

      // 默认禁用
      this.defaultDisabledNodes.forEach(id => {
        const node = this.getNodeById(id)
        if (node) {
          node.disabled = true
        }
      })
    },
    // 当在select组件嵌套tree时自动注册options，兼容select组件逻辑
    registryOptions (nodes) {
      const parent = this.$parent.$parent || this.$root
      const name = parent.$options.name
      if (name && name === 'bk-select' && parent.registerOption) {
        // 值变化时自动更新selectOptions
        parent.autoUpdate = true
        nodes.forEach(node => {
          parent.registerOption({
            id: node.id,
            name: node.name,
            disabled: node.disabled,
            unmatched: false,
            isHighlight: false
          })
        })
      }
    },
    recurrenceNodes (data, parent, nodes, map) {
      data.forEach((datum) => {
        const node = new TreeNode(datum, {
          level: parent ? parent.level + 1 : 0,
          parent: parent,
          index: nodes.length
        }, this)
        if (parent) {
          node.childIndex = parent.children.length
          parent.children.push(node)
        }
        nodes.push(node)
        map[node.id] = node

        const children = datum[this.nodeOptions.childrenKey]
        if (Array.isArray(children) && children.length) {
          this.recurrenceNodes(children, node, nodes, map)
        }
      })
    },
    getNodeById (id) {
      return this.map[id]
    },
    addNode (nodeData, parentId, trailing = true) {
      const options = typeof parentId === 'object' ? parentId : { parentId, trailing }
      const mergeOptions = Object.assign({
        parentId: null,
        trailing: true,
        expandParent: true
      }, options)
      const data = convertToArray(nodeData)
      if (!data.length) {
        return []
      }
      if (isNullOrUndefined(mergeOptions.parentId)) {
        return this.addRootNode(data, mergeOptions)
      }
      return this.addChildNode(data, mergeOptions)
    },
    addRootNode (data, { trailing }) {
      const lastNodes = [...this.nodes]
      const rootNodes = lastNodes.filter(node => node.level === 0)
      const offset = typeof trailing === 'number'
        ? Math.min(trailing, rootNodes.length)
        : trailing
          ? rootNodes.length
          : 0
      let insertIndex = 0
      if (offset > 0) {
        const referenceRoot = rootNodes[offset - 1]
        const referenceRootWithDescendants = [referenceRoot, ...referenceRoot.descendants]
        const referenceNode = referenceRootWithDescendants[referenceRootWithDescendants.length - 1]
        insertIndex = referenceNode.index + 1
      }
      const nodes = data.map(datum => {
        return new TreeNode(datum, {
          level: 0,
          parent: null
        }, this)
      })
      nodes.forEach(node => {
        this.map[node.id] = node
      })
      lastNodes.splice(insertIndex, 0, ...nodes)
      lastNodes.slice(insertIndex).forEach((node, index) => {
        node.index = insertIndex + index
      })
      this.nodes = Object.freeze(lastNodes)
      this.setVirtualScrollList()
      return nodes
    },
    addChildNode (data, options) {
      const { parentId, trailing } = options
      const parent = this.getNodeById(parentId)
      if (!parent) {
        console.warn('Unexpected parent id, add node failed')
        return
      }
      const children = parent.children
      const offset = typeof trailing === 'number'
        ? Math.min(trailing, children.length)
        : trailing
          ? children.length
          : 0
      let insertIndex
      if (offset > 0) {
        const referenceChild = children[offset - 1]
        const referenceChildWithDescendants = [referenceChild, ...referenceChild.descendants]
        const referenceNode = referenceChildWithDescendants[referenceChildWithDescendants.length - 1]
        insertIndex = referenceNode.index + 1
      } else {
        insertIndex = parent.index + 1
      }
      const nodes = data.map(datum => {
        return new TreeNode(datum, {
          level: parent.level + 1,
          parent: parent
        }, this)
      })
      parent.appendChild(nodes, offset, options)
      nodes.forEach(node => {
        this.map[node.id] = node
      })
      const lastNodes = [...this.nodes]
      lastNodes.splice(insertIndex, 0, ...nodes)
      lastNodes.slice(insertIndex).forEach((node, index) => {
        node.index = insertIndex + index
      })
      this.nodes = Object.freeze(lastNodes)
      this.setVirtualScrollList()
      return nodes
    },
    removeNode (nodeId) {
      try {
        const ids = convertToArray(nodeId)
        const nodes = []
        ids.forEach(id => {
          const node = this.getNodeById(id)
          if (node) {
            nodes.push(node)
          }
        })
        const lastNodes = [...this.nodes]

        // 从最大的node.index开始倒序splice
        nodes.sort((M, N) => N.index - M.index)
        nodes.forEach(node => {
          const removeNodes = [node, ...node.descendants]
          lastNodes.splice(node.index, removeNodes.length)
          if (node.parent) {
            node.parent.removeChild(node)
          }
        })
        const minChangedIndex = Math.min(...nodes.map(node => node.index))

        lastNodes.slice(minChangedIndex).forEach((node, index) => {
          node.index = minChangedIndex + index
        })
        this.nodes = Object.freeze(lastNodes)
        this.setVirtualScrollList()
      } catch (e) {
        console.warn(e.message)
      }
    },
    async setSelected (nodeId, options = {}) {
      try {
        if (!this.selectable || nodeId === this.selected) {
          return false
        }
        const mergeOptions = {
          emitEvent: false,
          beforeSelect: true,
          ...options
        }
        const node = this.getNodeById(nodeId)
        if (mergeOptions.beforeSelect && typeof this.beforeSelect === 'function') {
          const response = await this.beforeSelect(node)
          if (!response) {
            return false
          }
        }
        this.selected = nodeId
        if (mergeOptions.emitEvent) {
          this.$emit('select-change', node)
        }
      } catch (e) {
        console.warn(e.message)
      }
    },
    removeChecked (options = {}) {
      try {
        const mergeOptions = {
          emitEvent: true,
          ...options
        }
        this.checkedNodes.forEach(node => {
          node.checked = false
        })
        if (mergeOptions.emitEvent) {
          this.$emit('check-change', [], null, null)
        }
      } catch (e) {
        console.warn(e.message)
      }
    },
    async setChecked (nodeId, options = {}) {
      try {
        const isMultiple = Array.isArray(nodeId)
        const ids = isMultiple ? nodeId : [nodeId]
        if (ids.length) {
          const mergeOptions = {
            emitEvent: false,
            beforeCheck: true,
            checked: true,
            ...options
          }
          const nodes = ids.map(id => this.getNodeById(id))
          if (mergeOptions.beforeCheck && typeof this.beforeCheck === 'function') {
            const response = await this.beforeCheck(nodes.length > 1 ? nodes : nodes[0], mergeOptions.checked)
            if (!response) {
              return false
            }
          }
          nodes.forEach(node => {
            node.checked = mergeOptions.checked
          })
          if (mergeOptions.emitEvent) {
            // 延时执行，防止大量数据时check卡顿问题
            setTimeout(() => {
              this.$emit('check-change', this.checked, isMultiple ? nodes : nodes[0])
            }, 0)
          }
          const lastNodes = [...this.nodes]
          this.nodes = Object.freeze(lastNodes)
        }
      } catch (e) {
        console.warn(e.message)
      }
    },
    setExpanded (nodeId, options = {}) {
      try {
        const mergeOptions = {
          expanded: true,
          emitEvent: false,
          ...options
        }
        const node = this.getNodeById(nodeId)
        if (!node) {
          console.warn('Unexpected node id, set expanded failed.')
          return false
        }
        node.expanded = mergeOptions.expanded
        if (mergeOptions.emitEvent) {
          this.$emit('expand-change', node)
        }
        this.nodes = Object.freeze([...this.nodes])
        this.setVirtualScrollList()
      } catch (e) {
        console.warn(e.message)
      }
    },
    setDisabled (nodeId, options = {}) {
      try {
        const mergeOptions = {
          disabled: true,
          emitEvent: false,
          ...options
        }
        const ids = convertToArray(nodeId)
        const nodes = ids.map(id => this.getNodeById(id)).filter(node => !!node)
        nodes.forEach(node => {
          node.disabled = mergeOptions.disabled
        })
        if (mergeOptions.emitEvent) {
          this.$emit('disable-change', nodes.length > 1 ? nodes : nodes[0])
        }
        this.nodes = Object.freeze([...this.nodes])
      } catch (e) {
        console.warn(e.message)
      }
    },
    setDisableCheck (nodeId, options = {}) {
      try {
        const mergeOptions = {
          disabled: true,
          emitEvent: false,
          ...options
        }
        const ids = convertToArray(nodeId)
        const nodes = ids.map(id => this.getNodeById(id)).filter(node => !!node)
        nodes.forEach(node => {
          node.disableCheck = mergeOptions.disabled
        })
        if (mergeOptions.emitEvent) {
          this.$emit('disable-check-change', nodes.length > 1 ? nodes : nodes[0])
        }
        this.nodes = Object.freeze([...this.nodes])
      } catch (e) {
        console.warn(e.message)
      }
    },
    handleCalculateLine () {
      const calculateNodeLine = (node) => {
        const {
          children,
          isLeaf,
          expanded
        } = node
        if (isLeaf || !expanded) {
          node.line = 0
          return
        }
        const visibleChildren = children.filter(child => child.visible)
        if (!visibleChildren.length) {
          node.line = 0
          return
        }
        const firstChild = visibleChildren[0]
        const firstChildElement = this.$el.querySelector(`#${firstChild.uid}`)
        const lastChild = visibleChildren[visibleChildren.length - 1]
        const lastChildElement = this.$el.querySelector(`#${lastChild.uid}`)
        node.line = lastChildElement.getBoundingClientRect().bottom - firstChildElement.getBoundingClientRect().top
      }
      this.calculateTimer && clearTimeout(this.calculateTimer)
      if (this.needsCalculateNodes.length) {
        this.calculateTimer = setTimeout(() => {
          this.needsCalculateNodes.forEach(node => {
            calculateNodeLine(node)
          })
          this.needsCalculateNodes.splice(0)
          this.nodes = Object.freeze([...this.nodes])
        }, 0)
      } else {
        this.calculateTimer = null
      }
    },

    defaultFilterMethod (keyword, node) {
      return String(node.name).toLowerCase().indexOf(keyword) > -1
    },
    filter (keyword = '') {
      const lastNodes = [...this.nodes]
      const matchedNodes = []
      const filterMethod = this.filterMethod || this.defaultFilterMethod
      if (keyword === '') {
        this.inSearch = false
        lastNodes.forEach(node => {
          node.setState('matched', true)
          node.recalculateLinkLine()
          if (this.checkOnlyAvailableStrictly) {
            node.setState('checked', false)
          }
          matchedNodes.push(node)
        })
      } else {
        this.inSearch = true
        const convertKeyword = this.filterMethod ? keyword : String(keyword).toLowerCase()
        lastNodes.forEach(node => {
          const matched = filterMethod(convertKeyword, node)
          node.setState('matched', matched)
          if (this.checkOnlyAvailableStrictly) {
            node.setState('checked', false)
          }
          if (matched) {
            node.parent && (node.parent.expanded = true)
            matchedNodes.push(node)
            node.recalculateLinkLine()
          }
        })
      }
      this.isSearchEmpty = matchedNodes.length === 0
      this.nodes = Object.freeze(lastNodes)
      this.setVirtualScrollList()
      return matchedNodes
    },
    handleNodeClick (node) {
      if (node.disabled) {
        return false
      }
      this.$emit('node-click', node)
      this.setSelected(node.id, {
        emitEvent: true,
        beforeSelect: true
      })
      if (this.expandOnClick && !node.isLeaf) {
        this.setExpanded(node.id, {
          emitEvent: true,
          expanded: !node.expanded
        })
      }
      if (this.checkOnClick) {
        this.setChecked(node.id, {
          emitEvent: true,
          checked: !node.checked
        })
      }
    },
    handleNodeExpand (node) {
      this.setExpanded(node.id, {
        expanded: !node.expanded,
        emitEvent: true
      })
    },
    handleNodeCheck (node) {
      if (node.disabled || node.disableCheck) {
        return false
      }
      this.setChecked(node.id, {
        checked: node.indeterminate ? true : !node.checked,
        emitEvent: true,
        beforeCheck: true
      })
    },
    setVirtualScrollList () {
      // 未开启虚拟滚动不执行
      if (!this.height) return

      if (!this.$refs.virtualScroll) {
        console.warn('virtual dom is not ready')
        return
      }
      this.$nextTick(() => {
        this.$refs.virtualScroll.setListData(this.visibleNodes)
      })
    },
    // 虚拟滚动区域resize
    resize () {
      this.$refs.virtualScroll && this.$refs.virtualScroll.resize()
    }
  }
}
</script>

<style>
  @import '../../ui/big-tree.css';
</style>
