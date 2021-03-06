<template>
  <!-- Need to catch for ResponsiveRowSwitch @toggle events for small view -->
  <ResponsiveRowSwitch>
    <template #status>
      <v-card
        :color="status_color"
        class="pl-2 font-weight-bold"
        hover
        @click="$emit('toggle', !expanded)"
      >
        <v-card-text class="pa-2 font-weight-bold">
          {{ control.root.hdf.status }}
          <v-icon class="float-right">
            {{ expanded ? "mdi-chevron-down" : "mdi-chevron-up" }}
          </v-icon>
        </v-card-text>
      </v-card>
    </template>

    <template #severity>
      <v-card-text class="pa-2">
        <v-icon small v-for="i in severity_arrow_count" :key="i"
          >mdi-checkbox-blank-circle</v-icon
        >
        <v-icon small v-for="i in 4 - severity_arrow_count" :key="5 - i"
          >mdi-checkbox-blank-circle-outline</v-icon
        >
        <br />
        <v-divider class="lighten-4 mx-1"></v-divider>
        {{ control.hdf.severity.toUpperCase() }}
      </v-card-text>
    </template>

    <template #title>
      <v-card-text class="pa-2">{{ truncated_title }}</v-card-text>
    </template>

    <!-- ID and Tags -->
    <template #id>
      <v-card-text class="pa-2">{{ control.data.id }}</v-card-text>
    </template>
    <template #tags>
      <v-chip-group column active-class="NONE">
        <v-tooltip
          bottom
          v-for="(tag, i) in control.hdf.raw_nist_tags"
          :key="i"
        >
          <template v-slot:activator="{ on }">
            <v-chip v-on="on" active-class="NONE">
              {{ tag }}
            </v-chip>
          </template>
          <span>{{ tooltip(tag) }}</span>
        </v-tooltip>
      </v-chip-group>
    </template>
  </ResponsiveRowSwitch>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";
import { HDFControl, ControlStatus, Severity } from "inspecjs";
import ResponsiveRowSwitch from "@/components/cards/controltable/ResponsiveRowSwitch.vue";
import { context } from "inspecjs";
import { NIST_DESCRIPTIONS } from "@/utilities/nist_util";

// We declare the props separately to make props types inferable.
const ControlRowHeaderProps = Vue.extend({
  props: {
    control: {
      type: Object, // Of type HDFControl (but with added key field)
      required: true
    },
    expanded: {
      type: Boolean, // Whether or this control should be open
      required: false
    }
  }
});

@Component({
  components: {
    ResponsiveRowSwitch
  }
})
export default class ControlRowHeader extends ControlRowHeaderProps {
  /** Typed getter for control */
  get _control(): context.ContextualizedControl {
    return this.control;
  }

  // Get NIST tag description for NIST tag, this is pulled from the 800-53 xml
  // and relies on a script not contained in the project
  get tooltip(): (tag: string) => string {
    return (tag: string) => {
      return this.descriptionForTag(tag);
    };
  }

  get truncated_title(): string {
    if (this._control.data.title && this._control.data.title.length > 80) {
      return this._control.data.title.substr(0, 80) + "...";
    } else {
      return this._control.data.title || "Untitled";
    }
  }

  get status_color(): string {
    // maps stuff like "not applicable" -> "statusnotapplicable", which is a defined color name
    return `status${this._control.root.hdf.status.replace(" ", "")}`;
  }

  get severity_arrow_count(): number {
    switch (this._control.hdf.severity) {
      default:
      case "none":
        return 0;
      case "low":
        return 1;
      case "medium":
        return 2;
      case "high":
        return 3;
      case "critical":
        return 4;
    }
  }

  fmtNist(nist: string[]): string {
    return nist.join(", ");
  }

  // Get NIST tag description for NIST tag, this is pulled from the 800-53 xml
  // and relies on a script not contained in the project
  descriptionForTag(tag: string): string {
    return NIST_DESCRIPTIONS[tag] || "Unrecognized tag";
  }
}
</script>

<style scoped>
.lightened-row .v-card {
  background: var(--v-background-lighten-2);
}
</style>
