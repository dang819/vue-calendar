<template>

    <div>
        <table v-if="ready" >

            <div class="calendar-main-cal-container" v-for="(val, key) in calendarFormatedData" :key="key">
                <tr>
                    <td class="calendar-text-center" colspan="7">{{key.substr(0, 4)}}</td>
                </tr>
                <tr>
                    <td class="calendar-text-center" colspan="7">{{Object.values(val[1])[0].monthName}}</td>
                </tr>  
                <tr>
                    <th v-for="(days, index) in daysIndex" :key="index">{{days}}</th>
                </tr>
                <tr v-for="(weeks, keyMonths) in calendarFormatedData[key]" :key="keyMonths">
                    <td v-for="(weekDay, keyWeeks) in calendarFormatedData[key][keyMonths]" 
                        :key="keyWeeks" class="calendar-cell-width" 
                        :title="weekDay.holidayName"
                        :class="{'calendar-cell-isWeekend': weekDay.isWeekDay==false, 'calendar-cell-notInRange' : weekDay.isBetween==false, 'calendar-cell-inRange' : weekDay.isWeekDay==true, 'calendar-cell-isHoliday' : weekDay.isHoliday==true && weekDay.isBetween==true }"
                    >
                        {{showDate(weekDay.day, weekDay.isBetween)}}
                    </td>
                </tr>
            </div>
        </table>
    </div>
    
</template>

<script>
import moment from 'moment';
import _ from 'lodash';
import axios from 'axios';

const API_KEY = '518c9b2c-74b3-48d4-b8af-4cd5d9fb45e1';

