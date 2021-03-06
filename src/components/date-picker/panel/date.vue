<template>
    <div :class="classes">
        <div :class="[prefixCls + '-sidebar']" v-if="shortcuts.length">
            <div
                :class="[prefixCls + '-shortcut']"
                v-for="shortcut in shortcuts"
                @click="handleShortcutClick(shortcut)">{{ shortcut.text }}</div>
        </div>
        <div :class="[prefixCls + '-body']">
            <div :class="[datePrefixCls + '-header']" v-show="currentView !== 'time'">
                <span
                    :class="iconBtnCls('prev', '-double')"
                    @click="prevYear"><Icon type="ios-arrow-left"></Icon></span>
                <span
                    :class="iconBtnCls('prev')"
                    @click="prevMonth"
                    v-show="currentView === 'date'"><Icon type="ios-arrow-left"></Icon></span>
                <span
                    :class="[datePrefixCls + '-header-label']"
                    @click="showYearPicker">{{ yearLabel }}</span>
                <span
                    :class="[datePrefixCls + '-header-label']"
                    @click="showMonthPicker"
                    v-show="currentView === 'date'">{{ month + 1 + '月' }}</span>
                <span
                    :class="iconBtnCls('next', '-double')"
                    @click="nextYear"><Icon type="ios-arrow-right"></Icon></span>
                <span
                    :class="iconBtnCls('next')"
                    @click="nextMonth"
                    v-show="currentView === 'date'"><Icon type="ios-arrow-right"></Icon></span>
            </div>
            <div :class="[prefixCls + '-content']">
                <date-table
                    v-show="currentView === 'date'"
                    :year="year"
                    :month="month"
                    :date="date"
                    :value="value"
                    :selection-mode="selectionMode"
                    :disabled-date="disabledDate"
                    @on-pick="handleDatePick"
                    @on-pick-click="handlePickClick"></date-table>
                <year-table
                    v-ref:year-table
                    v-show="currentView === 'year'"
                    :year="year"
                    :date="date"
                    :selection-mode="selectionMode"
                    :disabled-date="disabledDate"
                    @on-pick="handleYearPick"
                    @on-pick-click="handlePickClick"></year-table>
                <month-table
                    v-ref:month-table
                    v-show="currentView === 'month'"
                    :month="month"
                    :date="date"
                    :selection-mode="selectionMode"
                    :disabled-date="disabledDate"
                    @on-pick="handleMonthPick"
                    @on-pick-click="handlePickClick"></month-table>
            </div>
            <Confirm
                v-if="confirm"
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
    import Confirm from '../base/confirm.vue';

    import Mixin from './mixin';

    const prefixCls = 'ivu-picker-panel';
    const datePrefixCls = 'ivu-date-picker';

    export default {
        mixins: [Mixin],
        components: { Icon, DateTable, YearTable, MonthTable, Confirm },
        data () {
            return {
                prefixCls: prefixCls,
                datePrefixCls: datePrefixCls,
                shortcuts: [],
                currentView: 'date',
                date: new Date(),
                value: '',
                showTime: false,
                selectionMode: 'day',
                disabledDate: '',
                year: null,
                month: null,
                confirm: false
            };
        },
        computed: {
            classes () {
                return [
                    `${prefixCls}-body-wrapper`,
                    {
                        [`${prefixCls}-with-sidebar`]: this.shortcuts.length
                    }
                ];
            },
            yearLabel () {
                const year = this.year;
                if (!year) return '';
                if (this.currentView === 'year') {
                    const startYear = Math.floor(year / 10) * 10;
                    return `${startYear}年 - ${startYear + 9}年`;
                }
                return `${year}年`;
            }
        },
        watch: {
            value (newVal) {
                if (!newVal) return;
                newVal = new Date(newVal);
                if (!isNaN(newVal)) {
                    this.date = newVal;
                    this.year = newVal.getFullYear();
                    this.month = newVal.getMonth();
                }
            }
        },
        methods: {
            resetDate () {
                this.date = new Date(this.date);
            },
            handleClear() {
                this.date = new Date();
                this.$emit('on-pick', '');
            },
            resetView() {
                if (this.selectionMode === 'month') {
                    this.currentView = 'month';
                } else if (this.selectionMode === 'year') {
                    this.currentView = 'year';
                } else {
                    this.currentView = 'date';
                }

                this.year = this.date.getFullYear();
                this.month = this.date.getMonth();
            },
            prevYear () {
                if (this.currentView === 'year') {
                    this.$refs.yearTable.prevTenYear();
                } else {
                    this.year--;
                    this.date.setFullYear(this.year);
                    this.resetDate();
                }
            },
            nextYear () {
                if (this.currentView === 'year') {
                    this.$refs.yearTable.nextTenYear();
                } else {
                    this.year++;
                    this.date.setFullYear(this.year);
                    this.resetDate();
                }
            },
            prevMonth () {
                this.month--;
                if (this.month < 0) {
                    this.month = 11;
                    this.year--;
                }
            },
            nextMonth () {
                this.month++;
                if (this.month > 11) {
                    this.month = 0;
                    this.year++;
                }
            },
            showYearPicker () {
                this.currentView = 'year';
            },
            showMonthPicker () {
                this.currentView = 'month';
            },
            handleYearPick(year, close = true) {
                this.year = year;
                if (!close) return;

                this.date.setFullYear(year);
                if (this.selectionMode === 'year') {
                    this.$emit('on-pick', new Date(year, 0, 1));
                } else {
                    this.currentView = 'month';
                }

                this.resetDate();
            },
            handleMonthPick (month) {
                this.month = month;
                const selectionMode = this.selectionMode;
                if (selectionMode !== 'month') {
                    this.date.setMonth(month);
                    this.currentView = 'date';
                    this.resetDate();
                } else {
                    this.date.setMonth(month);
                    this.year && this.date.setFullYear(this.year);
                    this.resetDate();
                    const value = new Date(this.date.getFullYear(), month, 1);
                    this.$emit('on-pick', value);
                }
            },
            handleDatePick (value) {
                if (this.selectionMode === 'day') {
                    if (!this.showTime) {
                        this.$emit('on-pick', new Date(value.getTime()));
                    }
                    this.date.setFullYear(value.getFullYear());
                    this.date.setMonth(value.getMonth());
                    this.date.setDate(value.getDate());
                }

                this.resetDate();
            }
        },
        compiled () {
            if (this.selectionMode === 'month') {
                this.currentView = 'month';
            }

            if (this.date && !this.year) {
                this.year = this.date.getFullYear();
                this.month = this.date.getMonth();
            }
        }
    };
</script>
