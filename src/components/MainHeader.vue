<template>
  <header
    class="flex flex-col"
    v-bind:style=" !isAuthenticated && offerURL == '' && homeHeroImg != '' ? { backgroundImage: 'url(' + homeHeroImg + ')' } : {}"
  >
    <div class="wrap header flex">
      <a>
        <img v-on:click="goHomePage" class="logo" :src="('./img/u+-logo.svg')" alt="U+">
      </a>
      <nav class="flex-grow-1">
        <ul class="flex flex-mid-align flex-grow-1">
          <MenuItem v-for="item in app.menuitems" v-bind:key="item.title" v-bind:title="item.title"></MenuItem>
        </ul>
      </nav>
      <a
        class="kmhelp"
        v-if="isAuthenticated && settings.kmhelp.url !==''"
        v-on:click="showKMHelp"
      >{{ $t('message.kmhelp') }}</a>
      <OperatorButton v-if="isAuthenticated"/>
      <LoginButton v-else/>
      <MainHeaderMenu/>
    </div>
  </header>
</template>

<script>
import MenuItem from './controls/MenuItem.vue';
import MainHeaderMenu from './widgets/MainHeaderMenu.vue';
import OperatorButton from './widgets/OperatorButton.vue';
import LoginButton from './widgets/LoginButton.vue';
import { mainconfig } from '../global';

export default {
  data() {
    return mainconfig;
  },
  mounted() {
    window.addEventListener('message', this.iFrameMessageListener);
  },
  destroyed() {
    window.removeEventListener('message', this.iFrameMessageListener);
  },
  methods: {
    /* Will listen for message from the Mashup iframe to force a reload back of the MashupComponent */
    iFrameMessageListener(e) {
      if (e.data === 'pegaMashupNavigateBack') {
        mainconfig.reloadAccountMashup += 1;
        this.goHomePage();
      } else if (
        typeof e.data === 'object' &&
        typeof e.data.key !== 'undefined'
      ) {
        if (e.data.key === 'Intent' && typeof e.data.value === 'string') {
          mainconfig.intent = e.data.value;
          mainconfig.reloadOffer += 1;
        } else if (
          e.data.key === 'PreviousPage' &&
          typeof e.data.value === 'string'
        ) {
          mainconfig.previousPage = e.data.value;
          mainconfig.reloadOffer += 1;
        }
      } else if (
        typeof e.data === 'object' &&
        typeof e.data.action !== 'undefined'
      ) {
        if (
          e.data.action === 'showkmarticle' &&
          typeof e.data.articleid === 'string'
        ) {
          mainconfig.KMArticleID = e.data.articleid;
          this.showKMHelp();
        }
      }
    },
    goHomePage() {
      mainconfig.quickLinkId = -1;
      mainconfig.viewBill = -1;
      mainconfig.homeHeroAction = -1;
      mainconfig.offerAction = -1;
      mainconfig.toDo = -1;
      mainconfig.viewKMHelp = -1;
      mainconfig.offerURL = '';
      if (window.history) {
        if (mainconfig.isAuthenticated) {
          window.history.pushState(
            { userId: mainconfig.userId },
            '',
            'account',
          );
        } else {
          window.history.pushState({}, '', 'index.html');
        }
      }
    },
    showKMHelp() {
      this.goHomePage();
      mainconfig.viewKMHelp = 1;
      mainconfig.reloadMashup += 1;
    },
  },
  components: {
    MenuItem,
    LoginButton,
    OperatorButton,
    MainHeaderMenu,
  },
};
</script>
