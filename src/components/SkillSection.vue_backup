<script setup lang="ts">
import { ref } from "vue";

const skillInfo = ref([
  {
    title: "Programming Languages Skills",
    listSkill: [
      {
        name: "HTML, CSS",
        percent: "90%",
      },
      {
        name: "Javascript",
        percent: "80%",
      },
      {
        name: "Typescript",
        percent: "70%",
      },
      {
        name: "PHP",
        percent: "50%",
      },
      {
        name: "Java",
        percent: "50%",
      },
    ],
  },
  {
    title: "Personal Skills",
    listSkill: [
      {
        name: "Git",
        percent: "90%",
      },
      {
        name: "Figma",
        percent: "80%",
      },
      {
        name: "Communicate",
        percent: "100%",
      },
      {
        name: "Teamwork",
        percent: "100%",
      },
      {
        name: "Responsibility",
        percent: "100%",
      },
    ],
  },
]);
</script>

<template>
  <section class="skills" id="skills">
    <h2 class="title">My <span>Skills</span></h2>
    <div class="skills-row">
      <div
        class="skills-column"
        v-for="(skill, i) in skillInfo"
        :key="skill.title + i"
        :id="`skills-column${i}`"
      >
        <h3 class="skills-title">{{ skill.title }}</h3>
        <div class="skills-box">
          <div class="skills-content">
            <div
              class="progress"
              v-for="(childSkill, i) in skill.listSkill"
              :key="childSkill.name + i"
            >
              <h3>
                {{ childSkill.name }} <span>{{ childSkill.percent }}</span>
              </h3>
              <div class="bar">
                <span :style="`width: ${childSkill.percent}`"></span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style lang="scss" scoped>
.skills {
  min-height: auto;
  padding-bottom: 7rem;
  background-color: var(--bg-color);

  .skills-row {
    display: flex;
    flex-wrap: wrap;
    gap: 5rem;

    .skills-column {
      flex: 1 1 40rem;

      .skills-title {
        font-size: 2.5rem;
        margin: 0 0 1.5rem;
        text-align: start;
      }
      .skills-box {
        .skills-content {
          position: relative;
          border-left: 0.2rem solid var(--main-color);
          padding: 0.5rem 1.5rem;
        }
        .progress::before {
          content: "";
          position: absolute;
          top: 50%;
          width: 2rem;
          height: 2rem;
          left: -2.5rem;
          background: var(--main-color);
          border-radius: 50%;
        }
        .progress {
          position: relative;
          padding: 1rem 0;
          h3 {
            font-size: 1.7rem;
            display: flex;
            justify-content: space-between;
          }
          .bar {
            height: 2.5rem;
            border-radius: 0.6rem;
            border: 0.2rem solid var(--main-color);
            padding: 0.5rem;
            margin: 1rem 0;

            span {
              display: block;
              height: 100%;
              border-radius: 0.3rem;
              background: var(--main-color);
            }
          }
        }
      }
    }
  }
}
</style>
