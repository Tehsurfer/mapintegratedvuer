<template>
  <div class="viewer-container">
    <div class="map-selector-container">
      <el-select 
        v-model="selectedMap" 
        placeholder="Select a map" 
        @change="onMapSelectionChange"
        style="width: 200px"
      >
        <el-option
          v-for="option in mapOptions"
          :key="option.value"
          :label="option.label"
          :value="option.value"
        />
      </el-select>
    </div>
    
    <div class="map-display-area">
      <div v-if="!selectedMap" class="placeholder-text">
        Please select a body system to display
      </div>
      <div v-else class="image-container">
        <img 
          :src="selectedMapImage" 
          :alt="`${selectedMapLabel} diagram`"
          @error="onImageError"
          @load="onImageLoad"
          class="system-image"
        />
        <div v-if="imageError" class="error-message">
          Image not found for {{ selectedMapLabel }}
          <br>
          <small>Expected: {{ selectedMapImage }}</small>
        </div>
      </div>
    </div>

    <HelpModeDialog
      v-if="helpMode && useHelpModeDialog"
      ref="multiflatmapHelp"
      :multiflatmapRef="multiflatmapRef"
      :lastItem="helpModeLastItem"
      @show-next="onHelpModeShowNext"
      @finish-help-mode="onFinishHelpMode"
    />
  </div>
</template>

<script>
/* eslint-disable no-alert, no-console */
import Tagging from '../../services/tagging.js';
import ContentMixin from "../../mixins/ContentMixin";
import EventBus from "../EventBus";
import {
  capitalise,
  availableSpecies,
  getBodyScaffoldInfo,
  transformObjToString
} from "../scripts/utilities";
import DyncamicMarkerMixin from "../../mixins/DynamicMarkerMixin";

import YellowStar from "../../icons/yellowstar";

import { MultiFlatmapVuer } from "@abi-software/flatmapvuer";
import "@abi-software/flatmapvuer/dist/style.css";
import { HelpModeDialog } from '@abi-software/map-utilities'
import '@abi-software/map-utilities/dist/style.css'

const getOpenMapOptions = (species) => {
  const options = [
    {
      display: "Open AC Map",
      key: "AC"
    },
    {
      display: "Open FC Map",
      key: "FC"
    },
    {
      display: "Open 3D Human Map",
      key: "3D"
    },
  ]
  return options;
}

