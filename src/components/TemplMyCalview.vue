<template>
  <v-col 
    class="col-12 col-sm-6 col-md-4 col-lg-4"
  >
    <v-card
      :dark="this.$vuetify.theme.dark"
      color="secondary"
    >
      <v-progress-linear
        height="7"
        :value="vals.mainLevel"
        :color="vals.mainColor"
        background-color="secondary darken-1"
      />

      <v-card-title 
        class="text-truncate"
      >
        <v-col>
          {{ vals.title }}
        </v-col>  
        <v-spacer />
        <v-btn
          small
          icon
          @click="expand"
        >
          <v-icon small>
            {{ maxIcon }}
          </v-icon>
        </v-btn>
      </v-card-title>
      <v-divider />

      <v-card-text
        v-if="expanded==false"
      >
        <v-row align="center">
            <v-col 
              v-if="vals.itemTodayCount == 0"
            >
              <span> Heute keine Termine </span>
            </v-col>
            <v-col 
              class="col-3" 
              v-if="(vals.itemTodayCount > 0) && (expanded==false)"
            >
              <span>{{ vals.vBeginDate }} </span>
            </v-col>
            <v-col 
              class="col-3" 
              v-if="(vals.itemTodayCount > 0) && (expanded==false)"
            >
              <span>{{ vals.vBeginTime }}</span>
            </v-col>
            <v-col 
              class="col-6" 
              v-if="(vals.itemTodayCount > 0) && (expanded==false)"
            >  
              <span>{{ vals.vDescription }}</span>
            </v-col>
        </v-row>
      </v-card-text>

      <v-divider 
        v-if="expanded==false" 
      />

      <v-card-text 
        v-if="expanded"
      >
        <v-row
          v-for="dayitem in vals.calview"
          :key="dayitem.iCounter"
          :class="dayitem.iColor"
          align="center"
        >
            <v-col 
              class="col-3"
              align="left"
            >
              <span>{{ dayitem.iBeginDate }} </span>
            </v-col>
            <v-col 
              class="col-3"
              align="left"
            >
              <span>{{ dayitem.iBeginTime }}</span>
            </v-col>
            <v-col 
              class="col-6 text-truncate"
              align="left"
            >
              <span>{{ dayitem.iDescription }}</span>
            </v-col>
        </v-row>
      </v-card-text>

      <v-divider />
      <v-system-bar color="secondary darken-1">
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-icon
              v-bind="attrs"
              v-on="on"
            >
              {{ vals.systemItemValues }}
            </v-icon>
          </template>
          Termine: heute/morgen
        </v-tooltip>
        <v-spacer />
        {{ vals.systemLastEvent }}
        <v-spacer />
          <v-icon>
              {{ vals.systemActivityIcon }}
            </v-icon>    
      </v-system-bar>
    </v-card>
  </v-col>
</template>

