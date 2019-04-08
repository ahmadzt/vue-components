/*
    Author: Ahmad Zaidi Ahmad Toha
    email: ahmadzt6210@gmail.com
*/

<template>
    <div
        class="modal-wrap"
        @click="clickAway">
        <div
            class="modal slideInLeft"
            :class="{ 'padding-0' : noPadding }">
            <button
                class="close-btn"
                @click="hideModal()">
                ✕
            </button>
            <div>
                <slot name="modal-header"></slot>
                <slot name="modal-content">
                   <h2>Modal content here</h2>
                </slot>
                <slot name="modal-footer"></slot>
            </div>
        </div>
    </div>
</template>
<style scoped>
.modal-wrap {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    overflow: auto;
    display: flex;
    background-color: rgba(218, 225, 231, 0.5);
    z-index: 11;
}

.modal {
    padding: 2rem;
    position: relative;
    margin: auto;
    width: 750px;
    background-color: #ffffff;
    border-radius: .25rem;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, .12), 0 2px 4px 0 rgba(0, 0, 0, .08);
}

.close-btn {
    position: absolute;
    right: -15px;
    top: -15px;
    padding: 4px;
    border-radius: 50%;
    background-color: #ccc;
    opacity: .8;
}

.padding-0 {
    padding: 0;
}

@keyframes slideInLeft {
  from {
    transform: translate3d(-100%, 0, 0);
    visibility: visible;
  }

  to {
    transform: translate3d(0, 0, 0);
  }
}

.slideInLeft {
  animation-name: slideInLeft;
}
</style>
<script>
    export default {
        props: {
            bodyScroll: {
                type: Boolean,
                default: false
            },
            noPadding: {
                type: Boolean,
                default: false
            }
        },
        methods: {
            clickAway(e) {
                this.event = function (event) {
                    if (event.target.className === "modal-wrap") {
                        this.hideModal()
                        document.body.removeEventListener("click", this.event);
                    }
                }.bind(this);

                document.body.addEventListener("click", this.event);
            },
            bodyNoScroll() {
                document.body.classList.toggle("overflow-hidden");
            },
            hideModal() {
                this.bodyNoScroll();
                this.$emit("hideModal");
            }
        },
        mounted() {

        }
    };
</script>