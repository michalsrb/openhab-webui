<template>
  <f7-page no-toolbar no-navbar no-swipeback no-swipe-panel login-screen class="setup-wizard" @page:init="pageBeforeIn" @page:beforeout="pageBeforeOut">
    <f7-tabs animated>
      <f7-tab id="intro" ref="intro" tab-active>
        <f7-login-screen-title>
          <img class="intro-logo" src="@/images/openhab-logo.svg" type="image/svg+xml">
        </f7-login-screen-title>
        <f7-list form style="margin-top: 4rem" v-if="i18nReady">
          <f7-list-item
            :title="$t('setupwizard.language')"
            smart-select
            :smart-select-params="{openIn: 'popup', searchbar: true, closeOnSelect: true}">
            <select name="language" @change="(evt) => language = evt.target.value">
              <option value="" :selected="!language" />
              <option
                v-for="option in availableLanguages"
                :key="option.value"
                :value="option.value"
                :selected="language === option.value">
                {{ option.label }}
              </option>
            </select>
          </f7-list-item>
          <f7-list-item
            :title="$t('setupwizard.region')"
            smart-select
            :smart-select-params="{openIn: 'popup', searchbar: true, closeOnSelect: true}">
            <select name="region" @change="(evt) => region = evt.target.value">
              <option value="" :selected="!region" />
              <option
                v-for="option in availableRegions"
                :key="option.value"
                :value="option.value"
                :selected="region === option.value">
                {{ option.label }}
              </option>
            </select>
          </f7-list-item>
          <f7-list-item
            :title="$t('setupwizard.timezone')"
            smart-select
            :smart-select-params="{openIn: 'popup', searchbar: true, virtualList: true, closeOnSelect: true, virtualListHeight: ($theme.aurora) ? 32 : undefined }">
            <select name="timezone" @change="(evt) => timezone = evt.target.value">
              <option value="" />
              <option
                v-for="option in availableTimezones"
                :key="option.value"
                :value="option.value"
                :selected="timezone === option.value">
                {{ option.label }}
              </option>
            </select>
          </f7-list-item>
        </f7-list>
        <f7-block class="display-flex flex-direction-column padding">
          <div>
            <f7-button large fill color="blue" :text="$t('setupwizard.beginSetup')" @click="beginSetup" />
            <f7-button large color="blue" :text="$t('setupwizard.skipSetup')" class="margin-top" @click="skipSetup" />
          </div>
        </f7-block>
      </f7-tab>

      <f7-tab id="location" ref="location">
        <f7-block>
          <f7-link
            icon-ios="f7:arrow_left"
            icon-aurora="f7:arrow_left"
            icon-md="material:arrow_back"
            tab-link="#intro"
            color="blue"
            tab-link-active />
          <f7-login-screen-title>
            <div class="padding">
              <f7-icon size="48" color="blue" f7="map_pin_ellipse" />
            </div>
            {{ $t('setupwizard.location.title') }}
          </f7-login-screen-title>
        </f7-block>
        <f7-block strong>
          {{ $t('setupwizard.location.header1') }}<br>{{ $t('setupwizard.location.header2') }}
        </f7-block>
        <f7-list>
          <parameter-location :value="location" :config-description="{ label: $t('setupwizard.location.parameterLabel'), name: 'Location' }" @input="(value) => location = value" :placeholder="$t('setupwizard.location.placeholder')" />
        </f7-list>
        <f7-block class="padding">
          <f7-row>
            <f7-col width="100">
              <f7-button large icon-f7="location_fill" icon-size="24" @click="getCurrentPosition()" :text="$t('setupwizard.location.retrieveFromDevice')" />
            </f7-col>
          </f7-row>
          <f7-block-footer>
            <small v-t="'setupwizard.location.footer'" />
          </f7-block-footer>
        </f7-block>
        <f7-block class="display-flex flex-direction-column padding">
          <div>
            <f7-button v-if="location" large fill color="blue" :text="$t('setupwizard.location.setLocation')" @click="setLocation" />
            <f7-button large color="blue" :text="$t('setupwizard.location.configureLater')" class="margin-top" @click="skipLocation" />
          </div>
        </f7-block>
      </f7-tab>

      <f7-tab id="addons" ref="addons">
        <f7-block>
          <f7-link
            icon-ios="f7:arrow_left"
            icon-aurora="f7:arrow_left"
            icon-md="material:arrow_back"
            tab-link="#location"
            color="blue"
            tab-link-active />
          <f7-login-screen-title>
            <div class="padding">
              <f7-icon size="48" color="blue" f7="bag_badge_plus" />
            </div>
            {{ $t('setupwizard.addons.title') }}
          </f7-login-screen-title>
        </f7-block>
        <f7-block strong>
          {{ $t('setupwizard.addons.header1') }}<br>{{ $t('setupwizard.addons.header2') }}<br><br>
          <a class="text-color-blue external" target="_blank" href="https://www.openhab.org/addons/" v-t="'setupwizard.addons.browseAddonsOnWebsite'" />
        </f7-block>
        <f7-block class="padding">
          <f7-row>
            <f7-col width="100">
              <f7-button ref="selectAddons" large icon-f7="cart_fill" icon-size="24" @click="selectAddons" :text="$t('setupwizard.addons.selectAddons')" />
            </f7-col>
          </f7-row>
          <f7-list class="search-list searchbar-found" ref="selectAddons" media-list v-show="!installingAddons">
            <f7-list-item media-item v-for="addon in selectedAddons" :key="addon.uid"
                          :header="addon.uid" :title="addon.label" :footer="addon.version">
              <f7-link slot="after" v-if="addon.link" icon-f7="doc_text_search" :external="true" color="gray" target="_blank" :href="addon.link" />
            </f7-list-item>
          </f7-list>
          <f7-block-footer class="margin-bottom">
            <small v-t="'setupwizard.addons.footer'" />
          </f7-block-footer>
          <div>
            <f7-button v-if="selectedAddons.length > 0" large fill color="blue" :text="$tc('setupwizard.addons.installAddons', selectedAddons.length)" @click="installAddons" />
            <f7-button large color="blue" :text="$t('setupwizard.addons.installLater')" class="margin-top" @click="skipAddons" />
          </div>
        </f7-block>
      </f7-tab>

      <f7-tab id="wait" ref="wait">
        <f7-block>
          <f7-link
            icon-ios="f7:arrow_left"
            icon-aurora="f7:arrow_left"
            icon-md="material:arrow_back"
            tab-link="#intro"
            color="blue"
            tab-link-active
            style="visibility: hidden" />
          <f7-login-screen-title class="text-color-gray">
            {{ $t('setupwizard.addons.pleaseWait') }}
          </f7-login-screen-title>
          <div class="display-flex justify-content-center flex-direction-column text-align-center text-color-gray" style="margin-top: 4rem">
            <div class="display-flex justify-content-center margin-bottom">
              <f7-preloader size="24" />
            </div>
            <div v-t="'setupwizard.addons.waitMessage'" />
          </div>
        </f7-block>
      </f7-tab>

      <f7-tab id="finish" ref="finish">
        <f7-block style="margin-top: 8rem">
          <!-- no going back on this last screen!
                  <f7-link
                  icon-ios="f7:arrow_left"
                  icon-aurora="f7:arrow_left"
                  icon-md="material:arrow_back"
                  tab-link="#package"
                  color="blue"
                  tab-link-active
                ></f7-link>-->
          <f7-login-screen-title>{{ $t('setupwizard.welcome.title') }}</f7-login-screen-title>
        </f7-block>

        <f7-block class="display-flex flex-direction-column padding" style="margin-top: 4rem">
          <div>
            <f7-button large color="blue" :text="$t('setupwizard.welcome.getStarted')" @click="finish" />
          </div>
        </f7-block>
      </f7-tab>
    </f7-tabs>
  </f7-page>
