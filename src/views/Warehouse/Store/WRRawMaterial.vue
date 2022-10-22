<template>
  <div class="WRRawMaterial">
    <div class="breadcrumb p-0">
      <router-link :to="{name: 'WRStore'}" class="breadcrumb__link"><i class="bx bx-home-alt"></i></router-link>
      <div class="breadcrumb__item">
        <i class="fa fa-angle-right"></i>
        <router-link :to="{name: 'WRStore'}" class="breadcrumb__link"><i class="bx bxs-cylinder"></i></router-link>
      </div>
      <div class="breadcrumb__item">
        <i class="fa fa-angle-right"></i>
        <p>{{ 'Сырье' }}</p>
      </div>
    </div>
    <div class="wrapper">
      <div class="card">
        <ul class="clps-menu">
          <li v-for="item in info" :key="item.index" class="clps-menu__item clps">
            <div @click="collapseToggle(item.name)" class="clps__button">
              <p class="d-flex align-items-center">{{ item.name }} <i class='ml-1 bx ' :class="[collapseActive == item.name ? 'bxs-down-arrow' : 'bxs-right-arrow']"></i></p>
            </div>
            <Transition duration="350" name="nested">
              <div v-if="collapseActive == item.name" class="clps__list list">
                <div class="custom-table">
                  <div v-if="item.items.length != 0" class="custom-table__header">
                    <p class="w-1">№</p>
                    <p class="w-2">Код</p>
                    <p class="w-3">Название</p>
                    <p class="w-4">Цвет</p>
                    <p class="w-5">Кол.</p>
                  </div>
                  <div v-else class="custom-table__header">
                    <p>Пусто</p>
                  </div>
                  <div v-for="(itemList,index) in item.items" :key="itemList.Id" class="custom-table__body">
                    <p class="w-1">{{ index + 1}}</p>
                    <p class="w-2">{{ itemList.Code }}</p>
                    <p class="w-3">{{ itemList.Name }}</p>
                    <p class="w-4">{{ itemList.ColorCode }} {{ itemList.Color }}</p>
                    <p class="w-5">{{ itemList.Count }} {{ itemList.Unit }}</p>
                  </div>
                </div>
              </div>
            </Transition>
          </li>
        </ul>
      </div>
    </div>
    <Loading v-show="loading"/>
  </div>
</template>

<script>
import axios from 'axios';
import { required } from 'vuelidate/lib/validators';
import Loading from '@/components/Loading.vue';

export default {
  name: 'WRRawMaterial',
  data() {
    return {
      loading: false,
      collapseActive: '',
      info: [],
      token: '',
      auth: [],
      api: ''
    }
  },
  components: {
    Loading
  },
  beforeMount() {
    this.api = window.MY_CONFIG_BUYURSIN.api;
    // this.token = localStorage.token;
  },
  mounted() {
    this.token = sessionStorage.getItem('token');
    this.auth = JSON.parse(sessionStorage.getItem('auth'));
    this.itemsbybranch();
  },
  methods: {
    // getMetterGroups() {
    //   // this.info = [];
    //   this.metterGroupOption = [];
    //   this.metterOption = [];
    //   axios.get(`${this.api}/itemgroups/`, {
    //       headers: {
    //         'Authorization': `Token ${this.token}`
    //       }
    //     })
    //     .then(response => {
    //       // console.log(response.data);
    //       // this.userGroup = response.data;
    //       for(let i = 0; i < response.data.length; i++) {
    //         const list = {
    //           'id': response.data[i].id,
    //           'name': response.data[i].name,
    //           'list': []
    //         }
    //         this.metter.push(list);
    //       }
          
    //       this.metterGroupOption = response.data;
    //       this.getMetter();
    //     })
    //     .catch(e => console.log(e))
    //     .finally(() => { this.loading = false; });
    // },
    // getMetter() {
    //   axios.get(`${this.api}/itemTemplates/`, {
    //       headers: {
    //         'Authorization': `Token ${this.token}`
    //       }
    //     })
    //     .then(response => {
    //       // console.log(response.data);
    //       for (let i = 0; i < this.metter.length; i++) {
    //         for(let t = 0; t < response.data.length; t++) {
    //           if(this.metter[i].id == response.data[t].group) {
    //             this.metter[i].list.push(response.data[t]);
    //           }
    //         }
    //       }
    //       // this.metterAll = response.data;
    //       this.getUnits();
    //     })
    //     .catch(e => console.log(e));
    // },
    // getUnits() {
    //   axios.get(`${this.api}/units/`, {
    //       headers: {
    //         'Authorization': `Token ${this.token}`
    //       }
    //     })
    //     .then(responce => {
    //       this.unitOption = responce.data;
    //     })
    //     .catch(e => console.log(e));
    // },
    itemsbybranch() {
      this.loading = true;
      this.info = [];
      axios.get(`${this.api}/itemsbybranch/?branchId=${this.auth.branchId}`, {
          headers: {
            'Authorization': `Token ${this.token}`
          }
        })
        .then(response => {
          // console.log(response.data);
          this.info = response.data;
        })
        .catch(error => {
          // console.log(e.response);
          if(error.response.status == 401) {
            this.$toast.error(error.response.data.detail);
            sessionStorage.setItem('token', '');
            this.$router.push({ name: 'Auth' });
          }
        })
        .finally(() => { this.loading = false; });
    },
    collapseToggle(id) {
      if(this.collapseActive == id) {
        this.collapseActive = 0
      } else {
        this.collapseActive = id;
      }
    },
  },
  watch: {
    metterGroupValue() {
      this.metterOption = [];
      if(this.metterGroupValue != '') {
        const metter = this.metter.find(item => {
          if(item.id == this.metterGroupValue.id) {
            return item
          }
        });
        // console.log(metter);
        this.metterOption = metter.list;
        if(this.castelmetterValue == false) {
          this.metterValue = '';
        } else {
          this.castelmetterValue = false;
        }
      }
    },
    metterValue() {
      if(this.metterValue != '') {
        this.unitValue = this.unitOption.find(item => {
          if(item.id == this.metterValue.unit) {
            return item
          }
        });
      }
    }
  },
  validations: {
    metterGroupValue: {
      required
    },
    metterValue: {
      required
    },
    count: {
      required
    }
  },
}
</script>

