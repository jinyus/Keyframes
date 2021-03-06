<!--
// 
// Shadows.vue
// _________________________
//
//	Generate CSS shadows
//
// 
// -->
<!--
// Shadows.vue
// _________________________
//
//	Generate CSS box-shadows. Use slides to adjust offset, blur, spread. Change colors, add multiple layers
//
//		Layout
// 			Split right-sidebar, sliders on right, shadow element on left. Option bar on top.
//
//		Scripts
//			Computed
//				searchCharacters - Returns filtered character list matching search - based on name
//			Methods
//				calculateShadow() Takes values and calculates layers for shadows
//					Only runs if layers is more than 1.
//				rgbNumberFormat(input) Pass it a number, it will return 0 if less than 0, or 255 if more
// 					Just a quick fix for people who format badly.
//				copyShadowOutput() Copies generated shadow CSS to clipboard
//				addLayer() Adds new shadow layer
//				deleteLayer() Deletes selected layer
//				tiltShadow() Detects compatibility (prompts permission on ios) then calls toggleTilt. Tilting device moves shadow.
//				toggleTilt() Called with option button click - only shown below LG and on touch devices
// -->
<template>
	<div class="app-page-split">

		<!--/////////////////////////////
			Left side of page - top bar, stage
		/////////////////////////////-->
		<div class="app-page-split-left">
			<div class="app-page-split-left-top">

				<!--/////////////////////////////
					Dropdown buttons - options, output
				/////////////////////////////-->
				<div class="option-dropdown">
					<button class="button small action-btn" @click="controlTab('options')" v-bind:class="{'red': controlToggles.options, 'black': !controlToggles.options}">
						<i v-bind:class="{'fas fa-edit': !controlToggles.options, 'fas fa-times-circle': controlToggles.options}"></i>
						<span>Options</span>
					</button>
					<button class="button small action-btn" @click="controlTab('output')" v-bind:class="{'red': controlToggles.output, 'green': !controlToggles.output}">
						<i v-bind:class="{'fas fa-brackets-curly': !controlToggles.output, 'fas fa-times-circle': controlToggles.output}"></i>
						<span>Get CSS</span>
					</button>
					<!-- Spacer Flex grow -->
					<div class="flex-grow"></div>
					<!-- Tilt mode button - only shown if touch sreen and device orientation sensors -->
					<button class="button small action-btn" @click="toggleTilt()" v-bind:class="{'red': tiltMode, 'blue': !tiltMode}" v-if="$store.getters.device.hasTouch">
						<i v-bind:class="{'fas fa-atom-alt': !tiltMode, 'fas fa-times-circle': tiltMode}"></i>
						<span>Tilt Mode</span>
					</button>

					<!--/////////////////////////////
						Dropdown Content
					/////////////////////////////-->
					<!-- Options -->
					<transition name="fromtop">
						<div class="option-dropdown-content" v-if="controlToggles.options">
							<div class="field horizontal">
								<label for="stageBackground">Background</label>
								<input type="text" id="stageBackground" v-model="options.stageBackground" placeholder="#FFFFFF"/>
							</div>
							<!-- Target Element -->
							<div class="field mtop-sm">
								<label for="customShadowTarget">
									Target Element
									<small class="block">Your element must have the id targetElement</small>
								</label>
								<textarea class="code" id="customShadowTarget" v-model="options.customTarget"></textarea>
							</div>
							<!-- Target CSS -->
							<div class="field mtop-sm">
								<label for="customShadowTargetCSS">
									Custom CSS
								</label>
								<textarea class="code" id="customShadowTargetCSS" v-model="options.customTargetCSS"></textarea>
							</div>
						</div>
					</transition>


					<!-- Output -->
					<transition name="fromtop">
						<div class="option-dropdown-content" v-if="controlToggles.output">
							<code class="wrap">
								<pre>
									box-shadow: 
									<div v-for="(layer, index) in layers" :key="index">{{layer.horizontal_offset}}px {{layer.vertical_offset}}px {{layer.blur}}px {{layer.spread}}px {{layer.color}}{{index != layers.length - 1 ? ', ' : ';'}}</div>
								</pre>
							</code>

							<!-- Copy to clipboard -->
							<button class="button green" @click="copyShadowOutput()">
								<i class="far fa-copy"></i>
								<span>Copy to Clipboard</span>
							</button>
						</div>
					</transition>
				</div>
			</div> <!-- End page split left top -->


			<!--/////////////////////////////
				Stage
			/////////////////////////////-->
			<div class="app-stage" v-html="options.customTarget">
			</div>
			
		</div> <!-- End page left -->



		<!--//////////////////////
			//////////////////////
			Right side - value editor
			//////////////////////
			//////////////////////-->
		<div class="app-page-split-right">
			<!-- Fields -->
			<div class="app-fields">

				<h4 class="mbottom-sm">Box Shadow</h4>

				<!-- Horizontal Offset -->
				<div class="field">
					<label for="horOffset" class="slider-label-value">
						<span>Horizontal Offset</span>
						<input type="number" v-model="layers[selectedLayer].horizontal_offset" min="-200" max="200"/>
						<!-- <b>{{options.horizontal_offset}}px</b>	 -->
					</label>
					<input type="range" id="horOffset" v-model="layers[selectedLayer].horizontal_offset" min="-200" max="200"/>
				</div>

				<!-- Vertical Offset -->
				<div class="field mtop-sm">
					<label for="vertOffset" class="slider-label-value">
						<span>Vertical Offset</span>
						<input type="number" v-model="layers[selectedLayer].vertical_offset" min="-200" max="200"/>
					</label>
					<input type="range" id="vertOffset" v-model="layers[selectedLayer].vertical_offset" min="-200" max="200"/>
				</div>

				<!-- Blur -->
				<div class="field mtop-sm">
					<label for="shadowBlur" class="slider-label-value">
						<span>Blur</span>
						<input type="number" v-model="layers[selectedLayer].blur" min="0" max="100"/>
					</label>
					<input type="range" id="shadowBlur" v-model="layers[selectedLayer].blur" min="0" max="100"/>
				</div>

				<!-- Spread -->
				<div class="field mtop-sm">
					<label for="shadowSpread" class="slider-label-value">
						<span>Spread</span>
						<input type="number" v-model="layers[selectedLayer].spread" min="-200" max="200"/>
					</label>
					<input type="range" id="shadowSpread" v-model="layers[selectedLayer].spread" min="-200" max="200"/>
				</div>

				<!-- Color -->
				<div class="field mtop-sm">
					<label for="shadowSpread">
						Shadow Color
					</label>
					<input type="text" id="shadowColor" v-model="layers[selectedLayer].color" @change="calculateShadow('color')"/>
				</div>

				<!-- Color Opacity -->
				<div class="field mtop-sm mbottom-sm">
					<label for="shadowTransparency" class="slider-label-value">
						<span>Color Opacity</span>
						<input type="number" v-model="layers[selectedLayer].opacity" min="0" max="1"/>
					</label>
					<input type="range" id="shadowTransparency" v-model="layers[selectedLayer].opacity" min="0" max="1" step="0.01" @input="calculateShadow('color')"/>
				</div>

				<!-- Layers -->
				<div class="field mtop-md mbottom-sm">
					<label>
						<span>Layers</span>
					</label>
					<!-- Layer controls -->
					<div class="layer-selector">
						<div class="layer-button" v-for="(layer, index) in layers" :key="index" :class="{'active': selectedLayer == index}" @click="selectedLayer = index">{{index + 1}}</div>
						<!-- Add layer -->
						<div class="layer-button" @click="addLayer()">
							<i class="far fa-plus"></i>
						</div>
					</div>

					<!-- Delete Layer -->
					<button class="button small red mtop-sm mbottom-sm" @click="deleteLayer()" v-if="layers.length > 1">
						<i class="far fa-trash-alt"></i>
						<span>Delete Layer</span>
					</button>
				</div>

			</div>
		</div>

		<!-- Append styles to page - options, shadow output, etc -->
		<v-style>
			{{options.customTargetCSS}}
			.app-stage{
				background: {{options.stageBackground}};
			}
			#targetElement{
				box-shadow:
				<template v-for="(layer, index) in layers">
					{{layer.horizontal_offset}}px {{layer.vertical_offset}}px {{layer.blur}}px {{layer.spread}}px {{layer.color}} {{index != layers.length - 1 ? ', ' : ';'}}				
				</template>
			}
		</v-style>


	</div>
