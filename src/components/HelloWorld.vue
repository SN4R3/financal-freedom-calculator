<style scoped>
  .money {
    color:green;
  }
  .row {
    margin:25px 0;
  }
  .header {
    padding-bottom:20px 0;
    display:flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  h1 {
    font-size: 45px;
    font-weight: bold;
  }
  
  .result-container div {
    padding: 5px 0;
  }
  /*Medium devices*/
  @media (min-width: 600px) {
    .header {
      flex-direction: row;
    }
    .header img {
      width:10%;
    }
  }
</style>
<template>
  <div class="inputs-container">

    <div class="header">
      <img src="/fire.png" width="30%"/>
      <h1>FIRE Roadmap</h1>
    </div>

    <hr/>
    
    <div class="row">
      <div class="col-12 col-md-6 offset-md-3">
        <b-field label="Starting Amount ($)">
          <b-numberinput v-model="starting" :step="50"></b-numberinput>
        </b-field>
      </div>
    </div>

    <div class="row">
      <div class="col-12 col-md-6">
        <b-field label="Current Age" 
          :type="currAge > age ? 'is-danger' : ''" 
          :message="currAge > age ? 'Current Age must be lower than Retirement Age' : ''"
        >
          <b-numberinput v-model="currAge"></b-numberinput>
        </b-field>
      </div>

      <div class="col-12 col-md-6">
        <b-field label="Retirement Age" 
          :type="currAge > age ? 'is-danger' : ''" 
          :message="currAge > age ? 'Current Age must be lower than Retirement Age' : ''"
        >
          <b-numberinput v-model="age"></b-numberinput>
        </b-field>
      </div>
    </div>

    <div class="row">
      <div class="col-12 col-md-6">
        <b-field label="Amount to Last Until (Age)"
          :type="age > ageUntil ? 'is-danger' : ''" 
          :message="age > ageUntil ? 'Retirement Age must be Lower than Amount to Last Until' : ''"
        >
          <b-numberinput v-model="ageUntil"></b-numberinput>
        </b-field>

        <p><strong>{{ageUntil - age}} </strong>years of retirement</p>
        <br/>
      </div>

      <div class="col-12 col-md-6">
        <b-field label="Expenses per Year ($)">
          <b-numberinput v-model="expenses" :step="50"></b-numberinput>
        </b-field>
      </div>
    </div>
    
    <h2>Result:</h2>
    <p><strong class="money">${{formatNumber(amount)}}</strong> required by <strong>{{this.yearsUntilRetirement + moment().year()}}</strong></p>
    <p>
      Save <strong class="money">${{formatNumber(savePerYear.toFixed(0))}}</strong> per year 
      (<strong class="money">${{formatNumber((savePerYear / 12).toFixed(0))}}</strong> per month)
    </p>
    
    <h2>Roadmap:</h2>
    <div class="result-container">
      <div v-for="(milestone, i) in milestones" :key="'ms'+i">
        <p>
          - End <strong>{{milestone.name}}</strong> with: <span class="money">${{formatNumber(milestone.amount)}}</span>
          (age {{parseInt(currAge) + (i + 1)}})
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment';

export default {
  name: 'InputsContainer',
  data() {
    return {
      currAge:25,
      age: 60,
      ageUntil: 80,
      expenses: 24000,
      starting: 0,
    }
  },
  computed: {
    amount: function() {
      return ((this.ageUntil - this.age) * this.expenses) - this.starting;
    },
    yearsUntilRetirement: function() {
      return  this.age - this.currAge;
    },
    savePerYear: function() {
      return this.amount / this.yearsUntilRetirement;
    },
    milestones: function() {
      let result = [];
      for(let year = 1; year <= this.yearsUntilRetirement; year++) {
        result.push({
          name: year + moment().year(),
          amount: (this.savePerYear * year).toFixed(0) - this.starting
        });
      }
      this.saveToLocal();
      return result;
    }
  },
  created() {
    if(localStorage.getItem('FIRERoadMap-currAge')) {
      this.currAge = parseInt(localStorage.getItem('FIRERoadMap-currAge'));
      this.age = parseInt(localStorage.getItem('FIRERoadMap-age'));
      this.ageUntil = parseInt(localStorage.getItem('FIRERoadMap-ageUntil'));
      this.expenses = parseInt(localStorage.getItem('FIRERoadMap-expenses'));
      this.starting = parseInt(localStorage.getItem('FIRERoadMap-starting'));
    }
  },
  methods: {
    saveToLocal() {
      localStorage.setItem('FIRERoadMap-currAge',this.currAge);
      localStorage.setItem('FIRERoadMap-age',this.age);
      localStorage.setItem('FIRERoadMap-ageUntil',this.ageUntil);
      localStorage.setItem('FIRERoadMap-expenses',this.expenses);
      localStorage.setItem('FIRERoadMap-starting',this.starting);
    },
    formatNumber(num) {
      num = num.toString();
      if(num.length == 4) {
        return num[0]+','+num.substr(1, 3);
      } else if(num.length == 5) {
        return num.substr(0, 2) +','+num.substr(2, 4);
      } else if(num.length == 6) {
        return num.substr(0, 3) +','+num.substr(3, 5);
      } else if(num.length == 7) {
        return num[0] +','+num.substr(1, 3) +','+num.substr(4, 6);
      } else if(num.length == 8) {
        return num[0] + '' + num[1] +'M+';
      } else if(num.length == 9) {
        return num[0] + '' +num[1] + num[2] + 'M+';
      } else if(num.length == 10) {
        return num[0] + 'B+';
      }
      return num;
    },
    moment() {
      return moment();
    }
  }
}
</script>
