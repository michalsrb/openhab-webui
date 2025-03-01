<template>
  <f7-page @page:afterin="onPageAfterIn" @page:afterout="onPageAfterOut">
    <f7-navbar :title="(createMode) ? 'Create rule' : rule.name" back-link="Back" no-hairline>
      <f7-nav-right v-if="isEditable">
        <f7-link @click="save()" v-if="$theme.md" icon-md="material:save" icon-only />
        <f7-link @click="save()" v-if="!$theme.md">
          Save<span v-if="$device.desktop">&nbsp;(Ctrl-S)</span>
        </f7-link>
      </f7-nav-right>
    </f7-navbar>
    <f7-toolbar tabbar position="top">
      <f7-link @click="switchTab('design', fromYaml)" :tab-link-active="currentTab === 'design'" class="tab-link">
        Design
      </f7-link>
      <f7-link @click="switchTab('code', toYaml)" :tab-link-active="currentTab === 'code'" class="tab-link">
        Code
      </f7-link>
    </f7-toolbar>
    <f7-tabs class="sitemap-editor-tabs">
      <f7-tab id="design" @tab:show="() => this.currentTab = 'design'" :tab-active="currentTab === 'design'">
        <f7-block v-if="ready && rule.status && !createMode" class="block-narrow padding-left padding-right" strong>
          <f7-col v-if="!createMode">
            <div class="float-right align-items-flex-start align-items-center">
              <!-- <f7-toggle class="enable-toggle"></f7-toggle> -->
              <f7-link :icon-color="(rule.status.statusDetail === 'DISABLED') ? 'orange' : 'gray'" :tooltip="((rule.status.statusDetail === 'DISABLED') ? 'Enable' : 'Disable') + (($device.desktop) ? ' (Ctrl-D)' : '')" icon-ios="f7:pause_circle" icon-md="f7:pause_circle" icon-aurora="f7:pause_circle" icon-size="32" color="orange" @click="toggleDisabled" />
              <f7-link :tooltip="'Run Now' + (($device.desktop) ? ' (Ctrl-R)' : '')" icon-ios="f7:play_round" icon-md="f7:play_round" icon-aurora="f7:play_round" icon-size="32" color="blue" @click="runNow" />
            </div>
            Status:
            <f7-chip class="margin-left"
                     :text="rule.status.status"
                     :color="ruleStatusBadgeColor(rule.status)" />
            <div>
              <strong>{{ (rule.status.statusDetail !== 'NONE') ? rule.status.statusDetail : '&nbsp;' }}</strong>
              <br>
              <div v-if="rule.status.description">
                {{ rule.status.description }}
              </div>
            </div>
          </f7-col>
        </f7-block>
        <!-- skeletons for not ready -->
        <f7-block v-else-if="!createMode" class="block-narrow padding-left padding-right skeleton-text skeleton-effect-blink" strong>
          <f7-col>
            ______:
            <f7-chip class="margin-left" text="________" />
            <div>
              <strong>____ _______</strong>
              <br>
            </div>
          </f7-col>
        </f7-block>

        <!-- skeletons -->
        <f7-block v-if="!ready" class="block-narrow">
          <f7-col class="skeleton-text skeleton-effect-blink">
            <f7-list inline-labels no-hairlines-md>
              <f7-list-input label="Unique ID" type="text" placeholder="Required" value="_______" required validate
                             :disabled="true" :info="(createMode) ? 'Note: cannot be changed after the creation' : ''"
                             @input="rule.uid = $event.target.value" :clear-button="createMode" />
              <f7-list-input label="Name" type="text" placeholder="Required" required validate
                             :disabled="true" @input="rule.name = $event.target.value" :clear-button="isEditable" />
              <f7-list-input label="Description" type="text" value="__ _____ ___ __ ___"
                             :disabled="true" @input="rule.description = $event.target.value" :clear-button="isEditable" />
            </f7-list>
          </f7-col>
        </f7-block>

        <f7-block v-else class="block-narrow">
          <f7-col>
            <f7-list inline-labels no-hairlines-md>
              <f7-list-input label="Unique ID" type="text" placeholder="Required" :value="rule.uid" required validate
                             :disabled="!createMode" :info="(createMode) ? 'Note: cannot be changed after the creation' : ''"
                             pattern="[A-Za-z0-9_\-]+" error-message="Required. A-Z,a-z,0-9,_,- only"
                             @input="rule.uid = $event.target.value" :clear-button="createMode" />
              <f7-list-input label="Name" type="text" placeholder="Required" :value="rule.name" required validate
                             :disabled="!isEditable" @input="rule.name = $event.target.value" :clear-button="isEditable" />
              <f7-list-input label="Description" type="text" :value="rule.description"
                             :disabled="!isEditable" @input="rule.description = $event.target.value" :clear-button="isEditable" />
            </f7-list>
          </f7-col>

          <f7-block-footer v-if="!isEditable" class="no-margin padding-left">
            <f7-icon f7="lock_fill" size="12" color="gray" />&nbsp;Note: this rule is not editable because it has been provisioned from a file.
          </f7-block-footer>
          <!-- <f7-col v-if="isEditable" class="text-align-right justify-content-flex-end">
          </f7-col> -->
          <f7-col v-if="createMode && templates.length > 0" class="new-rule-from-template">
            <f7-block-title medium class="margin-bottom">
              Create from Template
            </f7-block-title>
            <f7-list media-list>
              <f7-list-item title="No template" footer="Create a new rule from scratch"
                            radio :checked="!hasTemplate" radio-icon="start"
                            :value="''"
                            @change="selectTemplate(null)" />
            </f7-list>
            <f7-block-footer class="margin-left">
              or choose a rule template:
            </f7-block-footer>
            <f7-list media-list>
              <f7-list-item :key="template.uid" v-for="template in templates"
                            :title="template.label" :footer="template.description"
                            :value="template.uid"
                            radio :checked="hasTemplate && currentTemplate.uid === template.uid" radio-icon="start"
                            @change="selectTemplate(template.uid)" />
            </f7-list>
            <f7-block-title v-if="hasTemplate" medium class="margin-vertical padding-top">
              Template Configuration
            </f7-block-title>
            <f7-link v-if="templateTopicLink" target="_blank" class="external margin-left" color="blue" :href="templateTopicLink">
              Template Community Marketplace Topic
            </f7-link>
            <config-sheet v-if="hasTemplate"
                          :parameter-groups="[]" :parameters="currentTemplate.configDescriptions"
                          :configuration="rule.configuration" />
          </f7-col>
          <f7-col v-if="!hasTemplate" class="rule-modules">
            <div v-if="isEditable" class="no-padding float-right">
              <f7-button @click="toggleModuleControls" small outline :fill="showModuleControls" sortable-toggle=".sortable" style="margin-top: -3px; margin-right: 5px"
                         color="gray" icon-size="12" icon-ios="material:wrap_text" icon-md="material:wrap_text" icon-aurora="material:wrap_text">
                &nbsp;Reorder
              </f7-button>
            </div>
            <div v-for="section in ['triggers', 'actions', 'conditions']" :key="section">
              <f7-block-title medium style="margin-bottom: var(--f7-list-margin-vertical)" v-if="isEditable || rule[section].length > 0">
                {{ sectionLabels[section][0] }}
              </f7-block-title>
              <f7-list sortable swipeout media-list @sortable:sort="(ev) => reorderModule(ev, section)">
                <f7-list-item media
                              :title="mod.label || suggestedModuleTitle(mod, null, section)"
                              :footer="mod.description || suggestedModuleDescription(mod, null, section)"
                              v-for="mod in rule[section]" :key="mod.id"
                              :link="isEditable && !showModuleControls"
                              @click.native="(ev) => editModule(ev, section, mod)" swipeout>
                  <f7-link slot="media" v-if="isEditable" icon-color="red" icon-aurora="f7:minus_circle_filled" icon-ios="f7:minus_circle_filled" icon-md="material:remove_circle_outline" @click="showSwipeout" />
                  <f7-link slot="after" v-if="mod.type && mod.type.indexOf('script') === 0" icon-f7="pencil_ellipsis_rectangle" color="gray" @click.native="(ev) => editModule(ev, section, mod, true)" :tooltip="'Edit module'" />
                  <f7-link slot="after" v-if="mod.type === 'timer.GenericCronTrigger' && isEditable" icon-f7="pencil_ellipsis_rectangle" color="gray" @click.native="(ev) => editModule(ev, section, mod, true)" tooltip="Edit module" />
                  <f7-swipeout-actions right v-if="isEditable">
                    <f7-swipeout-button @click="(ev) => deleteModule(ev, section, mod)" style="background-color: var(--f7-swipeout-delete-button-bg-color)">
                      Delete
                    </f7-swipeout-button>
                  </f7-swipeout-actions>
                </f7-list-item>
              </f7-list>
              <f7-list v-if="isEditable">
                <f7-list-item link no-chevron media-item :color="($theme.dark) ? 'black' : 'white'" :subtitle="sectionLabels[section][1]" @click="addModule(section)">
                  <f7-icon slot="media" color="green" aurora="f7:plus_circle_fill" ios="f7:plus_circle_fill" md="material:control_point" />
                </f7-list-item>
                <!-- <f7-list-button :color="(showModuleControls) ? 'gray' : 'blue'" :title="sectionLabels[section][1]"></f7-list-button> -->
              </f7-list>
            </div>
          </f7-col>
          <f7-col v-if="(isEditable || rule.tags.length > 0) && (!createMode || !hasTemplate)">
            <f7-block-title>Tags</f7-block-title>
            <semantics-picker v-if="isEditable" :item="rule" />
            <tag-input :item="rule" :disabled="!isEditable" />
          </f7-col>
          <f7-col v-if="isEditable && !createMode">
            <f7-list>
              <f7-list-button color="red" @click="deleteRule">
                Remove Rule
              </f7-list-button>
            </f7-list>
          </f7-col>
        </f7-block>
      </f7-tab>
      <f7-tab id="code" @tab:show="() => { this.currentTab = 'code'; toYaml() }" :tab-active="currentTab === 'code'">
        <editor v-if="currentTab === 'code'" class="rule-code-editor" mode="application/vnd.openhab.rule+yaml" :value="ruleYaml" @input="onEditorInput" />
        <!-- <pre class="yaml-message padding-horizontal" :class="[yamlError === 'OK' ? 'text-color-green' : 'text-color-red']">{{yamlError}}</pre> -->
      </f7-tab>
    </f7-tabs>
  </f7-page>
