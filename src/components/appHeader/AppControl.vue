<template>

  <div
    :class="{'AppControl': !isFrontpage}"
  >

    <!-- New document dialog -->
    <newDocumentDialog
      :dialog-trigger="newObjectDialogTrigger"
      @trigger-dialog-close="newObjectDialogClose"
    />

    <!-- Existing document dialog -->
    <existingDocumentDialog
      :dialog-trigger="existingObjectDialogTrigger"
      @trigger-dialog-close="existingObjectDialogClose"
    />

    <!-- Project close dialog -->
    <projectCloseCheckDialog
     :dialog-trigger="projectCloseCheckDialogTrigger"
     :dialog-mode="'projectClose'"
      @trigger-dialog-close="projectCloseCheckDialogClose"
    />

    <!-- Keybind dialog -->
    <keybindCheatsheetDialog
      :dialog-trigger="keybindsDialogTrigger"
      @trigger-dialog-close="keybindsDialogClose"
    />

    <!-- Import project dialog -->
    <importProjectCheckDialog
      :dialog-trigger="importProjectDialogTrigger"
      @trigger-dialog-close="importProjectDialogClose"
    />

    <!-- Merge project dialog -->
    <mergeProjectCheckDialog
      :dialog-trigger="mergeProjectDialogTrigger"
      @trigger-dialog-close="mergeProjectDialogClose"
    />

    <!-- New project dialog -->
    <newProjectCheckDialog
      :dialog-trigger="newProjectDialogTrigger"
      @trigger-dialog-close="newProjectDialogClose"
    />

    <!-- About app dialog -->
    <aboutAppDialog
      :dialog-trigger="aboutAppDialogTrigger"
      @trigger-dialog-close="aboutAppDialogClose"
    />

    <!-- Changelog dialog -->
    <changeLogDialog
      :dialog-trigger="changeLogDialogTrigger"
      @trigger-dialog-close="changeLogDialogClose"
    />

    <!-- Program settings dialog -->
    <programSettingsDialog
      :dialog-trigger="programSettingsDialogTrigger"
      @trigger-dialog-close="programSettingsDialogClose"
    />

    <!-- Advanced search guide dialog -->
    <advancedSearchGuideDialog
      :dialog-trigger="advancedSearchGuideDialogTrigger"
      @trigger-dialog-close="advancedSearchGuideDialogClose"
    />

    <!-- Tips, Tricks & Trivia dialog -->
    <tipsTricksTriviaDialog
      :dialog-trigger="tipsTricksDialogTrigger"
      @trigger-dialog-close="tipsTricksDialogClose"
    />

    <!-- License dialog -->
    <licenseDialog
      :dialog-trigger="licenseDialogTrigger"
      @trigger-dialog-close="licenseDialogClose"
    />

    <!-- Repair project dialog -->
    <repairProjectDialog
      :dialog-trigger="repairProjectDialogTrigger"
      @trigger-dialog-close="repairProjectDialogClose"
    />

    <q-btn-group
      flat
      class="AppControl__buttons"
    >

      <!-- Options button -->
      <q-btn
        flat
        :ripple="false"
        dark
        size='md'
        no-caps
        @click="programSettingsDialogAssignUID"
       >
        <q-img
          :src="appLogo"
          style="height: 26px; width: 26px; margin: 0 -9px;"
        />
        <q-tooltip anchor="center right" self="center left" :delay="500">
          Program settings
        </q-tooltip>

      </q-btn>
      <q-separator color="primary" vertical dark style="opacity: 0.1;" />
      <!-- Project button-->
      <q-btn
        flat
        :ripple="false"
        dark
        size='md'
        no-caps
       >
        Project

        <q-menu
          @show="checkProjectStatus"
          anchor="bottom left"
          class="bg-gunmetal-light"
          dark
          square
          >
          <q-list class="bg-gunmetal-light" dark>

             <q-item
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              @click="newObjectAssignUID"
              :disable="!projectExists || isFrontpage"
            >
              <q-item-section>Quick-add new document</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-text-box-plus-outline" />
              </q-item-section>
            </q-item>

             <q-item
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              @click="existingObjectAssignUID"
              :disable="!projectExists || isFrontpage"
            >
              <q-item-section>Quick-search existing document</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-database-search" />
              </q-item-section>
            </q-item>

            <q-separator dark />

              <q-item
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              @click="toggleHierarchicalTree"
              :disable="!projectExists || isFrontpage"
            >
              <q-item-section>Toggle hierarchical tree</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-page-layout-sidebar-left" />
              </q-item-section>
            </q-item>

            <q-separator dark />

             <q-item
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              @click="newProjectAssignUID"
            >
              <q-item-section>New project</q-item-section>
               <q-item-section avatar>
                <q-icon name="mdi-plus" />
              </q-item-section>
            </q-item>

            <q-separator dark />

            <q-item
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              @click="commenceExport"
              :disable="!projectExists"
            >
              <q-item-section>Export current project</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-package-variant-closed" />
              </q-item-section>
            </q-item>

            <q-item
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              @click="importProjectAssignUID"
            >
              <q-item-section>Import existing project</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-package-variant" />
              </q-item-section>
            </q-item>

            <q-item
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              @click="mergeProjectAssignUID"
              :disable="!projectExists"
            >
              <q-item-section>Merge another project into the current one</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-folder-plus-outline" />
              </q-item-section>
            </q-item>

            <q-separator dark />

            <q-item clickable>
              <q-item-section>Advanced project tools</q-item-section>
              <q-item-section avatar>
                <q-icon name="keyboard_arrow_right" />
              </q-item-section>
              <q-menu anchor="top end" self="top start">
                <q-list class="bg-gunmetal text-accent">

                  <q-item
                    v-close-popup
                    clickable
                    active
                    active-class="bg-gunmetal-light text-cultured"
                    class="noHigh"
                    @click="repairProjectAssignUID"
                    :disable="!projectExists || isFrontpage"
                  >
                    <q-item-section>Repair legacy project</q-item-section>
                    <q-item-section avatar>
                      <q-icon name="mdi-folder-open-outline" />
                    </q-item-section>
                  </q-item>

                </q-list>
              </q-menu>
            </q-item>

            <q-separator dark />

            <q-item
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              @click="navigateToProjectPage"
              :disable="!projectExists || isProjectPage || SGET_allOpenedDocuments.docs.length > 0"
            >
              <q-item-section>Resume project</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-folder-open-outline" />
              </q-item-section>
            </q-item>

            <q-item
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              @click="projectCloseCheckDialogAssignUID"
              :disable="!projectExists || isFrontpage"
            >
              <q-item-section>Close project</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-exit-to-app" />
              </q-item-section>
            </q-item>
          </q-list>
        </q-menu>
      </q-btn>

      <!-- Help button-->
      <q-btn
        flat
        :ripple="false"
        dark
        size='md'
        no-caps
       >
        Help, Settings & Info
        <q-menu
          anchor="bottom left"
          class="bg-gunmetal-light"
          dark
          square
        >
          <q-list class="bg-gunmetal-light" dark>
            <q-item
              @click="programSettingsDialogAssignUID"
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              >
              <q-item-section>Program settings</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-tune" />
              </q-item-section>
            </q-item>

            <q-separator dark />

            <q-item
              @click="keybindsDialogAssignUID"
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              >
              <q-item-section>Show keybind cheatsheet</q-item-section>
                <q-item-section avatar>
                <q-icon name="mdi-keyboard-settings" />
              </q-item-section>
            </q-item>

            <q-item
              @click="advancedSearchGuideAssignUID"
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              >
              <q-item-section>Advanced search guide</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-file-question" />
              </q-item-section>
            </q-item>

            <q-item
              @click="tipsTricksAssignUID"
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              >
              <q-item-section>Tips, Tricks & Trivia</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-fire-alert" />
              </q-item-section>
            </q-item>

            <q-separator dark />

            <q-item
              @click="changeLogDialogAssignUID"
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              >
              <q-item-section>Changelog</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-clipboard-text" />
              </q-item-section>
            </q-item>

            <q-item
              @click="aboutAppDialogAssignUID"
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              >
              <q-item-section>About Fantasia Archive</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-information-variant" />
              </q-item-section>
            </q-item>

            <q-item
              @click="licenseAssignUID"
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              >
              <q-item-section>License</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-script-text-outline" />
              </q-item-section>
            </q-item>

            <q-separator dark />

            <q-item
              @click="toggleDevTools"
              v-close-popup
              clickable
              active
              active-class="bg-gunmetal-light text-cultured"
              class="noHigh"
              >
              <q-item-section>Toggle developer tools</q-item-section>
              <q-item-section avatar>
                <q-icon name="mdi-code-tags" />
              </q-item-section>
            </q-item>

          </q-list>
        </q-menu>

      </q-btn>
    </q-btn-group>

  </div>

