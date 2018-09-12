<template>

  <div>
    <transition
      name="slidein"
      mode="out-in"
    >
      <Snackbar
        v-if="currentSnackbar === SNACKBARS.POINTS"
        :key="SNACKBARS.POINTS"
        @close="currentSnackbar = SNACKBARS.NEXT_RESOURCE"
      >
        <template slot="icon">
          <ProgressIcon :progress="1" />
        </template>

        <template slot="content">
          <PointsIcon
            class="points-icon"
            :active="true"
          />
          <div class="points-amount">{{ $tr('plusPoints', { maxPoints }) }}</div>
        </template>

        <UiAlert
          v-if="!isUserLoggedIn"
          :dismissible="false"
          :removeIcon="true"
          type="warning"
          class="alert"
          slot="alert"
        >
          {{ $tr('signIn') }}
        </UiAlert>
      </Snackbar>

      <Snackbar
        v-else-if="currentSnackbar === SNACKBARS.NEXT_RESOURCE && nextContent"
        :key="SNACKBARS.NEXT_RESOURCE"
        @close="$emit('close')"
      >
        <template slot="icon">
          <ContentIcon
            class="content-icon icon-bg"
            :kind="nextContent.kind"
            :showTooltip="true"
            :style="{backgroundColor: iconBackgroundColor }"
          />
        </template>

        <template slot="content">
          <router-link
            class="rm-link-style"
            :to="nextContentLink"
          >
            <h2 class="next-content-heading">{{ $tr('next') }}</h2>
            <KRouterLink
              :text="nextContent.title"
              :to="nextContentLink"
              class="next-content-title"
              dir="auto"
            />
          </router-link>

        </template>
      </Snackbar>
    </transition>

  </div>

</template>


<script>

  import { mapGetters } from 'vuex';
  import { MaxPointsPerContent, ContentNodeKinds } from 'kolibri.coreVue.vuex.constants';
  import PointsIcon from 'kolibri.coreVue.components.PointsIcon';
  import ProgressIcon from 'kolibri.coreVue.components.ProgressIcon';
  import ContentIcon from 'kolibri.coreVue.components.ContentIcon';
  import KRouterLink from 'kolibri.coreVue.components.KRouterLink';
  import UiAlert from 'keen-ui/src/UiAlert';
  import Snackbar from './Snackbar';

  const SNACKBARS = { POINTS: 'POINTS', NEXT_RESOURCE: 'NEXT_RESOURCE' };

  export default {
    name: 'MasteredSnackbars',
    components: {
      PointsIcon,
      ProgressIcon,
      ContentIcon,
      KRouterLink,
      Snackbar,
      UiAlert,
    },
    props: {
      nextContent: {
        type: Object,
        required: false,
      },
      nextContentLink: {
        type: Object,
        required: false,
      },
    },
    data: () => ({
      currentSnackbar: null,
      pointsTimeout: null,
      nextContentTimeout: null,
    }),

    computed: {
      ...mapGetters(['isUserLoggedIn']),
      SNACKBARS() {
        return SNACKBARS;
      },
      maxPoints() {
        return MaxPointsPerContent;
      },
      iconBackgroundColor() {
        switch (this.nextContent.kind) {
          case ContentNodeKinds.EXERCISE:
            return '#0eafaf';
          case ContentNodeKinds.VIDEO:
            return '#3938A5';
          case ContentNodeKinds.AUDIO:
            return '#E65997';
          case ContentNodeKinds.DOCUMENT:
            return '#ED2828';
          case ContentNodeKinds.HTML5:
            return '#FF8B41';
          default:
            return '#262626';
        }
      },
    },
    watch: {
      currentSnackbar(val) {
        if (val === SNACKBARS.NEXT_RESOURCE) {
          this.nextContentTimeout = setTimeout(() => {
            this.currentSnackbar = null;
          }, 10000);
        }
      },
    },
    mounted() {
      this.currentSnackbar = SNACKBARS.POINTS;
      this.pointsTimeout = setTimeout(() => {
        if (this.currentSnackbar === SNACKBARS.POINTS) {
          this.currentSnackbar = SNACKBARS.NEXT_RESOURCE;
        }
      }, 4000);
    },
    beforeDestroy() {
      if (this.pointsTimeout) {
        clearTimeout(this.pointsTimeout);
      }
      if (this.nextContentTimeout) {
        clearTimeout(this.nextContentTimeout);
      }
    },
    $trs: {
      plusPoints: '+ { maxPoints, number } Points',
      next: 'Next:',
      signIn: 'Sign in or create an account to save points you earn',
    },
  };

</script>


<style lang="scss" scoped>

  @import '~kolibri.styles.definitions';

  .points-icon {
    display: inline-block;
    width: 24px;
    height: 24px;
    margin-right: 4px;
    vertical-align: middle;
  }

  .icon-bg {
    border-radius: 50%;
  }

  .points-amount {
    display: inline-block;
    font-weight: bold;
    color: $core-status-correct;
    vertical-align: middle;
  }

  .content-icon {
    display: block;
    width: 24px;
    height: 24px;
    font-size: 18px;
    color: white;
    text-align: center;
    border-radius: 50%;
  }

  .next-content-heading {
    margin: 0 0 4px;
    font-size: 12px;
    font-weight: normal;
    color: $core-text-annotation;
  }

  .next-content-title {
    font-weight: bold;
  }

  .slidein-enter-active {
    animation-name: slidein;
  }

  .slidein-leave-active {
    animation-name: slidein;
    animation-direction: reverse;
  }

  @keyframes slidein {
    from {
      visibility: visible;
      transform: translate3d(0, 100%, 0);
    }
    to {
      transform: translate3d(0, 0, 0);
    }
  }

  .alert {
    margin-top: 8px;
    margin-bottom: 0;
  }

  .rm-link-style {
    display: block;
    color: $core-text-default;
    text-decoration: none;
  }

</style>