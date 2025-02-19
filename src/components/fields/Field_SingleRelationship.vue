<template>
  <div>
    <div class="flex justify-start items-center text-weight-bolder q-mb-sm q-mt-md">
      <q-icon v-if="inputIcon" :name="inputIcon" :size="inputIcon.includes('fas')? '15px': '20px'" class="q-mr-md"/>
      {{inputDataBluePrint.name}}
       <q-icon v-if="toolTip && !disableDocumentToolTips" name="mdi-help-circle" size="16px" class="q-ml-md">
         <q-tooltip :delay="500">
           <span v-html="toolTip"/>
        </q-tooltip>
      </q-icon>
      <q-icon v-if="isOneWayRelationship" name="mdi-arrow-right-bold" size="16px" class="q-ml-md">
         <q-tooltip :delay="500" v-if="!disableDocumentToolTips">
          This is a one-way relationship. <br> Editing this value <span class="text-secondary">WILL NOT</span> have any effect on the connected document/s.
          <br>
          <br>
          Left-clicking the linked document in non-edit mode will open it in new tab and focuses on it.
          <br>
          Middle-clicking the linked document in non-edit mode will open it in new tab and not focus on it.
        </q-tooltip>
      </q-icon>
      <q-icon v-if="!isOneWayRelationship" name="mdi-arrow-left-right-bold" size="16px" class="q-ml-md">
         <q-tooltip :delay="500" v-if="!disableDocumentToolTips">
          This is a two-way relationship. <br> Editing this value <span class="text-secondary">WILL</span> also affect the connected document/s.
          <br>
          <br>
          Left-clicking the linked document in non-edit mode will open it in new tab and focuses on it.
          <br>
          Middle-clicking the linked document in non-edit mode will open it in new tab and not focus on it.
        </q-tooltip>
      </q-icon>
    </div>

    <q-list
      v-if="!editMode && localInput"
      class="connectionList"
      dense>
      <q-item
      clickable
      class="text-primary"
      >
        <q-item-section
          @click.left="openExistingDocumentRoute(localInput)"
          @click.middle="openNewTab(localInput)"
          >
          <span class="text-weight-medium">
            <span class="isDeadIndicator" v-if="localInput.isDead">
              †
            </span>
            <span :class="{'isDead': (localInput.isDead && !hideDeadCrossThrough)}">
                {{stripTags(localInput.label)}}
            </span>
          </span>
          <span class="inline-block q-ml-xs text-italic connectionNote">
            {{retrieveNoteText()}}
          </span>
          <q-btn
            tabindex="-1"
            round
            flat
            dense
            dark
            color="primary"
            icon="mdi-open-in-new"
            size="12px"
            class="relationshipOpeningButton"
            @click.stop.prevent.left="openNewTab(localInput)"
            >
            <q-tooltip :delay="500">
              Open in new tab without leaving this one
            </q-tooltip>
          </q-btn>
        </q-item-section>
         <q-menu
              touch-position
              context-menu
              auto-close
              separate-close-popup
            >

              <q-list class="bg-gunmetal-light text-accent">

                <template>
                  <q-item clickable  @click="copyName(fixGetCorrectDocument(localInput))">
                    <q-item-section>Copy name</q-item-section>
                    <q-item-section avatar>
                      <q-icon name="mdi-text-recognition" />
                    </q-item-section>
                  </q-item>
                  <q-item clickable @click="copyTextColor(fixGetCorrectDocument(localInput))">
                    <q-item-section>Copy text color</q-item-section>
                    <q-item-section avatar>
                      <q-icon name="mdi-eyedropper" />
                    </q-item-section>
                  </q-item>
                  <q-item clickable @click="copyBackgroundColor(fixGetCorrectDocument(localInput))">
                    <q-item-section>Copy background color</q-item-section>
                    <q-item-section avatar>
                      <q-icon name="mdi-format-color-fill" />
                    </q-item-section>
                  </q-item>
                  <q-separator dark />
                    <q-item clickable @click="openExistingInput(fixGetCorrectDocument(localInput))">
                    <q-item-section>Open document</q-item-section>
                    <q-item-section avatar>
                      <q-icon name="mdi-book-open-page-variant-outline" />
                    </q-item-section>
                  </q-item>
                  <q-item clickable @click="editExistingInput(fixGetCorrectDocument(localInput))">
                    <q-item-section>Edit document</q-item-section>
                    <q-item-section avatar>
                      <q-icon name="mdi-pencil" />
                    </q-item-section>
                  </q-item>
                  <q-item clickable @click="addNewUnderParent(fixGetCorrectDocument(localInput))">
                    <q-item-section>Create new document with this document as parent</q-item-section>
                    <q-item-section avatar>
                      <q-icon color="primary" name="mdi-file-tree" />
                    </q-item-section>
                  </q-item>
                  <q-item clickable @click="copyTargetDocument(fixGetCorrectDocument(localInput))">
                    <q-item-section>Copy this document</q-item-section>
                    <q-item-section avatar>
                      <q-icon color="primary" name="mdi-content-copy" />
                    </q-item-section>
                  </q-item>
                </template>
              </q-list>

            </q-menu>
      </q-item>
    </q-list>

    <div class="flex" v-if="editMode">
      <q-select
        class="singleRelashionshipSelect"
        menu-anchor="bottom middle"
        menu-self="top middle"
        dark
        popup-content-class="menuResizer"
        style="flex-grow: 1;"
        dense
        :ref="`singleRelationshipField${inputDataBluePrint.id}`"
        :options="filterList"
        :option-disable="opt => Object(opt) === opt ? disabledIDList.includes(opt._id) : true"
        use-input
        :outlined="!isDarkMode"
        :filled="isDarkMode"
        input-debounce="500"
        option-value="_id"
        v-model="localInput"
        @filter="filterSelect"
        @input="processSelectInteraction"
      >
        <template v-slot:append>
          <q-btn round dense flat v-slot:append v-if="!hideAdvSearchCheatsheetButton" icon="mdi-help-rhombus" @click.stop.prevent="SSET_setAdvSearchWindowVisible"
          >
            <q-tooltip :delay="500">
              Open search cheatsheet
            </q-tooltip>
          </q-btn>
        </template>
        <template v-slot:selected-item="scope">
          <q-chip
            v-if="scope.opt.label && scope.opt.label.length > 0"
            removable
            dense
            @click="openNewTab(scope.opt)"
            @remove="scope.removeAtIndex(scope.index)"
            :tabindex="scope.tabindex"
            color="accent"
            text-color="dark"
            class="text-bold"
          >
            <template v-if="scope.opt.isDead">
              †
            </template>
            {{ stripTags(scope.opt.label) }}
            <q-btn
              round
              dense
              flat
              class="z-15 relationshipChipNewTab"
              style="color: #000 !important;"
              size="sm"
              icon="mdi-open-in-new"
              @click.stop.prevent="openNewTab(scope.opt)"
            >
            <q-tooltip :delay="500">
                Open in new tab without leaving this one
              </q-tooltip>
            </q-btn>
            <q-menu
                touch-position
                context-menu
                auto-close
                separate-close-popup
              >

                <q-list class="bg-gunmetal-light text-accent">

                  <template>
                    <q-item clickable @click="copyName(fixGetCorrectDocument(scope.opt))">
                      <q-item-section>Copy name</q-item-section>
                      <q-item-section avatar>
                        <q-icon name="mdi-text-recognition" />
                      </q-item-section>
                    </q-item>
                    <q-item clickable @click="copyTextColor(fixGetCorrectDocument(scope.opt))">
                      <q-item-section>Copy text color</q-item-section>
                      <q-item-section avatar>
                        <q-icon name="mdi-eyedropper" />
                      </q-item-section>
                    </q-item>
                    <q-item clickable @click="copyBackgroundColor(fixGetCorrectDocument(scope.opt))">
                      <q-item-section>Copy background color</q-item-section>
                      <q-item-section avatar>
                        <q-icon name="mdi-format-color-fill" />
                      </q-item-section>
                    </q-item>
                    <q-separator dark />
                      <q-item clickable @click="openExistingInput(fixGetCorrectDocument(scope.opt))">
                      <q-item-section>Open document</q-item-section>
                      <q-item-section avatar>
                        <q-icon name="mdi-book-open-page-variant-outline" />
                      </q-item-section>
                    </q-item>
                    <q-item clickable @click="editExistingInput(fixGetCorrectDocument(scope.opt))">
                      <q-item-section>Edit document</q-item-section>
                      <q-item-section avatar>
                        <q-icon name="mdi-pencil" />
                      </q-item-section>
                    </q-item>
                    <q-item clickable @click="addNewUnderParent(fixGetCorrectDocument(scope.opt))">
                      <q-item-section>Create new document with this document as parent</q-item-section>
                      <q-item-section avatar>
                        <q-icon color="primary" name="mdi-file-tree" />
                      </q-item-section>
                    </q-item>
                    <q-item clickable @click="copyTargetDocument(fixGetCorrectDocument(scope.opt))">
                      <q-item-section>Copy this document</q-item-section>
                      <q-item-section avatar>
                        <q-icon color="primary" name="mdi-content-copy" />
                      </q-item-section>
                    </q-item>
                  </template>
                </q-list>

              </q-menu>
          </q-chip>
          </template>

        <template v-slot:option="{ itemProps, itemEvents, opt }" >
            <q-item
              :class="{'hasTextShadow': textShadow, 'isMinor':opt.isMinor}"
              v-bind="itemProps"
              v-on="itemEvents"
              :key="opt.id"
              :style="`background-color: ${opt.bgColor}`"
              :title="(disabledIDList.includes(opt._id)) ? 'This option is unavailable for selection as it is already paired to another.': ''"
            >
              <q-item-section avatar>
                <q-icon
                  :style="`color: ${retrieveIconColor(opt)}`"
                  :name="(opt.isCategory) ? 'fas fa-folder-open' : opt.icon"
                  />
              </q-item-section>
              <q-item-section>
                <q-item-label
                  :style="`color: ${opt.color}`"
                  >
                  <span class="isDeadIndicator" v-if="opt.isDead">
                    †
                  </span>
                  <span :class="{'isDead': (opt.isDead && !hideDeadCrossThrough)}" v-html="opt.label">
                  </span>
                </q-item-label>
                <q-item-label caption class="text-cultured" v-html="opt.hierarchicalPath"></q-item-label>
                <q-item-label caption class="text-cultured" v-if="opt.tags">
                  <q-chip
                  v-for="(input,index) in opt.tags" :key="index"
                  outline
                  style="opacity: 0.8;"
                  size="12px"
                  class="text-cultured noBounce"
                  v-html="`${input}`"
                  >
                  </q-chip>
                </q-item-label>
              </q-item-section>

              <q-menu
                touch-position
                context-menu
                auto-close
                separate-close-popup
              >

                <q-list class="bg-gunmetal-light text-accent">

                  <template>
                    <q-item clickable  @click="copyName(opt)">
                      <q-item-section>Copy name</q-item-section>
                      <q-item-section avatar>
                        <q-icon name="mdi-text-recognition" />
                      </q-item-section>
                    </q-item>
                    <q-item clickable @click="copyTextColor(opt)">
                      <q-item-section>Copy text color</q-item-section>
                      <q-item-section avatar>
                        <q-icon name="mdi-eyedropper" />
                      </q-item-section>
                    </q-item>
                    <q-item clickable @click="copyBackgroundColor(opt)">
                      <q-item-section>Copy background color</q-item-section>
                      <q-item-section avatar>
                        <q-icon name="mdi-format-color-fill" />
                      </q-item-section>
                    </q-item>
                    <q-separator dark />
                      <q-item clickable @click="openExistingInput(opt)">
                      <q-item-section>Open document</q-item-section>
                      <q-item-section avatar>
                        <q-icon name="mdi-book-open-page-variant-outline" />
                      </q-item-section>
                    </q-item>
                    <q-item clickable @click="editExistingInput(opt)">
                      <q-item-section>Edit document</q-item-section>
                      <q-item-section avatar>
                        <q-icon name="mdi-pencil" />
                      </q-item-section>
                    </q-item>
                    <q-item clickable @click="addNewUnderParent(opt)">
                      <q-item-section>Create new document with this document as parent</q-item-section>
                      <q-item-section avatar>
                        <q-icon color="primary" name="mdi-file-tree" />
                      </q-item-section>
                    </q-item>
                    <q-item clickable @click="copyTargetDocument(opt)">
                      <q-item-section>Copy this document</q-item-section>
                      <q-item-section avatar>
                        <q-icon color="primary" name="mdi-content-copy" />
                      </q-item-section>
                    </q-item>
                  </template>

                </q-list>

              </q-menu>
            </q-item>
          </template>
      </q-select>

      <table class="q-mt-sm" v-if="localInput && inputFieldID !== 'parentDoc'">
        <tr>
          <td>
            {{stripTags(localInput.label)}}
          </td>
          <td>
            <q-input
              label="Note"
              v-model="inputNote.value"
              dense
              @keydown="processInput"
              :outlined="!isDarkMode"
              :filled="isDarkMode"
              >
            </q-input>
          </td>

        </tr>
      </table>

    </div>

    <div class="separatorWrapper">
      <q-separator color="grey q-mt-md" />
    </div>

  </div>