</template>

<script lang="ts">

import { Component, Watch } from "vue-property-decorator"

import BaseClass from "src/BaseClass"
import projectCloseCheckDialog from "src/components/dialogs/ProjectCloseCheck.vue"
import keybindCheatsheetDialog from "src/components/dialogs/KeybindCheatsheet.vue"
import importProjectCheckDialog from "src/components/dialogs/ImportProjectCheck.vue"
import mergeProjectCheckDialog from "src/components/dialogs/MergeProjectCheck.vue"
import newProjectCheckDialog from "src/components/dialogs/NewProjectCheck.vue"
import repairProjectDialog from "src/components/dialogs/RepairProject.vue"
import aboutAppDialog from "src/components/dialogs/AboutApp.vue"
import changeLogDialog from "src/components/dialogs/ChangeLog.vue"
import programSettingsDialog from "src/components/dialogs/ProgramSettings.vue"
import advancedSearchGuideDialog from "src/components/dialogs/AdvancedSearchGuide.vue"
import newDocumentDialog from "src/components/dialogs/NewDocument.vue"
import existingDocumentDialog from "src/components/dialogs/ExistingDocument.vue"
import tipsTricksTriviaDialog from "src/components/dialogs/TipsTricksTrivia.vue"
import licenseDialog from "src/components/dialogs/License.vue"

