<template>
  <div class="body">

    <h1>Cadastro</h1>
    <form @submit.prevent="register()" class="form-cadastro">

      <div class="input-group form-cadastro2">
        <input @input="user.name = $event.target.value" type="text" class="form-control" id="name" aria-describedby="emailHelp" placeholder="Nome Completo" required>
      </div>

      <div class="input-group form-cadastro2" :class="inputActiveUser">
        <input 
          autocomplete="off" data-placement="bottom" data-toggle="popoverUser" data-trigger="focus" data-html="true" :data-content="'Usuário ' + '<span>' + user.user + '</span>' + ' já existe.'"
          class="form-control" :class="checkInputUser" @input="user.user = $event.target.value" @focus="inputActives(true, 'user')" @blur="inputActives(false, 'user')" type="text" id="user"placeholder="Usuário" required
        >
        <div class="input-group-prepend ">
            <i v-show="checkInputUser" class="input-group-text material-icons check-button check-i" :class="check.user">
              {{ check.user }}
            </i>
        </div>
      </div>

      <div class="input-group form-cadastro2" :class="inputActiveEmail">
        <input 
          autocomplete="off" data-placement="bottom" data-toggle="popoverEmail" data-trigger="focus" data-content="E-mail incorreto ou já existe."
          class="form-control" :class="checkInputEmail" @input="user.email = $event.target.value" @focus="inputActives(true, 'email')" @blur="inputActives(false, 'email')" type="text" id="user"placeholder="E-mail" required
        >
        <div class="input-group-prepend ">
            <i v-show="checkInputEmail" class="input-group-text material-icons check-button check-i" :class="check.email">
              {{ check.email }}
            </i>
        </div>
      </div>

      <div class="input-group form-cadastro2">
        <input @input="user.company = $event.target.value" type="text" class="form-control" id="company" placeholder="Empresa" required>
      </div>

      <div class="input-group form-cadastro2">
        <input @input="user.cnpj = $event.target.value" type="text" class="form-control" id="cnpj" placeholder="CNPJ" required>
      </div>

      <div class="input-group form-cadastro2">
        <input @input="user.phone = $event.target.value" type="text" class="form-control" id="phone" placeholder="Telefone" required>
      </div>
      
      <div class="input-group form-cadastro2">
        <input @input="user.password = $event.target.value" type="password" class="form-control mb-0" id="password" placeholder="Senha" required>
      </div>

      <div class="input-group form-cadastro2">
        <input @input="passwordConfirmation = $event.target.value" type="password" class="form-control" id="password" placeholder="Confirmar Senha" required>
      </div>
      
      <button type="submit" class="btn btn-primary">Cadastrar</button>
      
    </form>
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
  </div>
</template>

<script>
import $ from 'jquery';
import User from '../domain/User';

import { Observable } from 'rxjs';
import { pluck, empty, filter, debounceTime, distinctUntilChanged, switchMap, map } from 'rxjs/operators'

