/*
    Author: Ahmad Zaidi Ahmad Toha
    email: ahmadzt6210@gmail.com
    github: https://github.com/ahmadzt
*/

<template>
    <div class="paginate-container">
        {{ from || 0 }} to {{ to || 0 }} of {{ total }} {{ type }}
        <div class="btn-group">
            <button
                :disabled="current_page == 1"
                class="btn btn-default"
                type="button"
                @click="nextPage(1)">
                First
            </button>

            <button 
                v-for="(btn,index) in pages"
                :key="index+'b'"
                :class="{ 'active' : selected == btn }"
                class="btn btn-default"
                type="button"
                @click="nextPage(btn)">
                {{ btn }}
            </button>

            <button
                :disabled="current_page == last_page"
                class="btn btn-default"
                type="button"
                @click="nextPage(last_page)">
                Last
            </button>
        </div>
    </div>
</template>
<script>
    export default {
        name: "Paginate",
        props: {
            to: Number,
            from: Number,
            total: Number,
            last_page: Number,
            current_page: {  
                type: Number,
                default: 1
            },
            type: {
                type: String,
                default: 'items'
            },
            interval: {
                type: Number,
                default: 2
            }
        },
        data() {
            return {
                pages: [],
                selected: 1,
            }
        },
        watch: {
            current_page: function() {
                this.checkPages();
                this.checkMin();
                this.checkMax();
            },
            last_page: function() {
                this.checkPages();
                this.checkMin();
                this.checkMax();
            },
        },
        methods: {
            checkPages() {
                this.pages = [];
                this.selected = this.current_page;

                if (this.last_page < this.interval + 1) {
                    for (let a=0; a<this.last_page; a++) {
                        this.pages.push(a+1);
                    }
                }
                else {
                    for (let a=0; a<this.current_page + this.interval; a++) {
                        this.pages.push(a+1);
                    }
                }
            },
            checkMin() {
                let temp = this.pages.findIndex(item => item == this.current_page - this.interval);
                if (temp > -1) this.pages.splice(0, temp);
            },
            checkMax() {
                let temp = this.pages.findIndex(item => item > this.last_page);
                let interval = (this.pages.length-1)-temp;
                if (temp > -1) this.pages.splice(temp, this.pages.length-1);
            },
            nextPage(page) {
                this.$emit("nextPage", page);
            }
        },
        mounted() {
            this.checkPages();
        }
    };
</script>
<style scoped>
.paginate-container {
    width: 100%;
    display: flex;
    justify-content: flex-end;
    align-items: center;
}

.btn-group {
    position: relative;
    display: inline-flex;
    vertical-align: middle;
}

.btn {
    padding-top: .5rem;
    padding-bottom: .5rem;
    padding-left: 1rem;
    padding-right: 1rem;
    margin: .25rem;
    border-radius: .25rem;
    color: #083f8a;
    white-space: nowrap;
    font-size: .8rem;
    cursor: pointer;
}

.btn-default {
    background-color: #eee;
    color: #083f8a;
}

.btn-default:hover, .btn-default.active {
    background-color: #ccc;
    color: #083f8a;
}

.btn-group .btn:first-child {
    border-top-left-radius: 4px;
    border-bottom-left-radius: 4px;
}

.btn-group .btn {
    margin: 0px !important;
    border-radius: 0px;
}

.btn:disabled {
    opacity: .65;
    cursor: not-allowed;
}
</style>