</template>

<style lang="stylus">
.setup-wizard
  .intro-logo
    margin-top 3rem
    margin-bottom 2rem
    width 240px
  .page-content
    margin-top inherit
  .tabs-animated-wrap
    overflow-y auto !important

.view-master-detail
  .setup-wizard
    .intro-logo
      visibility hidden
      // margin-top 25%
    .tab
      padding-top 10%
</style>

<script>
import i18n from '@/components/i18n-mixin'
import { loadLocaleMessages } from '@/js/i18n'
export default {
  mixins: [i18n],
  components: {
    'parameter-location': () => import('@/components/config/controls/parameter-location.vue')
  },
  data () {
    return {
      i18nReady: false,
      availableLanguages: null,
      availableRegions: null,
      availableTimezones: null,
      language: null,
      region: null,
      timezone: null,
      location: null,
      autocompleteAddons: null,
      addons: [],
      selectedAddons: [],
      installingAddons: false
    }
  },
  i18n: {
    messages: loadLocaleMessages(require.context('@/assets/i18n/setup-wizard'))
  },
  computed: {
    locale () {
      if (!this.language) return null
      if (!this.region) return this.language
      return this.language + '-' + this.region.toLowerCase()
    }
  },
  watch: {
    locale (val) {
      this.$store.commit('setLocale', this.locale)
      this.updateLocale()
      this.$i18n.locale = val
    }
  },
  methods: {
    beginSetup () {
      this.$oh.api.put('/rest/services/org.openhab.i18n/config', {
        language: this.language,
        region: this.region,
        timezone: this.timezone
      }).then(() => {
        this.$f7.emit('localeChange')
        if (this.autocompleteAddons) {
          this.autocompleteAddons.params.pageTitle = this.$t('setupwizard.addons.selectAddons')
          this.autocompleteAddons.params.searchbarPlaceholder = this.$t('setupwizard.addons.selectAddons.placeholder')
          this.autocompleteAddons.params.searchbarDisableText = this.$t('dialogs.cancel')
          this.autocompleteAddons.params.popupCloseLinkText = this.$t('dialogs.close')
          this.autocompleteAddons.params.pageBackLinkText = this.$t('dialogs.back')
          this.autocompleteAddons.params.notFoundText = this.$t('dialogs.search.nothingFound')
        }
        this.$refs.location.show()
      })
    },
    getCurrentPosition () {
      if ('geolocation' in navigator) {
        navigator.geolocation.getCurrentPosition((position) => {
          this.location = position.coords.latitude + ',' + position.coords.longitude
        }, (error) => {
          this.$f7.dialog.alert(
            error.message,
            this.$t('setupwizard.location.retrieveFromDevice.error')
          )
        })
      } else {
        this.$f7.dialog.alert(this.$t('setupwizard.location.retrieveFromDevice.notAvailable.message'), this.$t('setupwizard.location.retrieveFromDevice.notAvailable.title'))
      }
    },
    skipSetup () {
      const self = this
      this.$f7.dialog.confirm(
        this.$t('setupwizard.skipSetup.confirm.message'), this.$t('setupwizard.skipSetup.confirm.title'),
        () => {
          self.$f7.panel.get('left').enableVisibleBreakpoint()
          this.$nextTick(() => {
            self.$f7.views.main.router.navigate('/', { transition: 'f7-circle', clearPreviousHistory: true })
          })
        })
    },
    setLocation () {
      this.$oh.api.put('/rest/services/org.openhab.i18n/config', {
        location: this.location
      }).then(() => {
        this.$refs.addons.show()
      })
    },
    skipLocation () {
      this.$refs.addons.show()
    },
    selectAddons () {
      if (this.autocompleteAddons) this.autocompleteAddons.open()
    },
    installAddons () {
      const self = this
      const checkInterval = 2 // check the add-ons statuses every 2 seconds

      this.installingAddons = true
      this.$refs.wait.show(false)

      const addonsCount = this.selectedAddons.length
      let progress = 0

      const progressDialog = this.$f7.dialog.progress(this.$t('setupwizard.addons.installing'), progress)

      const checkAddonStatus = function (addon) {
        return new Promise((resolve, reject) => {
          self.$oh.api.get('/rest/addons/' + addon.uid).then((data) => {
            if (data.installed) {
              console.log(`Add-on ${addon.uid} installed!`)
              resolve(data)
            } else {
              console.log(`Add-on ${addon.uid} still not installed. Trying again in ${checkInterval} secs...`)
              reject(data)
            }
          }).catch((err) => {
            console.log(`Error while querying API to check addon: ${addon.uid}: ${err}'. Trying again in ${checkInterval} secs...`)
            reject(err)
          })
        })
      }

      const installNextAddon = function () {
        // no more add-ons to install => go to next screen
        if (!self.selectedAddons.length) {
          progressDialog.close()
          progressDialog.destroy()
          self.$refs.finish.show()
          return
        }

        // install next add-on
        progressDialog.setText(self.$t('setupwizard.addons.progress', { current: addonsCount - self.selectedAddons.length + 1, total: addonsCount }))
        progressDialog.setProgress(((addonsCount - self.selectedAddons.length + 1) / addonsCount) * 100)
        const addon = self.selectedAddons.shift()
        console.log('Installing add-on: ' + addon.uid)
        progressDialog.setTitle(self.$t('setupwizard.addons.installingAddon', { addon: addon.label }))

        self.$oh.api.post('/rest/addons/' + addon.uid + '/install', {}, 'text').then((data) => {
          const checkTimer = setInterval(() => {
            checkAddonStatus(addon).then((addon) => {
              clearInterval(checkTimer)
              installNextAddon()
            }).catch(() => {
              // just keep going... TODO: implement failure mechanism after a number of failed checks?
            })
          }, checkInterval * 1000)
        })
      }

      progressDialog.open()
      installNextAddon()
    },
    skipAddons () {
      this.$refs.finish.show()
    },
    finish () {
      this.$f7.panel.get('left').enableVisibleBreakpoint()
      this.$nextTick(() => {
        this.$f7.views.main.router.navigate('/', { transition: 'f7-circle', clearPreviousHistory: true })
      })
    },
    pageBeforeIn () {
      this.$f7.panel.get('left').disableVisibleBreakpoint()
    },
    pageBeforeOut (e, page) {
      this.$f7.panel.get('left').enableVisibleBreakpoint()
      // create the overview page to prevent this setup wizard from being launched again
      this.$oh.api.post('/rest/ui/components/ui:page', {
        uid: 'overview',
        component: 'oh-layout-page',
        config: {
          label: 'Overview'
        },
        slots: {
          default: [],
          masonry: null
        }
      }).then((data) => {
        // this will force the pages to be refreshed
        this.$f7.emit('sidebarRefresh', null)
      })
    }
  },
  mounted () {
    this.$oh.api.get('/rest/config-descriptions/system:i18n').then((data) => {
      this.availableLanguages = data.parameters.find(p => p.name === 'language').options
      this.availableRegions = data.parameters.find(p => p.name === 'region').options
      this.availableTimezones = data.parameters.find(p => p.name === 'timezone').options
      if (Intl && Intl.DateTimeFormat().resolvedOptions()) {
        const intlOptions = Intl.DateTimeFormat().resolvedOptions()
        if (intlOptions.locale) {
          this.language = intlOptions.locale.split('-')[0]
          if (intlOptions.locale.split('-')[1]) this.region = intlOptions.locale.split('-')[1]
        }
        if (intlOptions.timeZone) {
          if (this.availableTimezones.find((tz) => tz.value === intlOptions.timeZone)) this.timezone = intlOptions.timeZone
        }
        this.i18nReady = true
      }
    })
    this.$oh.api.get('/rest/addons').then((data) => {
      this.addons = data.sort((a, b) => a.label.toUpperCase().localeCompare(b.label.toUpperCase()))
      const self = this
      this.autocompleteAddons = this.$f7.autocomplete.create({
        openIn: 'popup',
        pageTitle: this.$t('setupwizard.addons.selectAddons'),
        searchbarPlaceholder: this.$t('setupwizard.addons.selectAddons.placeholder'),
        openerEl: this.$refs.selectAddons,
        multiple: true,
        requestSourceOnOpen: true,
        source: (query, render) => {
          if (query.length === 0) {
            render(self.addons.filter((a) => !a.installed).map((a) => a.label))
          } else {
            render(self.addons.filter((a) => !a.installed && (a.label.toLowerCase().indexOf(query.toLowerCase()) >= 0 || a.uid.toLowerCase().indexOf(query.toLowerCase()) >= 0)).map((a) => a.label))
          }
        },
        on: {
          change (value) {
            const selected = value.map((label) => self.addons.find((a) => a.label === label))
            self.$set(self, 'selectedAddons', selected)
          }
        }
      })
    })
  }
}
</script>
