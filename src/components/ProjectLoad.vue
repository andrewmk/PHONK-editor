<template>
  <div id="project-load-container" class="editor_panel panel_above">
    <div class="container">
      <span class="debug" v-if="false">
        {{
        store.state.projects["playground"]
        }}
      </span>

      <div>
        <h3>Create a project</h3>
        <project-new></project-new>
      </div>

      <div class="section">
        <h3>Load a project</h3>
          <div v-show="showingAction" class="actionable">
            <!-- <button>rename</button>-->
            <transition name="scaleanim" mode="out-in">

              <div v-if="showingAction === 'general'" key="general" class = "confirmation">
                <button v-on:click="setAction('clone')">clone</button>
                <button v-on:click="setAction('rename')">rename</button>
                <button v-on:click="setAction('delete')">delete</button>
                <button v-on:click="cancelActions">cancel</button>
              </div>
              <div v-else-if = "showingAction === 'delete'" class="confirmation" key="delete">
                <p>Are you sure?</p>
                <button v-on:click="deleteActionSubmit">delete</button>
                <button v-on:click="backActions">cancel</button>
              </div>
              <div v-else-if = "showingAction === 'rename'" class = "confirmation" key = "rename">
                <input v-model = "newName" placeholder="New name..." class = "project-input"/>
                <button v-on:click="renameActionSubmit">rename</button>
                <button v-on:click="backActions">cancel</button>
              </div>
              <div v-else-if = "showingAction === 'clone'" class = "confirmation" key = "clone">
                <input v-model = "newName" placeholder="Clone name..." class = "project-input"/>
                <button v-on:click="cloneActionSubmit">clone</button>
                <button v-on:click="backActions">cancel</button>
              </div>
            </transition>
          </div>

        <div v-if="store.state.projects" id="project-load">
          <div class="left">
            <div class="project_list" v-for="(p, pindex) in projectsOrdered">
              <h1>{{ pindex }}</h1>
              <ul>
                <li
                  v-for="(f, index) in p"
                  v-bind:class="{
                    selected: selected == index && pselected == pindex,
                  }"
                  v-on:click="choose_folder(pindex, index, $event)"
                  v-bind:id="f.name"
                >
                  <span class="icon">
                    <i class="material-icons transparent">folder</i>
                  </span>
                  {{ f.name }}
                </li>
              </ul>
            </div>
          </div>
          <div class="right">
            <div class="project_info">
              <p>Double click to open</p>
              <div class="img-cover"></div>
              <div class="actions">
                <div class="action-element"></div>
                <div class="action-element"></div>
                <div class="action-element"></div>
              </div>
            </div>
            <ul v-if="pselected !== -1">
              <li
                v-bind:class="{ selected: actionOnProject === f }"
                v-for="f in folder_chosen"
                v-on:click="load_project(f)"
                class="project_item"
              >
                <span class="icon">{{ f.name.substr(0, 2) }}</span>
                <span>{{ f.name }}</span>
                <i v-on:click.stop.prevent="openActions(f)" class="action material-icons">more_vert</i>
              </li>
            </ul>
          </div>
        </div>
        <div v-else class="no-projects">
          <p>There is a problem loading your projects</p>
        </div>
      </div>
    </div>
    <!-- container -->
  </div>
</template>

<script>
import Store from '../Store'
import _ from 'lodash'
import ProjectNew from './ProjectNew'