export default {
  name: "MultiFlatmap",
  mixins: [ContentMixin, DyncamicMarkerMixin],
  components: {
    MultiFlatmapVuer,
    HelpModeDialog,
  },
  data: function () {
    return {
      zoomLevel: 6,
      flatmapReady: false,
      availableSpecies: availableSpecies(),
      scaffoldResource: { },
      showStarInLegend: false,
      openMapOptions: getOpenMapOptions("Human Male"),
      selectedMap: '',
      imageError: false,
      mapOptions: [
        { value: 'digestive', label: 'Digestive System', image: '/body-system-images/digestive.png'},
        { value: 'respiratory', label: 'Respiratory System', image: '/body-system-images/respiratory.png'},
        { value: 'lymphatic', label: 'Lymphatic System', image: '/body-system-images/lymphatic.png'},
        { value: 'musculoskeletal', label: 'Musculoskeletal System', image: '/body-system-images/musculoskeletal.png'},
        { value: 'central-nervous', label: 'Central Nervous System', image: '/body-system-images/central-nervous.png'},
        { value: 'circulatory', label: 'Circulatory System', image: '/body-system-images/circulatory.png'},
        { value: 'endocrine', label: 'Endocrine System', image: '/body-system-images/endocrine.png'},
        { value: 'neuron', label: 'Neuron', image: '/body-system-images/neuron.png'},
        { value: 'urinary', label: 'Urinary System', image: '/body-system-images/urinary.png'},
        { value: 'synapse', label: 'Synapse', image: '/body-system-images/synapse.png'},
        { value: 'skin', label: 'Skin', image: '/body-system-images/skin.png'},
      ]
    }
  },
  methods: {
    getState: function () {
      return {};
    },
    flatmapPanZoomCallback: function (payload) {
      return null;
    },
    /**
     * Perform a local search on this contentvuer
     */
    search: function (term) {
      return false;
    },
    /**
     * Append the list of suggested terms to suggestions
     */
    searchSuggestions: function (term, suggestions) {
      return null;
    },
    flatmaprResourceSelected: function (type, resource) {
    
      if (resource.eventType === 'click' && resource.feature.type === 'feature') {
        const eventData = {
          label: resource.label || '',
          id: resource.feature.id || '',
          featureId: resource.feature.featureId || '',
          taxonomy: resource.taxonomy || '',
          resources: resource.resource.join(', ')
        };
        const paramString = transformObjToString(eventData);
        // `transformStringToObj` function can be used to change it back to object
        Tagging.sendEvent({
          'event': 'interaction_event',
          'event_name': 'portal_maps_connectivity',
          'category': paramString,
          "location": type + ' ' + map.viewingMode
        });
      }
    },
    onPathwaySelectionChanged: function (data) {
      const { label, property, checked, selectionsTitle } = data;
      // GA Tagging
      // Event tracking for maps' pathway selection change
      Tagging.sendEvent({
        'event': 'interaction_event',
        'event_name': 'portal_maps_pathway_change',
        'category': label + ' [' + property + '] ' + checked,
        'location': selectionsTitle
      });
    },
    onSidebarAnnotationClose: function() {
      if (this.flatmapReady) {
  
      }
    },
    onOpenPubmedUrl: function (url) {
      // GA Tagging
      // Event tracking for open pubmed url from popup
      Tagging.sendEvent({
        'event': 'interaction_event',
        'event_name': 'portal_maps_pubmed_url',
        'file_path': url,
        'location': 'map_popup_button',
      });
    },
    displayTooltip: function (info) {
      if (info) {
        let name = info.name;
        if (name) {
          this.search(name);
        } else {
          const flatmap = this.$refs.multiflatmap.getCurrentFlatmap();
          flatmap.mapImp.clearSearchResults();
        }
      }
    },
    zoomToFeatures: function (info, forceSelect) {
      return null;
    },
    highlightFeatures: function (info) {
      return null;
    },
    updateProvCard: function() {
      return null;
    },
    flatmapChanged: async function (activeSpecies) {

    },
    multiFlatmapReady: function (flatmap) {
      if (flatmap) {
        this.flatmapReady = true;
        const flatmapImp = undefined
        EventBus.emit("mapLoaded", flatmap);
      }
    },
    getFlatmapImp: function () {
      if (this.entry.type === "MultiFlatmap" && this.flatmapReady && this.$refs.multiflatmap) {
        return this.$refs.multiflatmap.getCurrentFlatmap()["mapImp"];
      } else {
        return undefined;
      }
    },
    flatmapAreaSearch() {
      return null;
    },
    restoreFeaturedMarkers: function (flatmap) {

      return null;
    },
    // updateFeaturedMarkers will step through the featured markers and add them to the map
    updateFeaturedMarkers: function (markers, flatmap) {
      return null;
    },
    // addFeaturedMarker: add a featured marker to the map at the specified uberon location
    addFeaturedMarker: function (marker, index, flatmap) {
      return false;
    },
    /**
     * Change the view mode of the current flatmap
     */
    changeViewingMode: function (modeName) {
      return null;
    },
    showConnectivity: function (payload) {
      if (this?.alive && this.flatmapReady && this.$refs.multiflatmap) {
        const { featureIds, offset } = payload;
        const currentFlatmap = this.$refs.multiflatmap.getCurrentFlatmap();
        if (currentFlatmap) {
          currentFlatmap.moveMap(featureIds, {
            offsetX: offset ? -150 : 0,
            zoom: 4,
          });
        }
      }
    },
    showConnectivityTooltips: function (payload) {
      if (this?.alive && this.flatmapReady) {
        const flatmap = this.$refs.multiflatmap.getCurrentFlatmap();
        flatmap.showConnectivityTooltips(payload);
      }
    },
    showConnectivitiesByReference: function (payload) {
      if (this?.alive && this.flatmapReady && this.$refs.multiflatmap) {
        const currentFlatmap = this.$refs.multiflatmap.getCurrentFlatmap();
        if (currentFlatmap) {
          currentFlatmap.showConnectivitiesByReference(payload);
        }
      }
    },
    changeConnectivitySource: function (payload) {
      return null;
    },
    updateViewerSettings: function () {
      return null;
    },
    setVisibilityFilter: function (payload) {
      return null;
    },
    onMapSelectionChange: function(value) {
      console.log('Selected map:', value);
      this.selectedMap = value;
      this.imageError = false; // Reset error state when changing maps
      // You can emit an event or call other methods when the map selection changes
      this.$emit('map-changed', value);
    },
    onImageError: function() {
      this.imageError = true;
      console.warn(`Image not found: ${this.selectedMapImage}`);
    },
    onImageLoad: function() {
      this.imageError = false;
      console.log(`Image loaded successfully: ${this.selectedMapImage}`);
    },
  },
  computed: {
    facetSpecies() {
      return this.settingsStore.facets.species;
    },
    featuredMarkers() {
      return this.settingsStore.featuredMarkers;
    },
    selectedMapImage() {
      const option = this.mapOptions.find(opt => opt.value === this.selectedMap);
      return option ? option.image : '';
    },
    selectedMapLabel() {
      const option = this.mapOptions.find(opt => opt.value === this.selectedMap);
      return option ? option.label : this.selectedMap;
    }
  },
  watch: {
    featuredMarkers: function (markers) {
      if (!this.flatmapReady) {
        return;
      }
    },
  },
  mounted: function () {
    this.multiFlatmapReady()
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">

.viewer-container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.map-selector-container {
  padding: 16px;

  display: flex;
  align-items: center;
  gap: 12px;
}

.map-display-area {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  overflow: hidden;
}

.placeholder-text {
  font-size: 18px;
  color: #666;
  text-align: center;
}

.image-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.system-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.error-message {
  text-align: center;
  color: #e74c3c;
  font-size: 16px;
  
  small {
    color: #666;
    font-size: 12px;
  }
}

:deep(.maplibregl-popup) {
  z-index: 11;
}

:deep(.maplibregl-marker) {
  &.standard-marker {
    cursor: pointer !important;
    z-index: 2;
  }
  &.highlight-marker {
    visibility: visible !important;
    cursor: pointer !important;
    z-index: 1;
    div {
      scale: 0.5;
      width: 0;
    }
  }
}

</style>

<style src="../../assets/mapicon-species-style.css"></style>