</template>

<script lang="ts">
import { Component, Emit, Prop, Watch } from "vue-property-decorator"

import FieldBase from "src/components/fields/_FieldBase"

import { advancedDocumentFilter } from "src/scripts/utilities/advancedDocumentFilter"
import { extend } from "quasar"

import { I_OpenedDocument, I_ShortenedDocument } from "src/interfaces/I_OpenedDocument"
import { I_FieldRelationship, I_RelationshipPairSingle } from "src/interfaces/I_FieldRelationship"
import { createNewWithParent } from "src/scripts/documentActions/createNewWithParent"
import { copyDocumentName, copyDocumentTextColor, copyDocumentBackgroundColor } from "src/scripts/documentActions/uniqueFieldCopy"
import { copyDocument } from "src/scripts/documentActions/copyDocument"

@Component({
  components: { }
})
export default class Field_SingleRelationship extends FieldBase {
  /****************************************************************/
  // BASIC FIELD DATA
  /****************************************************************/

  /**
   * Already existing value in the input field (IF one is there right now)
   */
  @Prop({ default: "" }) readonly inputDataValue!: I_RelationshipPairSingle

  /**
   * ID of the document this field belongs to
   */
  @Prop({ default: "" }) readonly currentId!: ""

  /**
   * Current field ID
   * This is used for special handling of the "parentDoc" field used for hiearachical pathing
   */
  get inputFieldID () {
    return this.inputDataBluePrint?.id
  }

