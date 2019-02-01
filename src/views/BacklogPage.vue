<template>
  <div class="home">
    <h1>Backlog Page</h1>

    <button @click="filter('my')">my</button>
    <button>open</button>
    <button>closed</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Model } from "vue-property-decorator";

@Component
export default class BacklogPage extends Vue {
  public filter(preset: any) {
    //alert("filter " + preset);
    this.$router.push("/backlog/" + preset);
  }
}
</script>
