<template>
    <q-dialog
    v-model="dialogModel"
    @hide="triggerDialogClose"
    >
    <q-card dark>

      <q-card-section class="row justify-center">
        <h6 class="text-center q-my-sm">You have unsaved documents opened!</h6>
      </q-card-section>
      <q-card-section class="row justify-center q-mx-lg">
        All unsaved data will be lost upon closing the app unless the documents are saved first.
      </q-card-section>

      <q-card-section class="row q-mx-lg">
        <div class="q-mb-md text-bold">Affected documents:</div>
        <q-list class="projectCloseDalogList">
          <q-item
          v-for=" doc in openedDocsWithEdits"
          :key="doc._id"
          clickable
          v-ripple
          active
          class="noHigh"
          active-class="bg-primary-1 text-primary"
          v-close-popup
          :to="doc.url">
            <q-item-section avatar>
              <q-icon color="white" :name="doc.icon" />
            </q-item-section>

            <q-item-section class="text-primary">{{retrieveFieldValue(doc,'name')}}</q-item-section>
          </q-item>
        </q-list>

      </q-card-section>

      <q-card-actions align="around" class="q-mx-xl q-mt-lg q-mb-md">
        <q-btn
          flat
          label="Cancel"
          color="accent"
          v-close-popup />
        <q-btn
          outline
          :label="exitLabelText"
          color="secondary"
          v-close-popup
          @click="determineModeAction" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script lang="ts">

import { Component, Watch, Prop } from "vue-property-decorator"

import DialogBase from "src/components/dialogs/_DialogBase"
import { I_OpenedDocument } from "src/interfaces/I_OpenedDocument"
import { remote } from "electron"
@Component({
  components: { }
})
export default class ProjectCloseCheck extends DialogBase {
  /**
   * React to dialog opening request
   */
  @Watch("dialogTrigger")
  openDialog (val: string|false) {
    if (val) {
      this.SSET_setDialogState(true)
      this.checkForDocumentsWithEdits()
    }
  }

  /**
   * Determines if the dialog should be used in application closing mode or in project closing mode
   */
  @Prop() readonly dialogMode!: "appClose" | "projectClose"

  /**
   * Label text for the dialog
   */
  get exitLabelText () {
    if (this.dialogMode === "appClose") {
      return "Exit app without saving"
    }

    if (this.dialogMode === "projectClose") {
      return "Close project without saving"
    }
  }

  /**
   * List of opened documents with edits in them
   */
  openedDocsWithEdits: I_OpenedDocument[]= []

  /**
   * Check if we have any documents with edit. If not, skip the dialog and proceed.
   */
  checkForDocumentsWithEdits () {
    this.openedDocsWithEdits = this.SGET_allOpenedDocuments.docs.filter(doc => doc.hasEdits)

    if (this.openedDocsWithEdits.length > 0) {
      this.dialogModel = true
    }
    else {
      this.determineModeAction()
    }
  }

  /**
   * Decide what action to take depending on the dialog mode
   */
  determineModeAction () {
    if (this.dialogMode === "appClose") {
      this.closeApp()
    }
    if (this.dialogMode === "projectClose") {
      this.closeProject()
    }
  }

  /**
   * Close the project and navigate to the intro screen
   */
  closeProject () {
    this.SSET_resetDocuments()
    this.triggerDialogClose()
    this.$router.push({ path: "/" }).catch((e: {name: string}) => {
      if (e && e.name !== "NavigationDuplicated") {
        console.log(e)
      }
    })
  }

  /**
   * Close app
   */
  closeApp () {
    remote.getCurrentWindow().destroy()
  }
}
</script>

<style lang="scss">

  .projectCloseDalogList {
    width: 100%;
  }

</style>
