<template>
  <!-- Main content -->
  <section class="content">
    <!-- Info boxes -->
    <div class="row" v-if="userRole === 'STANDARD' && defaultEKPIData">
      <!-- <div class="alert alert-success alert-dismissible">
        <button type="button" class="close" data-dismiss="alert" aria-hidden="true">&times;</button>
        <h4><i class="icon fa fa-check"></i> CoPilot is open source!</h4>
        Click on icon to check it out on github. <a href="https://github.com/misterGF/CoPilot" target="_blank"><i class="fa fa-github fa-2x"></i></a>
      </div> -->
      <h4 style="margin-left: 15px; margin-top: -5px">Latest Economic Data Regarding your Organisation, by the year {{defaultEKPIData.date}}</h4>
      <div class="col-md-3 col-sm-6 col-xs-12">
        <div class="info-box">
          <span class="info-box-icon bg-aqua"><i class="fa fa-money"></i></span>

          <div class="info-box-content">
            <span class="info-box-text">Revenues in million SEK</span>
            <span class="info-box-number">{{defaultEKPIData.values[0].revenue}}</span>
          </div>
          <!-- /.info-box-content -->
        </div>
        <!-- /.info-box -->
      </div>
      <!-- /.col -->
      <div class="col-md-3 col-sm-6 col-xs-12">
        <div class="info-box">
          <span class="info-box-icon bg-red"><i class="fa fa-euro"></i></span>

          <div class="info-box-content">
            <span class="info-box-text">Yearly Result in million SEK</span>
            <span class="info-box-number">{{defaultEKPIData.values[1].profit}}</span>
          </div>
          <!-- /.info-box-content -->
        </div>
        <!-- /.info-box -->
      </div>
      <!-- /.col -->

      <!-- fix for small devices only -->
      <div class="clearfix visible-sm-block"></div>

      <div class="col-md-3 col-sm-6 col-xs-12">
        <div class="info-box">
          <span class="info-box-icon bg-green"><i class="fa fa-money"></i></span>

          <div class="info-box-content">
            <span class="info-box-text">Net Sales</span>
            <span class="info-box-number">{{defaultEKPIData.values[3].netSales}}</span>
          </div>
          <!-- /.info-box-content -->
        </div>
        <!-- /.info-box -->
      </div>
      <!-- /.col -->
      <div class="col-md-3 col-sm-6 col-xs-12">
        <div class="info-box">
          <span class="info-box-icon bg-yellow"><i class="fa fa-line-chart"></i></span>

          <div class="info-box-content">
            <span class="info-box-text">Equity Percentage</span>
            <span class="info-box-number">{{defaultEKPIData.values[2].equityPercentage}}</span>
          </div>
          <!-- /.info-box-content -->
        </div>
        <!-- /.info-box -->
      </div>
      <!-- /.col -->
    </div>
    <!-- /.row -->


    <div class="col-xs-12">
      <div class="box">
        <div class="box-header with-border">
          <h3 class="box-title" v-if="userRole === 'STANDARD'">Kommuninvest's Client</h3>
          <h3 class="box-title" v-if="userRole === 'MANAGER' || userRole === 'ADMIN'">Kommuninvest's Employees </h3>
          <div class="box-body" style="margin-bottom: -50px;">
            <div class="row">
              <div v-if="userRole === 'ADMIN'">
                <h4 class="text-center">File Upload</h4>
                <div class="form-group" style="text-align: center;">
                  <label>
                    <input type="file" id="file" ref="file" v-on:change="handleFileUpload()"/>
                  </label>
                    <button v-on:click="submitFile()" class="btn btn-success">Submit <i class="fa fa-cloud-upload"></i></button>
                </div>
                <hr>
              </div>
            </div>
            <div class="row">
              <div>
                <div class="col-sm-4 col-xs-12">
                  <p class="text-center">
                    <strong>Pick an Economic Key Indicator</strong>
                  </p>
                  <select class="form-control" v-show="picked === 'com'" v-model="selectedEKI">
                    <option v-for="economicKeyIndicatorCom in economicKeyIndicatorsCom" v-bind:value="economicKeyIndicatorCom.id">
                      {{economicKeyIndicatorCom.name}}
                    </option>
                  </select>
                </div>
                <div class="col-sm-4 col-xs-12">
                  <p class="text-center">
                    <strong>Pick a Financial Key Indicator</strong>
                  </p>
                  <div class="form-group">
                    <select class="form-control" v-model="selectedFKI">
                      <option v-for="financialKeyIndicator in financialKeyIndicators" v-bind:value="financialKeyIndicator.id">
                        {{financialKeyIndicator.name}}
                      </option>
                    </select>
                  </div>
                </div>
                <div class="col-sm-4 col-xs-12">
                  <button style="margin-top: 29px" type="button" v-on:click="generateComAveGraphs" class="btn btn-block btn-success">Generate Graphs</button>
                </div>
              </div>
              <div v-else>
                <div class="col-sm-5 col-xs-12">
                  <div v-if="userRole === 'MANAGER' || userRole === 'ADMIN'">
                    <p class="text-center">
                      <strong>Pick a Municipality Group(s)</strong>
                    </p>
                    <div class="form-group">
                      <multiselect
                        v-model="municipalitySelected"
                        placeholder="Pick an Entity"
                        :tagPosition='bottom'
                        :multiple="true"
                        :options="municipalityGroups"
                        :custom-label="customLabelGroups">
                      </multiselect>
                    </div>
                  </div>
                  <p class="text-center">
                    <strong>Pick a Municipality Entity(-ies)</strong>
                  </p>
                  <div>
                    <multiselect
                      v-model="entitiesSelected"
                      placeholder="Pick an Entity"
                      :tagPosition='bottom'
                      :multiple="true"
                      :options="entityOptions"
                      :max=1>
                    </multiselect>
                    <multiselect
                      v-else
                      v-model="entitiesSelected"
                      placeholder="Pick an Entity"
                      :tagPosition='bottom'
                      :multiple="true"
                      :options="entityOptions">
                    </multiselect>
                  </div>
                </div>
                <hr class="visible-xs-block">
                <div class="col-sm-4 col-xs-12">
                  <p class="text-center">
                    <strong>Pick an Economic Key Indicator</strong>
                  </p>
                  <div v-if="userRole === 'MANAGER' || userRole === 'ADMIN'">
                    <select class="form-control" v-show="picked === 'com'" v-model="selectedEKI">
                      <option v-for="economicKeyIndicatorCom in economicKeyIndicatorsCom" v-bind:value="economicKeyIndicatorCom.id">
                        {{economicKeyIndicatorCom.name}}
                      </option>
                    </select>
                    <select class="form-control" v-show="picked === 'muni'" v-model="selectedEKI">
                      <option v-for="economicKeyIndicatorMun in economicKeyIndicatorsMun"  v-bind:value="economicKeyIndicatorMun.id">
                        {{economicKeyIndicatorMun.name}}
                      </option>
                    </select><br/>
                    <p class="text-center">
                      Are you comparing Companies or Municipalities?
                    </p>
                    <p class="text-center">
                      <input type="radio" id="com" value="com" v-model="picked">
                      <label for="one">Companies</label> &nbsp;
                      <input type="radio" id="muni" value="muni" v-model="picked">
                      <label for="two">Municipalities</label>
                    </p>
                  </div>
                  
                  <div class="form-group" v-if="userRole === 'STANDARD'">
                    <select class="form-control" v-model="selectedEKI">
                      <option v-for="economicKeyIndicatorCom in economicKeyIndicatorsCom"  v-bind:value="economicKeyIndicatorCom.id">
                        {{economicKeyIndicatorCom.name}}
                      </option>
                    </select>
                  </div>
                  <!--<div class="checkbox" style="text-align: center;">
                    <label v-if="userRole === 'STANDARD' || userRole === 'MANAGER' || userRole === 'ADMIN'">
                      <input type="checkbox">Compare with peer group average<br/>
                    </label>
                     <label>
                      <input type="checkbox" v-model="scatterActive">Compare KPIs with a Scatterplot
                    </label> 
                  </div>-->
                </div>
                <hr class="visible-xs-block">
                <div class="col-sm-3 col-xs-12">
                  <p class="text-center">
                    <strong>Pick a Financial Key Indicator</strong>
                  </p>
                  <div class="form-group">
                    <select class="form-control" v-model="selectedFKI">
                      <option v-for="financialKeyIndicator in financialKeyIndicators" v-bind:value="financialKeyIndicator.id">
                        {{financialKeyIndicator.name}}
                      </option>
                    </select>
                  </div>
                  <button type="button" v-on:click="generateGraphs" class="btn btn-block btn-success">Generate Graphs</button>
                </div>
              </div>
            </div> <br><br>
            <div class="row">
              <div class="col-sm-6 col-xs-12">
                <div id="container0" class="graph"></div>
              </div>
              <div class="col-sm-6 col-xs-12">
                <div id="container1" class="graph"></div>
              </div>
            </div>
            <hr/>
            <div class="row" style="text-align: center;">
              <button type="button" v-on:click="generateScatterPlots" class="btn btn-block btn-success">
                Generate Scatter Plots
              </button> <br/>
            </div>
            <div class="row">
              <div class="col-sm-6 col-xs-12">
                <p class="text-center">
                  <strong>Pick 1 or 2 Economic Key Indicators</strong>
                </p>
                <div v-if="economicKeyIndicatorsCom">
                  <div class="col-md-10">
                    <multiselect
                      v-model="scatterEcoSelected"
                      placeholder="Pick here"
                      :tagPosition='bottom'
                      :multiple="true"
                      :options="economicKeyIndicatorsCom"
                      :max=2
                      :custom-label="customLabelGroups">
                    </multiselect>
                  </div>
                  <div class="col-md-2">
                    <select class="form-control" v-model="selectedScatterEYear">
                      <option value="2010">2010</option>
                      <option value="2011">2011</option>
                      <option value="2012">2012</option>
                      <option value="2013">2013</option>
                      <option value="2014">2014</option>
                      <option value="2015">2015</option>
                      <option value="2016">2016</option>
                    </select>
                  </div>
                </div><br/>
                <br/><br/>

                <div id="container2" class="graph"></div>
              </div>
              <div class="col-sm-6 col-xs-12">
                <p class="text-center">
                  <strong>Pick 1 or 2 Financial Key Indicators</strong>
                </p>
                <div v-if="financialKeyIndicators">
                  <div class="col-md-9">
                    <multiselect
                      v-model="scatterFinSelected"
                      placeholder="Pick here"
                      :tagPosition='bottom'
                      :multiple="true"
                      :options="financialKeyIndicators"
                      :max=2
                      :custom-label="customLabelGroups">
                    </multiselect>
                  </div>
                  <div class="col-md-3">
                    <select class="form-control" v-model="selectedScatterFYear">
                      <option value="2013-12-31">2013-12-31</option>
                      <option value="2014-03-31">2014-03-31</option>
                      <option value="2014-06-30">2014-06-30</option>
                      <option value="2014-09-30">2014-09-30</option>
                      <option value="2014-12-31">2014-12-31</option>
                      <option value="2015-03-31">2015-03-31</option>
                      <option value="2015-06-30">2015-06-30</option>
                      <option value="2015-09-30">2015-09-30</option>
                      <option value="2015-12-31">2015-12-31</option>
                      <option value="2016-03-31">2016-03-31</option>
                      <option value="2016-06-30">2016-06-30</option>
                      <option value="2016-09-30">2016-09-30</option>
                      <option value="2016-12-31">2016-12-31</option>
                      <option value="2017-03-31">2017-03-31</option>
                      <option value="2017-06-30">2017-06-30</option>
                    </select>
                  </div>
                </div><br/><br/><br/>

                <div id="container3" class="graph"></div><br/><br/>
              </div>
            </div><br><br>
          </div>

        </div>
      </div>
    </div>

    <div v-if="userRole === 'STANDARD' && defaultFKPIData">
      <h4 style="margin-left: 15px; margin-top: -5px">Latest Financial Data Regarding your Organisation, by the year {{defaultFKPIData.date}}</h4>
      <div class="row">
        <div class="col-md-3 col-sm-6 col-xs-12">
          <div class="info-box bg-yellow">
            <span class="info-box-icon"><i class="ion ion-ios-pricetag-outline"></i></span>

            <div class="info-box-content">
              <span class="info-box-text">Outstanding Debts</span>
              <span class="info-box-number">{{defaultFKPIData.values[0].outstandingDebt}}</span>

              <!-- <div class="progress">
                <div class="progress-bar" style="width: 50%"></div>
              </div>
                  <span class="progress-description">
                    50% Increase
                  </span> -->
            </div>
          </div>
        </div>
        <div class="col-md-3 col-sm-6 col-xs-12">
          <div class="info-box bg-green">
            <span class="info-box-icon"><i class="ion ion-ios-heart-outline"></i></span>

            <div class="info-box-content">
              <span class="info-box-text">Capital Commitment</span>
              <span class="info-box-number">{{defaultFKPIData.values[1].capitalCommitment}}</span>

              <!-- <div class="progress">
                <div class="progress-bar" style="width: 20%"></div>
              </div>
                  <span class="progress-description">
                    20% Increase
                  </span>-->
            </div> 
          </div>
        </div>
        <div class="col-md-3 col-sm-6 col-xs-12">
          <div class="info-box bg-red">
            <span class="info-box-icon"><i class="ion ion-ios-cloud-download-outline"></i></span>

            <div class="info-box-content">
              <span class="info-box-text">Rate Commitment</span>
              <span class="info-box-number">{{defaultFKPIData.values[2].rateCommitment}}</span>

              <!-- <div class="progress">
                <div class="progress-bar" style="width: 70%"></div>
              </div>
                  <span class="progress-description">
                    70% Increase
                  </span>-->
            </div> 
          </div>
        </div>
        <div class="col-md-3 col-sm-6 col-xs-12">
          <div class="info-box bg-aqua">
            <span class="info-box-icon"><i class="ion-ios-chatbubble-outline"></i></span>

            <div class="info-box-content">
              <span class="info-box-text">Current Rate Percentage</span>
              <span class="info-box-number">{{defaultFKPIData.values[3].currentRatePercentage}}</span>

              <!-- <div class="progress">
                <div class="progress-bar" style="width: 40%"></div>
              </div>
                  <span class="progress-description">
                    40% Increase
                  </span> -->
            </div>
          </div>
        </div>
      </div>
    </div>

  </section>
  <!-- /.content -->
