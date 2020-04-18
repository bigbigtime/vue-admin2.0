<template>
    <div id="header-wrap">
        <div class="pull-left header-icon" @click="navMenuState"><svg-icon iconClass="menu" className="menu" /></div>
        <div class="pull-right">
            <div class="user-info pull-left">
                <img src="../../../assets/images/face.jpg" alt="">
                {{ aaa }}
            </div>
            <div class="header-icon pull-left" @click="logout">
                <svg-icon iconClass="exit" className="exit" />
            </div>
        </div>
    </div>
</template>
<script>
import { computed } from '@vue/composition-api'
import { mapState } from 'vuex';
export default {
    name: 'layoutHeader',
    computed: {
        // 命名空间
        // ...mapState('app', ['username']),  
        // 非命名空间
        // ...mapState(['username']),  
        // 重命名
        ...mapState('app', {
            'aaa': 'username'
        })
    },
    methods: {
        navMenuState(){
            this.$store.commit('app/SET_COLLAPSE');
        },
        logout(){
            this.$store.dispatch('app/logout').then(response => {
                if(response.resCode === 0) {
                    this.$message({
                        message: "退出成功！",
                        type: "success"
                    })
                    this.$router.push({
                        name: 'Login'
                    })
                }
            })
        }
    }
}
</script>
<style lang="scss" scoped>
@import "../../../styles/config.scss";
#header-wrap {
    position: fixed;
    top: 0;
    right: 0;
    left: $navMenu;
    height: 75px;
    background-color: #fff;
    line-height: 75px;
    @include webkit(box-shadow, 0 3px 16px 0 rgba(0, 0, 0, .1));
    @include webkit(transition, all .3s ease 0s);
}
.open {
    #header-wrap { left: $navMenu; }
}
.close {
    #header-wrap { left: $navMenuMin; }
}

.header-icon {
    padding: 0 32px;
    svg { 
        margin-bottom: -8px;
        font-size: 25px;
        cursor: pointer;
    }

}
.user-info {
    height: 100%;
    padding: 0 32px;
    border-right: 1px solid #ededed;
    + .header-icon {
        padding: 0 28px;
    }
    img {
        display: inline-block;
        margin-bottom: -12px;
        margin-right: 18px;
        border-radius: 50px;
    }
}
</style>