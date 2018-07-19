<template>

  <div>
    <UiIconButton
      type="secondary"
      @click="showLanguageModal = true"
      class="globe"
    >
      <mat-svg
        name="language"
        category="action"
      />
    </UiIconButton>

    <span class="selected">
      {{ selectedLanguage }}
    </span>
    <KButton
      v-for="language in buttonLanguages"
      @click="switchLanguage(language.id)"
      :key="language.id"
      :text="language.lang_name"
      class="lang"
      appearance="basic-link"
    />
    <KButton
      :text="$tr('showMoreLanguagesSelector')"
      :primary="false"
      appearance="flat-button"
      class="more"
      @click="showLanguageModal = true"
    />
    <LanguageSwitcherModal
      v-if="showLanguageModal"
      @close="showLanguageModal = false"
      class="modal"
    />
  </div>

</template>


<script>

  import { availableLanguages, currentLanguage } from 'kolibri.utils.i18n';
  import KButton from 'kolibri.coreVue.components.KButton';
  import responsiveWindow from 'kolibri.coreVue.mixins.responsiveWindow';
  import UiIconButton from 'keen-ui/src/UiIconButton';
  import languageSwitcherMixin from './mixin';
  import LanguageSwitcherModal from './LanguageSwitcherList';

  export default {
    name: 'LanguageSwitcherList',
    $trs: {
      showMoreLanguagesSelector: 'More languages',
    },
    components: {
      KButton,
      LanguageSwitcherModal,
      UiIconButton,
    },
    mixins: [responsiveWindow, languageSwitcherMixin],
    data() {
      return {
        showLanguageModal: false,
      };
    },
    computed: {
      selectedLanguage() {
        return availableLanguages[currentLanguage].lang_name;
      },
      numVisibleLanguages() {
        if (this.windowSize.breakpoint <= 2) {
          return 2;
        }
        return this.windowSize.breakpoint;
      },
      buttonLanguages() {
        const prioritized_languages = ['en', 'ar', 'es-es', 'hi-in', 'fr-fr', 'sw-tz'];
        return prioritized_languages
          .filter(lang => availableLanguages[lang] !== undefined)
          .filter(lang => lang !== currentLanguage)
          .map(lang => availableLanguages[lang])
          .slice(0, this.numVisibleLanguages)
          .sort(this.compareLanguages);
      },
    },
  };

</script>


<style lang="scss" scoped>

  @import '~kolibri.styles.definitions';

  .globe {
    position: relative;
    top: -2px;
    right: -4px;
  }

  .selected {
    margin: 8px;
  }

  .lang {
    margin-right: 8px;
    margin-left: 8px;
  }

  .more {
    margin: 0;
    margin-top: 8px;
    margin-bottom: 8px;
  }

  .modal {
    text-align: left;
  }

</style>