</template>

<style lang="stylus">
.enable-toggle
  vertical-align inherit
.moduleconfig-popup
  .page-content
    overflow-x hidden !important
  .config-sheet, .parameter-group
    margin-top 0 !important
.rule-modules
  .swipeout-opened
    .sortable-handler
      display none
  .item-media .icon
    color var(--f7-theme-color)
  .media-list
    margin-bottom 0
  .list
    margin-top 0
.rule-code-editor.vue-codemirror
  display block
  top calc(var(--f7-navbar-height) + var(--f7-tabbar-height))
  height calc(100% - 2*var(--f7-navbar-height))
  width 100%
.yaml-message
  display block
  position absolute
  top 80%
  white-space pre-wrap

</style>

<script>
import YAML from 'yaml'
import cloneDeep from 'lodash/cloneDeep'
import fastDeepEqual from 'fast-deep-equal/es6'

import SemanticsPicker from '@/components/tags/semantics-picker.vue'
import TagInput from '@/components/tags/tag-input.vue'

import RuleModulePopup from './rule-module-popup.vue'

import ModuleDescriptionSuggestions from './module-description-suggestions'
import RuleStatus from '@/components/rule/rule-status-mixin'
import DirtyMixin from '../dirty-mixin'

import ConfigSheet from '@/components/config/config-sheet.vue'

