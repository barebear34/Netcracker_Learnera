<template>
  <v-container v-if="template">
    <v-layout row wrap>
      <v-flex d-flex justify-center align-center xs12>
        <h3 class="headline">{{ readOnly ? 'View' : 'Edit' }} template</h3>
      </v-flex>
      <v-flex xs12>
        <v-layout align-start justify-start row>
          <v-avatar size="100" color="teal">
            <img v-if="template.info && template.info.avatar" :src="template.info.avatar">
            <span v-else-if="template.name" class="ava">{{ template.name[0] }}</span>
          </v-avatar>
          <v-flex>
            <v-layout column>
              <v-layout justify-space-between row>
                <v-flex xs2 style="margin-left: 5px" fill-height>
                  <v-text-field v-if="!readOnly" v-model="template.name" label="Template name"/>
                  <div v-else class="head">{{ template.name ? template.name : '' }}</div>
                </v-flex>
                <v-spacer/>
                <v-flex fill-height xs2>
                  <div>
                    <v-btn v-if="teacher" :to="`/user/${teacher.id}`" color="primary">
                      Teacher: {{ teacherName }}
                    </v-btn>
                  </div>
                </v-flex>
              </v-layout>
              <v-flex>
                <v-layout justify-space-between row>
                  <v-flex fill-height="" xs2>
                    <v-btn v-if="!readOnly" small>
                      Update avatar
                    </v-btn>
                  </v-flex>
                  <v-spacer/>
                  <v-flex fill-height xs2 justify-end>
                    <v-chip :color="(template.completed ? 'orange' : 'green') + ' darken-3'" small text-color="white">
                      {{ template.completed ? 'FINAL' : 'EDITABLE' }}
                    </v-chip>
                  </v-flex>
                </v-layout>
              </v-flex>
            </v-layout>
          </v-flex>
        </v-layout>
      </v-flex>
      <v-flex style="margin: 15px 0 0 0">
        <v-textarea v-model="template.description" :readonly="readOnly" box label="Description" auto-grow/>
      </v-flex>
      <v-flex xs12>
        <v-card>
          <v-card-title><h3 class="headline mb-0">Weeks</h3></v-card-title>
          <v-responsive>
            <v-tabs color="cyan" dark slider-color="yellow">
              <template v-for="(week, i) in weeks">
                <v-tab :key="`tab-w-${i}`" ripple>
                  <v-layout row fill-height>
                    <v-flex d-flex align-center="">
                      <div>
                        {{ week.name ? week.name : 'Week ' + (week.weekNumber + 1) }}
                      </div>
                    </v-flex>
                    <v-flex v-if="!template.completed && !readOnly" d-flex align-center>
                      <div>
                        <v-btn style="margin-right: -5px; margin-left: 5px;" small flat icon 
                               @click.stop="removeWeek(i)">
                          <v-icon color="red darken-4">remove</v-icon>
                        </v-btn>
                      </div>
                    </v-flex>
                  </v-layout>
                </v-tab>
                <v-divider :key="`div-w-${i}`" class="cyan lighten-4" inset vertical/>
              </template>
              
              <v-tooltip v-if="!template.completed && !readOnly" bottom>
                <v-btn slot="activator" icon ripple @click.stop="addWeek">
                  <v-icon color="grey lighten-3">add</v-icon>
                </v-btn>
                <span>Add week</span>
              </v-tooltip>

              <v-tab-item v-for="(week, i) in weeks" :key="`tab-item-${i}`">
                <v-card>
                  <v-tabs color="teal lighten-1" dark slider-color="yellow">
                    <template v-for="(lesson, j) in week.lessons">
                      <v-tab :key="`tab-w-${i}-l-${j}`" ripple>
                        <v-layout row fill-height>
                          <v-flex d-flex align-center="">
                            <div>
                              {{ lesson.name ? lesson.name : ('Lesson ' + (lesson.ordering + 1)) }}
                            </div>
                          </v-flex>
                          <v-flex v-if="!template.completed && !readOnly" d-flex align-center>
                            <div>
                              <v-btn style="margin-right: -5px; margin-left: 5px;" small flat icon
                                     @click.stop="removeLesson({weekIndex: i, lessonIndex: j})">
                                <v-icon color="red darken-4">remove</v-icon>
                              </v-btn>
                            </div>
                          </v-flex>
                        </v-layout>
                      </v-tab>
                      <v-divider :key="`div-w-${i}-l-${j}`" 
                                 class="teal lighten-4" inset vertical/>
                    </template>

                    <v-tooltip v-if="!template.completed && !readOnly" bottom>
                      <v-btn slot="activator" icon ripple @click.stop="addLesson(i)">
                        <v-icon color="grey lighten-3">add</v-icon>
                      </v-btn>
                      <span>Add lesson</span>
                    </v-tooltip>

                    <v-tab-item v-for="(lesson, j) in week.lessons"
                                :key="`tab-item-w-${i}-l-${j}`">
                      <v-card flat>
                        <lesson-editor v-model="template.weeks[i].lessons[j]" :editable="!template.completed && !readOnly"/>
                      </v-card>
                    </v-tab-item>
                  </v-tabs>
                </v-card>
              </v-tab-item>
            </v-tabs>
          </v-responsive>
        </v-card>
      </v-flex>
      <v-flex v-if="!readOnly" xs12 style="margin: 15px 0 0 0">
        <v-card>
          <v-card-title><h3 class="headline mb-0">Confirm your action</h3></v-card-title>
          <v-responsive>
            <v-layout row>
              <v-flex style="margin: 0 10px 0 10px" xs4>
                <v-btn color="primary" block @click="onTemplatePost">Save template</v-btn>
              </v-flex>
              <v-flex style="margin: 0 10px 0 10px" xs4>
                <v-btn :disabled="!template.id || template.completed" color="secondary" block @click="onFinalize">
                  Finalize
                </v-btn>
              </v-flex>
              <v-flex style="margin: 0 10px 0 10px" xs4>
                <v-btn color="error" block @click="$router.back()">
                  Cancel
                </v-btn>
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
import LessonEditor from './base/LessonEditor.vue'

