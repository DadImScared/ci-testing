/*
 * sri sri guru gauranga jayatah
 */
<template>
  <div>
    <h1>TE Allotment Form</h1>
    <form @submit.stop.prevent="allot" v-show="submissionStatus != 'complete'">
      <!-- Track Editor -->
      <div class="form-group row">
        <label for="track-editor" class="col-sm-2 control-label">Track Editor</label>
        <div class="col-col-sm-offset-2 col-sm-10">
          <v-select
            name="track-editor"
            required
            :options="trackEditors"
            v-model="allotment.trackEditor"
            label="name"
          >
            <template slot="option" slot-scope="option">
              <div>
                <div class="row">
                  <div class="col-md-4">
                    <strong>{{option.name}}</strong>
                  </div>
                  <div class="col-md-4">{{option.emailAddress}}</div>
                </div>
              </div>
            </template>
          </v-select>
        </div>
      </div>
      <!-- List -->
      <div class="form-group row">
        <label class="col-sm-2 control-label">List</label>
        <div class="col-sm-10">
          <template v-if="lists">
            <div class="form-check form-check-inline" v-for="list in lists" :key="list">
              <input
                class="form-check-input"
                type="radio"
                v-model="filter.list"
                :value="list"
                name="list"
                :id="'list' + list"
                required
              >
              <label class="form-check-label" :for="'list' + list">{{ list }}</label>
            </div>
          </template>
          <p class="form-control-static" v-else>Loading…</p>
        </div>
      </div>
      <!-- Tasks -->
      <div id="tasks" class="form-group row" v-show="tasks != null || filter.list">
        <label for="tasks" class="col-sm-2 control-label">Tasks</label>
        <div class="col-sm-10" v-if="tasks">
          <div class="row mb-2" v-for="task in tasks" :key="task.id">
            <div class="col-md-3">
              <div class="form-check">
                <input
                  class="form-check-input"
                  type="checkbox"
                  v-model="allotment.tasks"
                  :id="'task-' + task.id"
                  :value="task"
                >
                <label class="form-check-label" :for="'task-' + task.id">{{ task.id }}</label>
              </div>
              <span class="badge badge-danger" v-if="task.sourceFiles.length === 0">No source files</span>
            </div>
            <div class="col-md-3">{{ task.action }} {{ task.language }}</div>
            <div class="col-md-6">
              <pre class="p-1 shadow-sm">{{ task.definition }}</pre>
            </div>
          </div>
          <span v-if="tasks.length === 0">No tasks in this list</span>
        </div>
        <span class="form-control-static col-sm-10" v-else>Loading…</span>
      </div>
      <!-- Comment -->
      <div class="form-group row">
        <label for="comment" class="col-sm-2 control-label">Comment</label>
        <div class="col-sm-10">
          <textarea v-model="allotment.comment" class="form-control" rows="3"></textarea>
        </div>
      </div>
      <!-- Buttons -->
      <div class="form-group row">
        <div class="col-sm-offset-2 col-sm-10">
          <button
            type="submit"
            class="btn btn-primary"
            :disabled="submissionStatus === 'inProgress'"
          >
            Allot
            <span
              class="glyphicon glyphicon-refresh rotate"
              aria-hidden="true"
              v-show="submissionStatus === 'inProgress'"
            ></span>
          </button>
        </div>
      </div>
    </form>
    <div
      class="alert alert-success col-sm-offset-2 col-sm-10"
      role="alert"
      v-show="submissionStatus === 'complete'"
    >
      <h4 class="alert-heading">TFC allotted succesfully</h4>
      <p class="mb-0">
        <button type="button" class="btn btn-link" @click="reset">Make another allotment</button>
      </p>
    </div>
  </div>
</template>

<style>
#tasks pre {
  background-color: #f9f2f4;
  color: #c7254e;
}
</style>

<script>
export default {
  name: "TE",
  data: () => ({
    trackEditors: [],
    lists: null,
    tasks: null,
    filter: {
      list: null
    },
    allotment: {
      trackEditor: null,
      tasks: [],
      comment: null
    },
    submissionStatus: null
  }),
  mounted: function() {
    // Getting TEs
    this.$http
      .jsonp(process.env.VUE_APP_SCRIPT_URL, {
        params: { path: "te/devotees", role: "TE" }
      })
      .then(response => {
        this.trackEditors = response.body;
      });

    // Getting lists
    this.$http
      .jsonp(process.env.VUE_APP_SCRIPT_URL, { params: { path: "te/lists" } })
      .then(response => {
        this.lists = response.body;
      });
  },
  methods: {
    allot() {
      this.submissionStatus = "inProgress";
      this.$http
        .post(process.env.VUE_APP_TE_ALLOTMENT_URL, this.allotment)
        .then(
          () => {
            this.submissionStatus = "complete";
          },
          response => {
            alert(response.text());
          }
        );
    },
    reset() {
      Object.assign(this.$data.allotment, this.$options.data().allotment);
      Object.assign(this.$data.filter, this.$options.data().filter);
      this.submissionStatus = null;
    }
  },
  watch: {
    filter: {
      deep: true,
      handler: function() {
        this.tasks = null;
        this.allotment.tasks = [];

        if (this.filter.list == null) return;

        this.$http
          .jsonp(process.env.VUE_APP_SCRIPT_URL, {
            params: {
              path: "te/tasks",
              list: this.filter.list
            }
          })
          .then(response => {
            this.tasks = response.body;
          });
      }
    }
  }
};
</script>
