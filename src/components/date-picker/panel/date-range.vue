<template>
    <div :class="classes" @mousedown.prevent>
        <div :class="[prefixCls + '-sidebar']" v-if="shortcuts.length">
            <div
                :class="[prefixCls + '-shortcut']"
                v-for="shortcut in shortcuts"
                @click="handleShortcutClick(shortcut)">{{ shortcut.text }}</div>
        </div>
        <div :class="[prefixCls + '-body']">
            <div :class="[prefixCls + '-content', prefixCls + '-content-left']" v-show="!isTime">
                <div :class="[datePrefixCls + '-header']" v-show="leftCurrentView !== 'time'">
                    <span
                        :class="iconBtnCls('prev', '-double')"
                        @click="prevYear('left')"><Icon type="ios-arrow-left"></Icon></span>
                    <span
                        :class="iconBtnCls('prev')"
                        @click="prevMonth('left')"
                        :style="{ visibility: leftCurrentView === 'date' ? 'visible' : 'hidden' }"><Icon type="ios-arrow-left"></Icon></span>
                    <span
                        :class="[datePrefixCls + '-header-label']"
                        @click="showYearPicker('left')">{{ leftYearLabel }}</span>
                    <span
                        :class="[datePrefixCls + '-header-label']"
                        @click="showMonthPicker('left')"
                        v-show="leftCurrentView === 'date'">{{ leftMonthLabel }}</span>
                    <span
                        :class="iconBtnCls('next', '-double')"
                        @click="nextYear('left')"
                        :style="{ visibility: new Date(leftYear + 1, leftMonth) < new Date(rightYear, rightMonth) || leftCurrentView !== 'date' ? 'visible' : 'hidden' }"><Icon type="ios-arrow-right"></Icon></span>
                    <span
                        :class="iconBtnCls('next')"
                        @click="nextMonth('left')"
                        :style="{ visibility: leftCurrentView === 'date' && (rightYear > leftYear || rightMonth > leftMonth + 1) ? 'visible' : 'hidden' }"><Icon type="ios-arrow-right"></Icon></span>
                </div>
                <date-table
                    v-show="leftCurrentView === 'date'"
                    :year="leftYear"
                    :month="leftMonth"
                    :date="leftDate"
                    :min-date="minDate"
                    :max-date="maxDate"
                    :range-state="rangeState"
                    selection-mode="range"
                    :disabled-date="disabledDate"
                    @on-changerange="handleChangeRange"
                    @on-pick="handleRangePick"
                    @on-pick-click="handlePickClick"></date-table>
                <year-table
                    ref="leftYearTable"
                    v-show="leftCurrentView === 'year'"
                    :year="leftTableYear"
                    :date="leftTableDate"
                    selection-mode="range"
                    :disabled-date="makeDisabledDate('year', 'left')"
                    @on-pick="handleLeftYearPick"
                    @on-pick-click="handlePickClick"></year-table>
                <month-table
                    ref="leftMonthTable"
                    v-show="leftCurrentView === 'month'"
                    :month="leftMonth"
                    :date="leftTableDate"
                    selection-mode="range"
                    :disabled-date="makeDisabledDate('month', 'left')"
                    @on-pick="handleLeftMonthPick"
                    @on-pick-click="handlePickClick"></month-table>
            </div>
            <div :class="[prefixCls + '-content', prefixCls + '-content-right']" v-show="!isTime">
                <div :class="[datePrefixCls + '-header']" v-show="rightCurrentView !== 'time'">
                     <span
                         :class="iconBtnCls('prev', '-double')"
                         @click="prevYear('right')"
                         :style="{ visibility: new Date(leftYear + 1, leftMonth) < new Date(rightYear, rightMonth) || rightCurrentView !== 'date' ? 'visible' : 'hidden' }"><Icon type="ios-arrow-left"></Icon></span>
                    <span
                        :class="iconBtnCls('prev')"
                        @click="prevMonth('right')"
                        :style="{ visibility: rightCurrentView === 'date' && (rightYear > leftYear || rightMonth > leftMonth + 1) ? 'visible' : 'hidden' }"><Icon type="ios-arrow-left"></Icon></span>
                    <span
                        :class="[datePrefixCls + '-header-label']"
                        @click="showYearPicker('right')">{{ rightYearLabel }}</span>
                    <span
                        :class="[datePrefixCls + '-header-label']"
                        @click="showMonthPicker('right')"
                        v-show="rightCurrentView === 'date'">{{ rightMonthLabel }}</span>
                    <span
                        :class="iconBtnCls('next', '-double')"
                        @click="nextYear('right')"><Icon type="ios-arrow-right"></Icon></span>
                    <span
                        :class="iconBtnCls('next')"
                        @click="nextMonth('right')"
                        :style="{ visibility: rightCurrentView === 'date' ? 'visible' : 'hidden' }"><Icon type="ios-arrow-right"></Icon></span>
                </div>
                <date-table
                    v-show="rightCurrentView === 'date'"
                    :year="rightYear"
                    :month="rightMonth"
                    :date="rightDate"
                    :min-date="minDate"
                    :max-date="maxDate"
                    :range-state="rangeState"
                    selection-mode="range"
                    :disabled-date="disabledDate"
                    @on-changerange="handleChangeRange"
                    @on-pick="handleRangePick"
                    @on-pick-click="handlePickClick"></date-table>
                <year-table
                    ref="rightYearTable"
                    v-show="rightCurrentView === 'year'"
                    :year="rightTableYear"
                    :date="rightTableDate"
                    selection-mode="range"
                    :disabled-date="makeDisabledDate('year', 'right')"
                    @on-pick="handleRightYearPick"
                    @on-pick-click="handlePickClick"></year-table>
                <month-table
                    ref="rightMonthTable"
                    v-show="rightCurrentView === 'month'"
                    :month="rightMonth"
                    :date="rightTableDate"
                    selection-mode="range"
                    :disabled-date="makeDisabledDate('month', 'right')"
                    @on-pick="handleRightMonthPick"
                    @on-pick-click="handlePickClick"></month-table>
            </div>
            <div :class="[prefixCls + '-content']" v-show="isTime">
                <time-picker
                    ref="timePicker"
                    v-show="isTime"
                    @on-pick="handleTimePick"
                    @on-pick-click="handlePickClick"></time-picker>
            </div>
            <Confirm
                v-if="confirm"
                :show-time="showTime"
                :is-time="isTime"
                :time-disabled="timeDisabled"
                @on-pick-toggle-time="handleToggleTime"
                @on-pick-clear="handlePickClear"
                @on-pick-success="handlePickSuccess"></Confirm>
        </div>
    </div>