export default {
  name: 'TemplateViewer',
  components: {LessonEditor},
  props: ['templateInput', 'templateIdStr', 'readOnly'],
  data() {
    return {
      template: {
        id: null,
        teacher: null,
        name: '',
        description: '',
        completed: false,
        avatar: null,
        weeks: []
      },
    };
  },
  computed: {
    ...mapState('account', {currentUser: 'user'}),
    ...mapState('users', {
      teacher(state) {
        if (this.template.teacher && this.template.teacher.id) {
          const ret = state.items.find(x => x.id === this.template.teacher.id);
          return ret;
        } else {
          return null;
        }
      }
    }),
    userId: function() {
      return this.currentUser.id;
    },
    weeks: function() {
      if (!this.template) {
        return undefined;
      }
      return this.template.weeks;
    },
    teacherName: function() {
      if (!this.teacher) { return ''; }
      if (this.teacher.info) {
        return this.teacher.info.firstName || this.teacher.info.lastName || this.teacher.email;
      }
      return this.teacher.email;
    }
  },
  watch: {
  },
  beforeMount() {
    if (this.templateInput) {
      this.template = this.templateInput;
      this.template.teacher = {id: this.currentUser.id};
    } else if (this.templateIdStr) {
      this.getTemplate(parseInt(this.templateIdStr)).then(t => {
        this.template = t;
        this.template.teacher = {id: this.template.teacher};
        this.getUser(this.template.teacher.id);
        this.transformTemplate();
      }).catch(e => console.err);
    }
  },
  methods: {
    ...mapActions('templates', {
      createTemplate: 'create',
      updateTemplate: 'update',
      getTemplate: 'get'
    }),
    ...mapActions('lessons', {
      getLessonsByWeekId: 'getByWeekId'
    }),
    ...mapActions('users', {
      getUser: 'get'
    }),
    transformTemplate() {
      if (this.template.weeks) {
        this.template.weeks[0].template = {id: this.template.id};
        this.template.weeks = this.template.weeks.map(w => {
          this.getLessonsByWeekId(w.id).then(ls => {
            if (ls) {
              ls[0].week = {id: w.id}
            }
            w.lessons = ls;
          });
          return w;
        });
        console.log('Template transformed');
      }
    },
    transformForPost() {
      this.template.teacher = {id: this.currentUser.id};
      if (this.template.weeks) {
        this.template.weeks = this.template.weeks.map(w => {
          w.template = null;

          if (w.lessons) {
            w.lessons = w.lessons.map(l => {
              l.week = null;
              l.messages = null;

              if (l.questions) {
                l.questions = l.questions.map(q => {
                  q.assignment = null;
                  q.attempts = null;

                  if (q.variants) {
                    q.variants = q.variants.map(v => {
                      v.question = null;
                      return v;
                    });
                  }

                  return q;
                });
              }
              return l;
            });
          }
          return w;
        });
      }
    },
    onTemplatePost() {
      this.transformForPost();
      if (this.template.id) {
        console.log(JSON.stringify(this.template));

        this.updateTemplate(this.template).then(template => {
          console.log('Template updated!');
          this.$emit('template-updated', template);

          this.template = template;
          this.transformTemplate();
        }).catch(e => console.error);
      } else {
        this.createTemplate(this.template).then(template => {
          console.log('Template created');
          this.$emit('template-created', template);
          // TODO: IMPLEMENT NOTIFICATION

          this.template = template;
          this.transformTemplate();
          this.$router.push(`/template/${template.id}/edit`);
        }).catch(e => console.error);
      }
    },
    onFinalize() {
      this.transformForPost();
      this.template.completed = true;
      this.updateTemplate(this.template).then(template => {
        console.log('Template finalized!');
        this.$emit('template-finalized', template);

        this.template = template;
        this.transformTemplate();
      })
    },
    addWeek() {
      this.template.weeks.push({
        template: this.template.id ? {id: this.template.id} : null,
        weekNumber: this.weeks.length,
        name: '',
        lessons: []
      });
    },
    removeWeek(index) {
      this.template.weeks.splice(index, 1);
      let i = 0;
      this.template.weeks.forEach(w => w.weekNumber = (i++));
    },
    addLesson(weekIndex) {
      this.weeks[weekIndex].lessons.push({
        ordering: this.weeks[weekIndex].lessons.length,
        lectureText: 'HELLO!',
        type: 'lecture'
      });
    },
    removeLesson({weekIndex, lessonIndex}) {
      this.weeks[weekIndex].lessons.splice(lessonIndex, 1);
      let i = 0;
      this.weeks[weekIndex].lessons.forEach(l => l.ordering = (i++));
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