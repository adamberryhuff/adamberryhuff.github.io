<template>
    <form v-on:keydown.enter.prevent="">
        <!-- expense label -->
        <div class="form-group">
            <label>{{ $t('common.chart_label') }}</label>
            <input id="new-expense-focus" v-model="label" type="text" class="form-control" :placeholder="$t('common.label')" v-on:keyup.enter="addExpense()">
            <small class="form-text text-muted">
                {{ $t('expenses.chart_label_examples') }}
            </small>
        </div>

        <!-- income stream amount -->
        <div class="form-group">
            <label>{{ $t('common.amount') }}</label>
            <input type="number" class="form-control" min="1" v-model="value" :placeholder="$t('common.amount')" v-on:keyup.enter="addExpense()">
            <small class="form-text text-muted">
                {{ amountExamples }}
            </small>
        </div>

        <!-- desktop buttons: save, cancel -->
        <div class="desktop-only-inline">
            <button v-if="expense" class="btn btn-link" v-on:click.enter.prevent="cancelEdit" v-on:keyup.enter="cancelEdit()">
                {{ $t('common.cancel') }}
            </button>
            <button class="btn btn-primary" v-on:click.enter.prevent="addExpense()" v-on:keyup.enter="addExpense()">
                {{ submitText }}
            </button>
        </div>

        <!-- mobile buttons: save, cancel -->
        <div class="mobile-only row">
            <br>
            <button class="btn btn-primary col-sm-12" v-on:click.enter.prevent="addExpense()" v-on:keyup.enter="addExpense()">
                {{ submitText }}
            </button>
            <button v-if="expense" class="btn btn-outline-primary col-sm-12" v-on:click.enter.prevent="cancelEdit" v-on:keyup.enter="cancelEdit()">
                {{ $t('common.cancel') }}
            </button>
        </div>
    </form>
</template>


<script>
import util from '../../util.js';

export default {
    name: 'AddExpense',
    props: ['unallocatedSum', 'mode', 'expense', 'timeline'],
    data () {
        return {
            // new income stream
            label: '',
            value: '',
        }
    },
    methods: {
        addExpense: function (e) {
            if (e) e.preventDefault();
            this.value = parseInt(this.value);

            // validate
            if (isNaN(this.value) || this.value < 0) {
                alert(this.$t('expenses.value_error'));
                return;
            }

            // make sure it's in the budget but take into account edit mode
            var err   = this.expense ? 'difference_error_edit' : 'difference_error_add';
            var value = this.expense ? this.value - this.expense.value : this.value;
            if (value > this.unallocatedSum) {
                var error = this.$t(`expenses.${err}`, {
                    budget: util.formatMoney(this.unallocatedSum, this.mode)
                });
                alert(error);
                return;
            }

            // emit
            this.$emit('addExpense', {
                label: this.label,
                value: this.value
            });

            // reset
            this.label = '';
            this.value = '';

            this.focusNewExpense();
        },
        cancelEdit: function () {
            this.$emit('editExpense', false);
        },
        focusNewExpense: function () {
            document.getElementById('new-expense-focus').focus();
        },
        processKeyPress: function (event) {
            if (event.keyCode == 13) this.addExpense();
        }
    },
    computed: {
        submitText: function () {
            return this.expense ? this.$t('expenses.update') : this.$t('expenses.add');
        },
        amountExamples: function () {
            var timeline = this.timeline == 'annual' ? 'common.annual' : 'common.monthly';
            return this.$t('expenses.amount_examples', {
                mode: this.mode ,
                timeline: this.$t(timeline)
            });
        }
    },
    watch: {
        expense: function () {
            this.value = this.expense ? this.expense.value : '';
            this.label = this.expense ? this.expense.label : '';
        }
    }
}

</script>

<style scoped>


.desktop-only-inline {
    float: right;
    margin-top: calc(30px - 1rem);
}

.desktop-only-inline .remove-expense {
    padding-left:5px;
    padding-right:17px;
    color:red;
}

.btn-outline-danger {
    margin-top:5px;
}

.btn-outline-primary {
    margin-top:5px;
}


</style>