export default {
  mixins: [ModuleDescriptionSuggestions, RuleStatus, DirtyMixin],
  components: {
    ConfigSheet,
    SemanticsPicker,
    TagInput,
    'editor': () => import(/* webpackChunkName: "script-editor" */ '@/components/config/controls/script-editor.vue')
  },
  props: ['ruleId', 'createMode', 'schedule'],
  data () {
    return {
      ready: false,
      loading: false,
      rule: {},
      ruleYaml: '',
      moduleTypes: {
        actions: [],
        conditions: [],
        triggers: []
      },
      showModuleControls: false,
      currentSection: 'actions',
      currentTab: 'design',
      currentModuleType: null,
      currentModule: null,
      currentModuleConfig: {},
      sectionLabels: {
        triggers: ['When', 'Add Trigger'],
        actions: ['Then', 'Add Action'],
        conditions: ['But only if', 'Add Condition']
      },
      eventSource: null,
      keyHandler: null,

      codeEditorOpened: false,
      cronPopupOpened: false,
      scriptCode: '',
      cronExpression: null,
      templates: null,
      currentTemplate: null
    }
  },
  watch: {
    rule: {
      handler: function (newRule, oldRule) {
        if (!this.loading) { // ignore initial rule assignment
          // create rule object clone in order to be able to delete status part
          // which can change from eventsource but doesn't mean a rule modification
          let ruleClone = cloneDeep(this.rule)
          delete ruleClone.status
          delete this.savedRule.status

          this.dirty = !fastDeepEqual(ruleClone, this.savedRule)
        }
      },
      deep: true
    }
  },
  methods: {
    onPageAfterIn () {
      if (window) {
        window.addEventListener('keydown', this.keyDown)
      }
      this.load()
    },
    onPageAfterOut () {
      this.stopEventSource()
      if (window) {
        window.removeEventListener('keydown', this.keyDown)
      }
    },
    onEditorInput (value) {
      this.ruleYaml = value
      this.dirty = true
    },
    load () {
      if (this.loading) return
      this.loading = true

      const loadModules1 = this.$oh.api.get('/rest/module-types?type=action')
      const loadModules2 = this.$oh.api.get('/rest/module-types?type=condition')
      const loadModules3 = this.$oh.api.get('/rest/module-types?type=trigger')

      const loadingFinished = () => {
        this.$nextTick(() => {
          this.savedRule = cloneDeep(this.rule)
          this.ready = true
          this.loading = false
        })
      }

      Promise.all([loadModules1, loadModules2, loadModules3]).then((data) => {
        this.moduleTypes.actions = data[0]
        this.moduleTypes.conditions = data[1]
        this.moduleTypes.triggers = data[2]
        if (this.createMode) {
          this.$set(this, 'rule', {
            uid: this.$f7.utils.id(),
            name: '',
            triggers: [],
            actions: [],
            conditions: [],
            tags: (this.schedule) ? ['Schedule'] : [],
            configuration: {},
            templateUID: null,
            visibility: 'VISIBLE',
            status: {
              status: 'NEW'
            }
          })
          this.$oh.api.get('/rest/templates').then((data2) => {
            this.$set(this, 'templates', data2)
            loadingFinished()
          })
          // no need for an event source, the rule doesn't exist yet
        } else {
          this.$oh.api.get('/rest/rules/' + this.ruleId).then((data2) => {
            this.$set(this, 'rule', data2)
            if (!this.eventSource) this.startEventSource()
            loadingFinished()
          })
        }
      })
    },
    save (noToast) {
      if (!this.isEditable) return Promise.reject()
      if (this.currentTab === 'code') {
        if (!this.fromYaml()) {
          return Promise.reject()
        }
      }
      if (!this.rule.uid) {
        this.$f7.dialog.alert('Please give an ID to the rule')
        return Promise.reject()
      }
      if (!this.rule.name) {
        this.$f7.dialog.alert('Please give a name to the rule')
        return Promise.reject()
      }
      const promise = (this.createMode)
        ? this.$oh.api.postPlain('/rest/rules', JSON.stringify(this.rule), 'text/plain', 'application/json')
        : this.$oh.api.put('/rest/rules/' + this.rule.uid, this.rule)
      return promise.then((data) => {
        this.dirty = false
        if (this.createMode) {
          this.$f7.toast.create({
            text: 'Rule created',
            destroyOnClose: true,
            closeTimeout: 2000
          }).open()
          this.$f7router.navigate(this.$f7route.url.replace('/add', '/' + this.rule.uid).replace('/schedule/', '/rules/'), { reloadCurrent: true })
          this.load()
        } else {
          if (!noToast) {
            this.$f7.toast.create({
              text: 'Rule updated',
              destroyOnClose: true,
              closeTimeout: 2000
            }).open()
          }
          this.savedRule = cloneDeep(this.rule)
        }
        // if (!stay) this.$f7router.back()
      }).catch((err) => {
        this.$f7.toast.create({
          text: 'Error while saving rule: ' + err,
          destroyOnClose: true,
          closeTimeout: 2000
        }).open()
      })
    },
    toggleDisabled () {
      if (this.createMode) return
      const enable = (this.rule.status.statusDetail === 'DISABLED')
      this.$oh.api.postPlain('/rest/rules/' + this.rule.uid + '/enable', enable.toString()).then((data) => {
        this.$f7.toast.create({
          text: (enable) ? 'Rule enabled' : 'Rule disabled',
          destroyOnClose: true,
          closeTimeout: 2000
        }).open()
      }).catch((err) => {
        this.$f7.toast.create({
          text: 'Error while disabling or enabling: ' + err,
          destroyOnClose: true,
          closeTimeout: 2000
        }).open()
      })
    },
    runNow () {
      if (this.createMode) return
      if (this.rule.status.status === 'RUNNING' || this.rule.status.status === 'UNINITIALIZED') {
        return this.$f7.toast.create({
          text: `Rule cannot be run ${(this.rule.status.status === 'RUNNING') ? 'while already running, please wait' : 'if it is uninitialized'}!`,
          destroyOnClose: true,
          closeTimeout: 2000
        }).open()
      }
      this.$f7.toast.create({
        text: 'Running rule',
        destroyOnClose: true,
        closeTimeout: 2000
      }).open()

      const savePromise = (this.isEditable && this.dirty) ? this.save(true) : Promise.resolve()

      savePromise.then(() => {
        this.$oh.api.postPlain('/rest/rules/' + this.rule.uid + '/runnow', '').catch((err) => {
          this.$f7.toast.create({
            text: 'Error while running rule: ' + err,
            destroyOnClose: true,
            closeTimeout: 2000
          }).open()
        })
      })
    },
    deleteRule () {
      this.$f7.dialog.confirm(
        `Are you sure you want to delete ${this.rule.name}?`,
        'Delete Rule',
        () => {
          this.$oh.api.delete('/rest/rules/' + this.rule.uid).then(() => {
            this.$f7router.back('/settings/rules/', { force: true })
          })
        }
      )
    },
    startEventSource () {
      this.eventSource = this.$oh.sse.connect('/rest/events?topics=openhab/rules/' + this.ruleId + '/*', null, (event) => {
        const topicParts = event.topic.split('/')
        switch (topicParts[3]) {
          case 'state':
            this.$set(this.rule, 'status', JSON.parse(event.payload))
            break
        }
      })
    },
    stopEventSource () {
      this.$oh.sse.close(this.eventSource)
      this.eventSource = null
    },
    selectTemplate (uid) {
      this.$set(this.rule, 'configuration', {})
      this.$set(this.rule, 'triggers', [])
      this.$set(this.rule, 'conditions', [])
      this.$set(this.rule, 'actions', [])
      if (!uid) {
        this.$set(this, 'currentTemplate', null)
        return
      }
      this.$set(this, 'currentTemplate', this.templates.find((t) => t.uid === uid))
      this.rule.templateUID = uid
    },
    keyDown (ev) {
      if ((ev.ctrlKey || ev.metaKey) && !(ev.altKey || ev.shiftKey)) {
        if (this.currentModule) return
        switch (ev.keyCode) {
          case 68:
            this.toggleDisabled()
            ev.stopPropagation()
            ev.preventDefault()
            break
          case 82:
            this.runNow()
            ev.stopPropagation()
            ev.preventDefault()
            break
          case 83:
            this.save()
            ev.stopPropagation()
            ev.preventDefault()
            break
        }
      }
    },
    toggleModuleControls () {
      this.showModuleControls = !this.showModuleControls
    },
    showSwipeout (ev) {
      let swipeoutElement = ev.target
      ev.cancelBubble = true
      while (!swipeoutElement.classList.contains('swipeout')) {
        swipeoutElement = swipeoutElement.parentElement
      }

      if (swipeoutElement) {
        this.$f7.swipeout.open(swipeoutElement)
      }
    },
    editModule (ev, section, mod, force) {
      if (this.showModuleControls) return
      if (!this.isEditable) return
      let swipeoutElement = ev.target
      ev.cancelBubble = true
      while (!swipeoutElement.classList.contains('swipeout')) {
        swipeoutElement = swipeoutElement.parentElement
      }
      if (swipeoutElement && swipeoutElement.classList.contains('swipeout-opened')) return
      this.currentSection = section
      this.currentModule = Object.assign({}, mod)
      this.currentModuleType = this.moduleTypes[section].find((m) => m.uid === mod.type)

      if (mod.type && mod.type.indexOf('script') === 0 && !force) {
        this.editScriptDirect(ev, mod)
        return
      }
      if (mod.type && mod.type === 'timer.GenericCronTrigger' && !force) {
        this.buildCronExpression(ev, mod)
        return
      }

      const popup = {
        component: RuleModulePopup
      }
      this.$f7router.navigate({
        url: 'module-config',
        route: {
          path: 'module-config',
          popup
        }
      }, {
        props: {
          currentSection: this.currentSection,
          ruleModule: this.currentModule,
          ruleModuleType: this.currentModuleType,
          moduleTypes: this.moduleTypes
        }
      })

      this.$f7.once('ruleModuleConfigUpdate', this.saveModule)
      this.$f7.once('ruleModuleConfigClosed', () => {
        this.$f7.off('ruleModuleConfigUpdate', this.saveModule)
        this.moduleConfigClosed()
      })
    },
    deleteModule (ev, section, mod) {
      let swipeoutElement = ev.target
      if (!this.isEditable) return
      ev.cancelBubble = true
      while (!swipeoutElement.classList.contains('swipeout')) {
        swipeoutElement = swipeoutElement.parentElement
      }
      this.$f7.swipeout.delete(swipeoutElement, () => {
        const idx = this.rule[section].findIndex((m) => m.id === mod.id)
        this.rule[section].splice(idx, 1)
      })
    },
    addModule (section) {
      if (this.showModuleControls) return
      if (!this.isEditable) return
      let moduleId = 1
      for (; ['triggers', 'actions', 'conditions'].some((s) => this.rule[s].some((m) => m.id === moduleId.toString())); moduleId++);
      console.debug('new moduleId=' + moduleId)
      const newModule = {
        id: moduleId.toString(),
        configuration: {},
        type: '',
        new: true
      }

      // this.rule[section].push(newModule)
      this.currentSection = section
      this.currentModule = newModule
      this.currentModuleType = null

      const popup = {
        component: RuleModulePopup
      }
      this.$f7router.navigate({
        url: 'module-config',
        route: {
          path: 'module-config',
          popup
        }
      }, {
        props: {
          currentSection: this.currentSection,
          ruleModule: this.currentModule,
          ruleModuleType: this.currentModuleType,
          moduleTypes: this.moduleTypes
        }
      })

      this.$f7.once('ruleModuleConfigUpdate', this.saveModule)
      this.$f7.once('editNewScript', this.saveAndEditNewScript)
      this.$f7.once('ruleModuleConfigClosed', () => {
        this.$f7.off('ruleModuleConfigUpdate', this.saveModule)
        this.$f7.off('editNewScript', this.saveAndEditNewScript)
        this.moduleConfigClosed()
      })
    },
    reorderModule (ev, section) {
      const newSection = [...this.rule[section]]
      newSection.splice(ev.to, 0, newSection.splice(ev.from, 1)[0])
      this.$set(this.rule, section, newSection)
    },
    saveModule (updatedModule) {
      if (!updatedModule.type) return
      if (!updatedModule.label) delete updatedModule.label
      if (!updatedModule.description) delete updatedModule.description
      if (updatedModule.new) {
        delete updatedModule.new
        this.rule[this.currentSection].push(updatedModule)
      } else {
        const idx = this.rule[this.currentSection].findIndex((m) => m.id === updatedModule.id)
        this.$set(this.rule[this.currentSection], idx, updatedModule)
      }
    },
    saveAndEditNewScript (updatedModule) {
      this.saveModule(updatedModule)
      this.save().then(() => {
        this.$f7router.navigate('/settings/rules/' + this.rule.uid + '/script/' + updatedModule.id, { transition: this.$theme.aurora ? 'f7-cover-v' : '' })
      })
    },
    moduleConfigClosed () {
      this.currentModule = null
      this.currentModuleType = null
    },
    editScriptDirect (ev, mod) {
      ev.cancelBubble = true
      this.currentModule = mod
      this.currentModuleType = mod.type
      this.scriptCode = mod.configuration.script

      const updatePromise = (this.rule.editable || this.createMode) ? this.save() : Promise.resolve()
      updatePromise.then(() => {
        this.$f7router.navigate('/settings/rules/' + this.rule.uid + '/script/' + mod.id, { transition: this.$theme.aurora ? 'f7-cover-v' : '' })
      })
    },
    buildCronExpression (ev, mod) {
      ev.cancelBubble = true
      this.currentModule = mod
      this.currentModuleType = mod.type
      this.cronExpression = mod.configuration.cronExpression
      import(/* webpackChunkName: "cronexpression-editor" */ '@/components/config/controls/cronexpression-editor.vue').then((c) => {
        const popup = {
          component: c.default
        }
        this.$f7router.navigate({
          url: 'cron-edit',
          route: {
            path: 'cron-edit',
            popup
          }
        }, {
          props: {
            value: this.cronExpression
          }
        })

        this.$f7.once('cronEditorUpdate', this.updateCronExpression)
        this.$f7.once('cronEditorClosed', () => {
          this.$f7.off('cronEditorUpdate', this.updateCronExpression)
          this.$nextTick(() => {
            this.currentModule = null
            this.currentModuleType = null
          })
        })
      })
    },
    updateScript (value) {
      if (this.isEditable) this.currentModule.configuration.script = value
    },
    updateCronExpression (value) {
      this.currentModule.configuration.cronExpression = value
    },
    toYaml () {
      this.ruleYaml = YAML.stringify({
        configuration: this.rule.configuration,
        triggers: this.rule.triggers,
        conditions: this.rule.conditions,
        actions: this.rule.actions
      })
    },
    fromYaml () {
      if (!this.isEditable) return
      try {
        const updatedRule = YAML.parse(this.ruleYaml)
        this.$set(this.rule, 'configuration', updatedRule.configuration)
        this.$set(this.rule, 'triggers', updatedRule.triggers)
        this.$set(this.rule, 'conditions', updatedRule.conditions)
        this.$set(this.rule, 'actions', updatedRule.actions)
        return true
      } catch (e) {
        this.$f7.dialog.alert(e).open()
        return false
      }
    }
  },
  computed: {
    isEditable () {
      return this.rule && this.rule.editable !== false
    },
    hasTemplate () {
      return this.rule && this.currentTemplate !== null
    },
    templateTopicLink () {
      if (!this.currentTemplate) return null
      if (!this.currentTemplate.tags) return null
      const marketplaceTag = this.currentTemplate.tags.find((t) => t.indexOf('marketplace:') === 0)
      if (marketplaceTag) return 'https://community.openhab.org/t/' + marketplaceTag.replace('marketplace:', '')
      return null
    },
    yamlError () {
      if (this.currentTab !== 'code') return null
      try {
        YAML.parse(this.ruleYaml, { prettyErrors: true })
        return 'OK'
      } catch (e) {
        return e
      }
    }
  }
}
</script>