  /**
   * Determines if this is a one or two way relationship
   */
  get isOneWayRelationship () {
    return (this.inputDataBluePrint.type === "singleToNoneRelationship" || this.inputDataBluePrint.type === "manyToNoneRelationship")
  }

  /****************************************************************/
  // INPUT HANDLING
  /****************************************************************/

  /**
   * Watch changes to the prefilled data already existing in the field and update local input accordingly
   * Also reload the local object list
   */
  @Watch("inputDataValue", { deep: true, immediate: true })
  reactToInputChanges () {
    // @ts-ignore
    this.localInput = (this.inputDataValue?.value) ? this.inputDataValue.value : ""

    this.inputNote = (!this.inputDataValue?.addedValues) ? this.inputNote : this.inputDataValue.addedValues

    this.reloadObjectListAndCheckIfValueExists()
  }

  /**
   * Reload the local object list based on blueprint changes
   */
  @Watch("inputDataBluePrint", { deep: true, immediate: true })
  reactToBlueprintChanges () {
    this.reloadObjectListAndCheckIfValueExists()
  }

  /**
   * Reload the local object list based on current document ID changes
   */
  @Watch("currentId")
  reactToIDChanges () {
    this.reloadObjectListAndCheckIfValueExists()
  }

  /**
   * Model for the local input
   */
  localInput = "" as unknown as I_FieldRelationship