</template>

<script>
// @ is an alias to /src
import metaMixin from "@/components/mixins/metaMixin.js";

export default {
	name: "shadowApp",

	components: {
	},

	mixins: [
		metaMixin,
	],

	data() {
		return {
			selectedLayer: 0,
			// Fun tilt mode on phones
			tiltMode: false,
			layers: [
				{
					horizontal_offset: 0,
					vertical_offset: 15,
					blur: 25,
					spread: -4,
					color: "rgba(30,30,60,0.25)",
					opacity: 0.25,
				},
			],
			options: {
				stageBackground: "white",
				customTarget: "<div id='targetElement' class='shadow-target'><i class='fal fa-hands-wash'></i></div>",
				customTargetCSS: "#targetElement{\n    display:block;\n    height: 140px;\n    width: 200px;\n    background-color: #ffffff;\n    border:var(--borderWidth) solid #f2f5f9;\n    color: #16023C;\n    border-radius: 3px;\n    margin: 0 auto;\n    font-size: 62px;\n    display: flex;\n    flex-direction: column;\n    justify-content: center;\n    text-align: center;\n}",
			},
			// Which controls are visible
			controlToggles: {
				options: false,
				output: false,
			},
		};
	},

	mounted() {
		this.updateMeta("Shadows | Keyframes.app", "Keyframes gives you a visual timeline to help you create, view, and run animations without having to go back and forth between your browser and editor.")
		// console.log(convert.hex.rgb('DEADBF'))
	},

	computed: {
	},

	watch: {
		
	},
	
	methods: {
		//////////////////////////
		// Switch Control Tab  //
		////////////////////////
		controlTab: function(tab){
			if(tab == 'options' && !this.controlToggles.options){
				this.controlToggles.options = true;
				this.controlToggles.output = false;
			}else if(tab == 'output' && !this.controlToggles.output){
				this.controlToggles.output = true;
				this.controlToggles.options = false;
			}else{
				this.controlToggles.options = false;
				this.controlToggles.output = false;
			}
		},
		
		/////////////////////////
		// Calculates Shadow  //
		///////////////////////
		// Runs shadow calculation - needed for multiple layers
		calculateShadow: function(type){

			var color = this.layers[this.selectedLayer].color;


			// Color only needs to update when color or opacity changes
			if(type == 'color'){

				var formattedColor = "";
				var rgbArray = []
				
				// Convert hex to rgb
				if(color.startsWith("#")){
					rgbArray = convert.hex.rgb(color.substring(1,7));
					this.layers[this.selectedLayer].opacity = 1;
				// Get numbers from RGB format
				}else{
					var numbers = color.replace(/[^\d,]/g, '').split(',')
					// If there are at least 3 numbers, good enough format
					if(numbers[2]){
						rgbArray = numbers;
					}else{
						this.toast("Bad Color Format", "Please enter your color in either rgb() or #HEX format.", "red", "far fa-tint");
					}
				}

				// Now actually format the string
				formattedColor = "rgba("
					+ this.rgbNumberFormat(rgbArray[0]) + ", "
					+ this.rgbNumberFormat(rgbArray[1]) + ", "
					+ this.rgbNumberFormat(rgbArray[2]) + ", "
					+ this.layers[this.selectedLayer].opacity + ")";

				// Set to options
				this.layers[this.selectedLayer].color_format = formattedColor;
				this.layers[this.selectedLayer].color = formattedColor;

				// Update color input
			}

			// Calculate rest

			

		},

		/////////////////////////
		// RGB Number Format  //
		///////////////////////
		// RGB Fix - returns 255 if more, or 0 if less
		rgbNumberFormat: function(input){
			var number = parseInt(input)
			if(number > 255){
				return 255;
			}else if(number < 0){
				return 0;
			}else{
				return number;
			}
		},

		/////////////////////////
		// Copy Shadow Output  //
		///////////////////////
		// copies CSS output to clipboard
		copyShadowOutput: function(){
			var output = "box-shadow: ";

			for (var index = 0; index < this.layers.length; index++) { 
				var l = this.layers[index];
				output = output + 	l.horizontal_offset + "px " +
					l.vertical_offset + "px " +
					l.blur + "px " +
					l.spread + "px " +
					l.color;

				// End line if end, otherwise comma
				if(index == this.layers.length - 1){
					output = output + ";"
				}else{
					output = output + ", ";
				}
			} 
			
			this.copyToClipboard("The shadow CSS", output);
		},

		/////////////////////
		// Add new Layer  //
		///////////////////
		addLayer: function(){
			// Copy selected layer to new layer
			var copyLayer = _.cloneDeep(this.layers[this.selectedLayer]);
			// Push copied layer to layers arrray
			this.layers.push(copyLayer);
			// Select new layer
			this.selectedLayer = this.layers.length - 1;
		},

		/////////////////////
		// Delete  Layer  //
		///////////////////
		deleteLayer: function(){

			// Select prev color unless 0
			if(this.selectedLayer != 0){
				this.selectedLayer--;
			}
			
			// Delete from pallete
			this.layers.splice(this.selectedLayer, 1);
		},

		////////////////////////
		// Toggle Tilt Mode  //
		//////////////////////
		tiltShadow:function(event){
			// x/hor y/vert
			var h, v;

			// Check for value
			// Webkit
			if(event.beta || event.gamma){
				v = Math.round(event.beta) * 2;
				h = Math.round(event.gamma) * 2;
			}else if(event.y || event.x){
				// Firefox uses orientation instead of event
				v = Math.round(orientation.y) * 2;
				h = Math.round(orientation.x) * 2;
			}

			// Make adjustments for orientation
			var deg = 0;
			// Device orientation
			if(window.orientation){
				deg = window.orientation;
			}

			var hOffset, vOffset;
			// Regular format is (h, v)
			// 90deg is -v,h
			if(deg == 90){
				hOffset = v;
				vOffset = -h;
			}else if(deg == 180){
				// 180deg is -h,-v
				hOffset = -h;
				vOffset = -v;
			}else if(deg == -90){
				// -90 is -v,-h
				hOffset = -v;
				vOffset = h;
			}else{
				// Regular orientation
				hOffset = h;
				vOffset = v;
			}


			// Set to shadow
			// Vertical offset has -30 because of how people naturally hold their devices.
			this.layers[this.selectedLayer].vertical_offset = (-vOffset) + 70;
			this.layers[this.selectedLayer].horizontal_offset = -hOffset;
		},
		toggleTilt: function(){

			var _this = this;

			// Get sensor type if its available
			var sensor = "";

			var addListener = function(sensor){
				_this.tiltMode = true;
				window.addEventListener(sensor, _this.tiltShadow)
			}
			
			// If it's already running, stop it
			if(_this.tiltMode){
				window.removeEventListener("deviceorientation", _this.tiltShadow); 
				window.removeEventListener("MozOrientation", _this.tiltShadow); 
				_this.tiltMode = false;
				this.tiltValues = [null, null];
			}
			// Else start it
			else{
				// iOS requires permission request before
				if (typeof DeviceOrientationEvent.requestPermission === 'function') {
					
					DeviceOrientationEvent.requestPermission()
					.then(permissionState => {
						if (permissionState === 'granted') {
							// _this.tiltMode = true;
							// window.addEventListener("deviceorientation", _this.tiltShadow)
							addListener("deviceorientation");
							_this.hello("Tilt your device to adjust the shadow.", "far fa-sync-alt")
						}else{
							_this.hello("Device orientation permission denied.", "far fa-times-circle")
						}
					})
					.catch(console.error);
				}
				// No permission needed for other OSes, just check for type
				else if (window.DeviceOrientationEvent){
					// Webkit
					addListener("deviceorientation");
					_this.hello("Tilt your device to adjust the shadow.", "far fa-sync-alt")
				}else if (window.MozOrientation){
					// Firefox
					addListener("MozOrientation");
					_this.hello("Tilt your device to adjust the shadow.", "far fa-sync-alt")
				}else{
					// Error 
					_this.hello("We can't access your device's sensors right now.", "far fa-exclamation-triangle")
				}
			}
		},
		
	}
};


</script>


<style lang="less">

	@import '~@/styles/variables.less';


	// Specific element styling
	#customShadowTarget{
		height: 80px;
		max-height: 80px;
		min-height: 80px;
	}
	#customShadowTargetCSS{
		height: 150px;
		max-height: 150px;
		min-height: 150px;
	}
	#shadowOutput{
		display: block;
		margin-top: 45px;

		code{
			background: var(--blue);
			margin-top: 15px;
		}
	}

</style>