export default {
  name: 'Cadastro',

  components: {
  },

  mounted: () => {
    $('[data-toggle="popoverUser"]').popover()
    $('[data-toggle="popoverEmail"]').popover()
  },
  
  data() {
    return {
      user: new User(),
      check: new User(),
      passwordConfirmation: '',
      inputActiveUser: '',
      inputActiveEmail: '',
      checkInputUser: '',
      checkInputEmail: '',
      iconUser: '',
      iconEmail: '',
      is_admin : null
    }
  },

  subscriptions () {
    return {
      results: this.$watchAsObservable('user.user').pipe(
        pluck('newValue'),
        debounceTime(300),
        distinctUntilChanged(),
        switchMap(value => {
          if(value.length > 0) {
            this.check.user = 'hourglass_empty'
            setTimeout( () => {
                this.checkField('user')
            }, 300);
            return value
          } else {
            $('[data-toggle="popoverUser"]').popover('hide')
            return 'null'
          }
        })
      ),

      results2: this.$watchAsObservable('user.email').pipe(
        pluck('newValue'),
        debounceTime(300),
        distinctUntilChanged(),
        switchMap(value => {
          if(value.length > 0) {
            this.check.email = 'hourglass_empty'
            setTimeout( () => {
                this.checkField('email')
            }, 300);
            return value
          }else {
            $('[data-toggle="popoverEmail"]').popover('hide')
            return 'null'
          }
        })
      ),
    }
  },

  methods: {
    register: function () {
      if (this.validate()) {
        let data = {
          'name': this.user.name,
          'user': this.user.user,
          'email': this.user.email,
          'company': this.user.company,
          'password': this.user.password,
          'cnpj': this.user.cnpj,
          'phone': this.user.phone
        }

        this.$store.dispatch('register', data)
        .then(check => {
          if(check.auth == true) {
            this.$router.push({ name: 'editarcadastro' });
          }
          else if(check.auth == false) {
            this.validate(true);
          }
        })
        .catch(res => {
          alert('Erro no sistema, por favor contate o admin.')
        })
      }
    },

    checkField(field_name) {
      this.checkInput()
      if (field_name == 'email' && !(/^[a-z0-9-_.]+@[a-z0-9]+\.[a-z]/.test(this.user.email))) {
        this.check.email = 'report_problem'
        $('[data-toggle="popoverEmail"]').popover('show')
      }
      else {
        this.$http.get(`bypass/users/validation?name=${field_name}&value=${this.user[field_name]}`)
        .then(res => res.json())
        .then(check => {
          if(check && check.check != false) {
            if(field_name == 'user') {
              $('[data-toggle="popoverUser"]').popover('show')
              this.check.user = 'report_problem'
            }
            else if(field_name == 'email') {
              $('[data-toggle="popoverEmail"]').popover('show')
              this.check.email = 'report_problem'
            }
          }

          else if(check.check == false) {
            if(field_name == 'user') {
              $('[data-toggle="popoverUser"]').popover('hide')
              if (this.user.user) this.check.user = `check_circle`
              else this.check.user = ''
            }
            if(field_name == 'email') {
              $('[data-toggle="popoverEmail"]').popover('hide')
              if (this.user.email) this.check.email = `check_circle`
              else this.check.email = ''  
            }
          }
        });
      }
    },

    inputActives(booleano, name) {
      if(booleano == true) {
        if (name == 'user') this.inputActiveUser = 'divv-control'
        else this.inputActiveEmail = 'divv-control'
      }
      else {
        if (name == 'user') this.inputActiveUser = ''
        else this.inputActiveEmail = ''
        $('[data-toggle="popoverUser"]').popover('hide')
        $('[data-toggle="popoverEmail"]').popover('hide')
      }
      if (!this.user.user) $('[data-toggle="popoverUser"]').popover('hide')
      if (!this.user.email) $('[data-toggle="popoverEmail"]').popover('hide')
    },

    checkInput() {
      if (this.user.user) this.checkInputUser = 'check-input'
      else {
        $('[data-toggle="popoverUser"]').popover('hide')
        this.checkInputUser = ''
      }
      if (this.user.email) this.checkInputEmail = 'check-input'
      else {
        $('[data-toggle="popoverEmail"]').popover('hide')
        this.checkInputEmail = ''
      }
    },

    validate (cadastrar=false) {
      if (cadastrar == true) {
        this.$notify({
            group: 'Warn',
            title: 'Alerta!',
            type: 'warn',
            text: 'Mude os campos que estão com erro.'
          });
        return false;
      }
      if (this.user.name === "") {
        this.$notify({
            group: 'Warn',
            title: 'Alerta!',
            type: 'warn',
            text: 'O campo nome não pode ficar vazio.'
          });
        return false;
      }
      if (this.check.user == "report_problem") {
        this.$notify({
            group: 'Warn',
            title: 'Alerta!',
            type: 'warn',
            text: 'Usuário já existe.'
          });
        return false;
      }

      if (this.check.email == "report_problem") {
        this.$notify({
            group: 'Warn',
            title: 'Alerta!',
            type: 'warn',
            text: 'E-mail incorreto ou já existe.'
          });
        return false;
      }
      if (this.user.email === "") {
        this.$notify({
            group: 'Warn',
            title: 'Alerta!',
            type: 'warn',
            text: 'O campo email não pode ficar vazio.'
          });
        return false;
      }
      if (!(/^[a-z0-9-_.]+@[a-z0-9]+\.[a-z]/.test(this.user.email))) {
        this.$notify({
            group: 'Warn',
            title: 'Alerta!',
            type: 'warn',
            text: 'Endereço de email inválido.'
          });
        return false;
      }
      if (this.user.password === "") {
        this.$notify({
          group: 'Warn',
          title: 'Alerta!',
          type: 'warn',
          text: 'O campo senha não pode ficar vazio.'
        });
        return false;
      }
      if (this.user.password !== this.passwordConfirmation) {
        this.$notify({
          group: 'Error',
          title: 'Alerta!',
          type: 'warn',
          text: 'As senhas informadas não correspondem.'
        });
        return false;
      }
      if (this.user.company == '') {
        this.$notify({
          group: 'Error',
          title: 'Alerta!',
          type: 'warn',
          text: 'O campo Empresa não pode ficar vazio.'
        });
        return false;
      }
      return true;
      }
  },

  beforeDestroy: () => {
    $('[data-toggle="popoverUser"]').popover('hide')
    $('[data-toggle="popoverEmail"]').popover('hide')
  }
  
}
</script>

<style> /* com scoped não funciona, analisar depois!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! */
  .body {
    position: relative;

    text-align: center;
  }
  .form-cadastro2 {
    width: 100%;
    max-width: 300px;
    margin: 10px auto;
    text-align: center;
  }
  .form-cadastro2 input[type="password"] {
    margin-bottom: 10px;
  }
  .check_circle {
    color: green;
  }
  .report_problem {
    color: #86181d;
  }
  .check-i {
    background-color: white;
    border-left: none;
    border-radius: 0 4px 4px 0!important;
  }
  .material-icons {

  }
  .check-input  {
    border-right: none !important;
  }
  .popover {
    transform: translate3d(0px, 400, 0px) !important;
  }
 .check-input:focus {
    border: 1px solid #ced4da;
    border-right: none !important;
    box-shadow: none !important;
    border-radius: 4px 0 0 4px !important;
  }
  .divv-control {
    color: #495057;
    background-color: #fff;
    border-color: #80bdff;
    outline: 0;
    box-shadow: 0 0 0 0.2rem rgba(0,123,255,.25);
    border-radius: 4px
  }
  .divv-control input, .divv-control .check-i{
    border-color: #80bdff!important;
  }
  .popover {
    background-color: #ffdce0;
    color: #cea0a5;
    border-color: #cea0a5;
    left: -25px !important;
  }
  .popover-content {
    background-color: #cea0a5 !important;
    color: #cea0a5!important;
}
  .popover.bottom .arrow::after {
    border-bottom-color: #ffdce0 !important;
    color: #cea0a5!important;
    border-color: #cea0a5 !important;
  }
  .arrow {
    border-right-color: #cea0a5 !important;
    left:  7px!important;
  }
  .bs-popover-auto[x-placement^="bottom"] .arrow::after, .bs-popover-bottom .arrow::after{
    border-bottom-color: #ffdce0 !important;
  }
  .popover-body{
    color: #86181d;
  }
  .popover-body span{
    font-weight: bold;
  }
</style>
