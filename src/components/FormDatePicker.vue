<template>
  <div class="form-group position-relative">
    <label v-uni-for="name">{{label}}</label>
    <date-picker
      v-model="date"
      :config="config"
      :disabled="disabled"
      :placeholder="placeholder"
      :data-test="dataTest"
    />
    <div v-if="errors.length > 0" class="invalid-feedback d-block">
      <div v-for="(error, index) in errors" :key="index">{{error}}</div>
    </div>
    <small v-if="helper" class="form-text text-muted">{{helper}}</small>
  </div>
</template>

<script>
import { createUniqIdsMixin } from 'vue-uniq-ids';
import ValidationMixin from './mixins/validation';
import DataFormatMixin from "./mixins/DataFormat";
import datePicker from 'vue-bootstrap-datetimepicker';
import moment from 'moment-timezone';
import { getLang, getTimezone, getUserDateFormat, getUserDateTimeFormat } from '../dateUtils';

const uniqIdsMixin = createUniqIdsMixin();

moment.tz.setDefault(getTimezone());

export default {
  mixins: [uniqIdsMixin, ValidationMixin, DataFormatMixin],
  components: {
    datePicker
  },
  props: {
    name: String,
    placeholder: String,
    label: String,
    error: String,
    helper: String,
    dataFormat: String,
    value: String,
    inputClass: {type: [String, Array, Object], default: 'form-control'},
    dataTest: String,
    disabled: null,
  },
  data() {
    return {
      validatorErrors: [],
      date: null,
      config: {
        format: this.getFormat(),
        timeZone: getTimezone(),
        locale: getLang(),
        useCurrent: false,
        showClose: true,
        icons: {
          time: 'far fa-clock',
          date: 'far fa-calendar',
          up: 'fas fa-arrow-up',
          down: 'fas fa-arrow-down',
          previous: 'fas fa-chevron-left',
          next: 'fas fa-chevron-right',
          today: 'fas fa-calendar-check',
          clear: 'far fa-trash-alt',
          close: 'far fa-times-circle'
        },
      },
    }
  },
  computed: {
    errors() {
      if (this.error) {
        return [...this.validatorErrors, this.error];
      }

      return this.validatorErrors;
    }
  },
  watch: {
    validator: {
      deep: true,
      handler() {
        this.validatorErrors = this.validator && this.validator.errors.get(this.name)
          ? this.validator.errors.get(this.name)
          : [];
      },
    },
    date() {
      if (this.value && !this.date) {
        this.$emit('input', '');
      }

      if (this.isDateAndValueTheSame()) {
        return;
      }

      const newDate = moment(this.date, this.config.format);
      this.$emit('input', newDate.toISOString());
    },
    value() {
      if (!this.value) {
        this.date = '';
        return;
      }

      const newDate = this.generateDate(this.value);

      if (!this.isDateAndValueTheSame()) {
        this.date = newDate.format(this.config.format);
      }
    },
    dataFormat: {
      immediate: true,
      handler() {
        this.config.format = this.getFormat();
        this.date = this.value
          ? this.generateDate().format(this.config.format)
          : '';
      }
    },
  },
  methods: {
    getFormat() {
      return this.dataFormat === 'datetime'
        ? getUserDateTimeFormat()
        : getUserDateFormat();
    },
    isDateAndValueTheSame() {
      if (!this.date && !this.value) {
        return true;
      }

      const currentDate = moment(this.date, this.config.format);
      const currentValue = this.value ? moment(this.value) : null;
      const comparatorString = this.dataFormat !== 'datetime' ? 'day' : null;

      return currentDate.isSame(currentValue, comparatorString);
    },
    generateDate(value = this.value) {
      let date = moment(value);

      if (!date.isValid()) {
        date = moment();
      }

      return date;
    },
  },
};
</script>

<style>
  @import '~pc-bootstrap4-datetimepicker/build/css/bootstrap-datetimepicker.css';

  .inspector-container .bootstrap-datetimepicker-widget.dropdown-menu {
    font-size: 11px;
  }
</style>
