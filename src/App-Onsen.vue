<style>
	#map-panel{

	}
	#data-panel{

	}
	/*for mobile*/
	.page-panel{
		width: 100%;
	}
	#nmw_logo{
		display: flex;
		justify-content: center;
		align-items: center;
		overflow: hidden
	}
	#nmw_logo img{
		flex-shrink: 1;
		max-width:225px;
	}
	.poly_text:hover{
		cursor: pointer;		
	}
	.map-boundary path:hover{
		cursor: pointer;
		fill:rgba(255,255,00,.25);
	}
	#map-container{
		background-color: black;
	}
	#data-panel{
		max-height:100vh;
		overflow-y: none;
		background-color: #d1e7dd;
	}
	#states_table tbody tr:hover{
		background-color: yellow;
		cursor: pointer;
	}
	#state_table{
		font-size: .75rem;
		overflow-x: scroll;
	}
	#state_table{
		overflow: auto; height: 100px;
	}
	#state_table thead th{
		position: sticky;
		top: 0;
		z-index: 1; 
		background-color: #fff;
		border-bottom: 1px solid #000;

	}
	#state_table thead th:hover{
		cursor: pointer;
		background-color: #fcc;
	}
	#state_table tbody tr:hover{
		cursor: pointer;
		background-color: #faa;
	}
	.ibp_btn{
		color: #fff;
		background-color: #0095ff;
	}
	.inat_btn{
		background-color: #f2720c;
	}
	.info-btn{
		font-size: 1.1rem !important;
		padding: 10px 10px;
	}
	.info-btn:hover{
		background-color: #0d5edd!important;
		cursor: pointer;
	}
	@media only screen and (min-width: 1000px) {
		/* For desktop: */
		.page-panel{
			width: 50%;
		}
	}

</style>