  /**
   * A note paired to the local input
   */
  inputNote: { pairedId: string; value: string; } = {
    pairedId: "",
    value: ""
  }

  /**
   * Retrieves note text
   */
  retrieveNoteText () {
    const pairedNote = this.inputNote
    return (pairedNote && pairedNote.value.length > 0) ? `(${pairedNote.value})` : ""
  }

  /**
   * A list of all retrieved documents without the current one
   */
  allTypeDocuments: I_ShortenedDocument[] = []

  /**
   * A copy of the list for the filter feed
   * A copy is needed here as the list gets modified as the filter returns highlights and similar
   */
  filterList: I_ShortenedDocument[] = []

  /**
   * Refocus after filtering to avoid un-intuitive focusing
   */
  async refocusSelect () {
    await this.$nextTick()
    /*eslint-disable */
    // @ts-ignore 
    this.$refs[`singleRelationshipField${this.inputDataBluePrint.id}`].setOptionIndex(-1)
    // @ts-ignore 
    this.$refs[`singleRelationshipField${this.inputDataBluePrint.id}`].moveOptionSelection(1, true) 
    /* eslint-enable */
  }

  /**
   * Filter the document list
   */
  filterSelect (val: string, update: (e: () => void) => void) {
    if (val === "") {
      update(() => {
        this.filterList = this.allTypeDocuments
          .filter((obj) => !obj.isMinor && obj._id !== this.currentId)

        if (this.$refs[`singleRelationshipField${this.inputDataBluePrint.id}`] && this.filterList.length > 0) {
          this.refocusSelect().catch(e => console.log(e))
        }
      })
      return
    }

    update(() => {
      const needle = val.toLowerCase()
      this.filterList = extend(true, [], this.allTypeDocuments)
      // @ts-ignore
      this.filterList = advancedDocumentFilter(needle, this.filterList, this.SGET_allBlueprints, this.filterList)
        .filter((obj) => obj._id !== this.currentId)

      if (this.$refs[`singleRelationshipField${this.inputDataBluePrint.id}`] && this.filterList.length > 0) {
        this.refocusSelect().catch(e => console.log(e))
      }
    })
  }

