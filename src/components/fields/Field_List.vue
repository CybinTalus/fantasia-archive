<template>
  <div>
    <div class="flex justify-start items-center text-weight-bolder q-mb-sm q-mt-md">
      <q-icon v-if="inputIcon" :name="inputIcon"  :size="inputIcon.includes('fas')? '15px': '20px'" class="q-mr-md"/>
      {{inputDataBluePrint.name}}
       <q-icon v-if="toolTip && !disableDocumentToolTips" name="mdi-help-circle" size="16px" class="q-ml-md">
         <q-tooltip :delay="500">
           <span v-html="toolTip"/>
        </q-tooltip>
      </q-icon>
    </div>

  <q-list
      v-if="!editMode"
      class="fieldList_list"
      dense>
      <q-item v-for="(input,index) in localInput" :key="index">
        <q-item-section
          class="fieldList_itemDot"
          side>
          <q-icon
            color="primary"
            name="mdi-menu-right"
             />
        </q-item-section>
        <q-item-section>
          <span class="text-weight-medium">
            {{input.value}}
          </span>
          <span v-if="localInput[index].affix" class="inline-block q-ml-xs text-italic listNote">
            ({{localInput[index].affix}})
            </span>
        </q-item-section>
      </q-item>
    </q-list>

  <div v-if="editMode">
    <div class="row q-mb-sm"
      v-for="(singleInput,index) in localInput"
      :key="index"
    >
      <div class="col-sm-12 col-md flex">
        <q-btn
          tabindex="-1"
          round
          flat
          dense
          :disable="index === 0"
          icon="mdi-arrow-up-bold"
          class="q-mr-xs self-center"
          size="10px"
          :color="(index !== 0) ? 'primary' : ''"
          @click="moveItem(index, 'up')"
        >
          <q-tooltip
            :delay="300"
            anchor="center left"
            self="center right"
          >
          Move the item one place up
          </q-tooltip>
        </q-btn>

        <q-btn
          tabindex="-1"
          round
          flat
          dense
          :disable="index === localInput.length - 1"
          icon="mdi-arrow-down-bold"
          class="q-mr-xs self-center"
          size="10px"
          :color="(index !== localInput.length - 1) ? 'primary' : ''"
          @click="moveItem(index, 'down')"
        >
          <q-tooltip
            :delay="300"
            anchor="center left"
            self="center right"
          >
          Move the item one place down
          </q-tooltip>
        </q-btn>

        <q-input
          v-model="localInput[index].value"
          class="grow-1"
          :class="`listField_input${index}_${inputDataBluePrint.id}`"
          dense
          @keydown="processInput"
          :outlined="!isDarkMode"
          :filled="isDarkMode"
          >
        </q-input>
      </div>

      <div
      class="q-ml-lg justify-end flex"
      style="max-width: 220px; width: 220px;"
      v-if="hasExtraInput">
        <q-select
          style="width: 100%;"
          dense
          class="listAtributeSelect"
          :options="localExtraInput"
          use-input
          :hide-dropdown-icon="!editMode"
          :outlined="editMode && !isDarkMode"
          :borderless="!editMode"
          :filled="editMode && isDarkMode"
          :readonly="!editMode"
          input-debounce="0"
          new-value-mode="add"
          dark
          @input="processInput"
          @keydown="processInput"
          :label="(inputAffix) ? inputAffix : ''"
          v-model="localInput[index].affix"
        />
      </div>

      <div style="width: 115px;" class="justify-end flex">
        <q-btn
          v-if="editMode"
          color="secondary"
          :outline="isDarkMode"
          @click="removeFromList(index)"
          label="Remove" />
      </div>
    </div>

    <div class="row q-mt-xs" v-if="editMode">
      <div class="col justify-start flex">
        <q-btn
        color="primary"
        :outline="isDarkMode"
        label="Add new"
        @click="addNewInput" />
      </div>
    </div>
  </div>

    <div class="separatorWrapper">
      <q-separator color="grey q-mt-md" />
    </div>

  </div>

</template>

<script lang="ts">
import { Component, Emit, Prop, Watch } from "vue-property-decorator"

import FieldBase from "src/components/fields/_FieldBase"
import { extend } from "quasar"

@Component({
  components: { }
})
export default class Field_List extends FieldBase {
  /****************************************************************/
  // BASIC FIELD DATA
  /****************************************************************/

  /**
   * Already existing value in the input field (IF one is there right now)
   */
  @Prop({
    default: () => {
      return []
    }
  }) readonly inputDataValue!: {
    value: string
    affix?: string
  }[]

  /****************************************************************/
  // INPUT HANDLING
  /****************************************************************/

  /**
   * Watch changes to the prefilled data already existing in the field and update local input accordingly
   */
  @Watch("inputDataValue", { deep: true, immediate: true })
  reactToInputChanges () {
    this.localInput = (this.inputDataValue) ? this.inputDataValue : []
  }

  /**
   * Model for the local input
   */
  localInput = [] as {
    value: string
    affix?: string
  }[]

  /**
   * Determine if the input has any extra values attached to it or not
   */
  get hasExtraInput () {
    // @ts-ignore
    this.localExtraInput = this.inputDataBluePrint?.predefinedListExtras?.extraSelectValueList
    return this.inputDataBluePrint?.predefinedListExtras?.extraSelectValueList
  }

  /**
   * List of extra input values
   */
  localExtraInput:string[] = []

  /**
   * Label for the extra input
   * EG: "Level" or "Skill tier"
   */
  get inputAffix () {
    return (this.inputDataBluePrint?.predefinedListExtras?.affix) || ""
  }

  /**
   * Remove an existing row from the input list
   */
  removeFromList (index: number) {
    this.localInput.splice(index, 1)
    this.processInput()
  }

  /**
   * Adds new row to the input list
   */
  async addNewInput () {
    this.localInput.push({
      value: "",
      affix: ""
    })

    const targetRefStringNamer = `.listField_input${this.localInput.length - 1}_${this.inputDataBluePrint.id}`

    await this.$nextTick()

    const newInput = document.querySelector(targetRefStringNamer) as HTMLInputElement

    if (newInput) {
      newInput.focus()
    }

    this.processInput()
  }

  moveItem (index: number, direction: "up" | "down") {
    const to = (direction === "up") ? index - 1 : index + 1
    const from = index

    this.localInput.splice(to, 0, this.localInput.splice(from, 1)[0])

    this.processInput()
  }

  /**
   * Debounce timer to prevent buggy input sync
   */
  pullTimer = null as any

  processInput () {
    clearTimeout(this.pullTimer)
    this.pullTimer = setTimeout(() => {
      this.signalInput()
    }, 500)
  }

  /**
   * Signals the input change to the document body parent component
   */
  @Emit()
  signalInput () {
    const dataCopy: {
      value: string
      affix?: string
    }[] = extend(true, [], this.localInput)

    // Fix hanging whitespaces in inputs
    const returnValue = dataCopy.map(e => {
      e.value = e.value.trim()
      if (e.affix) {
        e.affix = e.affix.trim()
      }
      return e
    })

    return returnValue
  }
}
</script>

<style lang="scss">
.fieldList_list {
  .q-item {
    padding-right: 10px;
    padding-left: 0;
  }

  .q-item__section {
    position: relative;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
  }

  .fieldList_itemDot {
    padding-right: 10px;
  }
}

</style>
