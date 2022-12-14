<template>
  <div id="app">
    <!-- Loading the navbar on the top of every page -->
    <fiori-nav-bar 
      :companyName="companyName" 
      :logoUrl="logoUrl"
      :showNavButtonStatus="showNavButtonStatus"
      @updateNavButtonStatus="updateNavButtonStatus($event)"
       />

    <!-- Inserting router with all props and events needed in the child components -->
    <router-view 
      :selectedFranchiseeId="selectedFranchiseeId"
      :allFranchises="allFranchises"
      :allMentors="allMentors"
      :allCoordinators="allCoordinators"
      :companyName="companyName"
      :logoUrl="logoUrl"
      :technicalEmail="technicalEmail"
      :technicalEmailPassword="technicalEmailPassword"
      @changeSelectedFranchiseeId="updateSelectedFranchiseeId($event)"
      @reloadAllFranchises="loadAllFranchises($event)"
      @reloadAllMentors="loadAllMentors($event)"
      @reloadAllCoordinators="loadAllCoordinators($event)"
      @updateCompanyDetails="updateCompanyDetails($event)"
      @updateNavButtonStatus="updateNavButtonStatus($event)"
      >
    </router-view>

    <!-- Loading the modal to start the app introduction -->
    <modal-intro />
  </div>
</template>

<script>
import FioriNavBar from '@/components/FioriNavBar.vue';
import ModalIntro from '@/components/ModalIntro.vue';
export default {
  name: 'App',
  data: function(){
    return {
      allFranchises: [],
      allMentors: [],
      allCoordinators: [],
      selectedFranchiseeId: "",
      companyName: "",
      logoUrl: "",
      technicalEmail: "",
      technicalEmailPassword: "",
      showNavButtonStatus: "false"
    }
  },
  components: {
    FioriNavBar,
    ModalIntro
  },
  methods: {
   // Calling Metering API to register the user
    logUser(){
      const apiUrl = this.$backendApi + "/meter-user-login";
      fetch(
          apiUrl,
          {
            method: "PUT"  
          }
        )
        .then(response => {
          console.log("[DEBUG] Login user in EF Service: " + response);
        })
        .catch(err => {
          console.log(err);
        });
    },

    // Checking if intro modal should be display or not based on maintained information
    async checkandFillCompanyDetails(){
      let apiUrl = this.$backendApi + "/config/franchisor";
      let response = await fetch(apiUrl);
      this.companyName = await response.text();
      console.log("[DEBUG] Loading companyName: " + this.companyName);

      apiUrl = this.$backendApi + "/config/logourl";
      response = await fetch(apiUrl);
      this.logoUrl = await response.text();
      console.log("[DEBUG] Loading logoUrl: " + this.logoUrl);

      apiUrl = this.$backendApi + "/config/notificationconfig";
      response = await fetch(apiUrl);
      let configDetails = await response.json();
      console.log("[DEBUG Loading configNotificationsDetails: " + configDetails)
      this.technicalEmail = configDetails.email;
      this.technicalEmailPassword = configDetails.password;
      console.log("[DEBUG] Loading technicalEmail and password: " + this.technicalEmail + " - " + this.technicalEmailPassword);

      apiUrl = this.$backendApi + "/mentor";
      response = await fetch(apiUrl);
      this.allMentors = await response.json();
      console.log("[DEBUG] Loading allMentors: " + this.allMentors);

      if (!this.companyName || !this.logoUrl || this.allMentors.length === 0) {
        console.log("[INFO] Company details are not 100% maintained");
        this.$fdModal.open("Introduction");
      }
    },
    // Loading all franchises details from backend
    loadAllFranchises() {
      const apiUrl = this.$backendApi + "/franchisee";
      fetch(apiUrl)
      .then(response => response.json())
      .then(data => {
        this.allFranchises = data;
        console.log("[DEBUG] Loading allFranchises: ", data);
      });
    },
    // Loading all mentors details from backend
    loadAllMentors() {
      const apiUrl = this.$backendApi + "/mentor";
      fetch(apiUrl)
      .then(response => response.json())
      .then(data => {
        this.allMentors = data;
        console.log("[DEBUG] Loading allMentors: ", data);
      });
    },
    // Loading all coordinators details from backend
    loadAllCoordinators() {
      const apiUrl = this.$backendApi + "/coordinator";
      fetch(apiUrl)
      .then(response => response.json())
      .then(data => {
        this.allCoordinators = data;
        console.log("[DEBUG] Loading allCoordinators: ", data);
      });
    },
    // Saving company name and logo in backend
    updateCompanyDetails(updatedCompanyDetails){
      if (!updatedCompanyDetails) {
        console.log("[DEBUG] Updated company details are empty")
      } else {
        console.log("[DEBUG] Sending updated company details: ", updatedCompanyDetails);
        this.companyName = updatedCompanyDetails.name;
        this.logoUrl = updatedCompanyDetails.logo;
        this.technicalEmail = updatedCompanyDetails.email;
        this.technicalEmailPassword = updatedCompanyDetails.password;

        let apiUrl = this.$backendApi + "/config/franchisor";
        fetch(
          apiUrl,
          {
            method: "PUT",
            mode: "cors",
            headers: {
              "Access-Control-Allow-Origin":"*",
              "Access-Control-Request-Method": "PUT",
              "Content-Type": "text/plain"
            },
            body: this.companyName
          }
        )
        .then(response => {
          console.log("[DEBUG] Loading company name: " + response);
        })
        .catch(err => {
          console.log(err);
        });

        apiUrl = this.$backendApi + "/config/logourl";
        fetch(
          apiUrl,
          {
            method: "PUT",
            headers: {
              "Content-Type": "text/plain"
            },
            body: this.logoUrl
          }
        )
        .then(response => {
          console.log("[DEBUG] Loading company logo: " + response);
        })
        .catch(err => {
          console.log(err);
        });

        apiUrl = this.$backendApi + "/config/notificationconfig";
        fetch(
          apiUrl,
          {
            method: "PUT",
            headers: {
              "Content-Type": "application/json"
            },
            body: JSON.stringify({
              email: this.technicalEmail,
              password: this.technicalEmailPassword
            })
          }
        )
        .then(response => {
          console.log("[DEBUG] Loading technical email: " + response);
        })
        .catch(err => {
          console.log(err);
        });
      }
    },
    // Updating selected franchisee Id
    updateSelectedFranchiseeId(id){
      console.log("[DEBUG] Updating selectedFranchiseeId: ", id);
      this.selectedFranchiseeId = id;
    },
    updateNavButtonStatus(newStatus){
        console.log("[DEBUG] Updating NavButtonStatus with: ", newStatus);
        this.showNavButtonStatus = newStatus;
    }
  }, 
  mounted: function() {
    this.loadAllFranchises();
    this.loadAllCoordinators();
    this.checkandFillCompanyDetails();
    this.logUser();
  }
}
</script>
  
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
a{
  text-decoration: none !important;
}
</style>