import { Loading, QSpinnerGears } from "quasar"
import { retrieveCurrentProjectName, exportProject } from "src/scripts/projectManagement/projectManagent"
import { toggleDevTools } from "src/scripts/utilities/devTools"

import appLogo from "src/assets/appLogo.png"

@Component({
  components: {
    projectCloseCheckDialog,
    keybindCheatsheetDialog,
    importProjectCheckDialog,
    mergeProjectCheckDialog,
    newProjectCheckDialog,
    aboutAppDialog,
    changeLogDialog,
    advancedSearchGuideDialog,
    programSettingsDialog,
    newDocumentDialog,
    existingDocumentDialog,
    tipsTricksTriviaDialog,
    licenseDialog,
    repairProjectDialog
  }
})
export default class AppControl extends BaseClass {
  /****************************************************************/
  // Import handling
  /****************************************************************/

  /**
   * Toggles the developer tools on and off
   */
  toggleDevTools = toggleDevTools

  /**
   * Retrieves the current project name
   */
  retrieveCurrentProjectName = retrieveCurrentProjectName

  /**
   * Just an image
   */
  appLogo = appLogo

  /****************************************************************/
  // Basic component functionality
  /****************************************************************/

  /**
   * Determines if the project exists or not
   */
  projectExists: undefined | string | boolean = false

  /**
   * Determines if we are on frontpage or not
   */
  isFrontpage = true

  /**
   * Determines if we are on project page or not
   */
  isProjectPage = true

  /**
   * Current project name
   */
  projectName = ""

  created () {
    this.checkProjectStatus().catch(e => console.log(e))
  }

  async checkProjectStatus () {
    this.projectName = await retrieveCurrentProjectName()
    this.projectExists = !!(await retrieveCurrentProjectName())
    this.isFrontpage = (this.$route.path === "/")
    this.isProjectPage = (this.$route.path === "/project")
  }

  /****************************************************************/
  // Local keybinds
  /****************************************************************/

  @Watch("SGET_getCurrentKeyBindData", { deep: true })
  processKeyPush () {
    // Keybind cheatsheet
    if (this.determineKeyBind("openKeybindsCheatsheet") && !this.SGET_getDialogsState) {
      this.keybindsDialogAssignUID()
    }

    // App options
    if (this.determineKeyBind("openAppOptions") && !this.SGET_getDialogsState) {
      this.programSettingsDialogAssignUID()
    }
  }

  /****************************************************************/
  // Navigate to project page action
  /****************************************************************/

  navigateToProjectPage () {
    this.$router.push({ path: "/project" }).catch((e: {name: string}) => {
      if (e && e.name !== "NavigationDuplicated") {
        console.log(e)
      }
    })
  }

  /****************************************************************/
  // Export project action
  /****************************************************************/

