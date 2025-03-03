<template>

  <div>
    <hr />

    <template v-if="step.type === 'TEXT' || step.type === 'RECIPE'">
      <div class="row" v-if="recipe.steps.length > 1">
        <div class="col col-md-8">
          <h5 class="text-primary">
            <template v-if="step.name">{{ step.name }}</template>
            <template v-else>{{ $t('Step') }} {{ index + 1 }}</template>
            <small style="margin-left: 4px" class="text-muted" v-if="step.time !== 0"><i class="fas fa-user-clock"></i>
              {{ step.time }} {{ $t('min') }}

            </small>
            <small v-if="start_time !== ''" class="d-print-none">
              <b-link :id="`id_reactive_popover_${step.id}`" @click="openPopover" href="#">
                {{ moment(start_time).add(step.time_offset, 'minutes').format('HH:mm') }}
              </b-link>
            </small>
          </h5>
        </div>
        <div class="col col-md-4" style="text-align: right">
          <b-button @click="details_visible = !details_visible" style="border: none; background: none"
                    class="shadow-none d-print-none"
                    :class="{ 'text-primary': details_visible, 'text-success': !details_visible}">
            <i class="far fa-check-circle"></i>
          </b-button>
        </div>
      </div>

    </template>

    <template v-if="step.type === 'TEXT'">

      <b-collapse id="collapse-1" v-model="details_visible">
        <div class="row">
          <div class="col col-md-4"
               v-if="step.ingredients.length > 0 && (recipe.steps.length > 1 || force_ingredients)">
            <table class="table table-sm">
              <!-- eslint-disable vue/no-v-for-template-key-on-child -->
              <template v-for="i in step.ingredients">
                <Ingredient v-bind:ingredient="i" :ingredient_factor="ingredient_factor" :key="i.id"
                            @checked-state-changed="$emit('checked-state-changed', i)"></Ingredient>
              </template>
              <!-- eslint-enable vue/no-v-for-template-key-on-child -->
            </table>
          </div>
          <div class="col" :class="{ 'col-md-8':  recipe.steps.length > 1, 'col-md-12':  recipe.steps.length <= 1,}">
            <compile-component :code="step.ingredients_markdown"
                               :ingredient_factor="ingredient_factor"></compile-component>
          </div>
        </div>
      </b-collapse>
    </template>

    <template v-if="step.type === 'TIME' || step.type === 'FILE'">
      <div class="row">
        <div class="col-md-8 offset-md-2" style="text-align: center">
          <h4 class="text-primary">
            <template v-if="step.name">{{ step.name }}</template>
            <template v-else>{{ $t('Step') }} {{ index + 1 }}</template>
          </h4>
          <span style="margin-left: 4px" class="text-muted" v-if="step.time !== 0"><i class="fa fa-stopwatch"></i>
              {{ step.time }} {{ $t('min') }}</span>
          <b-link class="d-print-none" :id="`id_reactive_popover_${step.id}`" @click="openPopover" href="#"
                  v-if="start_time !== ''">
            {{ moment(start_time).add(step.time_offset, 'minutes').format('HH:mm') }}
          </b-link>
        </div>

        <div class="col-md-2" style="text-align: right">
          <b-button @click="details_visible = !details_visible" style="border: none; background: none"
                    class="shadow-none d-print-none"
                    :class="{ 'text-primary': details_visible, 'text-success': !details_visible}">
            <i class="far fa-check-circle"></i>
          </b-button>
        </div>
      </div>

      <b-collapse id="collapse-1" v-model="details_visible">
        <div class="row" v-if="step.instruction !== ''">
          <div class="col col-md-12" style="text-align: center">
            <compile-component :code="step.ingredients_markdown"
                               :ingredient_factor="ingredient_factor"></compile-component>
          </div>
        </div>
      </b-collapse>
    </template>

    <div class="row" style="text-align: center">
      <div class="col col-md-12">
        <template v-if="step.file !== null">
          <div
              v-if="step.file.file.includes('.png') || recipe.file_path.includes('.jpg') || recipe.file_path.includes('.jpeg') || recipe.file_path.includes('.gif')">
            <img :src="step.file.file" style="max-width: 50vw; max-height: 50vh">
          </div>
          <div v-else>
            <a :href="step.file.file" target="_blank" rel="noreferrer nofollow">{{ $t('Download') }} {{
                $t('File')
              }}</a>
          </div>
        </template>
      </div>
    </div>


    <div class="card" v-if="step.type === 'RECIPE' && step.step_recipe_data !== null">
 <b-collapse id="collapse-1" v-model="details_visible">
      <div class="card-body">
        <h2 class="card-title">
          <a :href="resolveDjangoUrl('view_recipe',step.step_recipe_data.id)">{{ step.step_recipe_data.name }}</a>
        </h2>
        <div v-for="(sub_step, index) in step.step_recipe_data.steps" v-bind:key="`substep_${sub_step.id}`">
          <Step :recipe="step.step_recipe_data" :step="sub_step" :ingredient_factor="ingredient_factor" :index="index"
                :start_time="start_time" :force_ingredients="true"></Step>
        </div>
      </div>
    </b-collapse>

    </div>


    <div v-if="start_time !== ''">
      <b-popover
          :target="`id_reactive_popover_${step.id}`"
          triggers="click"
          placement="bottom"
          :ref="`id_reactive_popover_${step.id}`"
          :title="$t('Step start time')">
        <div>
          <b-form-group
              label="Time"
              label-for="popover-input-1"
              label-cols="3"
              class="mb-1">
            <b-form-input
                type="datetime-local"
                id="popover-input-1"
                v-model.datetime-local="set_time_input"
                size="sm"
            ></b-form-input>
          </b-form-group>
        </div>
        <div class="row" style="margin-top: 1vh">
          <div class="col-12" style="text-align: right">
            <b-button @click="closePopover" size="sm" variant="secondary" style="margin-right:8px">Cancel</b-button>
            <b-button @click="updateTime" size="sm" variant="primary">Ok</b-button>
          </div>
        </div>
      </b-popover>
    </div>
  </div>

</template>

<script>

import {calculateAmount} from "@/utils/utils";

import Ingredient from "@/components/Ingredient";
import {GettextMixin} from "@/utils/utils";

import CompileComponent from "@/components/CompileComponent";
import Vue from "vue";
import moment from "moment";
import Keywords from "@/components/Keywords";
import {ResolveUrlMixin} from "@/utils/utils";

Vue.prototype.moment = moment

export default {
  name: 'Step',
  mixins: [
    GettextMixin,
    ResolveUrlMixin,
  ],
  components: {
    Ingredient,
    CompileComponent,
  },
  props: {
    step: Object,
    ingredient_factor: Number,
    index: Number,
    recipe: Object,
    start_time: String,
    force_ingredients: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      details_visible: true,
      set_time_input: '',
    }
  },
  mounted() {
    this.set_time_input = moment(this.start_time).add(this.step.time_offset, 'minutes').format('yyyy-MM-DDTHH:mm')
  },
  methods: {
    calculateAmount: function (x) {
      // used by the jinja2 template
      return calculateAmount(x, this.ingredient_factor)
    },
    updateTime: function () {
      let new_start_time = moment(this.set_time_input).add(this.step.time_offset * -1, 'minutes').format('yyyy-MM-DDTHH:mm')

      this.$emit('update-start-time', new_start_time)
      this.closePopover()
    },
    closePopover: function () {
      this.$refs[`id_reactive_popover_${this.step.id}`].$emit('close')
    },
    openPopover: function () {
      this.$refs[`id_reactive_popover_${this.step.id}`].$emit('open')
    }
  }
}
</script>
