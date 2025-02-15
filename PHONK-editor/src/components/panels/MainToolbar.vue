<template>
  <div id="toolbar">
    <div class="left_side">
      <phonk-logo></phonk-logo>
    </div>

    <div class="central_side">
      <div class="project_actions">
        <transition name="upanim" mode="out-in">
          <button
            class="clean"
            key="show"
            v-if="!sharedState.show_load_project"
            v-on:click="toggle_load_project"
          >
            <p v-if="not_loaded">
              <span class="folder">/{{ sharedState.current_project.project.folder }}/</span>
              <span class="name">
                {{ sharedState.current_project.project.name }}
              </span>
            </p>
            <p v-else>
              <!--
              <span title="Open project" class="material-icons">folder</span>
              -->
              <span>load project</span>
            </p>
            <i class="material-icons closer">arrow_drop_down</i>
          </button>

          <button class="clean" key="hide" v-else v-on:click="toggle_load_project">
            <p class="name">back to editor</p>
            <i class="material-icons closer">arrow_drop_up</i>
          </button>
        </transition>
      </div>

      <!--
      <transition name = "banneranim" mode = "out-in">
        <div class = "dashboard" v-if = "sharedState.show_dashboard" key = "dashboard">
          <h1>Dashboard</h1>
        </div>
      </transition>
      -->
    </div>

    <div class="right_side">
      <div class = "app_info_msg_wrapper">
        <transition-group name="info-anim" mode="out-in" tag="div">
          <div
            v-for = "infoMsg in infoQueue"
            class="app_info_msg info-anim-item"
            :class = "{
              'progress': infoMsg['status'] === 'progress',
              'done': infoMsg['status'] === 'done',
              'error': infoMsg['status'] === 'error' }"
            :key="infoMsg.id">
              <span class="icon_left material-icons">{{ infoMsg.icon }}</span>
              <span class = "msg">{{ infoMsg.text }}</span>
          </div>
        </transition-group>
      </div>

      <!--
      <button class="material-icons icon" v-show="true" v-on:click="toggle_dashboard">dashboard</button>
      -->
      <button
        class="material-icons icon"
        v-bind:class="{
          rotate: is_rotated,
          enabled: sharedState.show_device_info,
          device_disabled: !sharedState.device_properties.connected,
        }"
        v-on:click="toggle_device_info"
      >phone_android</button>
      <button
        class="material-icons icon"
        v-on:click="sharedState.show_preferences = !sharedState.show_preferences"
        v-bind:class="{ enabled: sharedState.show_preferences }"
      >settings</button>
    </div>

    <transition name="downanim">
      <device-info-popover v-show="sharedState.show_device_info"></device-info-popover>
    </transition>

    <transition name="downanim">
      <preferences-popover v-show="sharedState.show_preferences"></preferences-popover>
    </transition>
  </div>
</template>

<script>
import Store from '../../Store'
import PhonkLogo from '../views/PhonkLogo'
import DeviceInfoPopover from '../popovers/DeviceInfoPopover'
import PreferencesPopover from '../popovers/PreferencesPopover.vue'

