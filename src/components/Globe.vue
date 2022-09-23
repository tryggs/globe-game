<script >
  import countries from '../assets/countries.js'
export default{
  props: ['foo'],
  data() {
    return {
      countries: countries(),
      countryToPlay: null,
      points: 0,
      score: 0,
      highScore: null,
      lost: false,
      playedCountries: [],
      pin: null,
      msgs: []
    }
  },
  watch: {
    points(newPoints) {
      if (newPoints > this.score){
        this.score = newPoints
      }
    },
  },
  methods: {
    load() {
      let globe
      let vm = this



      const deleteOverlay = (overlay, seconds) => {
      setTimeout(()=> {
      overlay.remove()
      },seconds*1000)
      }
      const blinkCountry = (country) => {
        globe.mapStyles = "#"+ country + " { fill: RGBA(74,128,245,1); stroke: #D6413E; }"
        globe.redrawMap()
        
        setTimeout(()=> {
          globe.mapStyles = "#"+ country + " { fill: #bbdaa4; stroke: #a7cdf2; }"
          globe.redrawMap()
        }, 5000)
      }
      
       globe = this.globe = new Earth("element", {
        location : {lat: 18, lng: 50},
        zoom: 0.8,
        light: 'sun',
        shadows:true,
        opacity:0.85,
        shininess: 0.3,
        quality: 6,
        transparent : true,
        mapSeaColor : 'RGBA(74,128,245, 0.85)',
        mapLandColor : '#bbdaa4',
        mapBorderColor : '#a7cdf2',
        mapBorderWidth : 0.85,
        mapHitTest : true,
        zoomable: true,
        autoRotate: true,
        autoRotateSpeed: 1,
        autoRotateDelay: 10000,
      });

      
      
      
      this.globe.addEventListener( 'click', function(event) {
        globe.redrawMap()
        const clickedCountry = countries().filter(obj => {
          return obj.AFF_ISO == event.id
        })
        let distance = Earth.getDistance(event.location,{lat: vm.countryToPlay.latitude, lng: vm.countryToPlay.longitude})
        console.log(distance)


        if (!(vm.lost)){
          
          if(!(event.id == "SEA")){
            
            if(vm.countryToPlay.AFF_ISO == event.id){
              //rätt
              vm.points += 2
              vm.msgs.unshift({msg: "You guessed the right country! +2", positive: true})
              newRound()
            } else {
              //fel
              if (Math.floor(distance) < 3000){
                vm.points += 1
                vm.msgs.unshift({msg: "You guessed the wrong country but you where only " + Math.floor(distance) + " km away. +1", positive: true})
              } else{
                vm.points -=1
                vm.msgs.unshift({msg: "You guessed the wrong country! -1", positive: false})
              }

              setTimeout(() => {
                this.goTo( { lat: vm.countryToPlay.latitude, lng: vm.countryToPlay.longitude }, { relativeDuration: 185, approachAngle: 10, zoom: 2.1} )
              blinkCountry(vm.countryToPlay.AFF_ISO)
              setTimeout(newRound, 2500)
              }, 500);

            if (vm.points <= 0){
              vm.lost = true
            } else {
            }


            const anwserText = this.addOverlay( {
              location: event.location,
              content : "<strong> That was "+ clickedCountry[0].COUNTRY +"</strong>",
              className : 'water-error',
              depthScale : 0.5
            } );      
            deleteOverlay(anwserText,4)
            
          }
          if (!vm.pin ) {
            vm.pin = this.addMarker( {
              mesh : ["Pin2", "Needle"],
              location : event.location,
              scale: 0.01
            } );
            vm.pin.animate( 'scale', 0.9, { easing: 'out-back' } );
          } else {
        vm.pin.animate( 'location', event.location, { duration: 200, relativeDuration: 50, easing: 'in-out-cubic' } );
			}
      
    } else {
      vm.msgs.unshift({msg: "click on land to guess", positive: false})
    }
    }
  })

  const newRound = () => {
    let country = vm.countries[Math.floor(Math.random()*(vm.countries.length)) + 1]
    
  if (vm.playedCountries.includes(country.AFF_ISO)) {
  newRound()
  } else {
  this.countryToPlay = country
  vm.playedCountries.push(country.AFF_ISO)
  }
  }

  newRound()
  },
  reset(){
    if (this.score > (localStorage.getItem("highScore"))){
      localStorage.setItem("highScore", this.score)
      this.highScore = this.score
    }

    this.points = 0
    this.score = 0
    this.lost = false
    this.playedCountries = []
    this.msgs = []
    this.globe.redrawMap()


  }
},
  mounted() {
    this.load();
    this.highScore = localStorage.getItem("highScore");
  }
}
</script>

<template>
  <p>High Score: {{highScore}}</p>
  <div id="wrapper">
    <div v-if="!lost" style="text-align:center">
      <h1>Country To Find: {{ countryToPlay == null ? "..." : countryToPlay.COUNTRY }}</h1>
      <div style="display: flex; justify-content: space-around">
        <h3>Points: {{points}}</h3>
        <h3>My Current Score: {{score}}</h3>
      </div>
      <div class="msgDiv" style="width: 100%;text-align: center;margin-bottom: -3rem; height: 5.5rem;">
      <p v-for="msg in msgs" :key="msg.msg" :style="msg.positive ? 'color:#5FD63E' : 'color:#D6413E' " >{{msg.msg}}</p>
      </div>
    </div>
    <div v-if="lost" style="text-align:center">
      <h1 style="color:#D6413E">You Lost!!</h1>
      <h3 style="color:#D6413E">score: {{score}}</h3>
      <button id="startAgain"  @click="reset">start again</button>
    </div>
		<div id="earth-col">
      <div id="element" class="earth-js"></div>
		</div>	
	</div>	
</template>

<style scoped>
  #wrapper {
    margin: auto;
    width: 60%;
  }

.msgDiv{
  overflow: hidden;
  position: relative;
}
.msgDiv:after {
  content  : "";
  position : absolute;
  z-index  : 1;
  bottom   : 0;
  left     : 0;
  pointer-events   : none;
  background-image : linear-gradient(to bottom, 
                    rgba(0,0,0, 0), 
                    rgba(000,000,000, 1) 90%);
  width    : 100%;
  height   : 4em;
}
#startAgain{
  background: #D6413E;
  color: black;
  width: 12rem;
  height: 4rem;
  font-size: 1.5rem;
  font-weight: bold;
  transition: all 0.4s ease-in-out 
}
#startAgain:hover{
  background: black;
  color: #D6413E;
  border: #D6413E 5px solid;
  border-radius: 2rem;
  width: 12rem;
  height: 4rem;
  font-size: 1.5rem;
  font-weight:200;
  cursor: pointer
}
</style>