<style lang="scss" scoped>
.WRRawMaterial {
  @media screen and (max-width: 991.5px) {
    margin-left: 0;
    padding: 0px 10px;
  }
  .breadcrumb {
    display: flex;
    align-items: center;
    background-color: transparent;
    margin: 0px 0 25px;
    &__item {
      display: flex;
      align-items: center;
      .fa {
        color: #002f34;
        font-size: 25px;
        margin: 0 10px;
      }
      p {
        font-family: 'Roboto';
        font-size: 18px;
        line-height: 18px;
        font-weight: 500;
        color: #002f34;
      }
    }
    &__link {
      display: flex;
      align-items: center;
      .bx {
        color: #002f34;
        font-size: 26px;
      }
    }
  }
  .wrapper {
    margin-bottom: 90px;
  }
  .card {
    background-color: #fff;
    background-clip: border-box;
    border: 0 solid transparent;
    border-radius: 0.25rem;
    box-shadow: 0 0.3rem 0.8rem rgba(0,0,0,0.12);
    padding: 1rem;
    overflow: scroll hidden;
    // position: relative;
    // display: flex;
    // flex-direction: column;
    // min-width: 0;
    // word-wrap: break-word;
    // background-color: rgba(0,0,0,0.15);
    // background-clip: border-box;
    // border: 0 solid transparent;
    // border-radius: 0.25rem;
    // margin-bottom: 4.5rem;
    // box-shadow: 0 0.3rem 0.8rem rgba(0,0,0,0.12);
    // padding: 1rem;
  }
  .clps-menu {
    
    min-width: 500px;
    .clps {
      padding: 5px;
      &__button {
        display: flex;
        justify-content: space-between;
        align-items: center;
        font-family: "Roboto";
        font-size: 20px;
        font-weight: 400;
        background: rgba($color: #002f34, $alpha: 0.3);
        padding: 5px 20px;
        cursor: pointer;
        .btns {
          width: 100px;
          display: flex;
          justify-content: space-between;
        }
        p {
          font-family: "Roboto";
          font-size: 20px;
          font-weight: 400;
          color: #002f34;
        }
      }
      .list {
        // height: 0;
        position: relative;
        &::before {
          content: "";
          width: 3px;
          height: 100%;
          background: rgba($color: #002f34, $alpha: 0.85);
          position: absolute;
          top: 0;
          left: 20px;
          @media screen and (max-width: 991.5px) {
            left: 10px;
          }
        }
        .custom-table {
          margin-left: 50px;
          margin-right: 200px;
          margin-bottom: 1px;
          @media screen and (max-width: 991.5px) {
            margin-left: 25px;
            margin-right: 0px;
          }
          &__header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: rgba(255, 255, 255, 0.12);
            padding: 5px 15px;
            border-bottom: 1px solid rgba(255, 255, 255, 0.15);
            p {
              font-weight: 600 !important;
            }
          }
          &__body {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: rgba(41, 115, 65, 0.1);
            padding: 5px 15px;
            position: relative;
            margin-bottom: 2px;
            transition: 0.3s all ease-in-out;
            border-bottom: 1px solid rgba(255, 255, 255, 0.15);
            &::before {
              content: "";
              width: 28px;
              height: 3px;
              background: rgba($color: #002f34, $alpha: 0.85);
              position: absolute;
              top: 50%;
              left: -28px;
              transform: translateY(-50%);
              @media screen and (max-width: 991.5px) {
                width: 15px;
                top: 50%;
                left: -15px;
              }
            }
            &:hover {
              background: rgba($color: #002f34, $alpha: 0.04);
            }
          }
          p {
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: "Roboto";
            font-size: 18px;
            font-weight: 400;
            color: rgba($color: #002f34, $alpha: 0.85);
          }
          .btns {
            display: flex;
            justify-content: space-between;
            .btn {
              padding: 0.2rem 0.5rem;
            }
          }
          .w-1 {
            width: 5%;
          }
          .w-2 {
            width: 20%;
          }
          .w-3 {
            width: 25%;
          }
          .w-4 {
            width: 25%;
          }
          .w-5 {
            width: 20%;
          }
          .w-6 {
            width: 80px;
          }
        }
      }
    }
  }
  .btn-add {
    width: 60px;
    height: 60px;
    position: fixed;
    bottom: 15px;
    right: 15px;
    padding: 10px;
    cursor: pointer;
    border-radius: 50%;
    background: #fff;
    display: flex;
    justify-content: center;
    align-items: center;
    .fa {
      font-size: 50px;
      color: #1d9d74;
    }
  }
  .modal-add {
    position: fixed;
    z-index: 11;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: #fff;
    border-radius: 0.5rem;
    padding: 12px;
    .text {
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-bottom: 2px solid #002f34;
      p {
        width: 60%;
        font-family: 'Roboto';
        font-size: 20px;
        font-weight: 400;
        padding: 5px 16px;
        color: rgb(129, 129, 129);
        text-align: center;
        transition: 0.3s all ease-in-out;
        cursor: pointer;
        @media screen and (max-width: 575.5px) {
          font-size: 18px;
        }
        &:hover {
          color: #000;
          background: rgb(230, 230, 230);
        }
        &:nth-child(2) {
          width: 40%;
        }
      }
      .active {
        color: #fff;
        background: #002f34;
        &:hover {
          color: #fff;
          background: #002f34;
        }
      }
    }
    .form {
      width: 400px;
      display: flex;
      flex-direction: column;
      // margin-top: 15px;
      @media screen and (max-width: 575.5px) {
        width: 290px;
      }
      &__title {
        font-family: 'Roboto';
        font-weight: 500;
        font-size: 18px;
        line-height: 18px;
        margin-bottom: 15px;
        color: #002f34;
      }
      .input {
        width: 100%;
        margin-bottom: 12px;
      }
      
      .selects {
        position: relative;
        width: 100%;
        select {
          width: 100%;
          font-family: 'Roboto';
          font-size: 18px;
          font-weight: 400;
          color: #5A5A5A;
          border: 2px solid rgba(0,0,0,.12);
          border-radius: 5px;
          padding: 0.5em 0.7em;
          position: relative;
          background: transparent;
          z-index: 1;
          margin-bottom: 12px;
          -webkit-appearance: none;
          -moz-appearance: none;
          appearance: none;
          cursor: pointer;
          &:hover {
            border-color: rgba(255, 255, 255, 0.2);
            transition: 0.3s all ease-in-out;
          }
          &:focus-visible {
            border-color: rgba(255, 255, 255, 0.2);
            outline: none;
          }
        }
        .fa {
          position: absolute;
          top: 7px;
          right: 14px;
          font-size: 30px;
          color: #5A5A5A;
        }
        select:hover + span + .fa,
        select:focus-visible + span + .fa {
          transition: 0.3s all ease-in-out;
          color: rgb(175, 175, 175);
        }
        select:hover + span,
        select:focus-visible + span {
          transition: 0.3s all ease-in-out;
          color: rgb(175, 175, 175) !important;
        }
        .multiselect {
          cursor: pointer;
          border: 2px solid rgba(0,0,0,.12);
          border-radius: 5px;
          margin-bottom: 12px;
          padding: 2px 0;
          font-family: 'Roboto';
          &:hover {
            border-color: rgb(175, 175, 175);
            transition: 0.3s all ease-in-out;
          }
        }
        .multiselect__single {
          font-family: 'Roboto';
        }
        span {
          padding: 2px 5px 0 5px !important;
          margin-bottom: 2px;
          z-index: 49;
        }
        .multiselect:hover+span {
          color: rgb(175, 175, 175);
          transition: 0.3s all ease-in-out;
        }
      }
      .btns {
        display: flex;
        justify-content: space-between;
        .btn {
          width: 48%;
          font-family: 'Roboto';
          padding: 8px 30px;
          text-transform: uppercase;
        }
      }
      .count {
        margin-bottom: 12px;
        .input {
          padding-right: 4em;
          margin-bottom: 0;
        }
        p {
          font-family: 'Roboto';
          font-size: 20px;
          font-weight: 400;
          color: #5A5A5A;
          position: absolute;
          right: 18px;
          top: calc(50% - 16px);
          transform: translateY(calc(50% - 16px));
        }
      }
      
      .count-btn {
        display: flex;
        align-items: center;
        .btn {
          width: 45px;
          height: 34px;
          display: flex;
          justify-content: center;
          align-items: center;
          font-family: 'Roboto';
          font-size: 1.5rem;
          line-height: 1.5rem;
          color: #fff;
          // background: gray;
          // padding: 5px 15px;
          border-radius: 0;
          margin-bottom: 15px;
          outline: none;
          &:focus {
            box-shadow: none;
          }
          &-left {
            border-bottom-left-radius: 10px;
            border-top-left-radius: 10px;
            // background-color: red;
          }
          &-right {
            border-bottom-right-radius: 10px;
            border-top-right-radius: 10px;
            // background-color: green;
          }
        }
        p {
          // width: 34px;
          height: 34px;
          font-family: 'Roboto';
          font-size: 1.5rem !important;
          line-height: 1.5rem !important;
          background: rgba(gray, 0.2);
          padding: 5px 10px;
          border-radius: none;
          margin-bottom: 15px !important;
        }
      }
    }
  }
  .modal-delete {
    position: fixed;
    z-index: 11;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: #fff;
    border-radius: 0.5rem;
    padding: 12px;
    .form {
      width: 400px;
      display: flex;
      flex-direction: column;
      @media screen and (max-width: 575.5px) {
        width: 290px;
      }
      p {
        font-family: 'Roboto';
        font-size: 24px;
        line-height: 24px;
        font-weight: 400;
        color: #212529;
        margin-bottom: 15px;
        text-align: center;
      }
      .btns {
        display: flex;
        justify-content: space-between;
        .btn {
          width: 48%;
          font-family: 'Roboto';
          padding: 8px 30px;
          text-transform: uppercase;
        }
      }
    }
  }
  .name,
  .selects,
  .number,
  .databasePassword,
  .search {
    position: relative;
    span {
      position: absolute;
      z-index: 2;
      top: 0%;
      left: 10px;
      transform: translateY(-50%);
      font-family: 'Roboto';
      font-size: 12px;
      font-weight: 400;
      color: rgba(0,0,0,.4);
      background: #fff;
      padding: 2px 5px;
    }
  }
  .input {
    width: 100%;
    font-family: 'Roboto';
    font-size: 18px;
    font-weight: 400;
    color: #5A5A5A;
    border: 2px solid rgba(0,0,0,.12);
    border-radius: 5px;
    padding: 0.5em 0.7em;
    position: relative;
    background: transparent;
    z-index: 1;
    margin: 0;
    &:hover {
      border-color: rgb(175, 175, 175);
      transition: 0.3s all ease-in-out;
    }
    &:focus-visible {
      border-color: rgb(175, 175, 175);
      outline: none;
    }
  }
  input:focus-visible+span{
    transition: 0.3s all ease-in-out;
    color: rgb(175, 175, 175) !important;
  }
  input:hover+span{
    transition: 0.3s all ease-in-out;
    color: rgb(175, 175, 175);
  }
  .modal-bg {
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.3);
    position: fixed;
    top: 0;
    left: 210px;
    z-index: 10;
    @media screen and (max-width: 991.5px) {
      left: 0px;
    }
  }
  .is-invalid {
    border-color: red !important;
  }
  .is-invalid + span {
    color: red !important;
  }
  .is-invalid + span + .fa{
    color: red !important;
  }
}

.nested-enter-active, .nested-leave-active {
	transition: all 0.2s ease-in-out;
}
/* delay leave of parent element */
.nested-leave-active {
  transition-delay: 0s;
}
.nested-enter-from,
.nested-leave-to {
  transform: translateY(30px);
  opacity: 0;
}

</style>