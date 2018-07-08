<template>
  <section class="content">
    
    <div class="row center-block">
      <div class="col-md-12">
        <div class="box">
          <!-- /.box-header -->
          <div class="box-body">
            <div class="dataTables_wrapper form-inline dt-bootstrap" id="mainContent">
              <div class="row col-sm-12 table-responsive">
                <div class="box-group" id="accordion" v-if="dataReady">
                    <!-- we are adding the .panel class so bootstrap.js collapse plugin detects it -->
                    <div class="panel box box-primary" v-for="group in groups">
                      <div class="box-header with-border">
                        <h4 class="box-title">
                          <a data-toggle="collapse" data-parent="#accordion" v-bind:href="'#collapse'+group.id">
                            {{group.name}} - {{group.org.length}} organisations <span class="fa fa-chevron-circle-right"></span>
                          </a>
                        </h4>
                      </div>
                      <div v-bind:id="'collapse' + group.id" class="panel-collapse collapse">
                        <div class="box-body">
                          <table v-bind:aria-describedby="'example' + group.id + '_info'" role="grid" v-bind:id="'example' + group.id" class="table table-bordered table-hover table-striped dataTable">
                            <thead>
                              <tr role="row">
                                <th aria-label="Rendering engine: activate to sort column descending" aria-sort="ascending"  colspan="1" rowspan="1" v-bind:aria-controls="'example' + group.id" tabindex="0" class="sorting_asc">Organisation ID</th>
                                <th aria-label="Browser: activate to sort column ascending" colspan="1" rowspan="1" v-bind:aria-controls="'example' + group.id" tabindex="0" class="sorting">Municipality ID</th>
                                <th aria-label="Platform(s): activate to sort column ascending" colspan="1" rowspan="1" v-bind:aria-controls="'example' + group.id" tabindex="0" class="sorting">Organization Type</th>
                                <th aria-label="Engine version: activate to sort column ascending" colspan="1" rowspan="1" v-bind:aria-controls="'example' + group.id" tabindex="0" class="sorting">SIN (if company)</th>
                              </tr>
                            </thead>
                            <tbody>
                              <tr v-for="organization in group.org">
                                <td class="sorting_1">{{organization.organization_id}}</td>
                                <td>{{organization.municipality_id}}</td>
                                <td>{{organization.organization_type}}</td>
                                <td>{{organization.SIN}}</td>
                              </tr>
                            </tbody>
                            <tfoot>
                              <tr>
                                <th colspan="1" rowspan="1">Organisation ID</th>
                                <th colspan="1" rowspan="1">Municipality ID</th>
                                <th colspan="1" rowspan="1">Organization Type</th>
                                <th colspan="1" rowspan="1">SIN (if company)</th>
                              </tr>
                            </tfoot>
                          </table>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <!-- /.box-body -->
          </div>
        </div>
      </div>
    </div>


<!--     <div class="row center-block">
      <h2>Simple</h2>
      <div class="col-md-12">
        <div class="box">
          <div class="box-header">
            <h3 class="box-title">Striped Full Width Table</h3>
          </div>
          <div class="box-body no-padding table-responsive">
            <table class="table table-striped">
              <tbody>
                <tr>
                  <th style="width: 10px">#</th>
                  <th>Task</th>
                  <th>Progress</th>
                  <th style="width: 40px">Label</th>
                </tr>
                <tr>
                  <td>1.</td>
                  <td>Update software</td>
                  <td>
                    <div class="progress progress-xs">
                      <div class="progress-bar progress-bar-danger" style="width: 55%"></div>
                    </div>
                  </td>
                  <td><span class="badge bg-red">55%</span></td>
                </tr>
                <tr>
                  <td>2.</td>
                  <td>Clean database</td>
                  <td>
                    <div class="progress progress-xs">
                      <div class="progress-bar progress-bar-yellow" style="width: 70%"></div>
                    </div>
                  </td>
                  <td><span class="badge bg-yellow">70%</span></td>
                </tr>
                <tr>
                  <td>3.</td>
                  <td>Cron job running</td>
                  <td>
                    <div class="progress progress-xs progress-striped active">
                      <div class="progress-bar progress-bar-primary" style="width: 30%"></div>
                    </div>
                  </td>
                  <td><span class="badge bg-light-blue">30%</span></td>
                </tr>
                <tr>
                  <td>4.</td>
                  <td>Fix and squish bugs</td>
                  <td>
                    <div class="progress progress-xs progress-striped active">
                      <div class="progress-bar progress-bar-success" style="width: 90%"></div>
                    </div>
                  </td>
                  <td><span class="badge bg-green">90%</span></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div> -->

    
  </section>
</template>

<script>
import $ from 'jquery'
import api from '../../api'
import 'datatables.net'
import 'datatables.net-bs'

export default {
  data () {
    return {
      organizations: null,
      groups: null,
      count: 0,
      dataReady: false
    }
  },
  mounted () {
    this.$nextTick(() => {
      $('#example9').DataTable()
    })

    api.request('get', '/municipality-types')
    .then(response => {
      this.groups = response.data
      this.getCompanies(this.groups)
    })
    .catch(error => {
      this.$store.commit('TOGGLE_LOADING')
      console.log(error)

      this.response = 'Server appears to be offline'
    })
  },
  updated () {
    // alert($('#AllTables').html() + ' -> one 2')
    $('#mainContent table').each(function () {
      var str = '#' + this.id.toString()
      $(str).DataTable()
    })
  },
  methods: {
    getCompanies (groups) {
      api.request('get', '/municipality-types/' + groups[this.count].id)
      .then(response => {
        console.log(response.data)
        var organizations = response.data
        this.groups[this.count].org = []

        for (var i = 0; i < organizations.length; i++) {
          if (organizations[i].companies.length > 0) {
            for (var j = 0; j < organizations[i].companies.length; j++) {
              var company = {
                organization_id: organizations[i].organization_id,
                municipality_id: organizations[i].id,
                organization_type: 'Company',
                SIN: organizations[i].companies[j].SIN
              }
              this.groups[this.count].org.push(company)
            }
          } else {
            var municipality = {
              organization_id: organizations[i].organization_id,
              municipality_id: organizations[i].id,
              organization_type: 'Municipality',
              SIN: null
            }
            this.groups[this.count].org.push(municipality)
          }
        }

        // this.groups[this.count].org = response.data
        this.count++

        if (this.count === this.groups.length) {
          console.log('All companies called')
          console.log(this.groups)

          this.dataReady = true
        } else {
          this.getCompanies(groups)
        }
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
