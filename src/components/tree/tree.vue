<template>
    <ul :class="classes">
        <li v-for="item in data" :class="itemCls(item)">
            <span :class="arrowCls(item)" @click="setExpand(item.disabled, $index)"></span>
            <span v-if="showCheckbox" :class="checkboxCls(item)" @click="setCheck(item.disabled||item.disableCheckbox,$index)">
                <span :class="[prefixCls + '-checkbox-inner']"></span>
            </span>
            <a :class="titleCls(item)" @click="setSelect(item.disabled, $index)">
                <span :class="[prefixCls + '-title']" v-html="item.title"></span>
            </a>
            <tree
                v-if="!item.isLeaf"
                v-show="item.expand"
                :class="expandCls(item)"
                :data.sync="item.node"
                :key="this.key+'.'+$index"
                :multiple="multiple"
                :show-checkbox="showCheckbox"
                transition="slide-up"></tree>
        </li>
    </ul>
</template>
<script>
    import { t } from '../../locale';

    const prefixCls = 'ivu-tree';

    export default {
        name: 'tree',
        props: {
            data: {
                type: Array,
                default () {
                    return [];
                }
            },
            key: {
                type: String,
                default: '0'
            },
            multiple: {
                type: Boolean,
                default: false
            },
            showCheckbox: {
                type: Boolean,
                default: false
            },
            onSelect: {
                type: Function,
                default () {
                    return {};
                }
            },
            onCheck: {
                type: Function,
                default () {
                    return {};
                }
            },
            emptyText: {
                type: String,
                default () {
                    return t('i.tree.emptyText');
                }
            }
        },
        data () {
            return {
                prefixCls: prefixCls
            };
        },
        computed: {
            classes () {
                if (this.key === '0') {
                    return this.prefixCls;
                } else {
                    return `${this.prefixCls}-child-tree`;
                }
            }
        },
        watch: {
            data(){
                if (this.key === '0') {
                    this.setKey();
                    this.preHandle();
                }
            }
        },
        methods: {
            itemCls (item) {
                return [
                    {
                        [`${prefixCls}-item-disabled`]: item.disabled
                    }
                ];
            },
            arrowCls (item) {
                return [
                    `${this.prefixCls}-switcher`,
                    {
                        [`${this.prefixCls}-switcher-disabled`]: item.disabled,
                        [`${this.prefixCls}-noline_close`]: !item.expand && !item.isLeaf,
                        [`${this.prefixCls}-noline_open`]: item.expand && !item.isLeaf,
                        [`${this.prefixCls}-switcher-noop`]: item.isLeaf
                    }
                ];
            },
            checkboxCls (item) {
                return [
                    `${this.prefixCls}-checkbox`,
                    {
                        [`${this.prefixCls}-checkbox-disabled`]: item.disabled || item.disableCheckbox,
                        [`${this.prefixCls}-checkbox-checked`]: item.checked && item.childrenCheckedStatus == 2,
                        [`${this.prefixCls}-checkbox-indeterminate`]: item.checked && item.childrenCheckedStatus == 1
                    }
                ];
            },
            titleCls (item) {
                return [
                    {
                        [`${this.prefixCls}-node-selected`]: item.selected
                    }
                ];
            },
            expandCls (item) {
                return [
                    {
                        [`${this.prefixCls}-child-tree-open`]: item.expand
                    }
                ];
            },
            setKey () {
                for (let i = 0; i < this.data.length; i++) {
                    this.data[i].key = `${this.key}.${i}`;
                }
            },
            preHandle(){
                for (let [i,item] of this.data.entries()) {
                    if (!item.node || !item.node.length) {
                        this.$set(`data[${i}].isLeaf`, true);
                        this.$set(`data[${i}].childrenCheckedStatus`, 2);
                        continue;
                    }
                    if (item.checked && !item.childrenCheckedStatus) {
                        this.$set(`data[${i}].childrenCheckedStatus`, 2);
                        this.$broadcast('parentChecked', true, `${this.key}.${i}`);
                    } else {
                        let status = this.getChildrenCheckedStatus(item.node);
                        this.$set(`data[${i}].childrenCheckedStatus`, status);
                        if (status !== 0) this.$set(`data[${i}].checked`, true);
                    }
                }
            },
            setExpand(disabled, index){
                if (!disabled) {
                    this.$set(`data[${index}].expand`, !this.data[index].expand);
                }
            },
            setSelect(disabled, index){
                if (!disabled) {
                    const selected = !this.data[index].selected;
                    if (this.multiple || !selected) {
                        this.$set(`data[${index}].selected`, selected);
                    } else {
                        for (let i = 0; i < this.data.length; i++) {
                            if (i == index) {
                                this.$set(`data[${i}].selected`, true);
                            } else {
                                this.$set(`data[${i}].selected`, false);
                            }
                        }
                    }
                    this.$dispatch('nodeSelected', this, selected);
                }
            },
            setCheck(disabled, index){
                if (disabled) return;
                const checked = !this.data[index].checked;
                this.$set(`data[${index}].checked`, checked);
                this.$set(`data[${index}].childrenCheckedStatus`, checked ? 2 : 0);
                this.$dispatch('childChecked', this, this.key);
                this.$broadcast('parentChecked', checked, `${this.key}.${index}`);
            },
            getNodes(data, opt){
                data = data || this.data;
                let res = [];
                for (let node of data) {
                    let tmp = true;
                    for (let [key, value] of Object.entries(opt)) {
                        if (node[key] != value) {
                            tmp = false;
                            break;
                        }
                    }
                    if (tmp) {
                        res.push(node);
                    }
                    if (node.node && node.node.length) {
                        res = res.concat(this.getNodes(node.node, opt));
                    }
                }
                return res;
            },
            getSelectedNodes(){
                return this.getNodes(this.data, {selected: true});
            },
            getCheckedNodes(){
                return this.getNodes(this.data, {checked: true, childrenCheckedStatus: 2});
            },
            getChildrenCheckedStatus(children){
                let checkNum = 0, child_childrenAllChecked = true;
                for (let child of children) {
                    if (child.checked) {
                        checkNum++;
                    }
                    if (child.childrenCheckedStatus !== 2) {
                        child_childrenAllChecked = false;
                    }
                }
                // select all
                if (checkNum == children.length) {
                    return child_childrenAllChecked ? 2 : 1;
                    // select some
                } else if (checkNum > 0) {
                    return 1;
                } else {
                    return 0;
                }
            }
        },
        ready(){
            this.setKey();
            this.preHandle();

            this.$on('nodeSelected', (ori, selected) => {
                if (this.key !== '0') return true;
                if (!this.multiple && selected) {
                    if (this !== ori) {
                        for (let i = 0; i < this.data.length; i++) {
                            this.$set(`data[${i}].selected`, false);
                        }
                    }
                    this.$broadcast('cancelSelected', ori);
                }
                if (this.onSelect) {
                    this.$nextTick(() => {
                        this.onSelect(this.getSelectedNodes());
                    });
                }
            });
            this.$on('cancelSelected', ori => {
                this.$broadcast('cancelSelected', ori);
                if (this !== ori) {
                    for (let i = 0; i < this.data.length; i++) {
                        this.$set(`data[${i}].selected`, false);
                    }
                }
            });
            this.$on('parentChecked', (status, key) => {
                if (this.key == key || this.key.startsWith(key + '.')) {
                    for (let i = 0; i < this.data.length; i++) {
                        this.$set(`data[${i}].checked`, status);
                        this.$set(`data[${i}].childrenCheckedStatus`, status ? 2 : 0);
                    }
                    this.$broadcast('parentChecked', status, key);
                }
            });
            this.$on('childChecked', (ori, key) => {
                if (this.key === '0' && this.onCheck) {
                    this.$nextTick(() => {
                        this.onCheck(this.getCheckedNodes());
                    });
                }
                if (this === ori) return;
                for (let [i,item] of this.data.entries()) {
                    if (this.key + '.' + i == key) {
                        let temp = this.getChildrenCheckedStatus(item.node);
                        if (temp != item.childrenCheckedStatus) {
                            this.$set(`data[${i}].checked`, !!temp);
                            this.$set(`data[${i}].childrenCheckedStatus`, temp);
                            if (this.key !== '0') this.$dispatch('childChecked', this, this.key);
                        }
                    }
                }
            });
        }
    };
</script>