</template>

<script>
// import Highcharts from 'highcharts/highstock'
var Highcharts = require('highcharts/highstock')
import api from '../../api'
import Multiselect from 'vue-multiselect'
require('highcharts/modules/exporting')(Highcharts)
require('highcharts-export-csv')(Highcharts)
require('highcharts/modules/histogram-bellcurve')(Highcharts)

export default {
  components: { Multiselect },
  data () {
    return {
      generateRandomNumbers (numbers, max, min) {
        var a = []
        for (var i = 0; i < numbers; i++) {
          a.push(Math.floor(Math.random() * (max - min + 1)) + max)
        }
        return a
      },
      username: this.$store.state.user.fullname,
      userRole: this.$store.state.user.role,
      userOrganization_id: this.$store.state.user.organization_id,
      economicKeyIndicatorsCom: null,
      economicKeyIndicatorsMun: null,
      financialKeyIndicators: null,
      municipalitySelected: null,
      entitiesSelected: null,
      municipalityGroups: [],
      entityOptions: [],
      bottom: 'bottom',
      timeSeries: '',
      scatterSeries: '',
      scatterActive: false,
      wholeActive: false,
      organisations: null,
      picked: 'com',
      selectedFKI: null,
      selectedEKI: null,
      defaultEKPIData: null,
      defaultFKPIData: null,
      scatterFinSelected: null,
      scatterEcoSelected: null,
      selectedScatterEYear: null,
      selectedScatterFYear: null,
      file: ''
    }
  },
  watch: {
    picked: function () {
      this.updateEntities()
    },
    municipalitySelected: function () {
      this.updateEntities()
    },
    entitiesSelected: function () {
      var url = null
      if (this.username === 'Tim Liberg') {  // TO be replaced with the MUNICIPLAITY EMPLOYEE role ID
        if (this.entitiesSelected.length === 1) {
          url = '/dashboard/details/' + this.entitiesSelected[0].split('-')[0]
        } else {
          url = '/dashboard/details/'
        }

        api.request('get', url)
        .then(response => {
          var data = response.data
          console.log(data)

          if (data.EKPI) {
            this.defaultEKPIData = data.EKPI
            this.defaultEKPIData.values[2].equityPercentage = Math.round(this.defaultEKPIData.values[2].equityPercentage * 100) + '%'
          }

          if (data.FKPI) {
            this.defaultFKPIData = data.FKPI
          }
        })
        .catch(error => {
          this.$store.commit('TOGGLE_LOADING')
          console.log(error)

          this.response = 'Server appears to be offline'
        })
      }
    }
  },
  computed: {
    coPilotNumbers () {
      return this.generateRandomNumbers(12, 1000000, 10000)
    },
    personalNumbers () {
      return this.generateRandomNumbers(12, 1000000, 10000)
    },
    isMobile () {
      return (window.innerWidth <= 800 && window.innerHeight <= 600)
    }
  },
  mounted () {
    if (this.userRole === 'STANDARD') {
      api.request('get', '/companies/')
      .then(response => {
        var data = response.data
        console.log(data)

        for (var j = 0; j < data.length; j++) {
          this.entityOptions.push(data[j].organization_id + '-' + data[j].municipality_id)
        }
      })
      .catch(error => {
        this.$store.commit('TOGGLE_LOADING')
        console.log(error)

        this.response = 'Server appears to be offline'
      })
    } else if (this.userRole === 'MANAGER' || this.userRole === 'ADMIN') {
      api.request('get', '/municipality-types')
      .then(response => {
        var data = response.data

        this.municipalityGroups = data
      })
      .catch(error => {
        this.$store.commit('TOGGLE_LOADING')
        console.log(error)

        this.response = 'Server appears to be offline'
      })
    }

    api.request('get', '/dashboard/details/')
    .then(response => {
      var data = response.data
      console.log(data)

      if (data.EKPI) {
        this.defaultEKPIData = data.EKPI
        this.defaultEKPIData.values[2].equityPercentage = Math.round(this.defaultEKPIData.values[2].equityPercentage * 100) + '%'
      }

      if (data.FKPI) {
        this.defaultFKPIData = data.FKPI
      }
    })
    .catch(error => {
      this.$store.commit('TOGGLE_LOADING')
      console.log(error)

      this.response = 'Server appears to be offline'
    })

    api.request('get', '/indicators')
    .then(response => {
      var data = response.data
      console.log(data)
      if (data) {
        this.economicKeyIndicatorsMun = data[1].indicators
        this.economicKeyIndicatorsCom = data[2].indicators
        this.financialKeyIndicators = data[3].indicators
      }
    })
    .catch(error => {
      this.$store.commit('TOGGLE_LOADING')
      console.log(error)

      this.response = 'Server appears to be offline'
    })
  },
  methods: {
    submitFile () {
      let formData = new FormData()
      formData.append('file', this.file)

      api.request('post', '/data',
        formData,
        {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        }
      ).then(function () {
        console.log('SUCCESS!!')
        window.alert('Data has been uploaded successfully! Please refresh the page for updated content.')
      })
      .catch(function () {
        console.log('FAILURE!!')
        window.alert('An error has occured in the file upload.')
      })
    },
    handleFileUpload () {
      this.file = this.$refs.file.files[0]
    },
    uploadExcel () {
      api.request('post', '/data')
      .then(response => {
        var data = response.data
        console.log(data)
        if (data) {
          this.economicKeyIndicatorsMun = data[1].indicators
          this.economicKeyIndicatorsCom = data[2].indicators
          this.financialKeyIndicators = data[3].indicators
        }
      })
      .catch(error => {
        this.$store.commit('TOGGLE_LOADING')
        console.log(error)

        this.response = 'Server appears to be offline'
      })
    },
    updateEntities () {
      this.entitiesSelected = null
      if (this.municipalitySelected.length === 0) {
        this.entityOptions = []
      } else if (this.municipalitySelected.length > 0) {
        this.entityOptions = []
        console.log('triggered')
        for (var i = 0; i < this.municipalitySelected.length; i++) {
          console.log(this.municipalitySelected[i].id)
          api.request('get', '/municipality-types/' + this.municipalitySelected[i].id)
          .then(response => {
            var organizations = response.data

            if (this.picked === 'muni') {
              for (var j = 0; j < organizations.length; j++) {
                this.entityOptions.push(organizations[j].organization_id + '-' + organizations[j].id + '-0')
                if (organizations[j].companies.length > 0) {
                  this.entityOptions.push(organizations[j].organization_id + '-' + organizations[j].id + '-1')
                }
              }
            } else if (this.picked === 'com') {
              for (var h = 0; h < organizations.length; h++) {
                if (organizations[h].companies.length > 0) {
                  for (var k = 0; k < organizations[h].companies.length; k++) {
                    this.entityOptions.push(organizations[h].companies[k].organization_id + '-' + organizations[h].id)
                  }
                }
              }
            }
          })
          .catch(error => {
            this.$store.commit('TOGGLE_LOADING')
            console.log(error)

            this.response = 'Server appears to be offline'
          })
        }
      }
    },
    customLabelGroups (option) {
      return `${option.name}`
    }, /*
    customLabelEntities (option) {
      return `${option.organization_id}-${option.municipality_id}`
    }, */
    generateComAveGraphs () {
      console.log(this.selectedFKI + ' ' + this.selectedEKI + ' ' + this.entitiesSelected)

      api.request('get', '/dashboard/peer-companies-graph/' + this.selectedEKI + '/' + this.selectedFKI)
      .then(response => {
        console.log(response.data)
        var ekiGraph = response.data.EKPI
        var fkiGraph = response.data.FKPI

        Highcharts.chart('container0', {
          exporting: {
            chartOptions: { // specific options for the exported image
              plotOptions: {
                series: {
                  dataLabels: {
                    enabled: true
                  }
                }
              }
            },
            fallbackToExportServer: false
          },

          title: {
            text: ekiGraph.title
          },

          tooltip: {
            crosshairs: [true, true]
          },
          // subtitle: {
            // text: 'Source: thesolarfoundation.com'
          // },
          scrollbar: {
            enabled: true
          },

          navigator: {
            enabled: true
          },

          rangeSelector: {
            enabled: true
          },

          /* yAxis: {
            title: {
              text: 'Number of Employees'
            }
          }, */
          legend: {
            layout: 'vertical',
            align: 'right',
            verticalAlign: 'middle'
          },

          plotOptions: {
            series: {
              label: {
                connectorAllowed: false
              },
              pointStart: 2010
            }
          },

          series: ekiGraph.series,

          responsive: {
            rules: [{
              condition: {
                maxWidth: 500
              },
              chartOptions: {
                legend: {
                  layout: 'horizontal',
                  align: 'center',
                  verticalAlign: 'bottom'
                }
              }
            }]
          }
        })

        Highcharts.chart('container1', {
          exporting: {
            chartOptions: { // specific options for the exported image
              plotOptions: {
                series: {
                  dataLabels: {
                    enabled: true
                  }
                }
              }
            },
            fallbackToExportServer: false
          },

          title: {
            text: fkiGraph.title
          },

          tooltip: {
            crosshairs: [true, true]
          },
          // subtitle: {
            // text: 'Source: thesolarfoundation.com'
          // },
          scrollbar: {
            enabled: true
          },

          navigator: {
            enabled: true
          },

          rangeSelector: {
            enabled: true
          },
          /* yAxis: {
            title: {
              text: 'Number of Employees'
            }
          }, */

          legend: {
            layout: 'vertical',
            align: 'right',
            verticalAlign: 'middle'
          },

          plotOptions: {
            series: {
              label: {
                connectorAllowed: false
              },
              pointStart: 2010
            }
          },

          series: fkiGraph.series,

          responsive: {
            rules: [{
              condition: {
                maxWidth: 500
              },
              chartOptions: {
                legend: {
                  layout: 'horizontal',
                  align: 'center',
                  verticalAlign: 'bottom'
                }
              }
            }]
          }
        })
      })
      .catch(error => {
        this.$store.commit('TOGGLE_LOADING')
        console.log(error)

        this.response = 'Server appears to be offline'
      })
    },
    generateScatterPlots () {
      var organization = this.userOrganization_id
      if (this.entitiesSelected) {
        organization = this.entitiesSelected[0].split('-')[0]
      }

      if (this.scatterEcoSelected.length === 2) {
        api.request('get', '/graphs/scatter-plot/' + organization + '/' + this.selectedScatterEYear + '/' + this.scatterEcoSelected[0].id + '/' + this.scatterEcoSelected[1].id)
        .then(response => {
          var data = response.data
          console.log(data)

          Highcharts.chart('container2', {
            exporting: {
              chartOptions: { // specific options for the exported image
                plotOptions: {
                  series: {
                    dataLabels: {
                      enabled: true
                    }
                  }
                }
              },
              fallbackToExportServer: false
            },
            chart: {
              type: 'scatter',
              zoomType: 'xy'
            },
            title: {
              text: this.scatterEcoSelected[0].name + ' vs. ' + this.scatterEcoSelected[1].name
            },
            tooltip: {
              crosshairs: [true, true]
            },
            xAxis: {
              title: {
                enabled: true,
                text: this.scatterEcoSelected[1].name
              },
              startOnTick: true,
              endOnTick: true,
              showLastLabel: true
            },
            yAxis: {
              title: {
                text: this.scatterEcoSelected[0].name
              }
            },
            legend: {
              layout: 'vertical',
              align: 'right',
              verticalAlign: 'top',
              floating: true,
              backgroundColor: (Highcharts.theme && Highcharts.theme.legendBackgroundColor) || '#FFFFFF',
              borderWidth: 1
            },
            plotOptions: {
              scatter: {
                marker: {
                  radius: 5,
                  states: {
                    hover: {
                      enabled: true,
                      lineColor: 'rgb(100,100,100)'
                    }
                  }
                },
                states: {
                  hover: {
                    marker: {
                      enabled: false
                    }
                  }
                },
                tooltip: {
                  headerFormat: '<b>{series.name}</b><br>',
                  pointFormat: '{point.x}, {point.y}'
                }
              }
            },
            series: [{
              name: data.name,
              color: 'rgba(223, 83, 83, .5)',
              data: data.data
            }]

          })
        })
        .catch(error => {
          this.$store.commit('TOGGLE_LOADING')
          console.log(error)

          this.response = 'Server appears to be offline'
        })
      } else if (this.scatterEcoSelected.length === 1) {
        api.request('get', '/graphs/distribution/' + organization + '/' + this.selectedScatterEYear + '/' + this.scatterEcoSelected[0].id)
        .then(response => {
          console.log(response.data)

          Highcharts.chart('container2', {
            title: {
              text: this.scatterEcoSelected[0].name
            },
            xAxis: [{
              title: { text: 'Data' },
              alignTicks: false
            }, {
              title: { text: 'Histogram' },
              alignTicks: false,
              opposite: true
            }],

            yAxis: [{
              title: { text: 'Data' }
            }, {
              title: { text: 'Histogram' },
              opposite: true
            }],

            series: [{
              name: organization + ' - Histogram',
              type: 'histogram',
              xAxis: 1,
              yAxis: 1,
              baseSeries: 's1',
              zIndex: -1
            }, {
              name: organization + ' - Data',
              type: 'scatter',
              data: response.data.data,
              id: 's1',
              marker: {
                radius: 5
              }
            }]
          })
        })
        .catch(error => {
          this.$store.commit('TOGGLE_LOADING')
          console.log(error)

          this.response = 'Server appears to be offline'
        })
      }

      if (this.scatterFinSelected.length === 2) {
        api.request('get', '/graphs/scatter-plot/' + organization + '/' + this.selectedScatterFYear + '/' + this.scatterFinSelected[0].id + '/' + this.scatterFinSelected[1].id)
        .then(response => {
          var data = response.data
          console.log(data)

          Highcharts.chart('container3', {
            exporting: {
              chartOptions: { // specific options for the exported image
                plotOptions: {
                  series: {
                    dataLabels: {
                      enabled: true
                    }
                  }
                }
              },
              fallbackToExportServer: false
            },
            chart: {
              type: 'scatter',
              zoomType: 'xy'
            },
            title: {
              text: this.scatterFinSelected[0].name + ' vs. ' + this.scatterFinSelected[1].name
            },
            tooltip: {
              crosshairs: [true, true]
            },
            xAxis: {
              title: {
                enabled: true,
                text: this.scatterFinSelected[1].name
              },
              startOnTick: true,
              endOnTick: true,
              showLastLabel: true
            },
            yAxis: {
              title: {
                text: this.scatterFinSelected[0].name
              }
            },
            legend: {
              layout: 'vertical',
              align: 'right',
              verticalAlign: 'top',
              floating: true,
              backgroundColor: (Highcharts.theme && Highcharts.theme.legendBackgroundColor) || '#FFFFFF',
              borderWidth: 1
            },
            plotOptions: {
              scatter: {
                marker: {
                  radius: 5,
                  states: {
                    hover: {
                      enabled: true,
                      lineColor: 'rgb(100,100,100)'
                    }
                  }
                },
                states: {
                  hover: {
                    marker: {
                      enabled: false
                    }
                  }
                },
                tooltip: {
                  headerFormat: '<b>{series.name}</b><br>',
                  pointFormat: '{point.x}, {point.y}'
                }
              }
            },
            series: [{
              name: data.name,
              color: 'rgba(223, 83, 83, .5)',
              data: data.data
            }]

          })
        })
        .catch(error => {
          this.$store.commit('TOGGLE_LOADING')
          console.log(error)

          this.response = 'Server appears to be offline'
        })
      } else if (this.scatterFinSelected.length === 1) {
        api.request('get', '/graphs/distribution/' + organization + '/' + this.selectedScatterFYear + '/' + this.scatterFinSelected[0].id)
        .then(response => {
          console.log(response.data)

          Highcharts.chart('container3', {
            title: {
              text: this.scatterFinSelected[0].name
            },
            xAxis: [{
              title: { text: 'Data' },
              alignTicks: false
            }, {
              title: { text: 'Histogram' },
              alignTicks: false,
              opposite: true
            }],

            yAxis: [{
              title: { text: 'Data' }
            }, {
              title: { text: 'Histogram' },
              opposite: true
            }],

            series: [{
              name: organization + ' - Histogram',
              type: 'histogram',
              xAxis: 1,
              yAxis: 1,
              baseSeries: 's1',
              zIndex: -1
            }, {
              name: organization + ' - Data',
              type: 'scatter',
              data: response.data.data,
              id: 's1',
              marker: {
                radius: 5
              }
            }]
          })
        })
        .catch(error => {
          this.$store.commit('TOGGLE_LOADING')
          console.log(error)

          this.response = 'Server appears to be offline'
        })
      }
    },
    generateGraphs () {
      console.log(this.selectedFKI + ' ' + this.selectedEKI + ' ' + this.entitiesSelected)
      var entities = []

      if (this.entitiesSelected) {
        for (var i = 0; i < this.entitiesSelected.length; i++) {
          var split = this.entitiesSelected[i].split('-')
          var entity = {}
          entity.id = Number(split[0])
          entity.type = null

          if (split[2]) {
            entity.type = Number(split[2])
          }

          entities.push(entity)
        }
      } else {
        entity = {}
        entity.id = this.userOrganization_id
        entity.type = null
        entities.push(entity)
      }

      var requestBody = {
        entityIDs: entities,
        ekpiID: this.selectedEKI,
        fkpiID: this.selectedFKI
      }
      console.log(requestBody)

      api.request('post', '/dashboard/graphs', requestBody)
      .then(response => {
        console.log(response.data)
        var ekiGraph = response.data.EKPI
        var fkiGraph = response.data.FKPI

        Highcharts.chart('container0', {
          exporting: {
            chartOptions: { // specific options for the exported image
              plotOptions: {
                series: {
                  dataLabels: {
                    enabled: true
                  }
                }
              }
            },
            fallbackToExportServer: false
          },

          title: {
            text: ekiGraph.title
          },

          tooltip: {
            crosshairs: [true, true]
          },
          // subtitle: {
            // text: 'Source: thesolarfoundation.com'
          // },
          scrollbar: {
            enabled: true
          },

          navigator: {
            enabled: true
          },

          rangeSelector: {
            enabled: true
          },

          /* yAxis: {
            title: {
              text: 'Number of Employees'
            }
          }, */
          legend: {
            layout: 'vertical',
            align: 'right',
            verticalAlign: 'middle'
          },

          plotOptions: {
            series: {
              label: {
                connectorAllowed: false
              },
              pointStart: 2010
            }
          },

          series: ekiGraph.series,

          responsive: {
            rules: [{
              condition: {
                maxWidth: 500
              },
              chartOptions: {
                legend: {
                  layout: 'horizontal',
                  align: 'center',
                  verticalAlign: 'bottom'
                }
              }
            }]
          }
        })

        Highcharts.chart('container1', {
          exporting: {
            chartOptions: { // specific options for the exported image
              plotOptions: {
                series: {
                  dataLabels: {
                    enabled: true
                  }
                }
              }
            },
            fallbackToExportServer: false
          },

          title: {
            text: fkiGraph.title
          },

          tooltip: {
            crosshairs: [true, true]
          },
          // subtitle: {
            // text: 'Source: thesolarfoundation.com'
          // },
          scrollbar: {
            enabled: true
          },

          navigator: {
            enabled: true
          },

          rangeSelector: {
            enabled: true
          },
          /* yAxis: {
            title: {
              text: 'Number of Employees'
            }
          }, */

          legend: {
            layout: 'vertical',
            align: 'right',
            verticalAlign: 'middle'
          },

          plotOptions: {
            series: {
              label: {
                connectorAllowed: false
              },
              pointStart: 2010
            }
          },

          series: fkiGraph.series,

          responsive: {
            rules: [{
              condition: {
                maxWidth: 500
              },
              chartOptions: {
                legend: {
                  layout: 'horizontal',
                  align: 'center',
                  verticalAlign: 'bottom'
                }
              }
            }]
          }
        })
      })
      .catch(error => {
        this.$store.commit('TOGGLE_LOADING')
        console.log(error)

        this.response = 'Server appears to be offline'
      })
    }
  }
}
</script>
<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<style type="text/css">
   .graph{
    width:101%; 
    height:450px; 
    border: 1px solid rgba(0,0,0,0.3);
    border-radius: 5px
   }
</style>
