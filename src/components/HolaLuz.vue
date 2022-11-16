<template>
  <div class="header-logo">
    <img src="@/assets/logo-gradient.svg" alt="header.logo.alt" width="112" height="26" class="logo">
  </div>
  <div> 
    <input 
      class="holaluz-input" 
      type="number" 
      placeholder="CUPS" 
      v-model="searchCUPS"  
      oninput="javascript: if (this.value.length > this.maxLength) this.value = this.value.slice(0, this.maxLength);"
      maxlength = "6">
      <div>
        <div v-for="client in filteredClient" :key="client.cups">
          <div class="box">
            <div class="one">
              <p> <b>CUPS :</b> {{ client.cups}} </p>
              <p> <b>Full name:</b>  {{ client.full_name }}</p>
              <p> <b>Adress:</b>  {{ client.address }}</p>
              <p> <b>Role:</b>  {{ client.role }}</p>
              <p> <b>Building type:</b>  {{ client.building_type }}</p>
            </div>
            <div class="two">
              <p> <b>Tariff:</b>  {{ client.tariff }}</p>
              <p> <b>Invoiced amount:</b> {{ client.invoiced_amount }}</p>
              <p> <b>Power:</b> P1: {{ client.power.p1 }} / P2: {{ client.power.p2 }}</p>
              <p> <b>Neighbors:</b> <span v-for="neighbor in client.neighbors" :key="neighbor"> {{ neighbor }} / </span></p>
            </div>  
            <div v-if="client.building_type == 'house' && client.neighbors.length >=1">
              <h2>Offer</h2>
              <div class="discount-percentage" v-for="neighbor in client.neighborsInfo" :key="neighbor">
                <p v-if="(client.power.p1 > neighbor.power.p1) && (client.power.p2 > neighbor.power.p2)" > Discount 5% </p>
                <p v-if=" Number(neighbor.invoiced_amount) > 100.00" > Discount 12% </p>
              </div>
            </div>
          </div>
        </div>
      </div>
    
      
    <div class="item-error" v-if="searchCUPS && !filteredClient.length">
       <p>No results found!</p> 
    </div>

    
  </div>
</template>

<script>
import axios from "axios";

  export default {
    data() {
     return {
      clients: [],
      points: [],
      searchCUPS: '',
      allClientInfo: [],
      discount: ''
     }
    },
    methods: {
      getClients () {
        axios.get("/data/clients.json")
            .then(res => this.clients = res.data)
            .catch(err => console.log(err))
      },
      getPoints () {
        axios.get("/data/supply-points.json")
            .then(res => {
              this.points = res.data;
              this.clients = this.clients.map(t1 => ({...t1, ...this.points.find(t2 => t2.cups === t1.cups)}));
            })
            .then( () => {
              this.clients.map(client => {
                const addNeighbor = [];
                client.neighbors.map( neighbor => {
                addNeighbor.push(this.clients.find(clientN => clientN.cups === neighbor));
                client.neighborsInfo = addNeighbor;
                })
              })
            }) 
            .catch(err => console.log(err))
      },
    },   
    
    computed: {
    filteredClient() {
      return this.clients.filter(client => {
        return client.cups.includes(this.searchCUPS)
      })
    },
  },
    mounted() { 
       this.getClients();
       this.getPoints(); 
    }
  };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .header-logo {
    margin-bottom: 16px;
  }
  .holaluz-input {
    align-items: center;
    height: 2.25rem;
    border: 1px solid #d4d7db;
    border-radius: 4px;
    background-color: #fff;
    padding: 0 2rem 0 0.5rem;
  }
  .holaluz-input:hover {
    border: 1px solid #e5007e;
}
  .box {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
    grid-auto-rows: minmax(100px, auto);
    box-shadow: 0 3px 17px 0 rgba(33,33,33,0.1);;
    background: #fff;
    padding: 2rem;
    border-radius: 8px;
    margin: 16px auto;
    width: 80%;
  }

  .one {
    grid-column: 1;
    grid-row: 1;
    text-align: left;
  }

  .two {
    grid-column: 2;
    grid-row: 1;
    text-align: left;
  }

  h2 {
    text-align: left;
  }
  div .box h2{
    color: #21A36B;
  }
  .discount-percentage {
    color: #e5007e;
    text-align: left;
  }

  input::-webkit-outer-spin-button,
  input::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
}

  /* Firefox */
  input[type=number] {
    -moz-appearance: textfield;
  }
</style>