<template>
	<div id="app" class="row">
		<div class="p-0 bg-dark page-panel" id="map-panel">
			<div id="map-container" class="svg-container"></div>        	
		</div>
		<div class="d-flex flex-column page-panel" id="data-panel">
			<div id="map_controls" class="border border-success bg-light text-center row m-0">
				<div class="col p-0">
					<table class="table text-center table-sm table-success m-0">
						<thead>
							<tr v-if="selected_state != ''">
								<th class="h4 text-success align-end" v-text="selected_state + ' Data'"></th>
								<th>
									<button class="btn btn-sm btn-danger" @click="selected_state = ''">Back to All States</button>
								</th>
							</tr>
							<tr v-else>
								<th colspan=2 class="h4">
									India Data
								</th>
							</tr>
							<tr class="h1">
								<th class="col-6" v-text="card['observations'].toLocaleString()"></th>
								<th class="col-6" v-text="card['users']"></th>
							</tr>
							<tr>
								<th>Observations</th>
								<th>Users</th>
							</tr>
						</thead>
					</table>
				</div>
				<div class="col-4 h-100 m-auto bg-dark" id="nmw_logo">
					<img src="https://nationalmothweek.org/wp-content/uploads/2021/03/cropped-nmw_logos_8_edited_white.png" alt="Big Butterfly Month">
					<span class="mt-2 mb-auto badge rounded-pill bg-primary info-btn" data-bs-toggle="modal" data-bs-target="#infoModal">ⓘ</span>
				</div>    
			</div>
			<div class="d-flex flex-column bg-light">
				<div class="btn-group my-2">
							<button class="btn btn-sm mx-1 rounded" 
								v-for="btn in labels"
								v-text="capatilize(btn)"
								:class="(selected_label == btn) ? 'btn-success': 'btn-outline-primary'"
								@click="selected_label = btn"
							></button>                    	
						</div>
				<div class="btn-group my-2">
					<button class="btn btn-sm mx-1 rounded" 
						:class="selectedPortal('inat')"
						@click="selectPortal('inat')"
					>iNaturalist</button>
					<button class="btn btn-sm mx-1 rounded" 
						:class="selectedPortal('ibp')"
						@click="selectPortal('ibp')"
					>India Biodiversity Portal</button>
				</div>
				<div class="btn-group my-2">
					<button class="btn btn-sm me-1 rounded" 
						v-for="btn in taxon_levels"
						v-text="capatilize(btn)"
						:class="selectedTaxon(btn)"
						@click="selected_level = btn"
					></button>                    	
				</div>
			</div>
			<div style="overflow: scroll;" class="border border-info flex-grow-1">
				<table class="table table-sm table-striped bg-light m-0" id="states_table" v-if="selected_state == ''">
					<thead>
						<tr>
							<th>State</th>
							<th>Observations</th>
							<th>Contributors</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="state in states_table" 
							:class="(state[1]==0)?'table-danger':''"
							@click="selectState(state[0])"
						>
							<td v-text="state[0]"></td>							
							<td v-text="state[1].toLocaleString()"></td>
							<td v-text="state[2]"></td>
						</tr>
					</tbody>
				</table>
				
				<table class="table table-sm table-striped table-hover bg-light m-0" id="state_table" v-else>
					<thead>
						<tr>
							<th class="text-center" style="width: 12% !important;" @click="sortTable(1)">
								<div class="d-flex justify-content-center">
									<span>User</span>
									<span v-if="(currentSort == 1)" class="ms-3 text-danger">
										<span v-if="(currentSortDir == 'asc')">▲</span>
										<span v-else>▼</span>
									</span>
								</div>
							</th>
							<th class="text-center" @click="sortTable(2)">
								<div class="d-flex justify-content-center">
									<span>Date</span>
									<span v-if="(currentSort == 2)" class="ms-3 text-danger">
										<span v-if="(currentSortDir == 'asc')">▲</span>
										<span v-else>▼</span>
									</span>
								</div>
							</th>
							<th @click="sortTable(3)">
								<div class="d-flex justify-content-center">
									<span>Order</span>
									<span v-if="(currentSort == 3)" class="ms-3 text-danger">
										<span v-if="(currentSortDir == 'asc')">▲</span>
										<span v-else>▼</span>
									</span>
								</div>
							</th>
							<th @click="sortTable(4)">
								<div class="d-flex justify-content-center">
									<span>Superfamily</span>
									<span v-if="(currentSort == 4)" class="ms-3 text-danger">
										<span v-if="(currentSortDir == 'asc')">▲</span>
										<span v-else>▼</span>
									</span>
								</div>
							</th>
							<th @click="sortTable(5)">
								<div class="d-flex justify-content-center">
									<span>Family</span>
									<span v-if="(currentSort == 5)" class="ms-3 text-danger">
										<span v-if="(currentSortDir == 'asc')">▲</span>
										<span v-else>▼</span>
									</span>
								</div>
							</th>
							<th @click="sortTable(6)">
								<div class="d-flex justify-content-center">
									<span>Genus</span>
									<span v-if="(currentSort == 6)" class="ms-3 text-danger">
										<span v-if="(currentSortDir == 'asc')">▲</span>
										<span v-else>▼</span>
									</span>
								</div>
							</th>
							<th @click="sortTable(7)">
								<div class="d-flex justify-content-center">
									<span>Species</span>
									<span v-if="(currentSort == 7)" class="ms-3 text-danger">
										<span v-if="(currentSortDir == 'asc')">▲</span>
										<span v-else>▼</span>
									</span>
								</div>
							</th>
						</tr>
						
					</thead>
					<tbody>
						<tr v-for="row in state_table_sorted" :class="(row[0]=='inat')?'inat_row':'ibp_row'" @click="GotoObservation(row)">
							<td v-text="row[1]"></td>
							<td v-text="row[2]" class="text-center"></td>
							<td v-text="row[3]"></td>
							<td v-text="row[4]"></td>
							<td v-text="row[5]"></td>
							<td v-text="row[6]"></td>
							<td v-text="row[7]"></td>
							
						</tr>
					</tbody>
				</table>
			</div>
			<div class="text-end text-muted my-2 ml-auto me-2">
				🄯 Rohit George
			</div>
		</div>
	</div>
