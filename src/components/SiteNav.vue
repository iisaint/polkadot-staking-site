<template>
  <div id="site-nav">
    <md-app md-mode="fixed" >
      <md-app-toolbar class="site-nav">
      <span class="md-title" @click="onClickTitle"><span style="color:#61ba89">Crypto</span><span style="color:#FAFAFA">Currency</span><span style="color:#61ba89">Lab</span></span>
      <div class="md-toolbar-section-end">
        <md-button class="md-button nav-button" @click="onClickTool">
          Tools
        </md-button>
        <md-button class="md-button nav-button" @click="onClickStake">
          Stake with Us
        </md-button>
        <!-- <md-button class="md-button nav-button" href="https://www.yaohsin.net/blog">
          Blog
        </md-button> -->
        <md-button class="md-button nav-button" @click="onClickContact">
          Contact
        </md-button>
      </div>
      </md-app-toolbar>
    </md-app>
  </div>
</template>

<script>
export default {
  name: 'SiteNav',
  methods: {
    onClickTool: function() {
      this.$router.push('/tools');
    },
    onClickStake: function() {
      this.$router.push('/');
    },
    onClickTitle: function() {
      this.$router.push('/');
    },
    onClickContact: function() {
      this.$router.push('/contact').catch(()=>{});
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .site-nav {
    background-color: #1f232c !important;
  }

  .md-title {
    cursor: pointer;
  }

  .nav-button {
    color: #FFFFFF !important;
  }
</style>
