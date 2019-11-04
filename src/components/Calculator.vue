<template>
    <div class="container">
        <div class="card">
            <div class="card-header">
                Loan Interest Calculator
            </div>
            <div class="card-body">
                <h5 class="card-title">Loan Interest Calculator</h5>
            </div>
            <div class="form-group">
                <label>Select Month</label>
                <Datepicker input-class="form-control" :minimumView="'month'" :maximumView="'year'"
                :initialView="'year'" v-model="currentMonth" :format="customFormatter"></Datepicker>
                <label>Loan Balance</label>
                <input type="number" class="form-control" v-model="loanBalance">
                <label>PA</label>
                <div class="input-group">
                    <input type="number" step="0.01" class="form-control" v-model="percentageAunual">
                    <div class="input-group-append">
                        <span class="input-group-text">%</span>
                    </div>
                </div>
                <label>Credit</label>
                <input type="number" class="form-control" step="0.01" v-model="newCreditAmount">
                <label>When</label>
                <Datepicker input-class="form-control"
                v-model="newCreditDate" :disabled-dates="currentDisabledDates"></Datepicker>
                <button @click="addCredit">Add</button>
                <div>
                    <table class="table">
                        <thead>
                            <tr>
                            <th scope="col">#</th>
                            <th scope="col">date</th>
                            <th scope="col">credit</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr
                                v-for="(item, index) in sortedCredits"
                                :key="index"
                            >
                                <th scope="row">{{ index }}</th>
                                <td>{{ item.displayDate }}</td>
                                <td>{{ item.amount }}</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
                <button class="btn btn-primary" @click="calculate">Calculate</button>
                <p><span>Total</span><b>{{ loanInterest }}</b></p>
            </div>
        </div>
    </div>
</template>
<script>
import Datepicker from 'vuejs-datepicker';
import moment from 'moment'
// import {setData} from '../data.js';
export default {
    data: function () {
        return {
            steps: {
                chooseMonth: true,
            },
            credits: [],
            newCreditAmount: null,
            newCreditDate: new Date(moment(this.currentMonth).startOf('month')),
            currentMonth: new Date(),
            loanBalance: "",
            percentageAunual: "",
            loanInterest: ""
        }
    },
    watch: {
        currentMonth: function () {
            this.newCreditDate = new Date(moment(this.currentMonth).startOf('month'));
        }
    },
    computed: {
        sortedCredits: function () {
            let target = [...this.credits];
            //return target;
            return target.sort((a, b) => a.date - b.date); 
        },
        currentDisabledTo: function () {
            return new Date(moment(this.currentMonth).startOf('month'));
        },
        currentDisabledFrom: function () {
            return new Date(moment(this.currentMonth).endOf('month'));
        },
        currentDisabledDates: function () {
            return {
                to: this.currentDisabledTo,
                from: this.currentDisabledFrom
            }
        },
        daysInYear: function () {
            let vm = this;
            let year = vm.currentMonth.getFullYear();
            if (year % 400 === 0 || (year % 100 !== 0 && year % 4 === 0)) {
                return 366;
            }
            return 365
        },
        percentagePerDay: function () {
            let vm = this;
            console.log("Days: "+vm.daysInYear);
            return ((parseFloat(vm.percentageAunual) / 100) / vm.daysInYear)
        },
    },
    components: {
        Datepicker
    },
    methods: {
        goToStep: function (step) {
            Object.keys(this.steps).map(key => key == step ? (this.steps[key] = true) : (this.steps[key] = false))
        },
        addCredit: function () {
            let vm = this;
            vm.credits.push({
                amount: vm.newCreditAmount,
                date: vm.newCreditDate,
                displayDate: moment(vm.newCreditDate).format('D MMMM YYYY')
            });
            vm.newCreditAmount = "";
        },
        customFormatter: function (date) {
            return moment(date).format('MMMM YYYY');
        },
        calculate: function () {
            let vm = this;
            let balance = vm.loanBalance;
            let day = vm.currentDisabledTo;
            let total = 0;
            while (day <= vm.currentDisabledFrom) {
                console.log("Balance: "+balance);
                console.log(vm.percentagePerDay);
                let interest = balance * vm.percentagePerDay;
                console.log(interest);
                total = parseFloat(total) + parseFloat(interest);
                balance = parseFloat(balance) + parseFloat(interest);
                for (let i = 0; i < vm.sortedCredits; i++) {
                    if (vm.sortedCredits[i].date == day) {
                        balance -= vm.sortedCredits[i].amount;
                    }
                }
                day = new Date(moment(day).add(1, 'd'));
                console.log(day);
                console.log(total);
            }
            vm.loanInterest = total;
        }
    },
    mounted: function () {
        // this.credits = setData;
    }
}
</script>