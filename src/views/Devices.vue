<template>
  <div>
    <v-row>
      <component
        :is="item.Options.component"
        v-for="item in app.data.deviceList"
        :key="item.Internals.FUUID"
        :item="item"
      />
    </v-row>
  </div>
</template>

<script>
export default {
  name: 'Devices',

  components: {
    templ_default: () => import('@/components/TemplDefault.vue'),
    templ_panel: () => import('@/components/TemplPanel.vue'),
    templ_chart: () => import('@/components/TemplChart.vue'),
    templ_weather: () => import('@/components/TemplWeather.vue'),
    templ_myweather: () => import('@/components/TemplMyWeather.vue'),
    templ_mycalview: () => import('@/components/TemplMyCalview.vue'),
    templ_scenes: () => import('@/components/TemplScenes.vue'),
    templ_sonos: () => import('@/components/TemplSonos.vue'),
    templ_sysmon: () => import('@/components/TemplSysmon.vue'),
    templ_hmlan: () => import('@/components/TemplHmLan.vue'),
    templ_cam: () => import('@/components/TemplCam.vue')
  },
  data: () => ({
    app: {
      session: {
        connect: false
      },
      options: {
        mobileHeader: false
      },
      data: {
        deviceList: [],
        header: ''
      }
    }
  }),

  watch: {
    $route() {
      this.subscribe();
    },

    'app.session.ready'(val) {
      if(val) this.subscribe();
    }
  },

  mounted() {
    this.app.session = this.$fhem.app.session;
    this.app.options = this.$fhem.app.options;
    this.app.data = this.$fhem.app.data;
  },

  methods: {
    setHeader() {
      if(this.app.options.mobileHeader) {
        if(this.$route.name === 'Devices') {
          if(this.$route.params.filter.match('device=')) {
            this.$fhem.app.appBar.header = '';
          } else {
            this.$fhem.app.appBar.header = this.$route.params.filter.split('=')[1];
          }
        } else {
          this.$fhem.app.appBar.header = this.$route.name;
        }
      }
    },

    subscribe() {
      let fltr = this.$route.params.filter || this.$route.path;

      this.setHeader();
      this.$fhem.getDevices(fltr);
    },
  }
}
</script>
