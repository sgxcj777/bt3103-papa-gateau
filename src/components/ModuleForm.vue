<template>
  <div>
    <form>
      <div v-show="allowedit">
        <md-autocomplete
          v-model="detailsForm.selectedModule"
          :md-options="modules"
          disabled
          :class="getValidationClass('detailsForm', 'selectedModule')"
        >
          <label v-show="!detailsForm.selectedModule">Search for Module</label>
          <label v-show="detailsForm.selectedModule">Module</label>
          <template slot="md-autocomplete-item" slot-scope="{ item, term }">
            <md-highlight-text :md-term="term">{{ item }}</md-highlight-text>
          </template>

          <template
            slot="md-autocomplete-empty"
            slot-scope="{ term }"
          >No Modules with "{{ term }}" were found.</template>

          <span
            class="md-error"
            v-if="!$v.detailsForm.selectedModule.required"
          >This field is required</span>
        </md-autocomplete>
      </div>
      <div v-show="disallowedit">
        <md-autocomplete
          v-model="detailsForm.selectedModule"
          :md-options="modules"
          :class="getValidationClass('detailsForm', 'selectedModule')"
        >
          <label v-show="!detailsForm.selectedModule">Search for Module</label>
          <label v-show="detailsForm.selectedModule">Module</label>
          <template slot="md-autocomplete-item" slot-scope="{ item, term }">
            <md-highlight-text :md-term="term">{{ item }}</md-highlight-text>
          </template>

          <template
            slot="md-autocomplete-empty"
            slot-scope="{ term }"
          >No Modules with "{{ term }}" were found.</template>

          <span
            class="md-error"
            v-if="!$v.detailsForm.selectedModule.required"
          >This field is required</span>
        </md-autocomplete>
      </div>

      <div class="md-layout">
        <div class="md-layout-item md-size-50">
          <md-field :class="getValidationClass('detailsForm', 'selectedGrade')">
            <label>Grade obtained</label>
            <md-select v-model="detailsForm.selectedGrade">
              <md-option v-for="g in grades" v-bind:key="g.id" v-bind:value="g.title">{{g.title}}</md-option>
            </md-select>
            <span
              class="md-error"
              v-if="!$v.detailsForm.selectedGrade.required"
            >This field is required</span>
          </md-field>
          <span style="color:red" v-show="showError">{{error}}</span>
        </div>
        <div class="md-layout-item md-size-10"></div>
        <div class="md-layout-item">
          <div style="margin-top:0.5vw">
            <md-radio
              v-model="detailsForm.selectedSU"
              class="md-primary"
              style="padding-left:1vw"
              value="Yes"
            >SU</md-radio>
            <md-radio
              v-model="detailsForm.selectedSU"
              class="md-primary"
              style="padding-left:1vw"
              value="No"
            >No SU</md-radio>
          </div>
        </div>
      </div>
      <br />
      <md-card-actions class="md-layout md-alignment-center">
        <md-button type="submit" class="addsem" v-on:click.prevent="submitForm">Submit</md-button>
      </md-card-actions>
    </form>
  </div>
</template>

<script>
// import FollowUpModal from "./FollowUpModal.vue"
import { validationMixin } from "vuelidate";
import { required } from "vuelidate/lib/validators";
import database from "../firebase.js";
export default {
  name: "ModuleForm",
  props: {
    //msg: String
    sem: String,
    year: String,
    grade: String,
    code: String,
    purpose: String,
    SUselect: String
  },
  components: {
    // FollowUpModal
  },
  data: function() {
    return {
      showModal: false,
      error: "",
      showError: false,
      searchlist: [],
      Years: [
        { id: 1, title: "AY1819" },
        { id: 1, title: "AY1920" }
      ],
      modules: [],
      grades: [
        {
          id: 1,
          title: "A+"
        },
        {
          id: 2,
          title: "A"
        },
        {
          id: 3,
          title: "A-"
        },
        {
          id: 4,
          title: "B+"
        },
        {
          id: 5,
          title: "B"
        },
        {
          id: 6,
          title: "B-"
        },
        {
          id: 7,
          title: "C+"
        },
        {
          id: 8,
          title: "C"
        },
        {
          id: 9,
          title: "D+"
        },
        {
          id: 10,
          title: "D"
        },
        {
          id: 11,
          title: "F"
        },
      ],
      SU: [
        { id: 1, title: "Yes" },
        { id: 2, title: "No" }
      ],
      semesters: [
        {
          id: 1,
          title: "Semester 1",
          examDuration: 120
        },
        {
          id: 2,
          title: "Semester 2",
          examDate: "2020-05-05T09:00:00.000Z",
          examDuration: 120
        }
      ],
      submitStatus: null,
      detailsForm: {
        selectedModule: this.code,
        selectedSemester: this.sem,
        selectedGrade: this.grade,
        selectedSU: this.SUselect,
        selectedYear: this.year
      }
    };
  },
  mixins: [validationMixin],
  validations: {
    detailsForm: {
      selectedModule: {
        required
      },

      selectedGrade: {
        required
      },
      selectedSU: {
        required
      }
    }
  },
  methods: {
    getValidationClass(formName, fieldName) {
      const field = this.$v[formName][fieldName];
      if (field) {
        return {
          "md-invalid": field.$invalid && field.$dirty
        };
      }
    },

    submitForm() {
      this.$v.$touch();
      if (!this.$v.$invalid) {
        console.log("ok");
        if (this.purpose == "Add") {
         
          database
            .addModuleResults(this.detailsForm)
            .then(e => {
              console.log(e);
             
              // create an alert saying you have already added this module
              this.showError = false;
              this.error = "";
              this.$root.$emit("closeModal1", {
                year: this.detailsForm.selectedYear,
                sem: this.detailsForm.selectedSemester
              });
            })
            .catch(error => {
              this.showError = true;
              this.error = error;

  
            });
        } else {
          console.log(this.detailsForm.selectedYear,)
          database
            .updateModuleResults(this.detailsForm)
            .then(e => {
              console.log(this.detailsForm)
              console.log(e);
              // create an alert saying you have already added this module

              this.$root.$emit("closeModal1", {
                year: this.detailsForm.selectedYear,
                sem: this.detailsForm.selectedSemester
              });
              // alert("Module Successfully Updated!");
            })
            .catch(error => {
              this.showError = true;
              this.error = error;
            
            });
        }

      }
    }
  },
  computed: {
    allowedit() {
      if (this.purpose == "Edit") {
        return true;
      }
      return false;
    },
    disallowedit() {
      if (this.purpose == "Edit") {
        return false;
      }
      return true;
    }
  },
  created() {
    //Accumulating dropdown with modules in DB

    database.firebase_data
      .collection("modules")
      .get()
      .then(querySnapShot => {
        var slookup = {};
        //Loop through each item
        querySnapShot.forEach(doc => {
          var name = doc.data().info.moduleCode + " " + doc.data().info.title;
          var sems = doc.data().info.semesterData;
          for (var sem in sems) {
           
            if ("Semester " + sems[sem].semester == this.detailsForm.selectedSemester) {
              if (!(name in slookup)) {
                slookup[name] = 1;
                this.modules.push(doc.data().info.moduleCode);
              }
              break;
            }
          }
        });
      });
  }
};
</script>

<style scoped>
.md-card {
  /* overflow: scroll; */
  display: block;
  /* min-height: 180px; */
}
</style>
<style lang="scss">
.md-button.addsem {
  background-color: teal !important;
  color: white !important;
  margin-top: 1vh;
  width: 8vw;
  font-size: 1vw;
}
.md-menu-content {
  z-index: 11 !important;
}
</style>