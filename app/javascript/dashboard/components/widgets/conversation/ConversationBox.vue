<template>
  <div class="conversation-details-wrap">
    <conversation-header
      v-if="currentChat.id"
      :chat="currentChat"
      :is-contact-panel-open="isContactPanelOpen"
      @contact-panel-toggle="onToggleContactPanel"
    />
    <div class="tabs-container">
      <button class="tab-scroll-button scroll-left" @click="slide('left')">
        <fluent-icon icon="chevron-left" size="16" />
      </button>
      <woot-tabs
        v-if="dashboardApps.length && currentChat.id"
        ref="dashboardAppsTabs"
        :index="activeIndex"
        class="dashboard-app--tabs"
        @change="onDashboardAppTabChange"
      >
        <woot-tabs-item
          v-for="tab in dashboardAppTabs"
          :key="tab.key"
          :name="tab.name"
          :show-badge="false"
        />
      </woot-tabs>
      <button class="tab-scroll-button scroll-right" @click="slide('right')">
        <fluent-icon icon="chevron-right" size="16" />
      </button>
    </div>
    <div v-if="!activeIndex" class="messages-and-sidebar">
      <messages-view
        v-if="currentChat.id"
        :inbox-id="inboxId"
        :is-contact-panel-open="isContactPanelOpen"
        @contact-panel-toggle="onToggleContactPanel"
      />
      <empty-state v-else />
      <div v-show="showContactPanel" class="conversation-sidebar-wrap">
        <contact-panel
          v-if="showContactPanel"
          :conversation-id="currentChat.id"
          :inbox-id="currentChat.inbox_id"
          :on-toggle="onToggleContactPanel"
        />
      </div>
    </div>
    <dashboard-app-frame
      v-else
      :key="currentChat.id"
      :config="dashboardApps[activeIndex - 1].content"
      :current-chat="currentChat"
    />
  </div>
</template>
<script>
import { mapGetters } from 'vuex';
import ContactPanel from 'dashboard/routes/dashboard/conversation/ContactPanel';
import ConversationHeader from './ConversationHeader';
import DashboardAppFrame from '../DashboardApp/Frame.vue';
import EmptyState from './EmptyState';
import MessagesView from './MessagesView';

export default {
  components: {
    ContactPanel,
    ConversationHeader,
    DashboardAppFrame,
    EmptyState,
    MessagesView,
  },

  props: {
    inboxId: {
      type: [Number, String],
      default: '',
      required: false,
    },
    isContactPanelOpen: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return { activeIndex: 0 };
  },
  computed: {
    ...mapGetters({
      currentChat: 'getSelectedChat',
      dashboardApps: 'dashboardApps/getRecords',
    }),
    dashboardAppTabs() {
      return [
        {
          key: 'messages',
          name: this.$t('CONVERSATION.DASHBOARD_APP_TAB_MESSAGES'),
        },
        ...this.dashboardApps.map(dashboardApp => ({
          key: `dashboard-${dashboardApp.id}`,
          name: dashboardApp.title,
        })),
      ];
    },
    showContactPanel() {
      return this.isContactPanelOpen && this.currentChat.id;
    },
  },
  watch: {
    'currentChat.inbox_id'(inboxId) {
      if (inboxId) {
        this.$store.dispatch('inboxAssignableAgents/fetch', [inboxId]);
      }
    },
    'currentChat.id'() {
      this.fetchLabels();
    },
  },
  mounted() {
    this.fetchLabels();
    this.$store.dispatch('dashboardApps/get');
  },
  methods: {
    fetchLabels() {
      if (!this.currentChat.id) {
        return;
      }
      this.$store.dispatch('conversationLabels/get', this.currentChat.id);
    },
    onToggleContactPanel() {
      this.$emit('contact-panel-toggle');
    },
    onDashboardAppTabChange(index) {
      this.activeIndex = index;
    },
    slide(direction) {
      var container = this.$refs.dashboardAppsTabs;
      let scrollCompleted = 0;
      // eslint-disable-next-line func-names
      var slideVar = setInterval(function() {
        if (direction === 'left') {
          container.scrollLeft -= 10;
        } else {
          container.scrollLeft += 10;
        }
        scrollCompleted += 10;
        if (scrollCompleted >= 100) {
          window.clearInterval(slideVar);
        }
      }, 5);
    },
  },
};
</script>
<style lang="scss" scoped>
@import '~dashboard/assets/scss/woot';

.conversation-details-wrap {
  display: flex;
  flex-direction: column;
  min-width: 0;
  width: 100%;
  border-left: 1px solid var(--color-border);
  background: var(--color-background-light);
}

.dashboard-app--tabs {
  background: var(--white);
  margin-top: -1px;
  flex-shrink: 0;
  display: flex;
  overflow-x: auto;
  padding: 1px 2.4rem;
  &::-webkit-scrollbar {
    display: none;
  }
  .tabs-title {
    flex-shrink: 0;
  }
}

.tabs-container {
  position: relative;
  .tab-scroll-button {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    height: 100%;
    padding: 0 0.5rem;
    background-color: #fff;
  }
  .scroll-left {
    left: 0;
    z-index: 999999999999999999;
  }
  .scroll-right {
    right: 0;
    z-index: 999999999999999999;
    &:before {
      content: '';
      background: linear-gradient(to left, #fff 20%, rgba(33, 33, 33, 0) 80%);
      height: 38px;
      width: 10px;
      pointer-events: none;
    }
  }
}

.messages-and-sidebar {
  display: flex;
  background: var(--color-background-light);
  margin: 0;
  height: 100%;
  min-height: 0;
}

.conversation-sidebar-wrap {
  height: auto;
  flex: 0 0;
  overflow: hidden;
  overflow: auto;
  background: white;
  flex-basis: 28rem;

  @include breakpoint(large up) {
    flex-basis: 30em;
  }

  @include breakpoint(xlarge up) {
    flex-basis: 31em;
  }

  @include breakpoint(xxlarge up) {
    flex-basis: 33rem;
  }

  @include breakpoint(xxxlarge up) {
    flex-basis: 40rem;
  }

  &::v-deep .contact--panel {
    width: 100%;
    height: 100%;
    max-width: 100%;
  }
}
</style>
