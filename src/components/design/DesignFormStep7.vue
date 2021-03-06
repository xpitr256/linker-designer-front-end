<template>
  <div class="mt-0">
    <nav aria-label="breadcrumb">
      <ol class="breadcrumb">
        <li class="breadcrumb-item">
          <a class="black-text" href="#" v-on:click="goToStep(1)">{{ $t("views.design.breadcrumb.step1") }}</a>
        </li>
        <li class="breadcrumb-item">
          <a class="black-text" href="#" v-on:click="getStepBack">{{ $t("views.design.breadcrumb.step2InitialData") }}</a>
        </li>
        <li class="breadcrumb-item active">
          {{ $t("views.design.breadcrumb.step3") }}
        </li>
      </ol>
    </nav>

    <h2 class="h-light mt-5">
      <span class="badge badge-secondary">3</span>
      {{ $t("views.design.rdStepISFS") }}
    </h2>

    <form class="mt-4" v-on:submit.prevent="onSubmit">
      <div class="form-row">
        <div class="form-group col-6">
          <label
            ><strong>{{ $t("views.design.rdStepLabelFS1") }}</strong></label
          >
          <fasta-uploader name="flankingSequence1" v-validate="'required'" v-model="flankingSequence1" :error="errors.first('flankingSequence1')">
          </fasta-uploader>
          <fasta-validator
            :fasta-file="flankingSequence1"
            id="flankingSequence1"
            :characters-in-line="50"
            :highlight-at-the-end="true"
            :highlighted-characters-amount="100"
            @newFastaValidation="updateFormValidation"
          ></fasta-validator>
        </div>
        <div class="form-group col-6">
          <label
            ><strong>{{ $t("views.design.rdStepLabelFS2") }}</strong></label
          >
          <fasta-uploader name="flankingSequence2" v-validate="'required'" v-model="flankingSequence2" :error="errors.first('flankingSequence2')">
          </fasta-uploader>
          <fasta-validator
            :fasta-file="flankingSequence2"
            id="flankingSequence2"
            :characters-in-line="50"
            :highlight-at-the-beginning="true"
            :highlighted-characters-amount="100"
            @newFastaValidation="updateFormValidation"
          ></fasta-validator>
        </div>
      </div>

      <div class="form-row">
        <div class="form-group col-6">
          <label><strong>Email</strong></label>
          <input class="form-control" placeholder="Email" ref="email" name="email" v-model="email" v-on:keypress="onEnterKeypress" type="email" />
        </div>
      </div>

      <div class="form-row">
        <div class="form-group col-5">
          <label
            ><strong>{{ $t("views.design.rdStepLabelIS") }}</strong></label
          >
          <fasta-uploader name="initialSequence" v-validate="'required'" v-model="initialSequence" :error="errors.first('initialSequence')"> </fasta-uploader>
        </div>
        <div class="form-group col-7">
          <label>&nbsp;</label>
          <fasta-validator
            :fasta-file="initialSequence"
            id="initialSequence"
            :characters-in-line="60"
            @newFastaValidation="updateFormValidation"
          ></fasta-validator>
        </div>
      </div>

      <div class="d-flex">
        <div>
          <a href="#" class="btn btn-light" v-on:click="getStepBack"> <i class="fas fa-chevron-left"></i> {{ $t("views.getBack") }} </a>
        </div>
        <div class="ml-auto">
          <button type="button" v-on:click="next" :disabled="errors.items.length > 0" class="btn btn-primary">
            {{ $t("views.design.next") }}
            <i class="fas fa-chevron-right"></i>
          </button>
        </div>
      </div>
    </form>
  </div>
</template>
<script>
import FastaUploader from "../../components/FastaUploader";
import FastaValidator from "../FastaValidator";
import FastaService from "../../services/FastaService";

export default {
  name: "DesignFormStep7",
  components: {
    FastaUploader,
    FastaValidator
  },
  methods: {
    getStepBack() {
      this.goToStep(3);
    },
    goToStep(step) {
      this.$emit("goToNextStep", {
        nextStep: step
      });
    },
    updateFormValidation: function(id, isValid) {
      if (!isValid) {
        this.errors.add({
          field: id,
          msg: "Please provide a fasta file according to the following suggestions"
        });
      }
    },
    onSubmit: function() {
      this.next();
    },
    onEnterKeypress: function(event) {
      if (event.key === "Enter") {
        this.next();
      }
    },
    next: async function() {
      let formIsValid = await this.$validator.validate();
      if (formIsValid) {
        const initialSequenceContent = await FastaService.getFastaFileContent(this.initialSequence);
        const flankingSequence1Content = await FastaService.getFastaFileContent(this.flankingSequence1);
        const flankingSequence2Content = await FastaService.getFastaFileContent(this.flankingSequence2);
        this.$emit("goToNextStep", {
          nextStep: "Final",
          formData: {
            stepFrom: 7,
            email: this.email,
            initialSequence: {
              name: FastaService.getSequenceName(initialSequenceContent),
              value: FastaService.getFirstSequence(initialSequenceContent)
            },
            flankingSequence1: {
              name: FastaService.getSequenceName(flankingSequence1Content),
              value: FastaService.getFirstSequence(flankingSequence1Content)
            },
            flankingSequence2: {
              name: FastaService.getSequenceName(flankingSequence2Content),
              value: FastaService.getFirstSequence(flankingSequence2Content)
            }
          }
        });
      }
    }
  },
  data: function() {
    return {
      initialSequence: null,
      flankingSequence1: null,
      flankingSequence2: null,
      email: null
    };
  }
};
</script>

<style scoped></style>