export default {
  name: 'MainToolbar',
  components: {
    PhonkLogo,
    DeviceInfoPopover,
    PreferencesPopover
  },
  data () {
    return {
      navitem: '',
      project: '',
      isConnected: false,
      isError: false,
      runShortcut: false,
      saveAsShortcut: false,
      sharedState: Store.state,
      infoQueue: [],
      isProjectRunning: false,
      device_properties: {}
    }
  },
  methods: {
    toggle_load_project: function () {
      this.sharedState.show_load_project = !this.sharedState.show_load_project
      Store.emit('toggle_load_project')
    },
    toggle_dashboard: function () {
      // Store.emit('toggle_dashboard')
      this.sharedState.show_dashboard = !this.sharedState.show_dashboard
      // console.log('toggling dashboard')
    },
    toggle_device_info: function () {
      this.sharedState.show_device_info = !this.sharedState.show_device_info
    },
    project_saved: function () {
      // this.show_info({ icon: 'save', text: 'Saved' })
    },
    show_info (msg) {
      // console.log('msg', msg)
      // check if id exists
      let obj = this.infoQueue.find(o => o.id === msg.id)

      if (obj) {
        setTimeout(() => {
          obj.text = msg.text
          obj.status = msg.status
        }, 500)
        setTimeout(() => {
          this.infoQueue = this.infoQueue.filter(el => {
            return el.id !== obj.id
          })
        }, 2000)
      } else {
        let infoMsg = {
          id: msg.id,
          icon: msg.icon,
          text: msg.text,
          status: msg.status
        }
        this.infoQueue.push(infoMsg)

        // if timesout => show error => remove
        setTimeout(() => {
          infoMsg.isShowing = false
          infoMsg.status = 'error'

          setTimeout(() => {
            this.infoQueue = this.infoQueue.filter(el => {
              return el.id !== infoMsg.id
            })
          }, 2000)
        }, 10000)
      }
    },
    device_update: function (data) {
      if (typeof data.info !== 'undefined') {
        this.device_properties = data
      } else {
        this.device_properties.connected = false
      }
    }
  },
  computed: {
    is_rotated: function () {
      return (
        this.sharedState.device_properties.info.screen.orientation ===
        'landscape'
      )
    },
    not_loaded: function () {
      if (!this.sharedState.current_project.project.folder) return false
      else return true
    }
  },
  created () {
    Store.on('project_saved', this.project_saved)
    Store.on('device', this.device_update)
    Store.on('show_info', this.show_info)
    // Store.on('toggle_load_project', this.toggle_load_project)
    Store.on('toggle_device_info', this.toggle_device_info)
  },
  destroyed () {
    Store.removeListener('project_saved', this.project_saved)
    Store.removeListener('device', this.device_update)
    Store.removeListener('show_info', this.show_info)
    // Store.removeListener('toggle_load_project', this.toggle_load_project)
    Store.removeListener('toggle_device_info', this.toggle_device_info)
  }
}
</script>

<style lang="less">
@import (reference) '../../assets/css/variables.less';

#toolbar {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  align-items: center;
  justify-content: space-between;
  color: var(--color-text-light);
  user-select: non;
  z-index: 2;
  height: 52px;
  font-size: 1.2em;
  border-bottom: 1px solid var(--color-lines);

  > * {
    flex: 1;
    display: inline-flex;
    align-self: center;
    align-items: center;
    padding: 10px;
  }

  .folder {
    color: var(--color-text-light-faded);
  }

  .name {
    color: var(--color-accent);
    display: inline-block;
  }

  // statuses
  // showing query (show waiting icon) => showing result (show tick result or convert to red if fail)
  // save / load / trying to reconnect

  .app_info_msg_wrapper {
    position: absolute;
    z-index: 20;
    right: 12px;
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    top: 9px;

    & > div {
      display: flex;
      flex-direction: column;
      align-items: flex-end;
    }
  }

  .app_info_msg {
    .font-mono-400;
    font-size: 0.7em;
    margin-bottom: 10px;
    padding: 5px 15px;
    border-radius: 20px;
    display: inline-flex;
    align-items: center;

    font-size: 0.8rem;

    &.progress {
      background: var(--color-text-light);
      color: var(--color-text-dark);
    }

    &.done {
      background: var(--color-accent);
      color: var(--color-text-dark);
    }

    &.error {
      background: var(--color-error);
    }

    .icon_left,
    .icon_right {
      padding-right: 10px;
    }

    .msg {
      white-space: nowrap;
      text-overflow: ellipsis;
      // overflow: hidden;
    }
  }

  .transparent {
    background: var(--color-transparent);

    &:hover {
    }

    &:active {
    }
  }

  .left_side {
    flex: 1;
  }

  .central_side {
    flex: 2;
    justify-content: center;
  }

  .right_side {
    flex: 1;
    justify-content: flex-end;

    button {
      padding: 8px 8px;
      // transition: all 0.3s ease-in-out;

      &.rotate {
        transform: rotate3d(0, 0, 1, 90deg);
      }

      &.rotate * {
        transform: translate3d(0px, 3px, 0px);
      }

      &.enabled {
        color: var(--color-accent);
      }

      &.device_disabled {
        opacity: 0.3;
      }
    }
  }

  .dashboard {
    display: inline-flex;
    margin: 5px;

    h1 {
      text-align: center;
      padding: 6px 15px;
      width: 100%;
      text-transform: uppercase;
    }
  }

  .project_actions {
    margin: 5px;

    button {
      display: inline-flex;
      align-items: center;
      .font-mono-400;
    }

    p {
      text-transform: none;
      flex: 2;
      padding: 0px 5px;
      font-size: 1em;
      text-overflow: ellipsis;
      overflow: hidden;
      white-space: nowrap;
      display: inline-flex;
    }

    .closer {
      margin-left: -5px;
    }
  }
}
</style>
