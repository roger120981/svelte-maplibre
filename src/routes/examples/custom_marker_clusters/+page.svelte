<script lang="ts">
  import MapLibre from '$lib/MapLibre.svelte';
  import GeoJSON from '$lib/GeoJSON.svelte';
  import CodeSample from '$site/CodeSample.svelte';
  import code from './+page.svelte?raw';
  import { mapClasses } from '../styles';
  import Popup from '$lib/Popup.svelte';
  import ClusterPopup from '../ClusterPopup.svelte';
  import clusterPopupCode from '../ClusterPopup.svelte?raw';
  import MarkerLayer from '$lib/MarkerLayer.svelte';
  import quakeImageUrl from '$site/earthquake.png';
  import tsunamiImageUrl from '$site/tsunami.png';
  import earthquakes from '$site/earthquakes.geojson?url';
  import type {
    ClusterFeatureProperties,
    ClusterProperties,
    SingleProperties,
  } from '../cluster_feature_properties.js';

  let clickedFeature: ClusterFeatureProperties | undefined = $state();

  let openOn: 'click' | 'dblclick' | 'contextmenu' | 'hover' = $state('hover');
</script>

<p>
  Data and layer configuration derived from <a
    href="https://maplibre.org/maplibre-gl-js-docs/example/cluster/">MapLibre Cluster Example.</a
  >
</p>

<fieldset class="mb-2 flex gap-x-4 self-start border border-gray-300 px-2">
  <legend>Show popup on</legend>
  <label><input type="radio" bind:group={openOn} value="hover" /> Hover</label>
  <label><input type="radio" bind:group={openOn} value="click" /> Click</label>
  <label><input type="radio" bind:group={openOn} value="dblclick" /> Double Click</label>
  <label
    ><input type="radio" bind:group={openOn} value="contextmenu" /> Context Menu (right-click)</label
  >
</fieldset>

<MapLibre
  style="https://basemaps.cartocdn.com/gl/positron-gl-style/style.json"
  class={mapClasses}
  zoomOnDoubleClick={openOn !== 'dblclick'}
  standardControls
>
  <GeoJSON
    id="earthquakes"
    data={earthquakes}
    cluster={{
      radius: 50,
      maxZoom: 14,
      properties: {
        // Sum the `mag` property from all the points in each cluster.
        total_mag: ['+', ['get', 'mag']],
      },
    }}
  >
    <MarkerLayer applyToClusters asButton onclick={(e) => (clickedFeature = e.feature?.properties)}>
      {#snippet children({
        feature,
      }: {
        feature: GeoJSON.Feature<GeoJSON.Point, ClusterProperties>;
      })}
        <div class="rounded-full bg-orange-200 p-1">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"
            ><path
              fill="currentColor"
              d="M14 11.5A2.5 2.5 0 0 0 16.5 9A2.5 2.5 0 0 0 14 6.5A2.5 2.5 0 0 0 11.5 9a2.5 2.5 0 0 0 2.5 2.5M14 2c3.86 0 7 3.13 7 7c0 5.25-7 13-7 13S7 14.25 7 9a7 7 0 0 1 7-7M5 9c0 4.5 5.08 10.66 6 11.81L10 22S3 14.25 3 9c0-3.17 2.11-5.85 5-6.71C6.16 3.94 5 6.33 5 9Z"
            /></svg
          >
        </div>
        <Popup {openOn} closeOnClickInside>
          <ClusterPopup {feature} />
        </Popup>
      {/snippet}
    </MarkerLayer>

    <MarkerLayer
      applyToClusters={false}
      asButton
      onclick={(e) => (clickedFeature = e.feature?.properties)}
    >
      {#snippet children({
        feature,
      }: {
        feature: GeoJSON.Feature<GeoJSON.Point, SingleProperties>;
      })}
        <img src={feature.properties?.tsunami ? tsunamiImageUrl : quakeImageUrl} alt="Earthquake" />
        <Popup {openOn} closeOnClickInside>
          {@const props = feature.properties}
          <p>
            Date: <span class="font-medium text-gray-800"
              >{new Date(props?.time).toLocaleDateString()}</span
            >
          </p>
          <p>Magnitude: <span class="font-medium text-gray-800">{props?.mag}</span></p>
          <p>
            Tsunami: <span class="font-medium text-gray-800">{props?.tsunami ? 'Yes' : 'No'}</span>
          </p>
        </Popup>
      {/snippet}
    </MarkerLayer>
  </GeoJSON>
</MapLibre>

{#if clickedFeature}
  {#if clickedFeature.cluster}
    <p>
      Number of Earthquakes:
      <span class="font-bold text-gray-800">{clickedFeature['point_count']}</span>
    </p>
    <p>
      Average Magnitude:
      <span class="font-bold text-gray-800">
        {(clickedFeature.total_mag / clickedFeature.point_count).toFixed(2)}
      </span>
    </p>
  {:else}
    <p>Magnitude: <span class="font-bold text-gray-800">{clickedFeature.mag}</span></p>
  {/if}
{/if}

<CodeSample {code} endBoundary="<CodeSample" omitEndBoundary />
<CodeSample
  code={clusterPopupCode}
  filename="ClusterPopup.svelte"
  startBoundary=""
  endBoundary=""
/>

<footer class="self-start">
  <p>
    <a
      class="text-sm"
      href="https://www.flaticon.com/free-icons/earthquake"
      title="earthquake icons">Earthquake icons created by Freepik - Flaticon</a
    >
  </p>
  <p>
    <a class="text-sm" href="https://www.flaticon.com/free-icons/tsunami" title="tsunami icons"
      >Tsunami icons created by surang - Flaticon</a
    >
  </p>
</footer>
