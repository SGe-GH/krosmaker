<template>
  <v-card-text>
    <h1>{{ $t("card.edit.powers") }}</h1>

    <v-expansion-panels
      class="pa-2 pt-4"
      accordion
      hover
      active-class="selected-power"
      v-model="activePower"
    >
      <v-expansion-panel v-for="(power, index) in powers" :key="index">
        <v-expansion-panel-header class="panel-header" ripple>
          {{ truncate(power.name, 21) }}
        </v-expansion-panel-header>
        <v-expansion-panel-content>
          <v-row>
            <v-col class="pa-0 pt-4" cols="12">
              <v-text-field
                :label="$t('card.edit.ability.name')"
                dense
                class="name-input"
                :value="power.name"
                @input="onNameChange($event, index)"
                maxlength="40"
                required
              />
            </v-col>
            <v-col class="pa-0" cols="12">
              <v-textarea
                :label="$t('card.edit.ability.description')"
                :value="power.description"
                @input="onDescriptionChange($event, index)"
                :rows="4"
                no-resize
                required
                persistent-hint
                :hint="$t('card.edit.ability.editHint')"
              />
            </v-col>
            <v-col class="pa-1" cols="12">
              <v-card-actions>
                <v-btn
                  color="red darken-1"
                  text
                  @click="showDeleteDialog = true"
                >
                  <v-icon dark left>mdi-delete</v-icon>
                  {{ $t("common.delete") }}
                </v-btn>
                <v-spacer></v-spacer>

                <v-btn text @click="moveUp(index)" :disabled="index === 0">
                  <v-icon dark left>mdi-chevron-up</v-icon>
                  {{ $t("common.up") }}
                </v-btn>

                <v-btn
                  text
                  @click="moveDown(index)"
                  :disabled="index === powers.length - 1"
                >
                  <v-icon dark left>mdi-chevron-down</v-icon>
                  {{ $t("common.down") }}
                </v-btn>
              </v-card-actions>
            </v-col>
          </v-row>
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>

    <transition name="fade">
      <v-card class="d-flex justify-center pa-3" v-if="!isFull" flat>
        <v-btn fab dark color="primary" @click="addPower">
          <v-icon dark>mdi-plus</v-icon>
        </v-btn>
      </v-card>
    </transition>

    <v-dialog v-model="showDeleteDialog" max-width="400">
      <v-card>
        <v-card-title class="headline">
          {{ $t("card.edit.ability.deletionHeader") }}
        </v-card-title>

        <v-card-text>
          <span>{{ $t("card.edit.ability.deletionPrompt1") }} </span>
          <strong>{{
            activePower != null ? powers[activePower].name : ""
          }}</strong
          >?
          <span> {{ $t("card.edit.ability.deletionPrompt2") }}</span>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>

          <v-btn text @click="showDeleteDialog = false">
            <v-icon dark left>mdi-cancel</v-icon>
            {{ $t("common.cancel") }}
          </v-btn>

          <v-btn color="red darken-1" text @click="deletePower">
            <v-icon dark left>mdi-delete</v-icon>
            {{ $t("common.delete") }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card-text>
</template>

<script lang="ts">
import Vue from "vue";
import { Component } from "vue-property-decorator";

import { Power } from "~/assets/src/data/fighters";
import {
  maxAbilitiesCount,
  maxMinionAbilitiesCount,
} from "~/assets/src/constants";

@Component
export default class PowersForm extends Vue {
  showDeleteDialog: boolean = false;

  get activePower(): number | null {
    return this.$store.state.sidebar.activePower;
  }

  set activePower(activePower: number | null) {
    this.$store.commit("sidebar/setActivePower", activePower);
  }

  get powers(): Power[] {
    return this.$store.state.krosmaster.powers;
  }

  get isFull(): boolean {
    const krosmaster = this.$store.state.krosmaster;
    const totalAbilities = krosmaster.powers.length + krosmaster.spells.length;
    const maxAbilities =
      krosmaster.type === "minion"
        ? maxMinionAbilitiesCount
        : maxAbilitiesCount;
    return totalAbilities >= maxAbilities;
  }

  truncate(text: string, length: number): string {
    return text.length > length ? text.substring(0, length) + "…" : text;
  }

  onNameChange(name: string, index: number) {
    this.$store.commit("export/setDirty", true);
    this.$store.commit("krosmaster/setPowerName", { index, name });
  }

  onDescriptionChange(description: string, index: number) {
    this.$store.commit("export/setDirty", true);
    this.$store.commit("krosmaster/setPowerDescription", {
      index,
      description,
    });
  }

  addPower() {
    if (!this.isFull) {
      this.$store.commit("export/setDirty", true);
      this.$store.commit("krosmaster/addPower", {
        name: this.$i18n.t("card.edit.ability.newPowerName").toString(),
        description: this.$i18n
          .t("card.edit.ability.newPowerDescription")
          .toString(),
      });
      this.activePower = this.powers.length - 1;
    }
  }

  moveUp(index: number) {
    this.movePower(index, index - 1);
  }

  moveDown(index: number) {
    this.movePower(index, index + 1);
  }

  private movePower(from: number, to: number) {
    if (this.isValidIndex(from) && this.isValidIndex(to)) {
      this.$store.commit("export/setDirty", true);
      this.$store.commit("krosmaster/switchPowers", { from, to });
      this.activePower = to;
    }
  }

  private isValidIndex(index: number) {
    return 0 <= index && index < this.powers.length;
  }

  deletePower() {
    const powerId = this.activePower;
    if (powerId != null) {
      const name = this.powers[powerId].name;
      this.$store.commit("export/setDirty", true);
      this.$store.commit("krosmaster/removePower", powerId);
      this.activePower = null;
      this.$store.commit("notification/add", {
        message: "card.edit.notification.delete",
        parameters: { name },
        color: "error",
      });
    }
    this.showDeleteDialog = false;
  }
}
</script>

<style lang="scss" scoped>
h1 {
  padding: 0.6em;
  padding-bottom: 1em;
}

.panel-header {
  > button {
    text-overflow: ellipsis;
  }
}

.selected-power {
  > button {
    background-color: #ffffff11;
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
