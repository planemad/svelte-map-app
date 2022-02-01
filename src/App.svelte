<script>

	import { onMount, setContext } from "svelte";
	import { contextKey } from "./mapbox.js";

	import { freeGeoIP } from "./lib/userLocale.js";
	import { Map, Geocoder, controls } from "@beyonk/svelte-mapbox";

	export let mapbox, map, geocoder;

export let appConfig = {
	map: {
		accessToken: "pk.eyJ1IjoicGxhbmVtYWQiLCJhIjoiY2l3ZmNjNXVzMDAzZzJ0cDV6b2lkOG9odSJ9.eep6sUoBS0eMN4thZUWpyQ",
		configUrl: "https://gist.githubusercontent.com/planemad/62be758755320cc052eecb8c8bde01bf/raw/9b6bcf880b83397167931975eed42fdb4762b85b/sample-map-config.js"
	}

}
	setContext(contextKey, {
		getMap: () => map,
		getMapbox: () => mapbox,
	});

	//
	// Initialize Mapbox compnent
	//

	onMount(() => {

	freeGeoIP((resp) => {
			console.log(resp)
			map.setCenter([resp.longitude, resp.latitude]);
		});

    // DEBUG: appConfig
    // console.log($settings,appConfig);

    // Load map config from external JSON
    if (appConfig.map.configUrl) {
      fetch(appConfig.map.configUrl)
        .then((resp) => resp.json())
        .then((data) => {
          Object.assign(appConfig.map, data);
        });
    }

    if (!window.location.hash) appConfig.map.autoLocate = true;

    const link = document.createElement("link");
    link.rel = "stylesheet";
    link.href =
      "https://raw.githack.com/korywka/mapbox-gl-controls/master/docs/controls.css";
    document.body.appendChild(link);


    return () => {
      map.remove();
      link.parentNode.removeChild(link);
    };
  });

	//
  // Map state change handlers
  //

  function onGeocoderResult(e) {
    map.fitBounds(e.detail.result.bbox);
    setLocationContext({
      lng: e.detail.result.center[0],
      lat: e.detail.result.center[1],
    });
  }

  function onGeolocate(e) {
    setLocationContext();
  }

  function onMapReady(e) {
    map = mapbox.getMap();
    mapbox = mapbox.getMapbox();

    // Enable RTL support
    // https://docs.mapbox.com/mapbox-gl-js/example/mapbox-gl-rtl-text/
    mapbox.setRTLTextPlugin(
      "https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-rtl-text/v0.2.3/mapbox-gl-rtl-text.js",
      null,
      true // Lazy load the plugin
    );
  }

</script>

<main>
	<div id="map-overlay">
		<div class="uk-width-large uk-grid-small" uk-grid>
			<div>
				<div
					class="uk-card uk-card-default uk-card-body uk-padding-small"
				>
					<h3 class="uk-card-title">World Water Map</h3>
					<p>
						as
					</p>
					<div class="uk-position-relative">
						{#if null}
						  <div
						  >
							{#await null}
							  <span>...</span>
							{:then result}
							  {#each result.features as feature}
								{#if feature.place_type.indexOf("country") > -1 || (feature.place_type.indexOf("region") > -1 && map.getZoom() > 6) || (feature.place_type.indexOf("district") > -1 && map.getZoom() > 9)}
								  <a
									target="_blank"
									style="color:grey;text-decoration: underline"
									href="https://www.wikidata.org/wiki/{feature.properties
									  .wikidata}"
								  >
									{feature.text}</a
								  >{feature.place_type.indexOf("country") == -1 ? ", " : ""}
								{/if}
							  {/each}
							{/await}
						  </div>
						{/if}
				  
						<Geocoder
						  bind:geocoder
						  accessToken={appConfig.map.accessToken}
						  types={[
							"country",
							"region",
							"postcode",
							"district",
							"place",
							"locality",
							"neighborhood",
						  ]}
						  options={{
							clearOnBlur: true,
							collapsed: true,
							marker: true,
							mapboxgl: mapbox,
						  }}
						  on:result={onGeocoderResult}
						/>
					  </div>
				</div>
			</div>
			<div>
				<div class="uk-card uk-card-default uk-card-body">Item</div>
			</div>
		</div>
	</div>

	<div id="map">
		<Map
			accessToken={appConfig.map.accessToken}
			bind:this={map}
			on:recentre={(e) =>
				console.log(e.detail.center.lat, e.detail.center.lng)}
			options={{ projection: "naturalEarth" }}
			style="mapbox://styles/planemad/ckd42fwa20n531iqrewerwla1"
			version="v2.6.1"
		/>
	</div>
</main>

<style>
	#map-overlay {
		position: absolute;
		margin: 10px;
		z-index: 1;
	}

	#map {
		height: 100vh;
		width: 100%;
		z-index: -1;
	}
</style>