  disabledIDList: string[] = []

  /**
   * Prepares the initial loading of the list for filtering and furhter use
   * Also remove the document itself from the list, checks if connected input fields even exist and altogether formats and clears the list
   */
  reloadObjectListAndCheckIfValueExists () {
    if (this.inputDataBluePrint?.relationshipSettings && this.currentId.length > 0) {
      // If this is the "parentDoc" field, include categories, otherwise, filter them out from the list
      const isBelongsUnder = (this.inputDataBluePrint.id === "parentDoc")
      const allDbObjects = (isBelongsUnder)
        ? this.SGET_allDocumentsByType(this.inputDataBluePrint.relationshipSettings.connectedObjectType)
        : this.SGET_allDocumentsByTypeWithoutCategories(this.inputDataBluePrint.relationshipSettings.connectedObjectType)

      // Map all of the documents to something more digestible for the select
      allDbObjects.docs.forEach((doc) => {
        const objectDoc = doc as unknown as I_ShortenedDocument

        const pairedField = (this.inputDataBluePrint?.relationshipSettings?.connectedField) || ""
        let isDisabled = false

        // If the paired field exists and if this is "singleToSingleRelationship", set it as disabled since it is already paired
        if (pairedField.length > 0) {
          const pairedFieldObject = objectDoc.extraFields.find(f => f.id === pairedField)
          const pairingType = this.inputDataBluePrint.type

          if (pairedFieldObject !== undefined && typeof pairedFieldObject?.value !== "string" && pairedFieldObject?.value !== null && pairedFieldObject?.value?.value !== null && pairingType === "singleToSingleRelationship") {
            const checkIfExists = allDbObjects.docs.find(f => f._id === pairedFieldObject?.value?.value?._id)

            if (checkIfExists) {
              isDisabled = true
            }
          }
        }

        if (isDisabled) {
          this.disabledIDList = [...new Set([
            ...this.disabledIDList,
            doc._id
          ])]
        }
      })
      // Proceed only if the local input is properly set up
      if (this.localInput._id) {
        // If the matched object doesn't exist in the object, assume it has been deleted or never existed
        if (!allDbObjects.docs.find(e => e._id === this.localInput._id)) {
          // @ts-ignore
          this.localInput = ""
        }
        // If the object does exist, make sure we have the newest available name by reasigning the label if it is different
        else {
          const matchedFieldContent = allDbObjects.docs.find(e => e._id === this.localInput._id)

          if (matchedFieldContent && (
            this.localInput.label !== matchedFieldContent.label ||
              this.localInput?.isDead !== matchedFieldContent.extraFields.find(e => e.id === "deadSwitch")?.value)
          ) {
            this.localInput.label = matchedFieldContent.label
            this.localInput.isDead = matchedFieldContent.extraFields.find(e => e.id === "deadSwitch")?.value
          }
        }
      }

      this.allTypeDocuments = allDbObjects.docs
    }
  }

  /****************************************************************/
  // FIELD ACTIONS
  /****************************************************************/

