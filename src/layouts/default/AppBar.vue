<template>
  <v-app-bar flat>
    <v-app-bar-nav-icon>
      <v-icon icon="mdi-circle-slice-6" />
    </v-app-bar-nav-icon>
    <v-app-bar-title>
      {{ navbar.title }}
    </v-app-bar-title>
    <div class="nav-buttons" v-if="navbar.status">
      <v-btn prepend-icon="mdi-vuetify" variant="tonal" v-for="item in items">
        {{ item }}
      </v-btn>
      
    </div>

    <v-btn prepend-icon="mdi-vuetify" @click="changeToggleState" variant="tonal" v-if="!navbar.status">
        {{screenWidth}}
    </v-btn>
    
  </v-app-bar>

  <v-navigation-drawer
        v-model="togglePanel"
        location="top"
        temporary
  >
  <div class="nav-buttons-list">
    <v-btn prepend-icon="mdi-vuetify" variant="tonal" v-for="item in items">
      {{ item }}
    </v-btn>
  </div>
  </v-navigation-drawer>

  
</template>

<script lang="ts" setup>
  import { ref, watch, onMounted} from 'vue'

  const navbar = ref({ title: 'AlocaSensores', status: true})
  const screenWidth = ref(window.innerWidth);
  const items = ref(["Página Inicial", "Sobre"])
  let togglePanel = ref<boolean>(false)

  
  onMounted(() => {
    window.addEventListener('resize', handleResize);
  });

  const handleResize = () => {
    screenWidth.value = window.innerWidth;
  };

  const changeToggleState = () => {
    togglePanel.value = !(togglePanel.value);
  }

  watch(screenWidth, (newValue) => {
    if(screenWidth.value < 600){
      navbar.value.status = false
    }else{
      navbar.value.status = true
      togglePanel.value = false
    }
  })

</script>

<style scoped>

.nav-buttons{
  display: flex;
  row-gap: 5px;
  column-gap: 5px;
  margin-right: 5px;
}

.nav-buttons-list{
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  row-gap: 5px;
  margin-top: 15px;
}

.nav-buttons-list button{
  max-width: 150px;
  width: 150px;
}
</style>
