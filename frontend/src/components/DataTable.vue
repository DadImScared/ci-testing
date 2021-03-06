<template>
  <v-data-table
    :headers="headers"
    :pagination.sync="pagination"
    hide-actions
    :items="items"
    v-bind="datatableProps"
  >
    <template
      slot="items"
      slot-scope="{ item }"
    >
      <td v-for="( value , key, index) in headers" :class="getStyles(value)" :key="index">
        <template v-if="computedComponent[value.value]">
          <table-data
            :item="item"
            :value="value.value"
            :Component="computedComponent[value.value]"
            :componentData="getComponentData(value)"
          >
          </table-data>
        </template>
        <template v-else>
          {{ getItem(item, value) }}
        </template>
      </td>
    </template>

    <template slot="no-data">
      <slot name="table-no-data"></slot>
    </template>

    <template slot="footer">
      <slot name="table-footer"></slot>
    </template>
  </v-data-table>
</template>

<script lang="ts">
import { Component, Vue, Prop } from "vue-property-decorator";
import _ from "lodash";
import TableData from "@/components/TableData";

interface IAnyObject {
  [key: string]: any;
}

@Component({
  name: "DataTable",
  components: { TableData }
})
export default class DataTable extends Vue {
  pagination = { rowsPerPage: -1 };

  // separator for array.join
  @Prop({ default: ", " })
  separator!: string;

  @Prop({ default: () => ({}) })
  styles!: { [key: string]: boolean };

  // Callback for missing value in all columns
  @Prop({ default: () => (value: any) => "" })
  missingFileCb!: (value: any) => any;

  // Callback for custom value per property
  @Prop({ default: () => ({}) })
  computedValue!: { [key: string]: (value: string, item: any) => any };

  // Pass item and value to component for additional control
  @Prop({ default: () => ({}) })
  computedComponent!: { [key: string]: any };

  @Prop({ default: () => ({}) })
  componentData!: { [key: string]: any };

  // Items for v-data-table component
  @Prop() items!: any[];

  // Props for v-data-table component
  @Prop({ default: () => ({}) })
  datatableProps!: IAnyObject;

  // Headers for v-data-table component
  @Prop() headers!: IAnyObject[];

  getStyles({ value }: { value: string }) {
    return this.styles[value] || {};
  }

  getComponentData(value: string) {
    return _.get(this.componentData, value, {});
  }

  getItem(item: any, { value }: { value: string }) {
    // display value based on cb from parent
    if (this.computedValue[value]) return this.computedCb(value, item);

    if (Array.isArray(item[value])) {
      return _.get(item, value).join(this.separator);
    }
    return _.get(item, value, this.missingFileCb(value));
  }

  computedCb(value: string, item: any) {
    return this.computedValue[value](value, item);
  }
}
</script>

<style scoped>
</style>
