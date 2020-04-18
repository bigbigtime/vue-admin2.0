<template>
    <div id="layout" :class="[menuStatus ? 'close' : 'open']">
        <LayoutHeader />
        <LayoutMain />
        <LayoutNav />
    </div>
</template>
<script>
import LayoutHeader from './Components/Header'
import LayoutMain from './Components/Main'
import LayoutNav from './Components/Nav'
// import { LayoutHeader, LayoutMain, LayoutNav } from './Components'
export default {
    name: 'layout',
    components: {LayoutHeader, LayoutMain, LayoutNav},
    // 2.0
    computed: {
        menuStatus(){
            return this.$store.state.app.isCollapse
        }
    },
    // setup(props, { root }){
    //     const menuStatus = computed(() => root.$store.state.app.isCollapse);
    //     return {
    //         menuStatus
    //     }
    // }
}
</script>
<style lang="scss" scoped></style>