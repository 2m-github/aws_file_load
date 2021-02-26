<template>
  <v-app>
    <router-link :to="{name:'Home'}">HOME</router-link>
    <router-link :to="{name:'ImageHandle'}">IMAGE</router-link>
    <v-main>
      <router-view />
    </v-main>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld';

export default {
  name: 'App',

 

  data: () => ({
    //
  }),
};
</script>
