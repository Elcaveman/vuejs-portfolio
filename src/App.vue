<template>
  <transition name="fade" tag="div" class="wrapper" mode="out-in">
    <div class="wrapper" v-if="isLoaded" id="app">
      <LandingPage :user="user" />
      <Description :user="user" :content="description" :links="links" />
      <Experience :content="experiences" />
      <Skills :skills="skills" />
      <Projects :projects="projects" />
      <Footer :user="user" :links="links" />
    </div>
  </transition>
</template>

<script>
import LandingPage from "./components/LandingPage.vue";
import Description from "./components/Description.vue";
import Experience from "./components/Experience.vue";
import Skills from "./components/Skills.vue";
import Projects from "./components/Projects.vue";
import Footer from "./components/Footer.vue";

import { bucket } from "./cosmic.js";

export default {
  name: "App",
  components: {
    LandingPage,
    Description,
    Experience,
    Skills,
    Projects,
    Footer,
  },
  data: () => ({
    isLoaded: false,
    user: {},
    experiences: [],
    skills: [],
    projects: [],
    description:{},
    links:[]
  }),
  methods: {
    fetchObject(slug) {
      return bucket.getObjects({query:{slug:slug}});
    },
    fetchUserData(){
      return this.fetchObject("user-data")
    },
    fetchSkills(){
      return this.fetchObject("skills")
    },
    fetchLinks(){
      return this.fetchObject("links")
    },
    fetchExperiences(){
      return this.fetchObject("experiences")
    },
    fetchDescription(){
      //for the about me section
      return this.fetchObject("description")
    },
    fetchProjects(){
      //for the about me section
      return this.fetchObject("projects")
    },
    extractFirstObject(objects) {
      if(objects.objects == null)
        return void 0;
      else
        return objects.objects[0];
    }
  },
  created() {
    document.body.classList.add("loading");

    Promise.all([this.fetchUserData(),this.fetchSkills(),this.fetchLinks(),
    this.fetchExperiences(),this.fetchDescription(),
    this.fetchProjects()]).then(([user_data,skills,links,experiences,
    description]) => {
      console.log("data",[user_data,skills,links,experiences,
    description])
      // this.posts = posts.objects;
      console.log("user_data",user_data)
      user_data = user_data.objects[0]
      this.user = {
        name: user_data.metadata.name,
        status: user_data.metadata.status,
        email: user_data.metadata.email,
        phone: user_data.metadata.phone,
        city: user_data.metadata.city,
        lang: user_data.metadata.lang,
        photo: user_data.metadata.photo,
      }
      this.skills = skills.objects[0].metadata.items;
      this.links = links.objects[0].metadata;
      this.experiences = experiences.objects[0].metadata;
      this.description = description.objects[0].metadata;
      console.log("user",this.user)
      console.log("links",this.links)
      console.log("experiences",this.experiences)
      console.log("description",this.description)
      this.isLoaded = true;
      this.$nextTick(() => document.body.classList.remove("loading"));
    });
  },
};
</script>

<style scoped lang="scss">
@import "./styles/constants.scss";

#app {
  font-family: Montserrat-Regular, serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

.wrapper {
  height: 100%;
}
</style>
