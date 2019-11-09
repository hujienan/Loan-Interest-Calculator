<template>
    <div class="container">
        <div class="card border-0">
            <div class="card-header bg-white border-0">
                <h2>Loan Interest Calculator</h2> 
            </div>
            <div class="card-body">
                <div class="form-group">
                    <div class="mb-4">
                        <p v-show="currentMonth">Selected Month: <b class="text-primary" @click="goToStep('showChooseMonth')">{{ formatedCurrentMonth }}</b></p>
                        <p v-show="loanBalance">Beginning Loan Balance: <b class="text-primary" @click="goToStep('showInputLoanBalance')">${{ loanBalance }}</b></p>
                        <p v-show="percentageAunual">Interest Percentage Aunual: <b class="text-primary" @click="goToStep('showInputPA')">{{ percentageAunual }}%</b></p>
                    </div>
                    <div v-show="steps.showChooseMonth">
                        <p class="font-weight-bold mb-0">Select Month</p>
                        <Datepicker input-class="form-control bg-white text-center mb-4" :minimumView="'month'" :maximumView="'year'"
                        :initialView="'year'" v-model="currentMonth" :format="customFormatter"></Datepicker>
                        <button class="btn btn-primary btn-block" :disabled="!currentMonth" @click="goToStep('showInputLoanBalance')">Next</button>
                    </div>
                    <div v-show="steps.showInputLoanBalance">
                        <p class="font-weight-bold mb-0">Loan Balance</p>
                        <div class="input-group mb-4">
                            <div class="input-group-prepend">
                                <span class="input-group-text">$</span>
                            </div>
                            <input type="number" class="form-control text-center" v-model="loanBalance">
                        </div>
                        <button class="btn btn-primary btn-block" :disabled="!loanBalance" @click="goToStep('showInputPA')">Next</button>
                    </div>
                    <div v-show="steps.showInputPA">
                        <p class="font-weight-bold mb-0">PA</p>
                        <div class="input-group mb-4">
                            <input type="number" step="0.01" class="form-control text-center" v-model="percentageAunual">
                            <div class="input-group-append">
                                <span class="input-group-text">%</span>
                            </div>
                        </div>
                        <button class="btn btn-primary btn-block" @click="goToStep('showAddCredits')">Next</button>
                    </div>
                    <div v-show="steps.showAddCredits">
                        <p class="font-weight-bold mb-0">When</p>
                        <Datepicker input-class="form-control bg-white text-center mb-4"
                        v-model="newCreditDate" :disabled-dates="currentDisabledDates"></Datepicker>
                        <p class="font-weight-bold mb-0">Credit</p>
                        <input type="number" class="form-control mb-4 text-center" step="0.01" v-model="newCreditAmount">
                        <button class="btn btn-primary mb-4 py-0" :disabled="!newCreditAmount || !newCreditDate" @click="addCredit">Add</button>
                        <div>
                            <table class="table">
                                <thead>
                                    <tr>
                                        <th scope="col">#</th>
                                        <th scope="col">date</th>
                                        <th scope="col">credit</th>
                                        <th scope="col"></th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr
                                        v-for="(item, index) in sortedCredits"
                                        :key="index"
                                    >
                                        <th scope="row">{{ index+1 }}</th>
                                        <td>{{ item.displayDate }}</td>
                                        <td>{{ item.amount }}</td>
                                        <td @click="deleteCredit(item.id)"><button type="button" class="btn btn-outline-primary py-0">X</button></td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                        <button class="btn btn-primary btn-block btn-lg" @click="calculate">Calculate</button>
                    </div>
                    <div v-show="steps.showResult">
                        <p><span>Total Interest</span> $<b>{{ loanInterest }}</b></p>
                        <table class="table">
                            <thead>
                                <tr>
                                    <th scope="col">Date</th>
                                    <th scope="col">Previous Balance</th>
                                    <th scope="col">Interest</th>
                                    <th scope="col">Current Balance</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr
                                    v-for="(item, index) in interestArr"
                                    :key="index"
                                >
                                    <th scope="row">{{ item.date }}</th>
                                    <td>{{ item.previous_balance }}</td>
                                    <td>{{ item.interest }}</td>
                                    <td>{{ item.current_balance }}</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import Datepicker from 'vuejs-datepicker';
import moment from 'moment';
let id = 0;
export default {
    data: function () {
        return {
            steps: {
                showChooseMonth: true,
                showInputLoanBalance: false,
                showInputPA: false,
                showAddCredits: false,
                showResult: false
            },
            credits: [],
            newCreditAmount: null,
            newCreditDate: null,
            currentMonth: new Date(),
            loanBalance: "",
            percentageAunual: "",
            loanInterest: "",
            interestArr: []
        }
    },
    watch: {
        currentMonth: function () {
            this.newCreditDate = new Date(moment(this.currentMonth).startOf('month'));
        }
    },
    computed: {
        formatedCurrentMonth: function () {
            return moment(this.currentMonth).format('MMMM YYYY')
        },
        sortedCredits: function () {
            let target = [...this.credits];
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
            return parseFloat(this.percentageAunual / 100 / this.daysInYear);
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
                id: id++,
                amount: vm.newCreditAmount,
                date: vm.newCreditDate,
                displayDate: moment(vm.newCreditDate).format('D MMMM YYYY')
            });
            vm.newCreditAmount = "";
        },
        deleteCredit: function (itemId) {
            let vm = this;
            let index = vm.credits.findIndex(item => item.id == itemId);
            vm.credits.splice(index, 1);
        },
        customFormatter: function (date) {
            return moment(date).format('MMMM YYYY');
        },
        isSameDay: function (d1, d2) {
            return d1.getFullYear() === d2.getFullYear() && d1.getMonth() === d2.getMonth() && d1.getDate() === d2.getDate();
        },
        calculate: function () {
            let vm = this;
            vm.interestArr = [];
            let balance = parseFloat(vm.loanBalance);
            let day = vm.currentDisabledTo;
            let total = 0;
            while (day <= vm.currentDisabledFrom) {
                let item = {};
                item.date = moment(day).format('D MMM YYYY');
                item.previous_balance = balance;
                let interest =  parseFloat((balance * vm.percentagePerDay).toFixed(2));
                item.interest = interest;
                total = total + interest;
                for (let i = 0; i < vm.sortedCredits.length; i++) {
                    if (vm.isSameDay(vm.sortedCredits[i].date, day)) {
                        balance -= vm.sortedCredits[i].amount;
                    }
                }
                item.current_balance = balance;
                vm.interestArr.push(item);
                day = new Date(moment(day).add(1, 'd'));
            }
            vm.loanInterest = total.toFixed(2);
            vm.goToStep("showResult");
        }
    }
}
</script>
<style>
    .vdp-datepicker__calendar {
        left: 50%;
        transform: translateX(-50%);
    }
</style>