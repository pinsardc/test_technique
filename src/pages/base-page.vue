<template>
  <div id="page">
    <div>
      <div class="col-md-5 offset-md-2 text-start">
        <h1>API Tracking Routes</h1>
        <h5>Choisissez une période pour commencer</h5>
      </div>

      <div class="row justify-content-around">
        <div class="col-md-4 text-start">
          Début
        </div>
        <div class="col-md-4 text-start">
          Fin
        </div>
      </div>

      <div class="row justify-content-around">
        <input class="col-4 mb-4 rounded" type="date" id="dateBegin"
        v-on:change="onChange()"
          v-model="dateBegin"
          value="2021-12-01"
          min="2018-01-01" max="2022-12-31">
        <input class="col-4 mb-4 rounded" type="date" id="dateEnd"
        v-on:change="onChange()"
          v-model="dateEnd"
          value="2021-12-09"
          min="2018-01-01" max="2022-12-31">
      </div>
      <div id="appels" style="width: 33%;" class="col-md-8 offset-md-4 bg-darken-xl rounded">
        <div id="textLightBlue">Total d'appels</div>
        <br>
        <div class="text-white">{{ this.sumOfCalls }}</div>
        <br>
        <div id="textLightBlue">Période</div>
        <br>
        <div class="text-white">{{ this.dateBegin }} - {{ new Date(this.dateEnd).toISOString() }}</div>
      </div>
      <canvas class="shadow m-4 rounded" id="barCanvas" width="400" height="200"></canvas>
      <canvas class="shadow m-4 rounded" id="myChart" width="400" height="200"></canvas>
    </div>
  </div>
</template>

<script>
const axios = require('axios');
import Chart from 'chart.js'
export default {
  name: 'base-page',
  props: {
  },
  data() {
    return {
      apiData: undefined,
      dateBegin: "2021-12-01",
      dateEnd: "2021-12-09",
      dataTreated: undefined,
      barCanvas: undefined,
      chart: undefined,
      sumOfCalls: undefined,
      dates: ['2021-12-01', '2021-12-02', '2021-12-03', '2021-12-04', '2021-12-05', '2021-12-06', '2021-12-07', '2021-12-08', '2021-12-09']
    }
  },
  mounted() {
    var self = this;
    axios.request({
      method: 'get',
      baseURL: 'http://api-buildings.homeys.io/api/tracking',
      headers: {
          "Authorization": "t8dwXjjIjrRq54LSq1Vt3SmWSYi9K7jbVzP5UdgD7ptPTJE5N0"
      }
    })
    .then(response => response.data)
    .then(async response => {
      this.apiData = await response;
      this.dataTreated = this.apiData.filter(function (el) {
      return el.date >= self.dateBegin && el.date <= self.dateEnd
      })
      this.sumAllCalls()
      this.createChart(this.dataTreated)
    });


    
  },
  methods: {
    listFilter(list, _path) {//filtre selon une route
      return list.filter(function (el) {
        return el.path == _path
      })
    },
    diffDates() {//calcule la période séléctionnée, et renvoie la liste des dates
      var days = [];
      for (var d = new Date(this.dateBegin); d <= new Date(this.dateEnd); d.setDate(d.getDate() + 1)) {
          days.push(d.toISOString().substring(0, 10));
      }
      this.dates = days
    },
    sumAllCalls() {//additionne tous les appels
      var sum = 0;
      for (var el of this.dataTreated) {
        sum += el.count;
      }
      this.sumOfCalls = sum;
    },
    sumCalls() {//additionne les appels pour une route donné
      var values = [];
      for (var day of this.dates) {
        var sum = 0;
          const found = this.dataTreated.filter(element => element.date == day)
          for (var el of found) {
            sum += el.count;
          }
          values.push(sum);
      }
      return values
    },
    getValues(list) {//prend les valeurs du nombre d'appels
      var values = [];
      for (var day of this.dates) {
        const found = list.find(element => element.date == day)
        if (found == undefined) {
          values.push(0)
        }
        else {
          values.push(found.count)
        }
      }
      return values
    },
    
    onChange() {//actualise les données
      this.barCanvas.destroy()
      this.chart.destroy()
      var self = this;
      this.dataTreated = this.apiData.filter(function (el) {
        return el.date >= self.dateBegin && el.date <= self.dateEnd
      })
      this.sumAllCalls()
      this.createChart(this.dataTreated)
    },
    createChart(list) {//crée les graphiques
      this.days = this.diffDates()
      let ctx = document.querySelector("#myChart")
      let barctx = document.querySelector("#barCanvas")

      this.barCanvas = new Chart(barctx, {
        type: "bar",
        data: {
          labels: this.dates,
          datasets: [
            {
              data: this.sumCalls(),
              backgroundColor: "darkblue",
              borderColor: "darkblue",
              borderWidth: 3
            },
          ]
        },
        options: {
          title: {
            display: true,
            text: 'Nombre d\'appels par jour',
            align: 'start',
            fontSize: 25
          },
          legend: {
            display: false
          },
          responsive: true,
          lineTension: 1,
          scales: {
            yAxes: [
              {
                gridLines: {
                  display: false
                },
                ticks: {
                  beginAtZero: true,
                }
              }
            ],
            xAxes: [
              {
                gridLines: {
                  display: false
                }
              }
            ]
          }
        }
      });

      //Pour ne pas répéter les mêmes informations, on crée une liste des routes et des couleurs choisies
      //et on crée la liste contenant l'ensemble des datasets
      const routes = ["/","/api/batimentfromadresse","/api/batiments/adresse","/api/batiments/infos","/api/infos"
                ,"/api/tracking","/api/parcelle/adresse","/api/adresse/infos"]
      const colors = ["lightblue","red","purple","green","darkblue","orange","grey","red"]
      const allDatasets = []
      for (var i = 0; i < routes.length; i++) {
        allDatasets.push({
              label: routes[i],
              data: this.getValues(this.listFilter(list, routes[i])),
              fill: false,
              borderColor: colors[i],
              borderWidth: 3
            })
      }
      this.chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: this.dates,
          datasets: allDatasets
        },
        options: {
          title: {
            display: true,
            text: 'Nombre d\'appels en fonction des routes',
            align: 'start',
            fontSize: 25
          },
          legend: {
                position: 'right',
          },
          responsive: true,
          lineTension: 1,
          scales: {
            yAxes: [
              {
                type: 'logarithmic',
                ticks: {
                  beginAtZero: true,
                }
              }
            ]
          }
        }
      });
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#page {
  padding-left: 10%;
  padding-right: 10%;
}
#appels {
  background-color: #0C2D48;
}
#textLightBlue {
  color: #5885AF;
}
html {
  font-size: 25px;
}
</style>
