<template>

  <CoreBase
    :immersivePage="false"
    :authorized="userIsAuthorized"
    authorizedRole="adminOrCoach"
    :showSubNav="true"
  >

    <TopNavbar slot="sub-nav" />

    <KPageContainer>
      <ReportsHeader />
      <KSelect
        v-model="filter"
        :label="$tr('show')"
        :options="filterOptions"
        :inline="true"
      />
      <CoreTable>
        <thead slot="thead">
          <tr>
            <td>{{ coachStrings.$tr('titleLabel') }}</td>
            <td>{{ coachStrings.$tr('avgScoreLabel') }}</td>
            <td>{{ coachStrings.$tr('progressLabel') }}</td>
            <td>{{ coachStrings.$tr('recipientsLabel') }}</td>
            <td>{{ coachStrings.$tr('statusLabel') }}</td>
          </tr>
        </thead>
        <transition-group slot="tbody" tag="tbody" name="list">
          <tr v-for="tableRow in table" :key="tableRow.id">
            <td>
              <KRouterLink
                :text="tableRow.title"
                :to="classRoute('ReportsQuizLearnerListPage', { quizId: tableRow.id })"
              />
            </td>
            <td>
              <Score :value="tableRow.avgScore" />
            </td>
            <td>
              <StatusSummary
                :tally="tableRow.tally"
                :verbose="true"
              />
            </td>
            <td><Recipients :groupNames="tableRow.groupNames" /></td>
            <td>
              <QuizActive :active="tableRow.active" />
            </td>
          </tr>
        </transition-group>
      </CoreTable>
    </KPageContainer>
  </CoreBase>

</template>


<script>

  import commonCoach from '../common';
  import ReportsHeader from './ReportsHeader';

  export default {
    name: 'ReportsQuizListPage',
    components: {
      ReportsHeader,
    },
    mixins: [commonCoach],
    data() {
      return {
        filter: 'allQuizzes',
      };
    },
    computed: {
      filterOptions() {
        return [
          {
            label: this.$tr('allQuizzes'),
            value: 'allQuizzes',
          },
          {
            label: this.$tr('activeQuizzes'),
            value: 'activeQuizzes',
          },
          {
            label: this.$tr('inactiveQuizzes'),
            value: 'inactiveQuizzes',
          },
        ];
      },
      table() {
        const filtered = this.exams.filter(exam => {
          if (this.filter.value === 'allQuizzes') {
            return true;
          } else if (this.filter.value === 'activeQuizzes') {
            return exam.active;
          } else if (this.filter.value === 'inactiveQuizzes') {
            return !exam.active;
          }
        });
        const sorted = this._.sortBy(filtered, ['title', 'active']);
        const mapped = sorted.map(exam => {
          const learnersForQuiz = this.getLearnersForGroups(exam.groups);
          const tableRow = {
            totalLearners: learnersForQuiz.length,
            tally: this.getExamStatusTally(exam.id, learnersForQuiz),
            groupNames: this.getGroupNames(exam.groups),
            avgScore: this.getExamAvgScore(exam.id, learnersForQuiz),
          };
          Object.assign(tableRow, exam);
          return tableRow;
        });
        return mapped;
      },
    },
    beforeMount() {
      this.filter = this.filterOptions[0];
    },
    $trs: {
      show: 'Show',
      allQuizzes: 'All quizzes',
      activeQuizzes: 'Active quizzes',
      inactiveQuizzes: 'Inactive quizzes',
    },
  };

</script>


<style lang="scss" scoped></style>