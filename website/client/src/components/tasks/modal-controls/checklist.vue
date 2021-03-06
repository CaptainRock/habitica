<template>
  <div class="checklist-component">
    <label
      v-once
      class="mb-1"
    >{{ $t('checklist') }}</label>
    <br>
    <draggable
      v-model="checklist"
      :options="{
        handle: '.grippy',
        filter: '.task-dropdown',
        disabled: disabled,
      }"
      @update="updateChecklist"
    >
      <div
        v-for="(item, $index) in checklist"
        :key="item.id"
        class="inline-edit-input-group checklist-group input-group"
      >
        <span
          v-if="!disabled"
          class="grippy"
          v-html="icons.grip"
        >
        </span>

        <checkbox
          :id="`checklist-${item.id}`"
          :checked.sync="item.completed"
          :disabled="disabled"
          class="input-group-prepend"
          :class="{'cursor-auto': disabled}"
        />

        <input
          v-model="item.text"
          class="inline-edit-input checklist-item form-control"
          type="text"
          :disabled="disabled"
        >
        <span
          v-if="!disabled"
          class="input-group-append"
          @click="removeChecklistItem($index)"
        >
          <div
            v-once
            class="svg-icon destroy-icon"
            v-html="icons.destroy"
          >
          </div>
        </span>
      </div>
    </draggable>
    <div
      v-if="!disabled"
      class="inline-edit-input-group checklist-group input-group new-checklist"
    >
      <span
        v-once
        class="input-group-prepend new-icon"
        v-html="icons.positive"
      >
      </span>

      <input
        v-model="newChecklistItem"
        class="inline-edit-input checklist-item form-control"
        type="text"
        :placeholder="$t('newChecklistItem')"
        @keypress.enter="setHasPossibilityOfIMEConversion(false)"
        @keyup.enter="addChecklistItem($event, true)"
        @blur="addChecklistItem($event, false)"
      >
    </div>
  </div>
</template>

<script>
// import clone from 'lodash/clone';
import draggable from 'vuedraggable';
import { v4 as uuid } from 'uuid';

import positiveIcon from '@/assets/svg/positive.svg';
import deleteIcon from '@/assets/svg/delete.svg';
import chevronIcon from '@/assets/svg/chevron.svg';
import gripIcon from '@/assets/svg/grip.svg';
import checkbox from '@/components/ui/checkbox';

export default {
  name: 'Checklist',
  components: {
    draggable,
    checkbox,
  },
  props: {
    disabled: {
      type: Boolean,
    },
    items: {
      type: Array,
    },
  },
  data () {
    return {
      checklist: this.items,
      hasPossibilityOfIMEConversion: true,
      newChecklistItem: null,
      icons: Object.freeze({
        positive: positiveIcon,
        destroy: deleteIcon,
        chevron: chevronIcon,
        grip: gripIcon,
      }),
    };
  },
  methods: {
    updateChecklist () {
      this.$emit('update:items', this.checklist);
    },
    setHasPossibilityOfIMEConversion (bool) {
      this.hasPossibilityOfIMEConversion = bool;
    },
    addChecklistItem (e, checkIME) {
      if (e) {
        e.preventDefault();
      }

      const newChecklistItemText = (this.newChecklistItem || '').trim();

      if ((checkIME && this.hasPossibilityOfIMEConversion)
        || !newChecklistItemText) {
        return;
      }

      const checkListItem = {
        id: uuid(),
        text: newChecklistItemText,
        completed: false,
      };
      this.checklist.push(checkListItem);
      this.newChecklistItem = null;
      this.setHasPossibilityOfIMEConversion(true);
      this.updateChecklist();
    },
    removeChecklistItem (i) {
      this.checklist.splice(i, 1);
      this.updateChecklist();
    },
  },
};
</script>

<style lang="scss">
  @import '~@/assets/scss/colors.scss';

  .checklist-component {

    .checklist-group {
      height: 2rem;
      border-top: 1px solid $gray-500;

      &.new-checklist {
        border-bottom: 1px solid $gray-500;
      }

      .inline-edit-input  {
        padding-left: 0.75rem;
      }

      .input-group-prepend {
        margin-left: 0.375rem;
        margin-top: 0.475rem;
        margin-right: 0;
        padding: 0;
        &:not(.new-icon) {
          width: 1.125rem;
          height: 1.125rem;
        }
        &.new-icon {
          margin-left: 0.688rem;
          margin-top: 0.625rem;
          margin-bottom: 0.625rem;
        }
      }

      .input-group-append,
      .input-group-prepend {
        background: inherit;
      }

      .checklist-item {
        padding: 0;
        margin-left: 0.75rem;
        margin-top: 0.188rem;
        margin-bottom: 0.25rem;
        height: 1.5rem;
        font-size: 14px;
        font-weight: normal;
        font-stretch: normal;
        font-style: normal;
        line-height: 1.71;
        letter-spacing: normal;
        color: $gray-50;
      }

      .new-icon {
        cursor: default;

        svg {
          width: 0.625rem;
          height: 0.625rem;
          object-fit: contain;
          fill: $gray-200;
        }
      }
    }

    span.grippy {
      position: absolute;
      left: -15px;
      width: 0.625rem;
      height: 1rem;
      object-fit: contain;
      color: $gray-200;
      top: 4px;
    }

    .checklist-item {
      margin-bottom: 0px;
      border-radius: 0px;
      border: none !important;
      padding-left: 36px;
    }

    .checklist-group {
      .grippy {
        opacity: 0;
        cursor: pointer;

        &:hover, &:active {
          opacity: 1;
        }
      }

      .destroy-icon {
        display: none;
      }

      &:hover {
        cursor: text;

        .destroy-icon {
          display: inline-block;
          color: $gray-200;

          &:hover {
            color: $maroon-50;
          }
        }

        .grippy {
          display: inline-block;
          opacity: 1;
        }
      }
    }

    label {
      height: 1.5rem;
      font-size: 14px;
      font-weight: bold;
      line-height: 1.71;
      letter-spacing: normal;
      color: $gray-50;
    }
  }
</style>
