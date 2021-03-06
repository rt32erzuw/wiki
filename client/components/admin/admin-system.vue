<template lang='pug'>
  v-container(fluid, grid-list-lg)
    v-layout(row, wrap)
      v-flex(xs12)
        .admin-header
          img.animated.fadeInUp(src='/svg/icon-tune.svg', alt='System Info', style='width: 80px;')
          .admin-header-title
            .headline.primary--text.animated.fadeInLeft {{ $t('admin:system.title') }}
            .subheading.grey--text.animated.fadeInLeft.wait-p2s {{ $t('admin:system.subtitle') }}
        v-layout.mt-3(row wrap)
          v-flex(lg6 xs12)
            v-card.animated.fadeInUp
              v-btn.animated.fadeInLeft.wait-p2s.btn-animate-rotate(fab, absolute, right, top, small, light, @click='refresh'): v-icon refresh
              v-list(two-line, dense)
                v-subheader Wiki.js
                v-list-tile(avatar)
                  v-list-tile-avatar
                    v-icon.blue.white--text system_update_alt
                  v-list-tile-content
                    v-list-tile-title {{ $t('admin:system.currentVersion') }}
                    v-list-tile-sub-title {{ info.currentVersion }}
                v-list-tile(avatar)
                  v-list-tile-avatar
                    v-icon.blue.white--text open_in_browser
                  v-list-tile-content
                    v-list-tile-title {{ $t('admin:system.latestVersion') }}
                    v-list-tile-sub-title {{ info.latestVersion }}
                  v-list-tile-action
                    v-list-tile-action-text {{ $t('admin:system.published') }} {{ info.latestVersionReleaseDate | moment('from') }}

                v-divider.mt-3

                v-subheader {{ $t('admin:system.hostInfo') }}
                v-list-tile(avatar)
                  v-list-tile-avatar
                    v-avatar.blue-grey(size='40')
                      img(:src='`/svg/icon-` + platformLogo + `-logo.svg`', alt='Platform', style='width: 24px;')
                  v-list-tile-content
                    v-list-tile-title {{ $t('admin:system.os') }}
                    v-list-tile-sub-title {{ (info.platform === 'docker') ? 'Docker Container (Linux)' : info.operatingSystem }}
                v-list-tile(avatar)
                  v-list-tile-avatar
                    v-icon.blue-grey.white--text computer
                  v-list-tile-content
                    v-list-tile-title {{ $t('admin:system.hostname') }}
                    v-list-tile-sub-title {{ info.hostname }}
                v-list-tile(avatar)
                  v-list-tile-avatar
                    v-icon.blue-grey.white--text nfc
                  v-list-tile-content
                    v-list-tile-title {{ $t('admin:system.cpuCores') }}
                    v-list-tile-sub-title {{ info.cpuCores }}
                v-list-tile(avatar)
                  v-list-tile-avatar
                    v-icon.blue-grey.white--text memory
                  v-list-tile-content
                    v-list-tile-title {{ $t('admin:system.totalRAM') }}
                    v-list-tile-sub-title {{ info.ramTotal }}
                v-list-tile(avatar)
                  v-list-tile-avatar
                    v-icon.blue-grey.white--text last_page
                  v-list-tile-content
                    v-list-tile-title {{ $t('admin:system.workingDirectory') }}
                    v-list-tile-sub-title {{ info.workingDirectory }}
                v-list-tile(avatar)
                  v-list-tile-avatar
                    v-icon.blue-grey.white--text settings
                  v-list-tile-content
                    v-list-tile-title {{ $t('admin:system.configFile') }}
                    v-list-tile-sub-title {{ info.configFile }}

          v-flex(lg6 xs12)
            v-card.pb-3.animated.fadeInUp.wait-p4s
              v-list(dense)
                v-subheader Node.js
                v-list-tile(avatar)
                  v-list-tile-avatar
                    v-avatar.light-green(size='40')
                      icon-node-js(fillColor='#FFFFFF')
                  v-list-tile-content
                    v-list-tile-title {{ info.nodeVersion }}

                v-divider.mt-3

                v-subheader {{ info.dbType }}
                v-list-tile(avatar)
                  v-list-tile-avatar
                    v-avatar.indigo.darken-1(size='40')
                      icon-database(fillColor='#FFFFFF')
                  v-list-tile-content
                    v-list-tile-title(v-html='dbVersion')
                    v-list-tile-sub-title {{ info.dbHost }}

                v-alert.mt-3(:value='isDbLimited', color='deep-orange', icon='warning') {{ $t('admin:system.dbPartialSupport') }}
</template>

<script>
import _ from 'lodash'

import IconDatabase from 'mdi/Database'
import IconNodeJs from 'mdi/Nodejs'

import systemInfoQuery from 'gql/admin/system/system-query-info.gql'

export default {
  components: {
    IconDatabase,
    IconNodeJs
  },
  data() {
    return {
      info: {}
    }
  },
  computed: {
    dbVersion() {
      return _.get(this.info, 'dbVersion', '').replace(/(?:\r\n|\r|\n)/g, '<br />')
    },
    platformLogo() {
      switch (this.info.platform) {
        case 'docker':
          return 'docker'
        case 'darwin':
          return 'apple'
        case 'linux':
          return 'linux'
        case 'win32':
          return 'windows'
        default:
          return ''
      }
    },
    isDbLimited() {
      return this.info.dbType === 'MySQL' && this.dbVersion.indexOf('5.') === 0
    }
  },
  methods: {
    async refresh() {
      await this.$apollo.queries.info.refetch()
      this.$store.commit('showNotification', {
        message: this.$t('admin:system.refreshSuccess'),
        style: 'success',
        icon: 'cached'
      })
    }
  },
  apollo: {
    info: {
      query: systemInfoQuery,
      fetchPolicy: 'network-only',
      update: (data) => data.system.info,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-system-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>

</style>