</template>

<script>
	import _ from 'lodash';
	import raw_data from "./data/data.json"
	import raw_species from "./data/species.json"
	import raw_users from "./data/users.json"
	import raw_tree from "./data/tree.json"
	import country from "./data/country.json"
	import * as d3 from "d3"
	import * as d3Legend from "d3-svg-legend"
	export default {
		name: 'app-onsen',
		data () {
			return {
				data: raw_data,
				species_list: raw_species,
				users: raw_users,
				tree:raw_tree,
				state_totals:{},
				state_max:{users:0,observations:0},
				states_table:[],
				state_table:[],
				card:{users:0,observations:0},
				svgWidth: 0,
				svgHeight: 0,
				svg: null,
				labels:["observations", "users"],
				taxon_levels:["order", "superfamily", "family", "genus", "species"],
				selected_label:"observations",
				selected_portal:["inat", "ibp"],
				selected_level:"order",
				selected_state:"",
				currentSort:2,
				currentSortDir:"asc",
			}
		},
		mounted(){
			this.initData();
		},
		watch: {
			selected_label: function (val) {
				this.initData();
			},
			selected_portal: function (val) {
				this.initData();
			},
			selected_level: function (val) {
				this.initData();
			},
			selected_state: function (val) {
				this.initData();
			},
		},
		computed:{
			computed_portal(){
				let portal = "all";
				if(this.selected_portal.length == 1){
					portal = this.selected_portal[0];
				}
				return portal;
			},
			state_table_sorted(){
				return _.orderBy(this.state_table, this.currentSort, this.currentSortDir)
			}
		},
		methods:{
			init(){
			},
			initData(){
				let totals = {};
				let overall_totals = {users: new Set(), observations:0};
				Object.keys(this.data).forEach(portal=>{
					if(this.selected_portal.indexOf(portal) != -1){
						Object.keys(this.data[portal]).forEach(state=>{
							this.data[portal][state].forEach(ob => {
								if(this.taxon_levels.indexOf(this.species_list[ob[2]][2]) >= this.taxon_levels.indexOf(this.selected_level)){
									if(totals[state] == undefined){
										totals[state] = {};
									}
									if(totals[state][portal] == undefined){
										totals[state][portal] = {
											users: new Set([this.users[portal][ob[0]]]),
											observations: 1
										};
									} else {
										totals[state][portal].users.add(this.users[portal][ob[0]]);
										totals[state][portal].observations++;
									}
									if(totals[state]["all"] == undefined){
										totals[state]["all"] = {
											users: new Set([this.users[portal][ob[0]]]),
											observations: 1
										};
									} else {
										totals[state]["all"].users.add(this.users[portal][ob[0]]);
										totals[state]["all"].observations++;
									}
									overall_totals.users.add(this.users[portal][ob[0]]);
									overall_totals.observations++;
								}
							})
						})						
					}
				});
				country.features.forEach(state=> {
					let name = state.properties.st_nm;
					this.state_totals[name] = {
						"all":{users:0,observations:0},
						"inat":{users:0,observations:0},
						"ibp":{users:0,observations:0},
					}
				});
				this.state_max = {users:0,observations:0};
				Object.keys(totals).forEach(state => {
					Object.keys(totals[state]).forEach(portal => {
						let users = totals[state][portal].users.size;
						let observations = totals[state][portal].observations 
						this.state_totals[state][portal] = {
							users: users ,
							observations: observations
						}
						if(users > this.state_max["users"]){
							this.state_max["users"] = users;
						}
						if(observations > this.state_max["observations"]){
							this.state_max["observations"] = observations;
						}
					})
				});
				this.card.users = overall_totals.users.size;
				this.card.observations = overall_totals.observations;
				
				this.states_table = [];
				Object.keys(this.state_totals).forEach(state => {
					this.states_table.push([state, this.state_totals[state][this.computed_portal].observations, this.state_totals[state][this.computed_portal].users])
				})
				this.states_table.sort((a, b) => b[this.labels.indexOf(this.selected_label)+1]-a[this.labels.indexOf(this.selected_label)+1]);
				if(this.selected_state != ""){
					this.initStateData();
					this.card = this.state_totals[this.selected_state][this.computed_portal];
				}
				this.renderMap();
			},
			initStateData(){
				this.state_table = [];
				Object.keys(this.data).forEach(portal=>{
					if(this.selected_portal.indexOf(portal) != -1 && this.data[portal][this.selected_state] != undefined){
						this.data[portal][this.selected_state].forEach(ob => {
							if(this.taxon_levels.indexOf(this.species_list[ob[2]][2]) >= this.taxon_levels.indexOf(this.selected_level)){
									let taxon = this.getHierrachy(ob[2]);
									this.state_table.push([
										portal, 
										this.users[portal][ob[0]],
										this.getDate(ob[1]),
										taxon["order"],
										taxon["superfamily"],
										taxon["family"],
										taxon["genus"],
										taxon["species"],
										ob[3]
									]);
							}
						});
					}
				});
			},
			renderMap(){
				this.svgWidth = window.innerWidth / 2 
				this.svgHeight = window.innerHeight
				if(window.innerWidth < 1000){
					this.svgWidth = window.innerWidth - 60
					this.svgHeight = window.innerHeight/1.01
				}
				if (!d3.select("#map-container svg").empty()) {
					d3.selectAll("svg").remove()
				}
				let colors = null

				this.svg = d3.select("#map-container").append("svg").attr("preserveAspectRatio", "xMinYMin meet")
					.attr("width", this.svgWidth)
					.attr("height", this.svgHeight)
					.style("background-color", "rgb(190, 229, 235)")
					.classed("svg-content", true)


				var projection = d3.geoMercator().scale(1000).center([89.0, 35])
				const path = d3.geoPath().projection(projection)
				if(this.state_max[this.selected_label] > 0){
				colors = d3.scaleLinear().domain([0, 1, this.state_max[this.selected_label]/2, this.state_max[this.selected_label]]).range(["#f55", "#cb7", "#6c4", "#4d6"]);
				} else {
					colors = d3.scaleLinear().domain([0, 0]).range(["#f55", "#f55"])
				}
				var legend = d3Legend.legendColor().scale(colors).shapeWidth(55).labelFormat(d3.format(".0f")).orient('horizontal').cells(6)
				let base = this.svg.append("g")
					.classed("map-boundary", true)

				let base_text = base.selectAll("text").append("g")
				base = base.selectAll("path").append("g")

				country.features.forEach(state=> {
					let s_name = state.properties.st_nm
					let shape = base.append("g")
						.data([state])
						.enter().append("path")
						.attr("d", path)
						.attr("stroke", "#333")
						.attr("id", s_name)
						.attr("title", s_name)
						.attr("stroke-width", .5)
						.on("click", (d) => this.selectState(s_name));

					if(s_name == this.selected_state){
						shape.attr("fill", "rgba(255,255,50,1)")
						.attr("stroke", "RED")
					} else if(this.state_totals[s_name] == undefined){
						shape.attr("fill", (d) => colors(-1))
					} else {
						shape.attr("fill", (d) => colors(this.state_totals[s_name][this.computed_portal][this.selected_label]))
					}
				})

				country.features.forEach(state=> {
					let s_name = state.properties.st_nm
					let label = base_text.append("g")
						.data([state])
						.enter().append("text")
						.classed("poly_text", true)
						.attr("x", (h) => path.centroid(h)[0] )
						.attr("y", (h) => path.centroid(h)[1] )
						.attr("text-anchor", "middle")
						.attr("font-size",12)
						.text(this.state_totals[s_name][this.computed_portal][this.selected_label])
						.on("click", (d) => this.selectState(s_name))
				})

				this.svg.append("g")
					.attr("transform", "translate("+this.svgWidth*.2+", 50)")
					.call(legend)
					.append("text")
					.classed("map_label", true)
					.attr("dx", 5)
					.attr("dy", -10)
					.classed("h1", true)
					.text(this.selected_label.replace(/(?:_| |\b)(\w)/g, function($1){return $1.toUpperCase().replace('_',' ');}))

				let that = this;
				let zoom = d3.zoom()
					.scaleExtent([.5, 7.5])
					.translateExtent([[-150,-150],[that.svgWidth*1.5,that.svgHeight*1.5]])
					.on('zoom', function() {
						that.svg.selectAll('.poly_text')
							.attr('transform', d3.event.transform),
						that.svg.selectAll('path')
							.attr('transform', d3.event.transform);
					});
				this.svg.call(zoom);
			},
			selectState(s){
				if(this.selected_state == s){
					this.selected_state = "";
				} else {
					this.selected_state = s;
				}
			},
			selectedPortal(p){
				let index = this.selected_portal.indexOf(p);
				let op = p+"_btn";

				if(index == -1){
					op = "btn-outline-danger";
				}
				return op;
			},
			selectPortal(p){
				let index = this.selected_portal.indexOf(p);
				if (index !== -1) {
					this.selected_portal.splice(index, 1);
				} else {
					this.selected_portal.push(p)
				}
			},
			selectedTaxon(t){
				let op = "btn-outline-danger";
				if(this.taxon_levels.indexOf(t) >= this.taxon_levels.indexOf(this.selected_level))
					op = "btn-success";
				return op;
			},
			capatilize(s){
				return s.charAt(0).toUpperCase() + s.slice(1);
			},
			getDate(d){
				let day = d+16
				return day+" Aug"
			},
			getHierrachy(sp_id){
				let name = this.species_list[sp_id][1];
				let rank = this.species_list[sp_id][2];
				let taxonomy = {
					order:null,
					superfamily:null,
					family:null,
					genus:null,
					species:null
				};
				switch(rank){
					case "order": Object.keys(this.tree).forEach(order => {
								taxonomy["order"] = order
							});
						break;
					case "superfamily": Object.keys(this.tree).forEach(order => {
								taxonomy["order"] = order
								taxonomy["superfamily"] = name;
							});
						break;
					case "family": Object.keys(this.tree).forEach(order => {
								taxonomy["order"] = order
								taxonomy["family"] = name;
								Object.keys(this.tree[order]).forEach(superfamily => {
									Object.keys(this.tree[order][superfamily]).forEach(family => {
										if(name == family){
											taxonomy["superfamily"] = superfamily;
										}
									});
								});
							});
						break;
					case "genus": Object.keys(this.tree).forEach(order => {
								taxonomy["order"] = order
								taxonomy["genus"] = name;
								Object.keys(this.tree[order]).forEach(superfamily => {
									Object.keys(this.tree[order][superfamily]).forEach(family => {
										Object.keys(this.tree[order][superfamily][family]).forEach(genus => {
											if(name == genus){
												taxonomy["superfamily"] = superfamily;
												taxonomy["family"] = family;
											}
										});
									});
								});
							});
						break;
					case "species": Object.keys(this.tree).forEach(order => {
								taxonomy["order"] = order
								taxonomy["species"] = name;
								Object.keys(this.tree[order]).forEach(superfamily => {
									Object.keys(this.tree[order][superfamily]).forEach(family => {
										Object.keys(this.tree[order][superfamily][family]).forEach(genus => {
											this.tree[order][superfamily][family][genus].forEach(species => {
												if(name == species){
													taxonomy["superfamily"] = superfamily;
													taxonomy["family"] = family;
													taxonomy["genus"] = genus;
												}
											});											
										});
									});
								});
							});
						break;
				}
				return taxonomy;
			},
			sortTable(s) {
				if(s === this.currentSort) {
					this.currentSortDir = this.currentSortDir==='asc'?'desc':'asc';
				}
				this.currentSort = s;
			},
			GotoObservation(row){
				let url = '';
				if(row[0] == "inat"){
					url = "https://www.inaturalist.org/observations/"
				} else {
					url = "https://indiabiodiversity.org/observation/show/"

				}
				url += row[8]
				window.open(url, '_blank').focus();
			}
		}
	}
</script>