export default {
    name: 'Calendar',

    data(){
        return { 
            daysIndex : {0:'S', 1:'M', 2:'T', 3:'W', 4:'T', 5:'F', 6:'S'},
            calendarData : null,
            calendarFormatedData: null,
            ready : false,
            monthsToRender : 0,
            yearsToRender : 0,
            year: moment(this.selectedDate, 'YYYY/MM/DD').year(),
            dataStartDate: null,
            dataEndDate: null,
            holidaysData: null,
        }
    },

    methods: {
        methodDate(date){
            date = moment(date, 'YYYY/MM/DD')
            let isBetween = false;
            if (date.isAfter(this.dataStartDate)  && date.isBefore(this.dataEndDate)  || (date.isSame(this.dataStartDate)) ) { 
                isBetween = true
            }
            let weekOfMonthValidated = moment(date, "YYYY/MM/DD").week() - moment(date, "YYYY/MM/DD").startOf('month').week()
            weekOfMonthValidated = weekOfMonthValidated <0 ? 4 : weekOfMonthValidated
            let isHoliday = false
            let holidayName = null
            for(let holidays in this.holidaysData){
                if (moment(this.holidaysData[holidays].date).format("MMM-DD") == moment(date, 'YYYY/MM/DD').format('MMM-DD')){
                    isHoliday = true
                    holidayName =  this.holidaysData[holidays].name
                }
            }

            return { 
                day: moment(date, 'YYYY/MM/DD').date(),
                month: moment(date, 'YYYY/MM/DD').month()+1,
                monthName: moment(date, 'YYYY/MM/DD ').startOf("month").format('MMMM'),
                year: moment(date, 'YYYY/MM/DD').year(),
                weekOfMonth: (moment(date, 'YYYY/MM/DD').startOf("month").format('MM')).toString() + (moment(date, "YYYY/MM/DD").week() - moment(date, "YYYY/MM/DD").startOf('month').week()).toString(), //isoWeek
                weekOfMonthNormal: weekOfMonthValidated,
                monthAndYear :  moment(date, 'YYYY/MM/DD').year().toString() + moment(date, 'YYYY/MM/DD ').startOf("month").format('MM').toString() ,
                weekDay: moment(date, "YYYY/MM/DD").weekday(), //isoWeekday
                weekDayLetter: this.daysIndex[moment(date, "YYYY/MM/DD").weekday()], //isoWeekday
                isWeekDay: moment(date, "YYYY/MM/DD").weekday() == 0 || moment(date, "YYYY/MM/DD").weekday() == 6  ? false : true, //isoWeekday
                isToday:  moment(date, "YYYY/MM/DD").format("YYYY/MM/DD") == moment().format("YYYY/MM/DD") ? true : false,
                isHoliday: isHoliday,
                holidayName: holidayName,
                isBetween : isBetween
            }
        },
        showDate(dayNumber, isBetween){
            return dayNumber<1 || !isBetween ? '': dayNumber;
        }

    },

    props:['selectedDate', 'numberOfDays', 'countryCode', 'counter'],

    watch: {
        'counter' : function(){

            axios.get('https://holidayapi.com/v1/holidays', {
                params:{
                    key: API_KEY,
                    country: this.countryCode,
                    year: 2018
                }
            }).then(response=>{
                this.holidaysData = response.data.holidays

                let endDate = moment(this.selectedDate, 'YYYY/MM/DD').add('days', this.numberOfDays)
                this.dataEndDate = endDate
                let startDate = moment(this.selectedDate, 'YYYY/MM/DD')
                this.dataStartDate = startDate

                let yearsDiff = (moment(endDate, 'YYYY/MM/DD').year() - moment(this.selectedDate, 'YYYY/MM/DD').year())
                let monthsCalc = ( moment(endDate, 'YYYY/MM/DD').month() - moment(this.selectedDate, 'YYYY/MM/DD').month() ) + (yearsDiff*12)
                this.monthsToRender = monthsCalc +1
                this.yearsToRender = yearsDiff

                let dataArray = [];
                for(let actualMonth = 0 ; actualMonth<=monthsCalc ; actualMonth++){
                    let newDate = moment(this.selectedDate, 'YYYY/MM/DD').add('months', actualMonth)
                    newDate = moment(newDate, "YYYY/MM/DD").format("YYYY/MM/DD")
                    for(let days=1 ; days<=moment(newDate, "YYYY/MM/DD").daysInMonth(); days++){
                        let data = this.methodDate(`${moment(newDate, "YYYY/MM/DD").year()}/${moment(newDate, "YYYY/MM/DD").month()+1}/${days}`)
                        dataArray.push(data)
                    }
                }

                let calendarByMonthAndYear = _.groupBy(dataArray, (el)=>{return el.monthAndYear})


                for(let monthAndYear in calendarByMonthAndYear){
                    calendarByMonthAndYear[monthAndYear] = _.groupBy(calendarByMonthAndYear[monthAndYear], (el)=>{ return el.weekOfMonthNormal})
                    let lastIndex =  Object.values(calendarByMonthAndYear[monthAndYear]).length -1
                    if(calendarByMonthAndYear[monthAndYear][0].length<7){
                        let diffDates = Math.abs(calendarByMonthAndYear[monthAndYear][0].length - 7)
                        for(let i=0; i<diffDates; i++){
                            calendarByMonthAndYear[monthAndYear][0].unshift({day:-100+i, weekOfMonth:calendarByMonthAndYear[monthAndYear][0][0].weekOfMonth, monthAndYear: calendarByMonthAndYear[monthAndYear][0][0].monthAndYear, isBetween: false})
                        }
                    }
                    if(calendarByMonthAndYear[monthAndYear][lastIndex].length<7){
                        let diffDates = Math.abs(calendarByMonthAndYear[monthAndYear][lastIndex].length - 7)
                        for(let i=0; i<diffDates; i++){
                            calendarByMonthAndYear[monthAndYear][lastIndex].push({day:-100+i, weekOfMonth:calendarByMonthAndYear[monthAndYear][lastIndex][0].weekOfMonth, monthAndYear: calendarByMonthAndYear[monthAndYear][lastIndex][0].monthAndYear, isBetween: false})
                        }
                    }

                }

                this.calendarFormatedData = calendarByMonthAndYear;
                this.ready=true
                
            })
        },immediate: true, deep: true
    }
}
</script>


<style scoped>
    .calendar-weekdays-style{
        display: flex;
    }

    .calendar-text-center{
        text-align: center;
    }
    .calendar-cell-width{
        width: 30px;
        height: 21px;
        text-align: center;
    }
    .calendar-cell-isWeekend{
        background-color: yellow;
    }
    .calendar-cell-notInRange{
        background-color: lightgray !important;
    }
    .calendar-cell-inRange{
        background-color: lightgreen;
    }
    .calendar-main-cal-container{
        margin:20px 0;
        border: 1px solid lightgray;
    }
    .calendar-cell-isHoliday{
        background-color: lightpink !important;
    }
</style>