<script>
  export default {
    props: {
      item: {
        type: Object,
        default: () => { return { name: 'mycalview' } }
      }
    },

    data: () => ({
      name: 'mycalview',
      app: {
        options: {
          lang: ''
        }
      },
      apiSet: {
        CALVIEW: {
          lastUpdate: 'lastUpdate',
          cItemCount: 'c-term',
          cItemTodayCount: 'c-today',
          cItemTomorrowCount: 'c-tomorrow',
          cBeginDate: '_bdate',
          cBeginTime: '_btime',
          cDescription: '_summary',
          cDaysLeft: '_daysleft'
          
        }
      },
      vals: {
        title: '',
        mainLevel: 0,
        mainColor: 'success',
        mainState: '',
        color1: 'red',
        vDescription: '',
        vBeginTime: '',
        vBeginDate: '',
        itemCount: '',
        itemTodayCount: '',
        itemTomorrowCount: '',
        systemItemValues: '',
        systemLastEvent: '',
        calview: []
      },
      maxIcon: 'mdi-arrow-expand',
      expanded: false,
      active: true,
      type: 'Calview'
    }),

    watch: {
      item: {
        immediate: true,
        deep: true,
        handler(val) {
          this.type = this.$fhem.getEl(val, 'Internals', 'TYPE');
          

          let lastevent = this.$fhem.getEl(val, 'Readings', this.apiSet[this.type].lastUpdate, 'Time');
          let alias = this.$fhem.getEl(val, 'Attributes', 'alias') || val.Name;
          
          console.log('LE: '+lastevent+' xx: '+ activity );

          let activity = this.$fhem.getEl(val, 'Readings', this.apiSet[this.type].lastUpdate, 'Value') || 'not up-to-date';
          
          this.vals.title = this.$fhem.getEl(val, 'Options', 'name') || alias;
          this.vals.color1 = this.$fhem.getEl(val, 'Options', 'color1') || "gelb";
          this.vals.vDescription = this.$fhem.getEl(val, 'Readings', 't_001' + this.apiSet[this.type].cDescription, 'Value') || 'des';
          this.vals.vBeginDate = this.$fhem.getEl(val, 'Readings', 't_001' + this.apiSet[this.type].cBeginDate, 'Value') || 'date';
          this.vals.vBeginTime = this.$fhem.getEl(val, 'Readings', 't_001' + this.apiSet[this.type].cBeginTime, 'Value') || 'time';
          this.vals.itemCount = this.$fhem.getEl(val, 'Readings', this.apiSet[this.type].cItemCount, 'Value') || 'icount';
          this.vals.itemTodayCount = this.$fhem.getEl(val, 'Readings', this.apiSet[this.type].cItemTodayCount, 'Value') || 'td';
          this.vals.itemTomorrowCount = this.$fhem.getEl(val, 'Readings', this.apiSet[this.type].cItemTomorrowCount, 'Value') || 'to';

          //if(this.type === 'CALVIEW') {
            //activity = 'up-to-date';
          //}
          
          this.vals.systemLastEvent = lastevent ? this.$fhem.getDateTime(lastevent) : 'no update time';
          
          console.log('LE:'+lastevent+'xx'+ activity + ' y: '+ this.$fhem.getDateTime(lastevent));
          this.vals.systemItemValues = this.vals.itemTodayCount + '/' + this.vals.itemTomorrowCount;
          this.vals.systemActivityIcon = activity === 'not up-to-date' ? 'mdi-sync-off' : 'mdi-sync';
          this.vals.systemActivityState = activity;

          if(activity == 'not up-to-date') {
            this.active = false;
            this.vals.mainLevel = 100;
            this.vals.mainColor = 'error';
            this.vals.mainState = 'nicht aktuell';
          }
        }
      }
    },

    methods: {
      getStdDate(input, format) {
        // get standard dateformat (format as Option?)
        let Format = format || 'dd.mm.yyyy'; // default format
        let Parts = input.match(/(\d+)/g), // split
        i = 0, formatParts = {};
        Format.replace(/(yyyy|dd|mm)/g, function(Part) { formatParts[Part] = i++; });

        return new Date(Parts[formatParts['yyyy']], Parts[formatParts['mm']]-1, Parts[formatParts['dd']]);
        },
      loadCalview() {
        this.vals.calview.splice(0);
        for(let i = 1; i <= this.vals.itemCount; i++) {
          let api = this.type || 'CALVIEW';
          let j = ("00000" + i).slice(-3);
          let iDaysLeft = this.$fhem.getEl(this.item, 'Readings', 't_' + j + this.apiSet[api].cDaysLeft, 'Value') ||'t_' + j + 'daysleft';
          console.log('daysleft ' + iDaysLeft);
          let iColor='white--text text--darken-1';
          if (iDaysLeft<1) 
             iColor='red--text text--darken-1';
          if (iDaysLeft==1) 
             iColor='orange--text text--darken-1';
          let iDescription = this.$fhem.getEl(this.item, 'Readings', 't_' + j + this.apiSet[api].cDescription, 'Value') ||'t_' + j + 'des';
          let iBeginDateLfhem = this.$fhem.getEl(this.item, 'Readings', 't_' + j + this.apiSet[api].cBeginDate, 'Value') || i + 'date';
          let iBeginDateL=this.getStdDate(iBeginDateLfhem);
          let day=iBeginDateL.getDate();
          let month=iBeginDateL.getMonth();
          month=month+1;
		      if((String(day)).length==1)
             day='0'+day;
          if((String(month)).length==1)
              month='0'+month;
          let iBeginDate = day + '.' + month + '.';
          let iBeginTime = this.$fhem.getEl(this.item, 'Readings', 't_' + j + this.apiSet[api].cBeginTime, 'Value') || i + 'time';
          if (iBeginTime === "00:00")
              iBeginTime=' '; 
          
          console.log('color ' + iColor);

          let dayitem = {
            iCounter: j,
            iDescription,
            iBeginDate,
            iBeginTime,
            iColor
          };

          this.vals.calview.push(dayitem);
        }

        
      },

      expand() {
        if(this.expanded) {
          this.expanded = false;
          this.maxIcon = 'mdi-arrow-expand';
        } else {
          this.expanded = true;
          this.maxIcon = 'mdi-arrow-collapse';
          this.loadCalview();
        }
      }
    },

    mounted() {
      this.app.options = this.$fhem.app.options;
      if(this.$fhem.getEl(this.item, 'Options','setup', 'expanded')) {
        this.expanded = false;
        this.expand();
      }
    }
  }
</script>
