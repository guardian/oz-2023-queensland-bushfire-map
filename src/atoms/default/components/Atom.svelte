<script>
  import { windowInnerWidth, windowInnerHeight } from '$lib/stores/dimensions.js';
  import * as d3 from "d3"
  import dayjs from 'dayjs'
  import utc from 'dayjs/plugin/utc'
  import * as topojson from "topojson"
  dayjs.extend(utc)
  export let name = "atom";
  export let mapSettings
  let data, places = null
  
  import { onMount } from "svelte";
  let ratio = 1
  let width = 620
  let height = width / ratio
  let loading = true
  
  
  onMount(() => {
  Promise.all([
    d3.json('__assetsPath__/data.json'),
    d3.json('__assetsPath__/places.json')
    ])
  .then((results) =>  {

	  data = results[0]

    // data.forEach(function(d) {
    //   d.lat = +d.latitude;
    //   d.lon = +d.longitude;
    //   d.date = dayjs.utc(d.time, "YYYY-MM-DD HHmm")
	  // })

	  places = results[1]
    loading = false
    // console.log(data)
    let settings = mapSettings.features[0]
    var mobile = (width < 861) ? true : false;
    ratio = settings.properties.width / settings.properties.height
    height = width / ratio
    console.log("ratio", ratio)
   
    var imageObj = new Image()
    imageObj.src = `__assetsPath__/${settings.properties.basemap_image}`
    
    function makeMap() {
      height = width / ratio
      console.log(width, height, ratio)
      console.log("making map?")               
      var filterPlaces = places.features.filter((d) => (mobile) ? d.properties.scalerank < 10 : d.properties.scalerank < 8 );

	    var context = d3.select("#mapContainer canvas").node().getContext("2d"); 	   

      var gradient = d3.scaleLinear()
						.range(['rgba(219, 0, 14, 1)', 'rgba(0, 0, 0, 1)'])
						.domain([0,48])

      let projection = d3.geoMercator()
            .scale(1)
            .translate([0,0])	
      
      projection.fitSize([width, height], settings);

      var nw = projection(settings.geometry.coordinates[0][0])
      var se = projection(settings.geometry.coordinates[0][2])    
      var sx = 0
      var sy = 0
      var sw = settings.properties.width
      var sh = settings.properties.height
      var dx = 0
      var dy = 0
      var dw = width
      var dh = height

      // -30.11827166186912, 152.775403734333
      // -30.118202549096353, 152.77148863814185
      var point2 = projection([152.775403734333,-30.11827166186912])[0]
      var point1 = projection([152.77148863814185,-30.118202549096353])[0]

      var rCircle = (point2 - point1)
      const pathGenerator = d3.geoPath(projection, context);

      // var totPixels = (width * height);
      // var totWorldMeters = 510000000000;
      // var metersPerPixel = (totWorldMeters / totPixels);
      // console.log(metersPerPixel)
      // var scaledRadius;
      // scaledRadius = (1000 * (375 / metersPerPixel));
      // Load basemap

      function drawMap() {
            console.log("draw map")
            context.clearRect(0,0,width,height);
            context.drawImage(imageObj, sx, sy, sw, sh, dx, dy, dw, dh); 

            filterPlaces.forEach(function(d,i) {
              context.beginPath();
              context.save();
              context.fillStyle="#767676";
              // context.strokeStyle = 'white';
              context.shadowColor="white";
              context.shadowBlur=5;
              // context.strokeText(d.properties.name,projection([d.properties.longitude,d.properties.latitude])[0],projection([d.properties.longitude,d.properties.latitude])[1]);
              context.fillText(d.properties.name,projection([d.properties.longitude,d.properties.latitude])[0],projection([d.properties.longitude,d.properties.latitude])[1]);
              context.font = "15px 'Guardian Text Sans Web' Arial";
                context.closePath();
                context.restore();
            })
     
            context.beginPath();
            pathGenerator(topojson.feature(data, data.objects.data));
            context.fillStyle = 'rgba(219, 0, 14, 1)'
            context.fill();
      
      }

      
  
      drawMap()
          
     
    

    } 
    imageObj.onload = function() {
      makeMap()
    };
   

    var to=null
      var lastWidth = document.querySelector("#mapContainer").getBoundingClientRect()
      window.addEventListener('resize', function() {
        var thisWidth = document.querySelector("#mapContainer").getBoundingClientRect()
        if (lastWidth != thisWidth) {
          window.clearTimeout(to);
          to = window.setTimeout(function() {
            makeMap()
            }, 100)
        }
      
      })

  });

  })
</script>

<svelte:window bind:innerWidth={$windowInnerWidth} bind:innerHeight={$windowInnerHeight} />

<div class="atom interactive-wrapper">
  <div id="outer-wrapper" class="inline">
  <div class="row">
    <div id="controls" class="main-col">
            <div class="row figureTitle">
              Bushfires in southern Queensland
            </div>
            <div class="row subTitle">
              Showing bushfire areas based on hotspots detected by satellite since 1 October. The size and shape of the fires is approximate only
            </div>	
    </div>
  </div>
  
  <div id="mapContainer" bind:clientWidth={width}> 
        {#if loading}
          <div id="statusMessage" class="btn">Loading...</div>
        {/if}
        <canvas width={width} height={height}></canvas>
  </div>

  <div class="row notes">
      <div class="main-col">
          <div class="row">
              Guardian graphic | Source: <a href="https://firms.modaps.eosdis.nasa.gov/active_fire/#firms-shapefile">Nasa</a>
          </div>
      </div>
  </div>
</div>
</div>

<style lang="scss">
  h2 {
    @include f-headline();
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 20px;
  }
</style>
