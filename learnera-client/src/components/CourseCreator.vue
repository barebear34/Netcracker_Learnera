<template>
  <v-container>
    <v-layout row wrap>
      <v-flex d-flex justify-center align-center xs8>
        <h3 class="headline">Create course based on template </h3>
      </v-flex>
      <v-flex d-flex justify-center align-center xs4>
        <v-select
          :items="userTemplatesSelection"
          v-model="selectedTemplate"
          label="Select template"
        />
      </v-flex>
      <v-flex xs12>
        {{ selectedTemplate + ' ' + courseName + ' ' + courseDescription }}
      </v-flex>
      <v-flex xs12>
        <v-text-field v-model="courseName" label="Course name"/>
      </v-flex>
      <v-flex xs12>
        <v-layout row>
          <v-flex>
            <v-avatar size="100" color="teal">
              <span>TODO</span>
            </v-avatar>
            <v-btn>Load avatar</v-btn>
          </v-flex>
        </v-layout>
      </v-flex>
      <v-flex style="margin: 15px 0 0 0">
        <v-textarea v-model="courseDescription" box label="Description" auto-grow/>
      </v-flex>
      <v-flex xs12>
        <v-card>
          <v-card-title><h3 class="headline mb-0">Deadlines</h3></v-card-title>
          <v-responsive>
            <v-layout column>
              <template v-for="weekDateElem in weekDateElems">
                <v-layout :key="weekDateElem.week.id" row>
                  <v-flex d-flex justify-center align-center xs2 style="margin: 0 0 0 0.5em">
                    <v-chip>{{ weekDateElem.week.name || 'Week ' + weekDateElem.week.weekNumber }}</v-chip>
                  </v-flex>
                  <v-flex d-flex justify-center align-center style="margin: 0 1em 0 1em">
                    <v-menu
                      :close-on-content-click="false"
                      v-model="weekDateElem.startMenu"
                      :nudge-right="40"
                      lazy
                      transition="scale-transition"
                      offset-y
                      min-width="250px"
                    >
                      <v-text-field
                        slot="activator"
                        v-model="weekDateElem.startDate"
                        label="Start date"
                        readonly/>
                      <v-date-picker v-model="weekDateElem.startDate" @input="weekDateElem.startMenu = false"/>
                    </v-menu>
                  </v-flex>
                  <v-flex d-flex justify-center align-center style="margin: 0 1em 0 1em">
                    <v-menu
                      :close-on-content-click="false"
                      v-model="weekDateElem.endMenu"
                      :nudge-right="40"
                      lazy
                      transition="scale-transition"
                      offset-y
                      min-width="250px"
                    >
                      <v-text-field
                        slot="activator"
                        v-model="weekDateElem.endDate"
                        label="End date"
                        readonly/>
                      <v-date-picker v-model="weekDateElem.endDate" @input="weekDateElem.endMenu = false"/>
                    </v-menu>
                  </v-flex>
                </v-layout>
              </template>
            </v-layout>
          </v-responsive>
        </v-card>
      </v-flex>
      <v-flex xs12 style="margin: 15px 0 0 0">
        <v-card>
          <v-card-title><h3 class="headline mb-0">Group management</h3></v-card-title>
          <v-responsive>
            <v-container>
              <v-layout row>
                <v-flex d-flex align-center justify-center>
                  <v-layout column>
                    <v-flex>
                      <h6 class="title">All groups</h6>
                    </v-flex>
                    <v-flex>
                      TODO: SHOW GROUPS
                    </v-flex>
                  </v-layout>
                </v-flex>
                <v-divider class="mx-2" vertical/>
                <v-flex>
                  <v-layout column>
                    <v-flex>
                      <h6 class="title">Participants</h6>
                    </v-flex>
                    <v-flex>
                      TODO: SHOW GROUPS
                    </v-flex>
                  </v-layout>
                </v-flex>
              </v-layout>
            </v-container>
          </v-responsive>
        </v-card>
      </v-flex>
      <v-flex xs12 style="margin: 15px 0 0 0">
        <v-card>
          <v-card-title><h3 class="headline mb-0">Confirm your action</h3></v-card-title>
          <v-responsive>
            <v-layout row>
              <v-flex style="margin: 0 10px 0 10px" xs6>
                <v-btn color="primary" block @click="onCourseCreate">Create course</v-btn>
              </v-flex>
              <v-flex style="margin: 0 10px 0 10px" xs6>
                <v-btn color="secondary" block @click="$router.back()">Cancel</v-btn>
              </v-flex>
            </v-layout>
          </v-responsive>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import {mapState, mapActions, mapGetters} from 'vuex';
import {store} from '../store'
import {router} from '../router'

export default {
  name: 'CourseCreator',
  data() {
    return {
      selectedTemplate: {},
      courseName: '',
      courseDescription: '',
      weekDateElems: [],
    };
  },
  computed: {
    ...mapState('account', ['user']),
    userTemplates: function() {
      return this.user.templates;
    },
    userTemplatesSelection: function() {
      return this.userTemplates.map(t => {return {text: t.name, value: t.id}; })
    },
    currentTemplate: function() {
      return this.userTemplates.find(t => t.id === this.selectedTemplate);
    },
    weeks: function() {
      if (!this.currentTemplate) {
        return undefined;
      }
      console.log(JSON.stringify(this.currentTemplate.weeks));
      return this.currentTemplate.weeks;
    }
  },
  watch: {
    user: function(val) {
      if (val && val.id) {
        this.getByTeacherId(val.id);
      }
    },
    weeks: function(val) {
      if (!val) {
        this.weekDateElems = [];
        return;
      }

      this.weekDateElems = val.map(w => { return {
        week: w,
        startDate: new Date().toISOString().substr(0, 10), 
        endDate: new Date().toISOString().substr(0, 10),
        startMenu: false,
        endMenu: false
      };});
    }
  },
  beforeMount() {
  },
  methods: {
    ...mapActions('courses', {
      createCourse: 'create'
    }),
    onCourseCreate() {
      // selectedTemplate: {},
      // courseName: '',
      // courseDescription: '',
      // weekDateElems: [],

      const course = {
        template: {id: this.currentTemplate.id},
        name: this.courseName,
        description: this.courseDescription,
        startDate: this.weekDateElems[0].startDate,
        endDate: this.weekDateElems[this.weekDateElems.length - 1].endDate,
        weekDates: this.weekDateElems.map(wde => {
          const {week, startDate, endDate} = wde;
          return {
            id: {
              weekId: week.id
            },
            week: {
              id: week.id
            },
            startDate,
            endDate
          };
        }),
        avatar: null, // TODO
        passPercent: 60, // TODO
        goodPercent: 75, // TODO
        excellentPercent: 90, // TODO
        groups: null, // TODO
      };

      this.createCourse(course).then(course => {
        // TODO: IMPLEMENT NOTIFICATION
        router.push(`/course/${course.id}`);
      }).catch(e => console.error);
    }
  },
}
</script>

<style>
.ava {
  font-size: 5em;
  color: white;
  text-align: center;
}

.mainInfo {
  padding-left: 0.4em;
}

.head {
  font-size: 1.5em;
  color: black;
}

</style>