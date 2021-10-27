<template>
   <div style align="center">
       <h1><img src="media/Logo.jpg" alt="НоваПошта" width="50" height="50">
       <span style="color: red; font-size: 2em">Н</span>ова <span style="color: red; font-size: 2em">П</span>ошта</h1> 
       <select v-model="dep">
           <option  v-for="city in cities" v-bind:key="city.Ref" v-bind:value="city.Description">{{ city.Description}}</option>
       </select>
       <br><button v-on:click="checkDeps">Вибрать отделение</button>
       <br><select>
           <option  v-for="dep in departments" v-bind:key="dep.CityRef" v-bind:value="dep.Description">{{ dep.Description}}</option>
       </select>
    <h2>
        <img src="media/sun.png" alt="Погода" width="50" height="50">
        <span style="color: blue; font-size: 2em">П</span>
        <span style="color: green; font-size: 1,5em">о</span>
        <span style="color: yellow; font-size: 1,5em">г</span>
        <span style="color: orange; font-size: 1,5em">о</span>
        <span style="color: red; font-size: 1,5em">д</span>
        <span style="color: purple; font-size: 1,5em">а</span>
    </h2>
       <table>
           <tr>
               <td style="border:none"><input type="text" v-model="wcityAdd">
                    <br><button v-on:click="addWCity">Добавить город</button></td>
                <td style="border:none; min-width:50px"> <select v-model="wcity" style="min-width:150px">
                        <option  v-for="city in wcities" v-bind:key="city" v-bind:value="city">{{ city}}</option>
                    </select>
                    <br><button v-on:click="getWeather">Погода </button>
                </td>
            </tr>
       </table>
       <table v-if="weather.cod >0">
                <tr>Город: {{weather.name}}</tr>
               <tr>Страна: {{weather.sys.country}}</tr>    
                <tr>Погода вообще: {{weather.weather[0].main}}</tr>
               <tr>Температура: {{weather.main.temp-273.15 | roundValue}} C</tr>
               <tr>Ощущается как: {{weather.main.feels_like-273.15 | roundValue}} C</tr>
               <tr>Минимальная Температура сегодня: {{weather.main.temp_min-273.15 | roundValue}} C</tr>
               <tr>Максимальная Температура сегодня: {{weather.main.temp_max-273.15 | roundValue}} C</tr>
               <tr>Давление: {{weather.main.pressure}} P</tr>
               <tr>Скорость ветра: {{weather.wind.speed}}</tr>
            
       </table>
    </div>
</template>

<script>
    import Vue from 'vue'
    import axios from 'axios'
    import VueAxios from 'vue-axios'
 
   export default {
        data: function() {
           return {
           cities:[],
           departments:[],
           dep:"Запоріжжя",
           wcityAdd:"",
           wcity:"",
           wcities:[],
           weather:[],
           testValue:0,
           latitude:0,
           longitude:0,
           link:"",
        }},
        mounted: function(){
            
                axios.post("https://api.novaposhta.ua/v2.0/json/Address/getCities", {
                        apiKey: "0555c556f433266e962e4ded47a8b3e9",
                        modelName: "Address",
                        calledMethod: "getCities",
                        methodProperties: {}
                })
                .then((response)=>{
                    console.log(response.data);
                    this.cities = response.data.data;
                })   
                              if(localStorage.getItem("wcities") !== '') this.wcities = JSON.parse(localStorage.getItem("wcities") || "[]");
                
               
                
                var getPosition = function (options) {
                    return new Promise(function (resolve, reject) {
                        navigator.geolocation.getCurrentPosition(resolve, reject, options);
                        }
                    );
                };
               getPosition()
                .then((position) => {
                    
                    this.latitude = position.coords.latitude;
                    this.longitude = position.coords.longitude;
                    console.log(this.latitude, this.longitude);
                    this.link = "https://api.openweathermap.org/data/2.5/weather?lat="+ this.latitude.toString()+"&lon="+ this.longitude.toString()+"&appid=7914d5a440960cfd5df3bd0388a7ad0f";
                        axios.get(this.link, {
                                units: "metric",
                        })
                        .then((response)=>{
                            console.log(response.data);
                            this.weather = response.data;
                        })
                })
                .catch((err) => {
                    console.error(err.message);
                });        
        },
        methods:{
            checkDeps: function() {
                
                 axios.post("https://api.novaposhta.ua/v2.0/json/Address/getWarehouses", {
                        apiKey: "0555c556f433266e962e4ded47a8b3e9",
                        modelName: "Address",
                        calledMethod: "getWarehouses",
                        methodProperties: {
                            CityName: this.dep,
                        }
                })
                .then((response)=>{
                    console.log(response.data);
                    this.departments = response.data.data;
                })
             },
            getWeather: function() {
                 axios.get("https://api.openweathermap.org/data/2.5/weather?q="+this.wcity+"&appid=7914d5a440960cfd5df3bd0388a7ad0f", {
                          units: "metric",
                })
                .then((response)=>{
                    console.log(response.data);
                    this.weather = response.data;
                    
                })
            },
            addWCity: function(){
                this.wcities.push(this.wcityAdd)
                localStorage.setItem("wcities",JSON.stringify(this.wcities))
            },
             
        },
        filters:{
            roundValue: function(value){
                return parseFloat(value.toFixed(2));
            },
        }
   }
</script>
<style scoped>

</style>