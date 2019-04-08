<template>
  <div id="app" class="container">
    <p>
      <a class="btn btn-primary" href="#" role="button" @click.prevent="showFirstCollapse = !showFirstCollapse">Toggle first element</a>
      <button class="btn btn-primary" type="button" @click="showSecondCollapse = !showSecondCollapse">Toggle second element</button>
      <button class="btn btn-primary" type="button" @click="showFirstCollapse = !showFirstCollapse; showSecondCollapse = !showSecondCollapse">Toggle both elements</button>
    </p>
    <div class="row">
      <div class="col">
        <div is="BsCollapse" :show=showFirstCollapse>
          <div class="card card-body">
            Anim pariatur cliche reprehenderit, enim eiusmod high life accusamus terry richardson ad squid.
            Nihil anim keffiyeh helvetica, craft beer labore wes anderson cred nesciunt sapiente ea proident.
          </div>
        </div>
      </div>
      <div class="col">
        <div is="BsCollapse" :show=showSecondCollapse>
          <div class="card card-body">
            Anim pariatur cliche reprehenderit, enim eiusmod high life accusamus terry richardson ad squid.
            Nihil anim keffiyeh helvetica, craft beer labore wes anderson cred nesciunt sapiente ea proident.
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue';
import BsCollapse from './components/bs-collapse.vue';

export default {
  name: 'app',
  data() {
    return {
      showFirstCollapse: false,
      showSecondCollapse: false,
    };
  },
  components: {
    BsCollapse,
  },
};
</script>