  async commenceExport () {
    const projectName = await retrieveCurrentProjectName()
    const setup = {
      message: "<h4>Exporting current project...</h4>",
      spinnerColor: "primary",
      messageColor: "cultured",
      spinnerSize: 120,
      backgroundColor: "dark",
      // @ts-ignore
      spinner: QSpinnerGears
    }
    exportProject(projectName, Loading, setup, this.$q)
  }

  /****************************************************************/
  // Close project dialog
  /****************************************************************/

  projectCloseCheckDialogTrigger: string | false = false
  projectCloseCheckDialogClose () {
    this.projectCloseCheckDialogTrigger = false
  }

  projectCloseCheckDialogAssignUID () {
    this.projectCloseCheckDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // Import project dialog
  /****************************************************************/

  importProjectDialogTrigger: string | false = false
  importProjectDialogClose () {
    this.importProjectDialogTrigger = false
  }

  importProjectAssignUID () {
    this.importProjectDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // Merge project dialog
  /****************************************************************/

  mergeProjectDialogTrigger: string | false = false
  mergeProjectDialogClose () {
    this.mergeProjectDialogTrigger = false
  }

  mergeProjectAssignUID () {
    this.mergeProjectDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // New project dialog
  /****************************************************************/

  newProjectDialogTrigger: string | false = false
  newProjectDialogClose () {
    this.newProjectDialogTrigger = false
  }

  newProjectAssignUID () {
    this.newProjectDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // Keybings cheatsheet dialog
  /****************************************************************/

  keybindsDialogTrigger: string | false = false
  keybindsDialogClose () {
    this.keybindsDialogTrigger = false
  }

  keybindsDialogAssignUID () {
    this.keybindsDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // About app dialog
  /****************************************************************/

  aboutAppDialogTrigger: string | false = false
  aboutAppDialogClose () {
    this.aboutAppDialogTrigger = false
  }

  aboutAppDialogAssignUID () {
    this.aboutAppDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // Changelog dialog
  /****************************************************************/

  changeLogDialogTrigger: string | false = false
  changeLogDialogClose () {
    this.changeLogDialogTrigger = false
  }

  changeLogDialogAssignUID () {
    this.changeLogDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // Program settings dialog
  /****************************************************************/

  programSettingsDialogTrigger: string | false = false
  programSettingsDialogClose () {
    this.programSettingsDialogTrigger = false
  }

  programSettingsDialogAssignUID () {
    this.programSettingsDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // Advanced search guide dialog
  /****************************************************************/

  advancedSearchGuideDialogTrigger: string | false = false
  advancedSearchGuideDialogClose () {
    this.advancedSearchGuideDialogTrigger = false
  }

  advancedSearchGuideAssignUID () {
    this.advancedSearchGuideDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // New document dialog
  /****************************************************************/

  newObjectDialogTrigger: string | false = false
  newObjectDialogClose () {
    this.newObjectDialogTrigger = false
  }

  newObjectAssignUID () {
    this.newObjectDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // Existing document dialog
  /****************************************************************/

  existingObjectDialogTrigger: string | false = false
  existingObjectDialogClose () {
    this.existingObjectDialogTrigger = false
  }

  existingObjectAssignUID () {
    this.existingObjectDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // Tips, Tricka & Trivia dialog
  /****************************************************************/

  tipsTricksDialogTrigger: string | false = false
  tipsTricksDialogClose () {
    this.tipsTricksDialogTrigger = false
  }

  tipsTricksAssignUID () {
    this.tipsTricksDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // License dialog
  /****************************************************************/

  licenseDialogTrigger: string | false = false
  licenseDialogClose () {
    this.licenseDialogTrigger = false
  }

  licenseAssignUID () {
    this.licenseDialogTrigger = this.generateUID()
  }

  /****************************************************************/
  // Repair project dialog
  /****************************************************************/

  repairProjectDialogTrigger: string | false = false
  repairProjectDialogClose () {
    this.repairProjectDialogTrigger = false
  }

  repairProjectAssignUID () {
    this.repairProjectDialogTrigger = this.generateUID()
  }
}
</script>

<style lang="scss" scoped>
.AppControl {
  background: rgba(0, 0, 0, 0.1);

  &__buttons {
    height: 40px;
    -webkit-app-region: no-drag;
  }
}
</style>
