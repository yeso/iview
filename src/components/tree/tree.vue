<template>
    <div :class="prefixCls">
        <Tree-node
            v-for="(item, i) in stateTree"
            :key="i"
            :data="item"
            visible
            :multiple="multiple"
            :show-checkbox="showCheckbox">
        </Tree-node>
        <div :class="[prefixCls + '-empty']" v-if="!stateTree.length">{{ localeEmptyText }}</div>
    </div>
</template>
<script>
    import TreeNode from './node.vue';
    import Emitter from '../../mixins/emitter';
    import Locale from '../../mixins/locale';

    const prefixCls = 'ivu-tree';

    export default {
        name: 'Tree',
        mixins: [ Emitter, Locale ],
        components: { TreeNode },
        props: {
            data: {
                type: Array,
                default () {
                    return [];
                }
            },
            multiple: {
                type: Boolean,
                default: false
            },
            checkMode: {
                type: String,
                default: 'default'
            },
            showCheckbox: {
                type: Boolean,
                default: false
            },
            emptyText: {
                type: String
            },
            loadData: {
                type: Function
            },
            render: {
                type: Function
            }
        },
        data () {
            return {
                prefixCls: prefixCls,
                stateTree: this.data,
                flatState: [],
            };
        },
        watch: {
            data: {
                deep: true,
                handler () {
                    this.stateTree = this.data;
                    this.flatState = this.compileFlatState();
                    this.rebuildTree();
                }
            }
        },
        computed: {
            localeEmptyText () {
                if (typeof this.emptyText === 'undefined') {
                    return this.t('i.tree.emptyText');
                } else {
                    return this.emptyText;
                }
            },
        },
        methods: {
            compileFlatState () { // so we have always a relation parent/children of each node
                let keyCounter = 0;
                const flatTree = [];
                function flattenChildren(node, parent) {
                    node.nodeKey = keyCounter++;
                    flatTree[node.nodeKey] = { node: node, nodeKey: node.nodeKey };
                    if (typeof parent != 'undefined') {
                        flatTree[node.nodeKey].parent = parent.nodeKey;
                        flatTree[parent.nodeKey].children.push(node.nodeKey);
                    }

                    if (node.children) {
                        flatTree[node.nodeKey].children = [];
                        node.children.forEach(child => flattenChildren(child, node));
                    }
                }
                this.stateTree.forEach(rootNode => {
                    flattenChildren(rootNode);
                });
                return flatTree;
            },
            updateTreeUp(nodeKey){
                const parentKey = this.flatState[nodeKey].parent;
                if (typeof parentKey == 'undefined') return;

                const node = this.flatState[nodeKey].node;
                const parent = this.flatState[parentKey].node;

                if (this.checkMode === 'default') {
                    // 默认模式
                    if (node.checked == parent.checked && node.indeterminate == parent.indeterminate) return; // no need to update upwards

                    if (node.checked == true) {
                        this.$set(parent, 'checked', parent.children.every(node => node.checked));
                        this.$set(parent, 'indeterminate', !parent.checked);
                    } else {
                        this.$set(parent, 'checked', false);
                        this.$set(parent, 'indeterminate', parent.children.some(node => node.checked || node.indeterminate));
                    }
                } else if (this.checkMode === 'bag') {
                    // 父节点单独控制，子节点选中时父节点自动选中
                    if (node.checked && parent.checked) return;
                    if (node.checked) {
                        this.$set(parent, 'checked', true);
                    }
                }

                this.updateTreeUp(parentKey);
            },
            rebuildTree () { // only called when `data` prop changes
                const checkedNodes = this.getCheckedNodes();
                checkedNodes.forEach(node => {
                    this.updateTreeDown(node, {checked: true});
                    // propagate upwards
                    const parentKey = this.flatState[node.nodeKey].parent;
                    if (!parentKey && parentKey !== 0) return;
                    const parent = this.flatState[parentKey].node;
                    const childHasCheckSetter = typeof node.checked != 'undefined' && node.checked;
                    if (childHasCheckSetter && parent.checked != node.checked) {
                        this.updateTreeUp(node.nodeKey); // update tree upwards
                    }
                });
            },

            getSelectedNodes () {
                /* public API */
                return this.flatState.filter(obj => obj.node.selected).map(obj => obj.node);
            },
            getCheckedNodes () {
                /* public API */
                return this.flatState.filter(obj => obj.node.checked).map(obj => obj.node);
            },
            updateTreeDown(node, changes = {}) {
                for (let key in changes) {
                    this.$set(node, key, changes[key]);
                }
                if (node.children) {
                    // bag模式 父节点取消选中，则子节点全取消；父节点选中，不处理子节点
                    if (this.checkMode === 'bag' && changes.checked) return;
                    node.children.forEach(child => {
                        this.updateTreeDown(child, changes);
                    });
                }
            },
            handleSelect (nodeKey) {
                const node = this.flatState[nodeKey].node;
                if (!this.multiple){ // reset previously selected node
                    const currentSelectedKey = this.flatState.findIndex(obj => obj.node.selected);
                    if (currentSelectedKey >= 0 && currentSelectedKey !== nodeKey) this.$set(this.flatState[currentSelectedKey].node, 'selected', false);
                }
                this.$set(node, 'selected', !node.selected);

                this.$emit('on-select-change', this.getSelectedNodes());
            },
            handleCheck({ checked, nodeKey }) {
                const node = this.flatState[nodeKey].node;
                this.$set(node, 'checked', checked);
                this.$set(node, 'indeterminate', false);

                this.updateTreeUp(nodeKey); // propagate up
                this.updateTreeDown(node, {checked, indeterminate: false}); // reset `indeterminate` when going down

                this.$emit('on-check-change', this.getCheckedNodes());
            }
        },
        created(){
            this.flatState = this.compileFlatState();
            this.rebuildTree();
        },
        mounted () {
            this.$on('on-check', this.handleCheck);
            this.$on('on-selected', this.handleSelect);
            this.$on('toggle-expand', node => this.$emit('on-toggle-expand', node));
        }
    };
</script>