  /**
   * Opens a new tab from a connected rleationship
   */
  openNewTab (input: I_FieldRelationship) {
    const retrievedObject = (this.SGET_openedDocument(input._id)) || this.SGET_document(input._id)

    const dataPass = {
      doc: retrievedObject,
      treeAction: false
    }

    // @ts-ignore
    this.SSET_addOpenedDocument(dataPass)
  }

  processSelectInteraction (input: null| I_ShortenedDocument) {
    if (this.inputDataBluePrint.type === "singleToSingleRelationship") {
      if (input) {
        this.disabledIDList.push(input._id)
      }
      else {
        const toRemoveIndex = this.disabledIDList.findIndex(id => id === this.inputDataValue.value._id)

        if (toRemoveIndex > -1) {
          this.disabledIDList.splice(toRemoveIndex, 1)
        }
      }
    }

    this.processInput()
  }

  /**
   * Debounce timer to prevent buggy input sync
   */
  pullTimer = null as any

  processInput () {
    this.inputNote = (this.localInput !== null) ? this.inputNote : { pairedId: "", value: "" }
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
    this.inputNote = (this.localInput !== null) ? this.inputNote : { pairedId: "", value: "" }

    const exportValue = (this.localInput && this.localInput._id)
      ? {
        _id: this.localInput._id,
        id: this.localInput._id,
        type: this.localInput.type,
        url: this.localInput.url,
        pairedField: (this.inputDataBluePrint?.relationshipSettings?.connectedField) || ""
      }
      : null

    return {
      value: exportValue,
      addedValues: this.inputNote
    }
  }

  /****************************************************************/
  // TRIGGER ACTIONS
  /****************************************************************/

  docToFind = null as unknown as I_OpenedDocument

  fixGetCorrectDocument (e: I_OpenedDocument | I_FieldRelationship) {
    this.docToFind = (this.allTypeDocuments.find(doc => doc._id === e._id)) as unknown as I_OpenedDocument
    return this.docToFind
  }

  /**
   * Opened the existing input
   */
  openExistingInput (e: I_OpenedDocument) {
    // @ts-ignore
    e = (Array.isArray(e)) ? e[0] : e
    this.openExistingDocumentRoute(e)
  }

  /**
   * Opened the existing input in two modes
   * Either as a focus with closure of the dialog.
   * Or as a background tab without closing of the dialog.
   */
  editExistingInput (e: I_OpenedDocument) {
    // @ts-ignore
    e = (Array.isArray(e)) ? e[0] : e
    this.openExistingDocumentRouteWithEdit(e)
  }

  documentPass = null as unknown as I_OpenedDocument

  /****************************************************************/
  // Add new document under parent
  /****************************************************************/
  addNewUnderParent (currentDoc: I_OpenedDocument) {
    createNewWithParent(currentDoc, this)
  }

  /****************************************************************/
  // Document field copying
  /****************************************************************/

  copyName (currentDoc: I_OpenedDocument) {
    copyDocumentName(currentDoc)
  }

  copyTextColor (currentDoc: I_OpenedDocument) {
    copyDocumentTextColor(currentDoc)
  }

  copyBackgroundColor (currentDoc: I_OpenedDocument) {
    copyDocumentBackgroundColor(currentDoc)
  }

  copyTargetDocument (currentDoc: I_OpenedDocument) {
    this.documentPass = extend(true, {}, currentDoc)

    const newDocument = copyDocument(this.documentPass, this.generateUID())

    const dataPass = {
      doc: newDocument,
      treeAction: false
    }

    // @ts-ignore
    this.SSET_addOpenedDocument(dataPass)
    this.$router.push({
      path: newDocument.url
    }).catch((e: {name: string}) => {
      const errorName : string = e.name
      if (errorName === "NavigationDuplicated") {
        return
      }
      console.log(e)
    })
  }
}
</script>

<style lang="scss" scoped>
table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0 8px;
}
</style>

<style lang="scss">
.connectionList {
  .q-item {
    padding-right: 30px;
    padding-left: 10px;
  }

  .q-item__section {
    position: relative;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;

    .relationshipOpeningButton {
      position: absolute;
      right: -30px;
    }
  }
}

.connectionList .connectionNote {
  color: #000;
  opacity: 0.8;
}
</style>