export default {
  components: {
    ProjectNew
  },
  name: 'ProjectLoad',
  data () {
    return {
      store: Store,
      id: Store.state.id,
      pselected: -1,
      selected: -1,
      uri: {
        type: '',
        folder: '',
        fullpath: ''
      },
      showingAction: null,
      actionOnProject: null,
      newName: ''
    }
  },
  computed: {
    folder_chosen: function () {
      let files = this.store.state.projects[this.pselected][this.selected]
        .files
      return _.orderBy(files, [(files) => files.name.toLowerCase()], ['asc'])
    },
    projectsOrdered: function () {
      // return _.sortKeysBy(this.store.state.projects)
      // const ordered = {}
      /*
      Object.keys(this.store.state.projects).sort().forEach((key) => {
        ordered[key] = this.store.state.projects[key]
      });
      */
      const ordered = {
        playground: this.store.state.projects['playground'],
        examples: this.store.state.projects['examples']
      }
      return ordered
    }
  },
  methods: {
    new_project: function () {
      return true
    },
    choose_folder: function (pindex, index, event) {
      // console.log(pindex, index, this.projects[pindex].files[index].name)
      this.pselected = pindex
      this.selected = index

      this.uri.type = pindex
      this.uri.folder = this.store.state.projects[pindex][index].name

      // console.log(this.uri.type, this.uri.folder)
      // console.log(this.state.projects[pindex][index].files)
    },
    load_project: function (project) {
      // this.uri.fullpath = this.uri.folder + '/' + folder.name
      // Store.emit('project_action', '/run', this.uri.fullpath)
      // Store.emit('project_load', this.uri.fullpath)
      // console.log(this.uri.fullpath, this.uri.type, this.uri.folder, project.name)

      this.close()
      var to = {
        name: 'editor.load',
        params: {
          type: this.uri.type,
          folder: this.uri.folder,
          project: project.name
        }
      }
      this.$router.push(to)
    },
    // load from app
    load_project_from_app: function (data) {
      console.log('loading... ' + data.type)
      var to = {
        name: 'editor.load',
        params: { type: data.type, folder: data.folder, project: data.name }
      }
      this.$router.push(to)
    },
    close: function () {
      this.store.state.show_load_project = false
      Store.emit('toggle', 'load_project')
    },
    openActions: function (f) {
      // console.log('openActions', f)
      this.showingAction = 'general'
      this.actionOnProject = f
    },
    cloneAction: function () {
    },
    setAction: function (action) {
      this.showingAction = action
    },
    deleteActionSubmit: function () {
      console.log('deleteSubmit')
      Store.project_delete(this.actionOnProject.path)
    },
    renameActionSubmit: function () {
      console.log('renameSubmit')

      if (this.newName !== '') {
        Store.project_rename(this.actionOnProject.path, this.newName)
        this.backActions()
      }
    },
    cloneActionSubmit: function () {
      console.log('cloneSubmit')

      if (this.newName !== '') {
        Store.project_clone(this.actionOnProject.path, this.newName)
        this.backActions()
      }
    },
    backActions: function () {
      this.showingAction = 'general'
      this.newName = ''
    },
    cancelActions: function () {
      this.showingAction = null
      this.actionOnProject = null
      this.newName = ''
    }
  },
  mounted () {
    this.cancelActions()
  },
  created () {
    Store.on('new_project', this.new_project)
    Store.on('load_project_from_app', this.load_project_from_app)
  },
  destroyed () {
    Store.removeListener('new_project', this.new_project)
    Store.removeListener('load_project_from_app', this.load_project_from_app)
  }
}
</script>

<style lang="less">
@import (reference) '../assets/css/variables.less';
@import (reference) '../assets/css/hacks.less';

