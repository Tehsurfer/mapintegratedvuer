<template>
  <div>
    <div id='heart' class="heart-invis"></div>
    <div id='pathway' class="pathway-invis"></div>
    <div id='icn' class="icn-invis"></div>
    <div id='rna' class="rna-invis"></div>
    <v-tour name="onboarding-tour" :steps="steps" :callbacks="tourCallbacks"></v-tour>
  </div>
</template>

<script>
/* eslint-disable no-alert, no-console */
import EventBus from './EventBus';
import VueTour from '@tehsurfer/vue-tour';
import tourSteps from './tourSteps';
import '@tehsurfer/vue-tour/dist/vue-tour.css';
import Vue from "vue";
import {
  Container,
  Header,
  Main
} from "element-ui";
Vue.use(Container);
Vue.use(Header);
Vue.use(Main);
Vue.use(VueTour);

export default {
  name: "Tutorial",
  props: ['parentRefs', 'fullscreen'],
  methods: {
    /**
     * Callback when an action is performed (open new dialogs).
     */
    flowChange: function(action) {
      if (action) {
        if (action === "startTutorial") {
          this.startTutorial();
        } else if (this.tour.isRunning){
          if (action === "flatmapChanged"){
            this.updateStep(4);
          } else if (action === "createNewEntry") {
            this.updateStep(5);
          } else if (action === "dialogMaximise") {
            this.updateStep(6);
          } else if (action === "resourceSelected") {
            this.tour.nextStep();
          }
        }
      }
    },
    updateStep: function(stepNumber){
      if (this.tour.isRunning && this.tour.currentStep === stepNumber ){
        this.tour.nextStep();
      }
    },
    checkForFirstTimeVisitor(){
      var hasVisted = localStorage.getItem('hasVisitedMaps')
      if (hasVisted === null || hasVisted === false){
          localStorage.setItem('hasVisitedMaps', true);
          this.startTutorial();
      }
    },
    startTutorial: function(){
      this.flow.resetApp()
      this.tour.start();
    },
    previousStepCallback: function(currentStep){
      console.log(currentStep);
    },
    nextStepCallback: function(currentStep){
      //The list of callbacks below is customised for each step manipulate the app to brin the user through it

      // Step one goes to full screen
      if (currentStep === 1){
        if (!this.fullscreen){
          this.flow.onFullscreen();
        }
      }

      // Hack on step three to show tooltip for the heart
      if (currentStep === 4){
        this.flow.$refs.dialogs[0].showTooltip({'resource':{'feature':{'id':"263#10018"}}});
        this.flow.$refs.dialogs[0].$refs.popover.updateFromTerm("UBERON:0000948");
      }

      // Step 4 brings up the heart scaffold
      if (currentStep === 5){
        if(this.flow.entries.length === 1){
          this.flow.actionClick({
            title: "View 3D scaffold",
            resource: "https://mapcore-bucket1.s3-us-west-2.amazonaws.com/others/29_Jan_2020/heartICN_metadata.json",
            type: "Scaffold"

          });
          this.tour.previousStep(); //I can't distinquish between a user and the tutorial, so we go back one step after opening it
        }
      }

      // Step 5 docks the dialog
      if (currentStep === 6){
        this.flow.dockDialog(2);
        this.flow.maximiseDialog(2);
      }

      // Step 7 pulls up the RNA seq data
      if (currentStep === 8){
        this.flow.$refs.dialogs[1].showTooltip({'resource':['unused']});
        this.flow.$refs.dialogs[1].$refs.popover.updateFromTerm("ICN");
        this.flow.$refs.dialogs[1].setTooltipCoords(650,300);
        // this.steps[7].target = this.$refs.dialogs[1].$refs.popover.$refs.tooltip.$el.children[0].children[0].children[0].children[1].children[1],
      }

      // Step 8 pulls opens up the plot vuer dailog 
      if (currentStep === 9){
        if(this.flow.entries.length === 2){
          this.flow.actionClick({
            title: "View plot",
            resource: "https://mapcore-bucket1.s3-us-west-2.amazonaws.com/ISAN/csv-data/use-case-4/RNA_Seq.csv",
            type: "Plot",
            plotType: "heatmap",
            datasetTitle: "Molecular Phenotype Distribution of Single Rat Intracardiac Neurons",
            datasetDescription: "Images collected from serial cryostat sectioning of a cryopreserved heart was used to reconstruct the 3D context. Transcriptional profiles taken from isolated single neurons and mapped back into the previously generated 3D context.",
            datasetUrl: "https://discover.blackfynn.com/datasets/29",
            datasetImage: "https://assets.discover.blackfynn.com/dataset-assets/29/6/revisions/1/banner.jpg"
          });
        }
      }
    },
    finishTutorialCallback(){
      this.flow.resetApp();
      document.querySelector('.mapboxgl-popup-close-button').click();
      this.flow.onFullscreen();
    }
      
      
  },
  data: function() {
    return {
      flow: undefined,
      steps: tourSteps,
      tour: this.$tours['onboarding-tour'],
      tourCallbacks: {
        onPreviousStep: this.previousStepCallback,
        onNextStep: this.nextStepCallback,
        onFinish: this.finishTutorialCallback
      }
    }
  },
  mounted: function() {
    this.flow = this.parentRefs.flow
    EventBus.$on("FlowChange", (payLoad) => {
      this.flowChange(payLoad);
    });
    this.tour = this.$tours['onboarding-tour'];
    this.checkForFirstTimeVisitor();
  }
};
</script>

<style scoped>

.dialog-header {
  color: #333;
  line-height: 20px;
  border-bottom: solid 0.7px #dcdfe6;
  background-color: #f5f7fa;
}

.dialog-main {
  padding:0px;
  width:100%;
  height:100%;
}

.heart-invis {
  position: absolute;
  right:40%;
  top:65%;
  height: 0px;
  width: 0px;
}

.pathway-invis {
  position: absolute;
  right:30%;
  top:50%;
  height: 0px;
  width: 0px;
}

.icn-invis {
  position: absolute;
  right:50%;
  top:45%;
  height: 0px;
  width: 0px;
}

.rna-invis {
  position: absolute;
  right:60%;
  top:45%;
  height: 0px;
  width: 0px;
}

</style>