</template>
<script>
    import Icon from '../../icon/icon.vue';
    import DateTable from '../base/date-table.vue';
    import YearTable from '../base/year-table.vue';
    import MonthTable from '../base/month-table.vue';
    import TimePicker from './time-range.vue';
    import Confirm from '../base/confirm.vue';
    import { toDate, prevMonth, nextMonth, initTimeDate } from '../util';

    import Mixin from './mixin';
    import Locale from '../../../mixins/locale';

    const prefixCls = 'ivu-picker-panel';
    const datePrefixCls = 'ivu-date-picker';

    export default {
        name: 'DatePicker',
        mixins: [ Mixin, Locale ],
        components: { Icon, DateTable, YearTable, MonthTable, TimePicker, Confirm },
        data () {
            return {
                prefixCls: prefixCls,
                datePrefixCls: datePrefixCls,
                shortcuts: [],
                leftDate: null,
                rightDate: null,
                value: '',
                minDate: '',
                maxDate: '',
                confirm: false,
                rangeState: {
                    endDate: null,
                    selecting: false
                },
                showTime: false,
                disabledDate: '',
                leftCurrentView: 'date',
                rightCurrentView: 'date',
                selectionMode: 'range',
                leftTableYear: null,
                rightTableYear: null,
                isTime: false,
                format: 'yyyy-MM-dd'
            };
        },
        computed: {
            classes () {
                return [
                    `${prefixCls}-body-wrapper`,
                    `${datePrefixCls}-with-range`,
                    {
                        [`${prefixCls}-with-sidebar`]: this.shortcuts.length
                    }
                ];
            },
            leftYear () {
                return this.leftDate.getFullYear();
            },
            leftTableDate () {
                if (this.leftCurrentView === 'year' || this.leftCurrentView === 'month') {
                    return new Date(this.leftTableYear);
                } else {
                    return this.leftDate;
                }
            },
            leftYearLabel () {
                const tYear = this.t('i.datepicker.year');
                if (this.leftCurrentView === 'year') {
                    const year = this.leftTableYear;
                    if (!year) return '';
                    const startYear = Math.floor(year / 10) * 10;
                    return `${startYear}${tYear} - ${startYear + 9}${tYear}`;
                } else {
                    const year = this.leftCurrentView === 'month' ? this.leftTableYear : this.leftYear;
                    if (!year) return '';
                    return `${year}${tYear}`;
                }
            },
            leftMonth () {
                return this.leftDate.getMonth();
            },
            leftMonthLabel () {
                const month = this.leftMonth + 1;
                return this.t(`i.datepicker.month${month}`);
            },
            rightYear () {
                return this.rightDate.getFullYear();
            },
            rightTableDate () {
                if (this.rightCurrentView === 'year' || this.rightCurrentView === 'month') {
                    return new Date(this.rightTableYear);
                } else {
                    return this.rightDate;
                }
            },
            rightYearLabel () {
                const tYear = this.t('i.datepicker.year');
                if (this.rightCurrentView === 'year') {
                    const year = this.rightTableYear;
                    if (!year) return '';
                    const startYear = Math.floor(year / 10) * 10;
                    return `${startYear}${tYear} - ${startYear + 9}${tYear}`;
                } else {
                    const year = this.rightCurrentView === 'month' ? this.rightTableYear : this.rightYear;
                    if (!year) return '';
                    return `${year}${tYear}`;
                }
            },
            rightMonth () {
                const m = this.rightDate.getMonth();
                return m;
            },
            rightMonthLabel () {
                const month = this.rightMonth + 1;
                return this.t(`i.datepicker.month${month}`);
            },
            timeDisabled () {
                return !(this.minDate && this.maxDate);
            }
        },
        watch: {
            value(newVal) {
                if (!newVal) {
                    this.minDate = null;
                    this.maxDate = null;
                } else if (Array.isArray(newVal)) {
                    this.minDate = newVal[0] ? toDate(newVal[0]) : null;
                    this.maxDate = newVal[1] ? toDate(newVal[1]) : null;
//                    if (this.minDate) this.leftDate = new Date(this.minDate);
//                    if (this.maxDate) this.rightDate = new Date(this.maxDate);
                }
                if (this.showTime) this.$refs.timePicker.value = newVal;
            },
            minDate (val) {
                if (this.showTime) this.$refs.timePicker.date = val;
            },
            maxDate (val) {
                if (this.showTime) this.$refs.timePicker.dateEnd = val;
            },
            format (val) {
                if (this.showTime) this.$refs.timePicker.format = val;
            },
            isTime (val) {
                if (val) this.$refs.timePicker.updateScroll();
            }
        },
        methods: {
            initLeftDate () {
                return initTimeDate();
            },
            initRightDate () {
                const newDate = new Date(this.leftDate);
                const month = newDate.getMonth();
                newDate.setDate(1);

                if (month === 11) {
                    newDate.setFullYear(newDate.getFullYear() + 1);
                    newDate.setMonth(0);
                } else {
                    newDate.setMonth(month + 1);
                }
                return newDate;
            },
            resetDate () {
                this.leftDate = new Date(this.leftDate);
                this.rightDate = new Date(this.rightDate);
                this.leftTableYear = this.leftDate.getFullYear();
                this.rightTableYear = this.rightDate.getFullYear();
            },
            handleClear() {
                this.minDate = null;
                this.maxDate = null;
                this.leftDate = this.initLeftDate();
                this.rightDate = this.initRightDate();
                this.handleConfirm();
                if (this.showTime) this.$refs.timePicker.handleClear();
            },
            resetView(reset = false) {
                this.leftCurrentView = 'date';
                this.rightCurrentView = 'date';

                this.leftTableYear = this.leftYear;
                this.rightTableYear = this.rightYear;

                if (reset) this.isTime = false;
            },
            prevYear (direction) {
                if (this[`${direction}CurrentView`] === 'year') {
                    this.$refs[`${direction}YearTable`].prevTenYear();
                } else if (this[`${direction}CurrentView`] === 'month') {
                    this[`${direction}TableYear`]--;
                } else {
                    const date = this[`${direction}Date`];
                    date.setFullYear(date.getFullYear() - 1);
                    this.resetDate();
                }
            },
            nextYear (direction) {
                if (this[`${direction}CurrentView`] === 'year') {
                    this.$refs[`${direction}YearTable`].nextTenYear();
                } else if (this[`${direction}CurrentView`] === 'month') {
                    this[`${direction}TableYear`]++;
                } else {
                    const date = this[`${direction}Date`];
                    date.setFullYear(date.getFullYear() + 1);
                    this.resetDate();
                }
            },
            prevMonth (direction) {
                let date = this[`${direction}Date`];
                this[`${direction}Date`] = prevMonth(date);
            },
            nextMonth (direction) {
                let date = this[`${direction}Date`];
                this[`${direction}Date`] = nextMonth(date);
            },
            handleLeftYearPick (year, close = true) {
                this.handleYearPick(year, close, 'left');
            },
            handleRightYearPick (year, close = true) {
                this.handleYearPick(year, close, 'right');
            },
            handleYearPick (year, close, direction) {
                this[`${direction}TableYear`] = year;
                if (!close) return;

                this[`${direction}CurrentView`] = 'month';
            },
            handleLeftMonthPick (month) {
                this.handleMonthPick(month, 'left');
            },
            handleRightMonthPick (month) {
                this.handleMonthPick(month, 'right');
            },
            handleMonthPick (month, direction) {
                let year = this[`${direction}TableYear`];
                this[`${direction}Date`].setYear(year);
                this[`${direction}Date`].setMonth(month);
                this[`${direction}CurrentView`] = 'date';
                this.resetDate();
            },
            showYearPicker (direction) {
                this[`${direction}CurrentView`] = 'year';
                this[`${direction}TableYear`] = this[`${direction}Year`];
            },
            showMonthPicker (direction) {
                this[`${direction}CurrentView`] = 'month';
            },
            handleConfirm(visible) {
                this.$emit('on-pick', [this.minDate, this.maxDate], visible);
            },
            handleRangePick (val, close = true) {
                if (this.maxDate === val.maxDate && this.minDate === val.minDate) return;

                this.minDate = val.minDate;
                this.maxDate = val.maxDate;

                if (!close) return;
//                if (!this.showTime) {
//                    this.handleConfirm(false);
//                }
                this.handleConfirm(false);
            },
            handleChangeRange (val) {
                this.minDate = val.minDate;
                this.maxDate = val.maxDate;
                this.rangeState = val.rangeState;
            },
            handleToggleTime () {
                this.isTime = !this.isTime;
            },
            handleTimePick (date) {
                this.minDate = date[0];
                this.maxDate = date[1];
                this.handleConfirm(false);
            },
            makeDisabledDate (type, direction) {
                return date => {
                    if (direction === 'left') {
                        if (type === 'year') {
                            return date.getFullYear() > this.rightYear;
                        } else if (type === 'month') {
                            return new Date(this.leftTableYear, date.getMonth()) >= new Date(this.rightDate.getFullYear(), this.rightDate.getMonth());
                        }
                    } else if (direction === 'right') {
                        if (type === 'year') {
                            return date.getFullYear() < this.leftYear;
                        } else if (type === 'month') {
                            return new Date(this.rightTableYear, date.getMonth()) <= new Date(this.leftDate.getFullYear(), this.leftDate.getMonth());
                        }
                    }
                };
            }
        },
        created () {
            this.leftDate = this.initLeftDate();
            this.rightDate = this.initRightDate();
        },
        mounted () {
            if (this.showTime) {
                // todo 这里也到不了
                this.$refs.timePicker.date = this.minDate;
                this.$refs.timePicker.dateEnd = this.maxDate;
                this.$refs.timePicker.value = this.value;
                this.$refs.timePicker.format = this.format;
                this.$refs.timePicker.showDate = true;
            }
        }
    };
</script>