#project-load-container {
  display: flex;
  flex-direction: column;
  margin-bottom: 12px;
  padding: 0px;
  font-size: 1rem;
  color: #222;
  overflow: auto;
  z-index: 5;

  .container {
    max-width: 1000px;
    margin: 0 auto;
    width: 100%;
  }

  h3,
  #editor_panel_new,
  #project-load {
    margin: 8px 12%;
  }

  .project-input {
    flex: 2;
    width: 100%;
    min-width: 20%;
    outline: none;
    border: none;
    width: 100%;
    box-sizing: border-box;
    color: black;
    font-size: 1rem;
    background: white;
    padding: 10px;
    border: 1px solid @accentColor_1;
  }

  h3 {
    // text-transform: uppercase;
    font-weight: 400;
    font-size: 0.8rem;
    color: @primaryTextColor;
    margin-top: 50px;
    margin-bottom: 15px;
  }

  .debug {
    font-size: 0.7em;
  }

  .section {
    position: relative;

    .actionable {
      position: absolute;
      top: -10px;
      right: 12%;
      display: flex;
      flex-direction: row;
      align-items: center;
      height: 38px;
      overflow: hidden;
      justify-content: end;

      button {
        padding: 12px 8px;
      }

      p {
        padding: 5px;
      }

      .confirmation {
        align-items: center;
        // background: @accentColor;
        color: @accentColor;
        display: flex;

        input {
          // color: white;
        }
      }
    }
  }
}

#project-load {
  display: flex;
  flex-direction: row;
  border: 1px solid @secondaryColor;
  font-size: 1em;
  font-weight: 100;
  // max-height: 100%;
  height: calc(100vh - 288px);

  .no-projects {
    p {
      font-size: 1.5em;
      line-height: 1.3em;
      font-weight: 600;
      padding: 2em;
      text-align: center;
    }
  }

  ul {
    text-align: left;
    list-style: none;
    margin: 0;
    padding: 0;
    cursor: pointer;
    user-select: none;

    li {
      color: @primaryTextColor;
      padding: 5px 10px;
      font-weight: 400;
      display: flex;
      line-height: 1.2em;
      align-items: center;

      .icon {
        max-width: 30px;
        min-width: 30px;
        height: 30px;
        border: 1px solid @secondaryColor;
        color: @accentColor;
        font-weight: 800;
        display: flex;
        justify-content: center;
        align-content: center;
        align-items: center;
        margin-right: 10px;
        padding: 5px;
      }

      span {
        flex: 2;
      }

      &:hover,
      &.selected {
        color: @accentColor;
        border-radius: 1px;
        position: relative;

        .action {
          display: block;
          margin: 0;
          position: absolute;
          padding: 12px;
          right: 0px;
          // background: rgba(255, 255, 255, 0.5);
        }
      }

      &.selected {
        color: @secondaryTextColor;
        background: darken(@accentColor, 10%);

        .icon {
          color: @backgroundColor;
          border-color: @backgroundColor;
        }
      }
    }
  }

  .left {
    .scrollbar;

    h1 {
      color: @accentColor;
      font-weight: 600;
      text-transform: uppercase;
      font-size: 0.7em;
      padding: 18px 18px;
      margin-bottom: 0px;

      // border-bottom: 0px solid @accentColor;
    }

    .project_list {
      margin-bottom: 22px;
    }
  }
  .right {
    text-align: center;
    border-left: 1px solid @secondaryColor;
    position: relative;
    overflow-y: auto;
    .scrollbar;

    .action {
      display: none;
      opacity: 1;
      position: sticky;
      right: -10px;
      color: white;
      padding: 2px 10px;

      &:hover {
        color: darken(@accentColor, 20%);
      }
    }

    .project_info {
      display: none;
    }

    .img-cover {
      width: 100px;
      height: 100px;
      background-color: blue;
    }

    .actions {
      margin-top: 10px;

      .action-element {
        width: 20px;
        height: 20px;
        background-color: green;
        display: inline-block;
      }
    }

    ul {
      font-weight: 500;
      height: 100%;
    }
  }

  #new_folder {
    font-size: 0.8em;
    border: 1px dotted #bbbbbb;
    display: inline-block;
    border-radius: 2px;
    margin-left: 10px;
    margin-top: 28px;
    color: #bbbbbb;
    padding: 8px;
    text-transform: uppercase;
    font-weight: 600;

    &:hover {
      color: #fff;
      border-color: transparent;
    }

    i {
      margin-right: 5px;
    }
  }
}
</style>
