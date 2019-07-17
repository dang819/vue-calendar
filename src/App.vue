<template>
    <div>
        <div class="flex"><h1 class="actions-element-labels" >VUE</h1></div>
        <div>
            <div class="flex">
                <label class="actions-element-labels">Start Date:</label>
                <input class="actions-element-inputs" type="date" min="2000-01-01" max="2022-12-31" ref="dateInput" :value="currentDate" />
            </div>
            <div class="flex">
                <label class="actions-element-labels">Number of dates:</label>
                <input class="actions-element-inputs" ref="numberOfDaysInput" type="number" min="0" max="1095" placeholder="10" />
            </div>
            <div class="flex">
                <label class="actions-element-labels">Country code:</label>
                <input class="actions-element-inputs" ref="countryCodeInput" type="text" minlength="2" maxlength="2" placeholder="US" />
            </div>
        </div>
        <div class="actions-button-container">
            <button class="actions-button-style" @click="onClick">Submit</button>
        </div>
        <hr>
        <Calendar :counter="counter" :selectedDate="currentDate" :numberOfDays="numberOfDays" :countryCode="countryCode"></Calendar>
    </div>
</template>

<script>
import Calendar from  './components/Calendar'
import moment from 'moment'

export default {
    name: 'App' ,
    data(){
        return {currentDate : moment().format("YYYY-MM-DD"), numberOfDays:10, countryCode:'US', counter:0}
    },
    methods: {
        onClick(){
            this.currentDate = this.$refs.dateInput.value
            this.numberOfDays = this.$refs.numberOfDaysInput.value
            this.countryCode = this.$refs.countryCodeInput.value ? this.$refs.countryCodeInput.value : 'US' 
            this.counter++; // intentionally left to force update
        }
    },
    components : {
        Calendar
    },
  }

</script>

<style scoped>
.actions-container{
    display: flex;
    justify-content: center;
    flex-direction: column;
}

.actions-element-labels{
    flex: 0.1;
    text-align: right;
    margin-right: 10px;
}

.actions-element-inputs{
    flex: 0.1;
}

.flex{
    display: flex;
}

.actions-button-container{
    display: flex;
    width: 295px;
    justify-content: flex-end;
    margin: 10px 0;
}

.actions-button-style{
    width: 100px;
}
</style>
