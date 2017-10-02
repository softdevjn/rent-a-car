<template>
    <div class="vehicle-types">
        <div v-if="!loaded">
            <h3 class="text-center">Loading data...</h3>
        </div>
        <vehicle-types-form :edit="vehicleTypeToEdit !== null" :vehicle-type="vehicleTypeToEdit"
                            @type-created="addVehicleType" @type-updated="updateVehicleType">
        </vehicle-types-form>

        <hr>

        <table-list v-if="loaded" :columns="propsToDisplay" v-bind:tableData="vehicleTypes"
                    :use-actions="true"></table-list>
    </div>
</template>
<script>
    import TableList from './../Views/TableList.vue'
    import VehicleTypesForm from './Forms/VehicleTypesForm.vue'

    const vehicleTypesColumns = ['Ord', 'Description', 'State', 'Actions'];

    export default {
        components: {
            VehicleTypesForm,
            TableList
        },
        data() {
            return {
                loaded: false,
                propsToDisplay: [...vehicleTypesColumns],
                vehicleTypes: [],
                vehicleTypeToEdit: null
            };
        },
        mounted() {
            this.$axios.get('http://localhost:8000/api/types').then((res) => {
                    this.vehicleTypes = res.data.map((type, position) => {
                        const {id, description, state} = type,
                            vehicleType = {ord: position + 1, id, description, state};

                        return this.addActionsTo(vehicleType);
                    });
                    this.loaded = true;
                }
            );
        },
        methods: {
            addVehicleType(data) {
                const {id, description, state} = data,
                    vehicleType = {ord: this.vehicleTypes.length + 1, id, description, state: state || 'ACTIVE'};

                this.vehicleTypes.push(vehicleType);
            },
            updateVehicleType(data) {
                this.vehicleTypes = this.vehicleTypes.map(type => {
                    if (type.id === data.id) {
                        type = Object.assign(type, {description: data.description});
                    }

                    return type;
                });
            },
            addActionsTo(vehicleType) {
                return Object.assign(vehicleType, {
                    actions: [
                        {
                            click: () => this.edit(vehicleType),
                            classes: 'text-warning',
                            text: '<i class="fa fa-edit"></i>Edit'
                        },
                        {
                            click: () => this.remove(vehicleType),
                            classes: 'text-danger',
                            text: '<i class="fa fa-remove"></i>Remove'
                        }
                    ]
                });
            },
            edit(vehicleType) {
                this.vehicleTypeToEdit = vehicleType;
            }
        }
    };
</script>
<style scoped lang="scss"></style>